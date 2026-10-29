# クライアントから接続する

[Windows](02-windows-host.md)、[WSL](03-wsl-host.md)、[Linux マシン](04-linux-host.md) で公開したトンネルに、別のマシンから接続する手順を説明します。  
クライアント側でも、ホストと同じアカウントでサインインします。

## デスクトップ版 VS Code から接続する

1. Remote - Tunnels 拡張機能 (`ms-vscode.remote-server`) をインストールします。

   ```bash
   code --install-extension ms-vscode.remote-server
   ```

2. コマンド パレット (F1) で "Remote Tunnels: Connect to Tunnel..." を実行します。
3. ホストと同じ GitHub アカウントまたは Microsoft アカウントでサインインします。
4. 登録済みのトンネルの一覧から、接続先 (例: `my-win`、`my-wsl`) を選択します。

Remote Explorer ビュー ("Remote Explorer: Focus on Remote View") からも、トンネルの一覧を確認して接続できます。  
不要になったマシンは、Remote Explorer で右クリックして "unregister" を選択すると登録を解除できます。

接続後は、UI がクライアント側で動作し、ファイル操作、ターミナル、ワークスペース側の拡張機能はホスト側で動作します。

## ブラウザーから接続する

ブラウザーで次の形式の URL を開きます。

```text
https://vscode.dev/tunnel/<マシン名>/<フォルダー>
```

初回はサインインを求められます。  
ブラウザー版の `vscode.dev` には Remote - Tunnels 拡張機能があらかじめ含まれています。

## 接続後に Dev Containers を使う

ホスト上のフォルダーに Dev Container の設定がある場合は、接続後に "Dev Containers: Reopen in Container" を実行します。

## 接続時の注意点

- 1 つのインスタンスに同時接続できるクライアントは 1 台のみです。
- ホスト側の VS Code (方法 1 の場合) や `code tunnel` (方法 2 の場合) が停止していると、トンネルの一覧に表示されていても接続できません。

## 参考資料

- [Developing with Remote Tunnels](https://code.visualstudio.com/docs/remote/tunnels)
- [Remote - Tunnels (Marketplace)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server)
