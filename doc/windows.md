Windows 用 セットアップマニュアル
===

本マニュアルでは、Windows 環境におけるセットアップ手順を解説します

**なお、Windows 版は現時点で実験的であり、安定動作を保証するものではありません  
また、今後のアップデート等で非対応となる可能性があります**

本マニュアル内では、以下のソフトウェアを利用したセットアップを行います

- [TeraPad](http://www5f.biglobe.ne.jp/~t-susumu/), [VSCode](https://code.visualstudio.com/) など
    - config.json 等のファイルを編集するため
    - Windows 標準のメモ帳では正しく改行されません
- [Git for Windows](https://git-for-windows.github.io/)
    - EPGStation をアップデートするときに便利です
- Windows PowerShell もしくは コマンドプロンプト


## セットアップ

ここでは Windows PowerShell を用いたセットアップを解説します

1. **Node.js (LTS 版推奨), Mirakurun, windows-build-tools, FFmpeg/FFprobe** がインストール済みであることを確認する

	```
	> node --version
	> Invoke-WebRequest http://<MirakurunIP>:<Port>/api/version
	> npm info windows-build-tools
	```

	FFmpeg/FFprobe については config.json でファイルの場所を指定するので適切な場所に配置すること

2. EPGStation のインストール

	```
	> git clone https://github.com/l3tnun/EPGStation.git
	> cd EPGStation
	> npm install
	> npm run build

	```
3. 設定ファイルの作成

	```
	> copy .\config\config.sample.json .\config\config.json
	> copy .\config\operatorLogConfig.sample.json .\config\operatorLogConfig.json
	> copy .\config\serviceLogConfig.sample.json .\config\serviceLogConfig.json
	```

4. 設定ファイルの編集
	- 詳細な設定は [詳細マニュアル](conf-manual.md) を参照
	- 以下の最低限の動作に必須な項目について編集する

	```json
	"serverPort": 8888,
	"mirakurunPath": "http://localhost:40772",
    "ffmpeg": "C:\\ffmpeg\\ffmpeg.exe",
    "ffprobe": "C:\\ffmpeg\\ffprobe.exe"
	```

    - Mirakurun について、名前付きパイプを使用するなら `\\\\.\\pipe\\mirakurun`

## EPGStationの起動/終了

- 手動で起動する場合

	```
	> npm start
	```

- 自動で起動する場合
	- [winser](https://github.com/jfromaniello/winser) を利用して自動起動設定が可能です
	- 以下のコマンドを管理者権限で実行するとサービス化できます

    ```
    > npm install winser -g
    > npm run install-win-service
    > net start epgstation
    ```

- 手動で終了する場合

	```
	> npm stop
	```

- 自動起動した EPGStation を終了する場合

	```
	> net stop epgstation
	```

    - サービスから削除する場合は以下のコマンドを管理者権限で実行します

    ```
    > npm run uninstall-win-service
    ```

## Tips

### ファイアウォールの設定
EPGStation を別のコンピュータから使用する場合はファイアウォールを開放してください

### パス名区切り文字

unix 系では `/` を使用するため *.sample.json では `/hoge/huga/piyo` と書かれていますが、windows では `\\hoge\\huga\\piyo` このように書いてください

### config.json
#### encode

[enc.sh](../config/enc.sh) を起動するようになっていますが、 Windows では動作しないため `config/enc.js` へ書き換えでください

```
"cmd": "C:\\PROGRA~1\\nodejs\\node.exe %ROOT%\\config\\enc.js main"
```

この `PROGRA~1` は 8.3 形式の表記方法で、 Program Files を指しています。  
cmd.exe にて `dir /x c:\` と打ち込むと確認できます

## MySQL 使用時の注意

EPGStation 使用中は MySQL のバイナリログが大量に生成されてディスクを圧迫するので、MySQL の設定 (my.ini) を変えることを推奨します

```
expire_logs_days = 1
```