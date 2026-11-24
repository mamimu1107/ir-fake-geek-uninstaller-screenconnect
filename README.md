# ir-fake-geek-uninstaller-screenconnect
# Incident Report: Fake Geek Uninstaller Leading to ScreenConnect Abuse, GPU Cryptojacking, and Cookie Theft

Windows 11 Home 端末において、SEO汚染された検索結果から偽の Geek Uninstaller 配布サイトへ遷移し、DLLサイドローディング経由で ScreenConnect を無音導入されたインシデントの解析・対応記録です（2026年11月）。

> **Disclaimer:** 本リポジトリの情報はセキュリティ調査・教育・防御確認を目的としています。スクリプトやコマンドを実行する際は、対象環境への影響を十分に確認したうえで自己責任で行ってください。

## インシデント概要

- **対象環境:** Windows 11 Home、Microsoft Defender（改ざん防止有効）、Microsoft Edge
- **滞在期間:** 2026-11-10 21:41 〜 2026-11-23 14:24（JST、約13日間）
- **初期侵入:** `geekuninstaller.app/jp/`（正規は `geekuninstaller.com`）から入手した配布物を実行
- **実行手口:** 正規署名の `SqlSetupBootstrapper.exe`（`geek.exe` にリネーム）に悪性 `SqlSetupBootstrapper.dll` をサイドロードさせ、`msiexec` で偽「Microsoft Visual C++ 2022 Redistributable (x64) 14.38.33135.0」MSI（実体は ScreenConnect 24.1.6.8875）を無音インストール
- **C2:** `442.fast-ports.com:8041`（`104.249.59.120/TCP`）、サービス名 `14.38.33135.0`（`LocalSystem` 権限）、セッションID `<REDACTED>`
- **主な活動・被害:**
  - Defender 除外60件超・通知抑止・UAC無効化・ScriptBlockLogging無効化・Windows Update停止試行・FW許可ルール追加・LSA保護（`RunAsPPL=0`）無効化・イベントログ消去
  - タスク・Runキー・スタートアップ `.lnk` による永続化
  - .NET ローダーおよび正規ツール（`InstallUtil`、`RegAsm`、`MSBuild` 等）へのプロセスホロウイング
  - GPU/CPU マイニング（SRBMiner-MULTI、lolMiner、WinRing0）
  - `ccv.exe`（NirSoft ChromeCookiesView）による Edge の `Cookies` / `Local State` 読み出しおよび書き出し
- **関連検体:** 同一の悪性 DLL と同一 C2 を利用する `HiBitUninstaller-setup-4.0.10.zip` も確認

## 主なタイムライン（JST）

- **11/10 21:40〜21:50:** 偽サイトから入手・実行。偽 `geek.exe` 起動、偽 VC++ MSI 導入、ScreenConnect 初回接続、囮の本物 `geek.exe` 起動、再起動後の自動接続確認
- **11/11 19:58:** ScreenConnect 経由で STEP1 スクリプト実行、Defender 除外初回追加
- **11/12〜11/14:** 再接続ごとに自動実行（SCAutoRun とみられる）で Defender 無効化スクリプトを実行
- **11/15 10:45 / 12:13:** コマンド実行（計10件）、`ccv.exe` / `mzcv.exe` やマイナー配置
- **11/21 19:49〜20:48:** `RuntimeHost.exe`・マイナー起動、`wsc.ps1` で UAC 無効化、`wevtutil` によるイベントログ消去（20:00）、`ccv.exe` 実行（20:48）
- **11/22〜11/23 11:56:** ヘッドレス Edge 起動、`NtSvc4643.exe` による Cookie 書き出し
- **11/23 11:42〜14:29:** 検知・資格情報変更と全セッション失効・永続化停止・ペイロード削除・設定/ポリシー復旧・ScreenConnect 停止と偽 VC++ アンインストール
- **11/23 15:00〜11/24:** `sfc` / `DISM`、Amcache/VirusTotal 調査、STEP1 復元、残存 FW ルール削除・設定復旧、Malwarebytes 確認
