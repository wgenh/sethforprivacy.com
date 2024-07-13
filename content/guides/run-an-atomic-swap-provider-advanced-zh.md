---
author: Seth For Privacy
comments: false
date: "2021-08-24T22:00:00-04:00"
summary: 本指南将引导您如何作为提供者（也称为卖家或“自动交换后端”（ASB））进行操作。
draft: false
hidemeta: false
showToc: true
tags:
- 原子交换
- 比特币
- 门罗币
title: 运行原子交换提供商（高级）
---

# 引言

***免责声明 -- 如果您阅读了本指南，但对安装过程和ASB工具的工作原理没有扎实的理解，请首先在测试网上测试软件，而不是主网。本指南旨在为那些认真运行ASB并理解其含义和能够处理前沿软件的人提供。***

经过长时间的等待，它终于来了。您可以*今天*直接在点对点的基础上，通过Tor，无需托管或可信的第三方，无需KYC（了解您的客户）信息，进行比特币和门罗币的交换。本指南将指导您如何作为提供者（也称为卖家或“自动交换后端”[ASB]）进行操作。

这是跨链交换的未来，今天就可以实现。

原子交换开启了广泛的跨链应用场景，但关键在于它们是无信任、不可审查且完全匿名/伪匿名的。

有关原子交换的更多信息，请查看以下链接：

- <https://localmonero.co/knowledge/monero-atomic-swaps>
- <https://www.monerooutreach.org/stories/monero-atomic-swaps.html>
- <https://github.com/comit-network/xmr-btc-swap>
- <https://comit.network/blog/2020/10/06/monero-bitcoin/>

# 背景

本指南旨在简化并提炼我从运行ASB进行自我测试中学到的内容，并为快速启动和运行提供更简单的复制粘贴格式。这个初始构建稍微高级一些，但我正在开发一个Docker Compose设置，应该会简单得多，更容易上手并保持更新。

官方指南可以在这里找到，如果您更愿意参考这些文档，足以让您开始：

- <https://github.com/comit-network/xmr-btc-swap/blob/master/docs/asb/README.md>

为了更好地理解`asb`工具在做什么，为什么您需要运行它来出售XMR，以及它如何与`swap`工具交互，请阅读官方指南的以下部分：

- <https://github.com/comit-network/xmr-btc-swap/blob/master/docs/asb/README.md#setup-details>

如果您想深入了解协议的工作原理以及交换过程中每个步骤的具体细节（我推荐这样做），可以在下面的博客文章中进行：

- <https://comit.network/blog/2020/10/06/monero-bitcoin>

# 理解交换过程中的步骤

我不会在本指南中重写这部分内容，因为它在COMIT网络博客文章中已经很好地布局了。我*强烈*建议熟悉交换过程并尽可能多地阅读上述文档，但最相关的部分在这里：

- <https://comit.network/blog/2020/10/06/monero-bitcoin/#long-story-short>

# 维护隐私/匿名性

重要的是您要理解，运行此工具将允许CLI端的用户将交易ID与IP地址关联，除非您使用Tor进行所有网络通信。以下是一些保护您隐私和/或匿名性的快速建议：

- *切勿*在家中运行此工具，除非您仅使用Tor，并且不暴露任何明网地址
  - 如果您在家中运行，*切勿*在任何可能的情况下通过SSH将运行此软件的机器暴露在互联网上
- 除非您有特定原因或希望确保那些没有Tor访问权限/不了解Tor的人可以与您交换，否则仅在Tor后面使用`asb`工具
- 如果您需要分享日志，请确保删除：
  - 交换ID
  - 交易ID
  - IP地址
- [运行您自己的门罗币节点]({{< ref "/content/guides/run-a-monero-node.md" >}})
- 如果可能，运行您自己的比特币节点和[ElectrumX服务器](https://electrumx-spesmilo.readthedocs.io/en/latest/)
- 使用比特币隐私工具，如[Samourai Wallet](https://www.samouraiwallet.com/)，以保护您的隐私并在从交换中接收资金后防止受污染的比特币
  - 有关比特币隐私的更多信息，请查看BitcoinQnA关于该主题的文章：<https://bitcoiner.guide/privacy/>
  - 有关如何使用Samourai Wallet的具体指南，请参见他的指南：<https://bitcoiner.guide/privacy/separate/>

# 前提条件

本指南假设以下事项已经就绪：

- 您已经有一台计算机/服务器来托管此工具（最好是VPS或为您托管的专用服务器）
- 您能够访问您想要用于该工具的主机的命令行
- 如果您想使用DNS，您已经有一个域名并且熟悉DNS配置
- 您要么运行自己的门罗币节点，要么有一个您信任的节点
  - 任何人都可以使用我的[公共门罗币节点]({{< ref "/content/about.md#high-performance-monero-nodes" >}})
- 您已经有愿意通过ASB出售的门罗币
- 您熟悉发送和接收门罗币
- 您熟悉使用[Samourai Wallet](https://www.samouraiwallet.com/)等工具处理可能受污染的比特币

# 获取工具

本指南还假设您使用的是Linux，但命令对于macOS应该类似，对于Windows大致相似。开始的第一步是获取您需要的所有工具。

## 自动交换经纪人（ASB）

1. 创建一个文件夹来保存所有相关文件

    ```bash
    mkdir ~/asb
    cd ~/asb
    ```

2. 下载最新版本的`asb`工具，例如`asb_0.11.0_Linux_x86_64.tar`通过浏览器
    1. <https://github.com/comit-network/xmr-btc-swap/releases/latest>
    2. 或者您可以通过CLI下载该工具

        ```bash
        wget https://github.com/comit-network/xmr-btc-swap/releases/download/0.10.0/asb_0.10.0_Linux_x86_64.tar
        ```

3. 解压`asb`二进制文件
    1. 打开终端
    2. 运行以下命令：

        ```bash
        tar xvf asb_0.11.0_Linux_x86_64.tar
        rm asb_0.11.0_Linux_x86_64.tar
        sudo chmod +x asb
        sudo cp asb /usr/local/bin/
        ```

4. 验证二进制文件是否正常工作

    ```bash
    asb --version
    ```

{{< figure src="/run-an-atomic-swap-provider-advanced/asb-version.png" align="center" style="border-radius: 8px;" >}}

## monero-wallet-rpc

1. 下载最新版本的门罗币二进制文件，例如`monero-linux-x64-v0.18.1.0.tar.bz2`
    1. <https://github.com/monero-project/monero/releases/latest>
    2. 或者您可以通过CLI下载该工具

        ```bash
        cd ~/asb
        wget https://downloads.getmonero.org/cli/monero-linux-x64-v0.18.1.0.tar.bz2
        ```

2. 解压`monero-wallet-rpc`二进制文件
    1. 打开终端
    2. 运行以下命令：

        ```bash
        tar xvf monero-linux-x64-v0.18.1.0.tar.bz2
        sudo chmod +x monero-x86_64-linux-gnu-v0.18.1.0/monero-wallet-rpc
        sudo cp monero-x86_64-linux-gnu-v0.18.1.0/monero-wallet-rpc /usr/local/bin
        rm monero-linux-x64-v0.18.1.0.tar.bz2
        rm -rf monero-x86_64-linux-gnu-v0.18.1.0
        ```

3. 验证二进制文件是否正常工作

    ```bash
    monero-wallet-rpc --version
    ```

{{< figure src="/run-an-atomic-swap-provider-advanced/monero-wallet-rpc-version.png" align="center" style="border-radius: 8px;" >}}

## Tor守护进程

- 如果您使用的是Debian，只需运行以下命令来安装并启动Tor

    ```bash
    sudo apt-get install tor
    sudo systemctl enable tor
    sudo systemctl start tor
    ```

- 如果您使用的是Ubuntu，请按照Tor提供的官方文档使用Tor提供的仓库
  - <https://support.torproject.org/apt/tor-deb-repo/>
  - 配置好Tor仓库后，运行以下命令来安装并启动Tor

    ```bash
    sudo apt install tor deb.torproject.org-keyring
    sudo systemctl enable tor
    sudo systemctl start tor
    ```

- 如果您使用的是CentOS/RHEL，请按照Tor提供的官方文档使用Tor提供的仓库
  - <https://support.torproject.org/rpm/>
  - 配置好Tor仓库后，运行以下命令来安装并启动Tor

    ```bash
    sudo yum install tor
    sudo systemctl enable tor
    sudo systemctl start tor
    ```

# 通过UFW进行初始强化

我们希望通过确保防火墙仅允许SSH和`asb`所需的端口访问，使用UFW以简单的方式强化系统。

有关开始使用UFW的优秀介绍，请参见[Landchad.net](https://landchad.net/ufw)。

运行以下命令来添加一些基本的UFW规则并启用防火墙：

```bash
# 拒绝所有非明确允许的端口
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 允许SSH访问
sudo ufw allow ssh

# 允许默认的ASB端口（如果仅在Tor上运行，请删除以下两行，因为它们不需要）
sudo ufw allow 9939/tcp
sudo ufw allow 9940/tcp

# 启用UFW
sudo ufw enable
```

# 配置工具

## 设置asb用户和目录

我们将为这两个工具和下面需要的目录设置一个唯一的用户。

```bash
# 创建一个系统用户和组来运行asb和monero-wallet-rpc
sudo addgroup --system asb
sudo adduser --system asb --home /var/lib/asb

# 创建asb工具所需的目录
sudo mkdir /var/run/asb
sudo mkdir /var/log/asb
sudo mkdir /etc/asb

# 设置新目录的权限
sudo chown asb:asb /var/run/asb
sudo chown asb:asb /var/log/asb
sudo chown -R asb:asb /etc/asb
```

## monero-wallet-rpc systemd配置

`monero-wallet-rpc`是`asb`工具用来连接门罗币区块链、管理门罗币资金并在每次交换时根据需要签名/发送交易的工具。

运行此工具并使用正确选项的最简单方法是简单地复制下面的systemd脚本内容并将其保存到`/etc/systemd/system/monero-wallet-rpc.service`中，使用vim或nano：

```bash
sudo nano /etc/systemd/system/monero-wallet-rpc.service
```

*要退出nano shell并保存文件，按`ctrl+x`。*

***注意：如果您没有在同一主机上运行门罗币节点，请确保将`127.0.0.1:18089` daemon-host参数替换为适当的门罗币守护程序URL，例如`node.sethforprivacy.com:18089`。***

```systemd
[Unit]
Description=Monero Wallet RPC (Mainnet)
After=network.target

[Service]
# 进程管理
####################

Type=forking
PIDFile=/var/run/asb/monero-wallet-rpc.pid
ExecStart=/usr/local/bin/monero-wallet-rpc --pidfile /var/run/asb/monero-wallet-rpc.pid --daemon-host 127.0.0.1:18089 --rpc-bind-port 18083 --disable-rpc-login --wallet-dir /etc/asb --detach --log-file /var/log/asb/monero-wallet-rpc.log
Restart=on-failure
RestartSec=30

# 目录创建和权限
####################################

# 以asb:asb运行
User=asb
Group=asb

# /run/asb
RuntimeDirectory=asb
RuntimeDirectoryMode=0710

# /var/lib/asb
StateDirectory=asb
StateDirectoryMode=0710

# /var/log/asb
LogsDirectory=asb
LogsDirectoryMode=0710

# /etc/asb
ConfigurationDirectory=asb
ConfigurationDirectoryMode=0710

# 强化措施
####################

# 提供私有的/tmp和/var/tmp。
PrivateTmp=true

# 将/usr、/boot/和/etc挂载为只读。
ProtectSystem=full

# 拒绝访问/home、/root和/run/user。
ProtectHome=true

# 禁止进程及其所有子进程通过execve()获得新权限。
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

## 自动交换经纪人（ASB）systemd配置

运行此工具并使用正确选项的最简单方法是简单地复制下面的systemd脚本内容并将其保存到`/etc/systemd/system/asb.service`中，使用vim或nano：

```bash
sudo nano /etc/systemd/system/asb.service
```

*要退出nano shell并保存文件，按`ctrl+x`。*

```systemd
[Unit]
Description=Automated swap broker (ASB)
After=network.target monero-wallet-rpc.service

[Service]
# 进程管理
####################

Type=simple
ExecStart=/usr/local/bin/asb --config /etc/asb/config.toml start
StandardOutput=append:/var/log/asb/asb.log

# 目录创建和权限
####################################

# 以asb:asb运行
User=asb
Group=asb

# /var/log/asb
LogsDirectory=asb
LogsDirectoryMode=0710

# /etc/asb
ConfigurationDirectory=asb
ConfigurationDirectoryMode=0710

# 强化措施
####################

# 提供私有的/tmp和/var/tmp。
PrivateTmp=true

# 将/usr、/boot/和/etc挂载为只读。
ProtectSystem=full

# 拒绝访问/home、/root和/run/user。
ProtectHome=true

# 禁止进程及其所有子进程通过execve()获得新权限。
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

## ASB配置文件

此配置文件将决定`asb`工具的工作方式，因此请确保根据需要更改参数。

以下是您*必须*更改的关键参数：

- `external_addresses`应反映可访问的外部地址
  - 如果进行Tor-only ASB，您需要启动ASB一次，复制那里列出的`/onion3/`地址，并像这样添加它们：

    ```conf
    external_addresses = ["/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9939", "/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9940"]
    ```

  - 如果使用IPv4地址而不使用DNS，使用类似条目：

    ```conf
    external_addresses = ["/ip4/5.9.120.18/tcp/9939", "/ip4/5.9.120.18/tcp/9940/ws"]
    ```

  - 如果使用DNS，使用类似条目：

    ```conf
    external_addresses = ["/dns4/swap.sethforprivacy.com/tcp/9939", "/dns4/swap.sethforprivacy.com/tcp/9940/ws"]
    ```

  - 如果您熟悉高级DNS配置，可以探索使用`/dnsaddr`格式，参考以下文档：
    - <https://github.com/multiformats/multiaddr/blob/master/protocols/DNSADDR.md>

以下是一些您应该更改的关键参数：

- `min_buy_btc`应反映您希望交换参与者能够提供的最小BTC数量
- `max_buy_btc`应反映您希望交换参与者能够提供的最大BTC数量
- `ask_spread` 应设置为您希望的点差（在市场价格之上收取的百分比）
  - `0.05` 等于 5%，`0.10` 等于 10%，依此类推
- `electrum_rpc_url` 如果您运行自己的 Electrum 服务器，或者有一个您信任的 Electrum 服务器，可以替换默认的 URL

1. 创建 asb 进程的配置

    ```bash
    sudo nano /etc/asb/config.toml
    ```

    *要退出 nano shell 并保存文件，按 `ctrl+x`。*

    ```ini
    [data]
    dir = "/etc/asb"

    [network]
    listen = ["/ip4/0.0.0.0/tcp/9939", "/ip4/0.0.0.0/tcp/9940/ws"]
    rendezvous_point = "/dnsaddr/swap.sethforprivacy.com/p2p/12D3KooWCULyZKuV9YEkb6BX8FuwajdvktSzmMg4U5ZX2uYZjHeu"
    # 示例 external_addresses: 
    external_addresses = ["/onion3/example.onion/tcp/9939", "/onion3/example.onion/tcp/9940/ws"]

    [bitcoin]
    electrum_rpc_url = "ssl://electrum.blockstream.info:50002"
    target_block = 3
    network = "Mainnet"

    [monero]
    wallet_rpc_url = "http://127.0.0.1:18083/json_rpc"
    network = "Mainnet"

    [tor]
    control_port = 9051
    socks5_port = 9050

    [maker]
    min_buy_btc = 0.0005
    max_buy_btc = 0.001
    ask_spread = 0.05
    price_ticker_ws_url = "wss://ws.kraken.com/"
    ```

    其他推荐的 rendezvous 节点，可以在上述配置中替换我的节点：

    ```bash
    /dns4/rendezvous.xmr.radio/tcp/8888/p2p/12D3KooWN3n2MioS515ek6LoUBNwFKxtG2ribRpFkVwJufSr7ro7
    ```

2. 重新加载 `systemd` 以启用新的 systemd 脚本：

    ```bash
    sudo systemctl daemon-reload
    ```

## Tor 配置

为了使 `asb` 工具能够正确配置其隐藏服务，您需要在 `/etc/tor/torrc` 文件中添加 3 行，将新创建的用户添加到 `debian-tor` 组，并重新启动 `tor`。

1. 使用 vim 或 nano 编辑 `/etc/tor/torrc` 文件，配置 Tor 允许 `asb` 设置和配置隐藏服务：

    ```bash
    sudo nano /etc/tor/torrc
    ```

    *要退出 nano shell 并保存文件，按 `ctrl+x`。*

    ```conf
    # 允许 asb 工具配置隐藏服务
    ControlPort 9051
    CookieAuthentication 1
    CookieAuthFileGroupReadable 1
    ```

2. 运行以下命令将 `asb` 用户添加到 `debian-tor` 组并重新启动 `tor`：

    ```bash
    sudo adduser asb debian-tor
    sudo systemctl restart tor
    ```

# 使用工具

## 启动 monero-wallet-rpc 和 asb

为了启动工具，只需运行以下命令：

1. `monero-wallet-rpc` 应始终首先启动：

    ```bash
    sudo systemctl start monero-wallet-rpc
    ```

2. 然后启动 `asb`：

    ```bash
    sudo systemctl start asb
    ```

## 重新启动 monero-wallet-rpc 和 asb

为了重新启动工具，只需运行以下命令：

1. `monero-wallet-rpc` 应始终首先重新启动：

    ```bash
    sudo systemctl restart monero-wallet-rpc
    ```

2. 然后重新启动 `asb`：

    ```bash
    sudo systemctl restart asb
    ```

# 为您的门罗币钱包充值

在启动时，ASB 工具将提供一个门罗币地址，用于向门罗币钱包充值。

1. 要获取地址，运行以下命令：

    ```bash
    sudo grep monero_address /var/log/asb/asb.log
    ```

2. 确保保存地址，因为一旦您充值资金，它将不再显示。要在您的机器上本地获取地址的二维码，可以运行以下命令（当然，将地址替换为您上面获取的地址）：

    ```bash
    qrencode "4A4tLy1b2PFFdHHvZubb85enYMroBZ3b3i8AV45gBATb2Kas1jNmVP3BwGq4HhSMwsfuedh2hK6MBMmG8M6KAvGGDVBqLDw" -t ascii -o -
    ```

    如果 `qrencode` 未安装，可以使用 `sudo apt install qrencode` 或 `sudo dnf install qrencode` 安装

    如果您在充值资金之前未能保存地址，可以直接从 `monero-wallet-rpc` 获取地址，运行以下命令：

    ```bash
    curl http://127.0.0.1:18083/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"get_address","params":{"account_index":0,"address_index":[0]}}' -H 'Content-Type: application/json'
    ```

3. 向提供的地址发送门罗币，但请记住，这个钱包将是热钱包且没有密码保护 -- 您应尽量减少 ASB 钱包中的门罗币数量。

# 将您的新 ASB 添加到 unstoppableswap.net

***注意：目前只能添加基于 IPv4 和 DNS 地址的 ASB，因此如果您正在进行仅限洋葱的 ASB，请暂时跳过此步骤。***

1. 导航到 <https://unstoppableswap.net/>
2. 点击“Swap provider”框

    {{< figure src="/run-an-atomic-swap-provider-advanced/unstoppable-home.png" align="center" style="border-radius: 8px;" >}}

3. 点击“Submit a swap provider”

    {{< figure src="/run-an-atomic-swap-provider-advanced/unstoppable-providers.png" align="center" style="border-radius: 8px;" >}}

4. 输入您的 external_address 和 peer ID

    要获取您的 peer ID，只需运行以下命令：

    ```bash
    sudo grep peer_id /var/log/asb/asb.log
    ```

    {{< figure src="/run-an-atomic-swap-provider-advanced/unstoppable-submit.png" align="center" style="border-radius: 8px;" >}}

5. 点击“Submit”

# 处理交换过程中的问题

重要的是要关注日志，特别是在前几次交换中，以确保您的配置正常。

要查看日志，只需运行以下命令：

```bash
sudo tail -f /var/log/asb/asb.log
```

如果在失败的交换中看到以 `ERROR` 开头的行，请在以下 URL 中搜索已报告的现有问题：

- <https://github.com/comit-network/xmr-btc-swap/issues>

如果没有针对您遇到的问题的开放问题，请务必打开一个新问题，并尽可能详细地提供以下信息：

- `asb` 的版本
  - 通过运行 `asb --version` 获取
- 失败交换/问题周围的所有日志行
  - 请确保删除 IP 地址、交换 ID 等，如本文开头所述！
- 您运行 `asb` 的操作系统和版本
- 您可以提供的有关问题的任何其他详细信息

大多数问题可以通过简单地重新启动 `asb` 工具来解决，但在重新启动之前收集日志，以确保您以后可以追踪问题。

# 从 ASB 钱包中提取比特币

当交换完成时，`asb` 工具将比特币存储在内部钱包中。每当您想从该钱包中提取 BTC 时，您需要停止 `asb` 工具，提取 BTC，然后重新启动 `asb` 工具。

为此，运行以下命令，替换您的比特币地址和首选金额：

```bash
sudo systemctl stop asb
asb withdraw-btc --address BITCOINADDRESS --amount "0.XX BTC"
sudo systemctl start asb
```

如果您想提取全部余额，只需运行：

*注意：目前有一个错误阻止此命令工作，因此现在只需通过上述命令集提取金额：<https://github.com/comit-network/xmr-btc-swap/issues/662>*

```bash
sudo systemctl stop asb
asb withdraw-btc --address <BITCOINADDRESS>
sudo systemctl start asb
```

# 检查比特币和门罗币余额

检查两个钱包中当前余额的最简单方法是停止 ASB 进程并运行 `asb balance`：

```bash
sudo systemctl stop asb
asb balance
sudo systemctl start asb
```

如果您不想停止 ASB，可以运行以下命令检查门罗币余额：

```bash
curl http://127.0.0.1:18083/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"get_balance","params":{"account_index":0,"address_indices":[0]}}' -H 'Content-Type: application/json'
```

# 升级工具

两个工具都需要保持最新，因此为了简化过程，这里有一组快速命令来更新两者。只需将下载 URL 替换为最新版本的 URL。

## monero-wallet-rpc

```bash
cd ~/asb
wget https://downloads.getmonero.org/cli/monero-linux-x64-0.18.1.0.tar.bz2
tar xvf monero-linux-x64-0.18.1.0.tar.bz2
sudo chmod +x monero-x86_64-linux-gnu-0.18.1.0/monero-wallet-rpc
sudo mv -f monero-x86_64-linux-gnu-0.18.1.0/monero-wallet-rpc /usr/local/bin/
rm monero-linux-x64-0.18.1.0.tar.bz2
rm -rf monero-x86_64-linux-gnu-0.18.1.0
```

## asb

```bash
cd ~/asb
wget https://github.com/comit-network/xmr-btc-swap/releases/download/0.11.0/asb_0.11.0_Linux_x86_64.tar
tar xvf asb_0.11.0_Linux_x86_64.tar
rm asb_0.11.0_Linux_x86_64.tar
sudo chmod +x asb
sudo mv -f asb /usr/local/bin/
```

# 高级配置选项

本节在启动 ASB 时绝对不需要遵循，但将包含一些作为 ASB 所有者可用的高级选项。

## 使用 /dnsaddr 格式作为 external_address

`libp2p` 的一个很酷的功能是能够使用一个统一的地址来描述您的 ASB 所有可能的可达方法。这允许您的 ASB 通过 IP、DNS 和 Onionv3 可达，同时为交换用户提供一个单一的统一地址，使他们的客户端能够根据其网络配置和/或 Tor 使用情况选择最佳选项。

有关规范和可用/需要的配置的更多详细信息，请参见官方文档：<https://github.com/multiformats/multiaddr/blob/master/protocols/DNSADDR.md>

要配置此功能，您需要为您的域名添加 TXT DNS 记录，每个地址对应您希望通过 `/dnsaddr` 条目广告的地址。

1. 配置所需的可达性通过 `listen` 和 Tor 配置

    通常这只是默认的监听选项并启用之前提到的 Tor 配置。

2. 通过您的域名提供商添加 TXT 记录，主机条目为 `_dnsaddr`，记录如下，根据您的洋葱地址和其他首选可达端点进行定制：

    *注意：此示例适用于 Namecheap，但所有域名提供商应允许类似的 TXT 记录定制。*

    {{< figure src="/run-an-atomic-swap-provider-advanced/dnsaddr-entries.png" align="center" style="border-radius: 8px;" >}}

    ```bash
    dnsaddr=/ip4/5.9.120.18/tcp/9939/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW
    dnsaddr=/ip4/5.9.120.18/tcp/9940/ws/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW
    dnsaddr=/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9939/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW
    dnsaddr=/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9940/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW
    ```

    每个条目必须以 `dnsaddr=` 开头，并包含如上所示的 `/p2p/peer_id` 后缀。

3. 通过 `dig` 和 `swap` 测试验证 DNS 条目是否正常工作

    `dig +short txt _dnsaddr.DOMAIN.NAME` 应返回类似以下的输出：

    ```bash
    dig +short txt _dnsaddr.swap.sethforprivacy.com
    "dnsaddr=/ip4/5.9.120.18/tcp/9939/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW"
    "dnsaddr=/ip4/5.9.120.18/tcp/9940/ws/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW"
    "dnsaddr=/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9939/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW"
    "dnsaddr=/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9940/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW"
    ```

    针对您注册的 rendezvous 点进行测试，并验证 ASB 显示为在线：

    ```bash
    ./swap list-sellers --rendezvous-point /dnsaddr/swap.sethforprivacy.com/p2p/12D3KooWCULyZKuV9YEkb6BX8FuwajdvktSzmMg4U5ZX2uYZjHeu
    ```

# 免责声明

虽然此软件对我来说运行良好，并且可以用于主网，但它当然不是万无一失的，并且仍在积极开发中。交换应始终以完全交换或双方取回资金结束，但请注意可能存在的错误，并且您在原子交换方面非常早期。

我对因交换中涉及的比特币/门罗币处理而导致的任何资金丢失或问题不承担责任，但如果您遇到问题，我会尽力帮助。

# 结论

希望这已经是一个不错的（相对）简单指南，让您开始为那些希望通过原子交换无信任地交换比特币换门罗币的人提供门罗币流动性！原子交换是消除交易所信任和消除潜在未来监管权力的重要工具，因此我对它们终于成为可能并且运行良好感到非常兴奋。

如果您有具体问题或需要帮助，请通过 [Signal, Matrix, Threema, 或电子邮件]({{< ref "/content/about.md#how-to-contact-me" >}}) 联系我。
