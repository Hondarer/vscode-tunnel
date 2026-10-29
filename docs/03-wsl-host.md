# WSL の VS Code を公開する

WSL 自体を Remote Tunnels のホストとして公開する手順を説明します。  
接続後は WSL の Linux ファイル システムとツールチェーンを使用し、拡張機能も WSL 側で動作します。  
物理マシンや VM で直接動作している Linux の場合は、[Linux マシンの VS Code を公開する](04-linux-host.md) を参照してください。

## 注意: WSL の code コマンドは Windows 版を指す場合がある

WSL では、Windows 側の VS Code の CLI が `PATH` に含まれています。  
そのため WSL 上でそのまま `code tunnel` を実行すると、Windows 版の VS Code が起動し、トンネルのホストが Windows になる場合があります。

まず、`code` コマンドが参照している実体を確認します。

```bash
type -a code
```

実行結果の例は次のとおりです。

```text
code is /home/user/.vscode-server/bin/<commit>/bin/remote-cli/code
code is /mnt/c/Users/<user>/AppData/Local/Programs/Microsoft VS Code/bin/code
```

| パス | 実体 | トンネルのホストに使えるか |
|---|---|---|
| `/mnt/c/.../Microsoft VS Code/bin/code` | Windows 版 VS Code の起動スクリプト | 使用不可 (ホストが Windows になる) |
| `~/.vscode-server/bin/<commit>/bin/remote-cli/code` | VS Code の WSL 拡張機能で開いたターミナルにだけ現れる、接続元の VS Code を操作するためのコマンド | 使用不可 |
| 後述の手順で導入する Linux 版 standalone CLI | Linux 用の VS Code CLI | 使用可能 |

WSL をホストにするには、Linux 版の standalone CLI を WSL 内に導入して使用します。  
Windows 版の `code` と混同しないよう、このリポジトリでは `code-tunnel` という名前で配置します。

```text
code         -> Windows 版 VS Code の起動スクリプト (そのまま残す)
code-tunnel  -> WSL 内の Linux 版 VS Code CLI (トンネル用)
```

## Linux 版 standalone CLI を導入する

x64 環境では次のコマンドで CLI をダウンロードして配置します。  
Arm64 環境の場合はダウンロード URL のパラメーターを `os=cli-alpine-arm64` に変更します。  
配布されているバイナリは Alpine 用のビルドですが、静的リンクされているため一般的なディストリビューションでも動作します。

```bash
mkdir -p ~/.local/bin
cd "$(mktemp -d)"
curl -fL 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz
tar -xf vscode_cli.tar.gz
mv code ~/.local/bin/code-tunnel
```

`~/.local/bin` が `PATH` に含まれていることを確認します。

```bash
type -a code-tunnel
code-tunnel --version
```

## トンネルを起動する

配置した `code-tunnel` コマンドでトンネルを起動します。

```bash
code-tunnel tunnel --name my-wsl --accept-server-license-terms
```

`--name` には、Windows 用のトンネルと区別できるトンネル名 (マシン名) を指定します。

初回実行時はサインインを求められます。  
ターミナルに表示されるコードを使用し、ブラウザーで GitHub アカウントまたは Microsoft アカウントにサインインします。  
起動すると `https://vscode.dev/tunnel/my-wsl/...` 形式の URL が表示されます。  
終了する場合は Ctrl+C で停止します。

## サービスとして常駐させる

ターミナルを閉じてもトンネルを維持したい場合は、サービスとして登録します。

```bash
code-tunnel tunnel service install
```

Linux では systemd のユーザー サービスとして登録されるため、WSL で systemd が有効になっている必要があります。  
`/etc/wsl.conf` に次の設定があることを確認してください。

```ini
[boot]
systemd=true
```

設定を変更した場合は、Windows 側で `wsl --shutdown` を実行してから WSL を再起動します。

ログインしていない間もユーザー サービスを稼働させ続けるには、lingering を有効にします。

```bash
loginctl enable-linger "$USER"
```

サービスの登録を解除する場合は、次のコマンドを実行します。

```bash
code-tunnel tunnel service uninstall
```

## 管理用のコマンド

| 目的 | コマンド |
|---|---|
| ヘルプを表示する | `code-tunnel tunnel --help` |
| このマシンとトンネルの登録を解除する | `code-tunnel tunnel unregister` |
| スリープを防いで起動する | `code-tunnel tunnel --no-sleep` |

## 運用上の注意点

- WSL は Windows 上の VM として動作するため、Windows がスリープまたはシャットダウンするとトンネルも停止します。
- 同じ PC で Windows 用のトンネルも公開する場合は、別のトンネル名を設定します。1 アカウントで登録できるトンネルは 10 個までです。
- トンネルに接続できるユーザーは、WSL 上でシェルを実行できます。サインインに使用するアカウントは二要素認証などで保護してください。

## 未検証の項目

- `code tunnel service install` が Linux で生成するユニットの名前と配置先の詳細は未確認です。
- systemd のユーザー サービスだけで WSL ディストリビューションが停止せずに稼働し続けるかは未確認です。Windows の再起動後やアイドル時に WSL が停止するとトンネルも停止するため、必要に応じて Windows のタスク スケジューラーでログオン時に WSL を起動しておくなどの対策を検討してください。

## 参考資料

- [Developing with Remote Tunnels](https://code.visualstudio.com/docs/remote/tunnels)
- [Visual Studio Code Server](https://code.visualstudio.com/docs/remote/vscode-server)
- [Advanced settings configuration in WSL](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)
