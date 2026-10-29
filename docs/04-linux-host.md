# Linux マシンの VS Code を公開する

物理マシンや VM で直接動作している Linux を、Remote Tunnels のホストとして公開する手順を説明します。  
基本的な流れは [WSL の VS Code を公開する](03-wsl-host.md) と同じですが、WSL 固有の注意点は不要になり、デスクトップ環境の有無によって手順を選択します。

## WSL の手順との違い

| 項目 | WSL | Linux マシン |
|---|---|---|
| `code` コマンドの混同 | Windows 版 VS Code の CLI が `PATH` に含まれるため注意が必要 | 発生しない |
| 使用する CLI | Linux 版 standalone CLI | デスクトップ版 VS Code の `code`、または Linux 版 standalone CLI |
| systemd | `/etc/wsl.conf` で有効化が必要 | 多くのディストリビューションでは標準で有効 |
| 稼働条件 | Windows と WSL ディストリビューションの稼働に依存 | マシン自身の電源管理に依存 |
| 対応アーキテクチャー | x64、Arm64 | x64、Arm64、Arm32 |

## デスクトップ環境の有無による手順の選択

| マシンの状態 | 使用する手順 |
|---|---|
| デスクトップ環境があり、VS Code デスクトップ版をインストール済み | 方法 1 または方法 2 |
| デスクトップ環境のないサーバー (ヘッドレス) | 方法 3 |

## 方法 1: VS Code の画面から有効にする

操作は Windows と同じです。

1. Linux で VS Code を起動します。
2. 左下のアカウント メニューから "Turn on Remote Tunnel Access..." を選択します。
3. GitHub アカウントまたは Microsoft アカウントでサインインします。
4. トンネルが開始されたら、`vscode.dev` のリンクが通知に表示されることを確認します。

この方法で公開したトンネルには、VS Code が起動している間だけ接続できます。

## 方法 2: デスクトップ版 VS Code の code コマンドを使う

VS Code デスクトップ版 (deb / rpm / snap などでインストールしたもの) に付属する `code` コマンドにも `tunnel` サブコマンドがあります。  
WSL と異なり Windows 版の CLI と混同するおそれはないため、そのまま使用できます。

```bash
type -a code
code tunnel --name my-linux --accept-server-license-terms
```

サービスとしての常駐や管理用のコマンドは、方法 3 の `code-tunnel` を `code` に置き換えて実行します。

## 方法 3: standalone CLI を使う (ヘッドレス)

### CLI を導入する

アーキテクチャーに応じたダウンロード URL のパラメーターを選択します。

| アーキテクチャー | `os=` の値 |
|---|---|
| x64 | `cli-alpine-x64` |
| Arm64 | `cli-alpine-arm64` |
| Arm32 | `cli-linux-armhf` |

x64 環境の例は次のとおりです。  
コマンド名は任意ですが、WSL の手順とコマンドをそろえるため、このリポジトリでは `code-tunnel` という名前で配置します。

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

### トンネルを起動する

```bash
code-tunnel tunnel --name my-linux --accept-server-license-terms
```

`--name` には、Windows や WSL のトンネルと区別できるトンネル名 (マシン名) を指定します。

初回実行時はサインインを求められ、ターミナルにコードが表示されます。  
ヘッドレスのマシンにはブラウザーがないため、手元の PC やスマートフォンのブラウザーでコードを入力し、GitHub アカウントまたは Microsoft アカウントにサインインします。  
終了する場合は Ctrl+C で停止します。

### サービスとして常駐させる

```bash
code-tunnel tunnel service install
```

systemd のユーザー サービスとして登録されるため、ログインしていない間や再起動後もトンネルを稼働させるには lingering を有効にします。  
ヘッドレスのサーバーでは、通常はログイン セッションが存在しないため、この設定が重要です。

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

- デスクトップ環境のある Linux では、電源管理によってスリープするとトンネルも停止します。必要に応じて `--no-sleep` を指定するか、電源設定を見直してください。
- CLI 自体は静的リンクされていますが、ホストで動作する VS Code Server にはディストリビューションや glibc などの動作要件があります。古いディストリビューションを使用する場合は、公式の Linux 前提条件を確認してください。
- 組織のネットワークでは、Dev Tunnels のドメイン (`global.rel.tunnels.api.visualstudio.com`) への通信が制限されている場合があります。
- トンネルに接続できるユーザーは、そのマシン上でシェルを実行できます。サインインに使用するアカウントは二要素認証などで保護してください。

## 未検証の項目

- `code tunnel service install` が Linux で生成するユニットの名前と配置先の詳細は未確認です。
- 方法 2 (デスクトップ版の `code`) で `service install` を実行した場合に、standalone CLI と同じ構成で登録されるかは未確認です。

## 参考資料

- [Developing with Remote Tunnels](https://code.visualstudio.com/docs/remote/tunnels)
- [Visual Studio Code Server](https://code.visualstudio.com/docs/remote/vscode-server)
- [Remote Development with Linux](https://code.visualstudio.com/docs/remote/linux)
