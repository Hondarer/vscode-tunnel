# Remote Tunnels の仕組み

VS Code Remote Tunnels は、手元の VS Code から別のマシン上の開発環境へ接続する仕組みです。  
開発環境を公開する側 (ホスト) と接続する側 (クライアント) の間は、Microsoft が運用する Dev Tunnels の中継サービス (relay) が仲介します。

クライアント側では、拡張機能 Remote - Tunnels を使用します。

| 項目 | 値 |
|---|---|
| 名前 | Remote - Tunnels |
| ID | `ms-vscode.remote-server` |
| 説明 | Connect to a remote machine through a Tunnel |
| 確認時のバージョン | 1.5.4 |
| パブリッシャー | Microsoft |
| Marketplace | <https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server> |

## 外向き接続による通信モデル

クライアントからホストへ直接 TCP 接続するのではなく、ホストとクライアントの双方が relay に向けて外向き (outbound) の接続を確立します。

```text
ホスト (Windows / WSL)
    |
    | outbound
    v
Microsoft Dev Tunnels relay
    ^
    | outbound / 認証済みの接続
    |
クライアント (VS Code / ブラウザー)
```

このため、ホスト側でインターネット向けの受信ポート (inbound port) を開放する必要はなく、ファイアウォールの設定変更も通常は不要です。  
通信の仕組みは SSH のリバース トンネルに類似しています。

ただし、単純な TCP のポート転送とは異なり、ホスト上では VS Code Server が稼働してファイル システム、ターミナル、言語サーバー、デバッガー、ワークスペース側の拡張機能を提供します。  
クライアントは UI の表示と操作だけを担当します。

## ホストとクライアントの役割

| 役割 | 動作するもの | 使用するもの |
|---|---|---|
| ホスト | VS Code Server、トンネルを確立する VS Code CLI | VS Code デスクトップ版の「Turn on Remote Tunnel Access」、または `code tunnel` コマンド |
| クライアント | VS Code の UI | VS Code デスクトップ版と Remote - Tunnels 拡張機能、またはブラウザー (`vscode.dev`) |

クライアントにはブラウザー版 (`vscode.dev`) のほか、Remote - Tunnels 拡張機能をインストールしたデスクトップ版 VS Code も使用できます。

## Windows を公開するか、WSL を公開するか

同じ PC であっても、Windows と WSL のどちらをホストにするかによって、接続後に参照できるファイル システムやツールチェーンが異なります。

```text
[Windows をホストにした場合]

クライアント --> relay --> Windows
                             +-- VS Code Server (Windows 版)
                             +-- C:\ のファイル
                             +-- Windows のツールチェーン

[WSL をホストにした場合]

クライアント --> relay --> WSL
                             +-- VS Code Server (Linux 版)
                             +-- Linux のファイル システム
                             +-- Linux のツールチェーン
```

このリポジトリでは、Windows と WSL をそれぞれ別のトンネルとして公開する手順を説明します。

- [Windows の VS Code を公開する](02-windows-host.md)
- [WSL の VS Code を公開する](03-wsl-host.md)
- [Linux マシンの VS Code を公開する](04-linux-host.md) (物理マシンや VM で直接動作している Linux をホストにする場合)
- [クライアントから接続する](05-connect-from-client.md)

公式ドキュメントに「Remote - Tunnels 越しに WSL や Dev Containers へ接続できる」と記載されているとおり、Windows のトンネルに接続したあと、WSL 拡張機能を経由して WSL へ接続することも可能です。  
ただし、この方法は Windows 側の VS Code が稼働していることを前提とします。  
WSL だけで接続を完結させたい場合は、WSL 自体をホストとして構成します。

## 認証と利用制限

ホストとクライアントの双方で、GitHub アカウントまたは Microsoft アカウント (Microsoft Entra ID を含む) によるサインインが必要です。  
匿名ではトンネルを作成できず、原則としてトンネルを作成したアカウントでサインインしたクライアントだけが接続できます。

公式ドキュメントに記載されている主な制限は次のとおりです。

- 1 アカウントで登録できるトンネルは 10 個までです。上限に達すると、CLI は使用されていないトンネルを 1 つ選択して削除します。
- 1 つのインスタンスに同時接続できるクライアントは 1 台のみです。
- 通信帯域などにも使用量の上限があります。

2026-10-30 時点の Microsoft Learn に記載されている Dev Tunnels 全体の上限は次のとおりです。

| 項目 | 制限 |
|---|---:|
| Bandwidth | 5 GB / user / month |
| Tunnels | 10 / user |
| Ports | 10 / tunnel |
| Active connections | 1000 / port |
| HTTP request rate | 1500 / min / port |
| Data transfer rate | 最大 20 MB/s / tunnel |
| Web forwarding HTTP request body | 最大 16 MB |

Microsoft は、Dev Tunnels が開発・テスト用途向けであり、本番ワークロード向けではないことを明示しています。  
2026-10-30 時点では public preview であり、SLA は提供されていません。

## Microsoft が relay を無償で提供できる理由

Microsoft が提供しているのは、認証、制御、relay、および通信帯域です。  
ホストの CPU、メモリ、ストレージ、コンパイラ、デバッガーは、すべてユーザー側のマシンのリソースを使用します。

```text
ユーザー側 PC / WSL
  +-- CPU / RAM / storage
  +-- compiler / debugger
  +-- VS Code Server
        |
        v
   Microsoft relay
        |
        v
   VS Code client
```

GitHub Codespaces のようにクラウド側で VM を用意するサービスとは、コスト構造が大きく異なります。  
前述の制限値からも、大量転送用の VPN ではなく開発用途向けに設計されていることが分かります。

## 参考資料

- [Developing with Remote Tunnels](https://code.visualstudio.com/docs/remote/tunnels)
- [VS Code Remote Development](https://code.visualstudio.com/docs/remote/remote-overview)
- [Visual Studio Code Server](https://code.visualstudio.com/docs/remote/vscode-server)
- [What are dev tunnels?](https://learn.microsoft.com/en-us/azure/developer/dev-tunnels/overview)
- [Dev tunnels FAQ](https://learn.microsoft.com/en-us/azure/developer/dev-tunnels/faq)
- [Dev tunnels security](https://learn.microsoft.com/en-us/azure/developer/dev-tunnels/security)
- [Azure subscription and service limits](https://learn.microsoft.com/azure/azure-subscription-service-limits)
