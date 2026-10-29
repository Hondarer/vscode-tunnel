# Windows の VS Code を公開する

Windows にインストールした VS Code を、Remote Tunnels のホストとして公開する手順を説明します。  
接続後は Windows のファイル システムとツールチェーンを使用します。  
トンネルの公開には、VS Code の画面から有効にする方法と、`code tunnel` コマンドを実行する方法があります。

## 準備する環境とアカウント

- Windows に VS Code デスクトップ版をインストールしておきます。
- ホストとクライアントの双方でサインインに使用する GitHub アカウントまたは Microsoft アカウントを用意します。

## 方法 1: VS Code の画面から有効にする

1. Windows で VS Code を起動します。
2. 左下のアカウント メニューから "Turn on Remote Tunnel Access..." を選択します。コマンド パレット (F1) で "Remote Tunnels: Turn on Remote Tunnel Access..." を実行しても同じ操作を行えます。
3. GitHub アカウントまたは Microsoft アカウントでサインインします。
4. トンネルが開始されたら、`vscode.dev` のリンクが通知に表示されることを確認します。

この方法で公開したトンネルには、Windows で VS Code が起動している間だけ接続できます。  
トンネルを無効にする場合は、コマンド パレットで "Remote Tunnels: Turn off Remote Tunnel Access..." を実行します。

## 方法 2: code tunnel コマンドを使う

Windows の VS Code に付属する `code` コマンドの `tunnel` サブコマンドを PowerShell で実行します。

```powershell
code tunnel --name my-win --accept-server-license-terms
```

- `--name` にはトンネルの名前 (マシン名) を指定します。WSL 用のトンネルと区別できる名前を設定します。
- `--accept-server-license-terms` を指定すると、初回起動時のライセンス確認を省略できます。

初回実行時はサインインを求められるため、画面の指示に従って認証を行います。  
トンネルが起動すると `https://vscode.dev/tunnel/<マシン名>/...` 形式の URL が表示されます。  
終了する場合は Ctrl+C で停止します。

### サービスとして常駐させる

ログオン時に自動でトンネルを開始したい場合は、サービスとして登録します。

```powershell
code tunnel service install
```

サービスの登録を解除する場合は、次のコマンドを実行します。

```powershell
code tunnel service uninstall
```

### スリープを防ぐ

接続の待ち受け中にマシンがスリープしないようにするには、`--no-sleep` オプションを指定します。

```powershell
code tunnel --no-sleep
```

## 管理用のコマンド

| 目的 | コマンド |
|---|---|
| ヘルプを表示する | `code tunnel --help` |
| このマシンとトンネルの登録を解除する | `code tunnel unregister` |

状態確認や名前変更などその他のサブコマンドについては、`code tunnel --help` の出力で手元のバージョンの仕様を確認してから使用してください。

## 運用上の注意点

- このトンネルのホストは Windows です。WSL のファイルやツールチェーンを直接使いたい場合は、[WSL の VS Code を公開する](03-wsl-host.md) の手順に従って WSL 自体をホストとして構成します。
- トンネルに接続できるユーザーは、ホスト マシン上でシェルを実行できます。サインインに使用するアカウントは二要素認証などで保護してください。
- 使用しなくなったトンネルは、ホスト側で `code tunnel unregister` を実行するか、クライアント側の Remote Explorer から登録を解除します。

## 参考資料

- [Developing with Remote Tunnels](https://code.visualstudio.com/docs/remote/tunnels)
