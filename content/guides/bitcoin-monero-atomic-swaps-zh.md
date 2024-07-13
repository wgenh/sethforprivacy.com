---
author: Seth For Privacy
comments: false
date: "2021-08-17T10:00:00-04:00"
summary: 期待很久了，但它终于来了。*今天*，你可以直接点对点交换比特币 <> 门罗币，通过Tor，无需托管人或受信任的第3方，无需KYC（了解你的客户）资讯，什么都不需要。
draft: false
hidemeta: false
showToc: true
tags:
- 原子交换
- 比特币
- 门罗币
title: 执行比特币 <> 门罗原子交换
---

# 引言

这一刻已经等待了很久，但终于到来了。今天，你可以直接通过Tor进行点对点的比特币与门罗币交换，无需托管或信任第三方，无需KYC（了解你的客户）信息，什么都不需要。本指南将引导你作为门罗币买家，或向门罗币提供者出售比特币。

这是跨链交换的未来，现在就可以实现。

原子交换开启了广泛的跨链应用场景，但关键在于它们是无信任、不可审查且完全匿名/伪匿名的。

更多关于原子交换的信息，请查看以下链接：

- <https://localmonero.co/knowledge/monero-atomic-swaps>
- <https://www.monerooutreach.org/stories/monero-atomic-swaps.html>
- <https://github.com/comit-network/xmr-btc-swap>
- <https://comit.network/blog/2020/10/06/monero-bitcoin/>

# 工具

对于这个特定的原子交换实现，我们将使用[COMIT网络](https://comit.network/)提供的工具，这些工具可以在他们的Github仓库发布部分找到：

<https://github.com/comit-network/xmr-btc-swap/releases>

我（希望很快其他人也能）正在运行提供门罗币交换侧的`asb`工具，因此为了深入并购买门罗币，你只需下载最新版本的`swap`工具。

我提供的说明假设你使用的是Linux，但Windows/macOS应该类似。

# 获取工具

1. 下载最新版本的`swap`工具，例如`swap_0.11.0_Linux_x86_64.tar`

    <https://github.com/comit-network/xmr-btc-swap/releases/latest>

2. 解压swap二进制文件
    打开终端并运行以下命令：

    ```bash
    cd ~/Downloads
    tar xvf swap_0.11.0_Linux_x86_64.tar
    ```

3. 验证二进制文件是否正常工作

    ```bash
    ./swap --version
    ```

    {{< figure src="/bitcoin-monero-atomic-swaps/version.png" align="center" style="border-radius: 8px;" >}}

# 进行交换（CLI）

1. 列出卖家并选择一个（如果你知道想要交换的对等方，可以跳过此步骤）

    原始测试汇合点：

    ```bash
    ./swap list-sellers --rendezvous-point /dnsaddr/rendezvous.coblox.tech/p2p/12D3KooWQUt9DkNZxEn2R5ymJzWj15MpG6mTW84kyd8vDaRZi46o
    ```

    我的生产汇合点：

    ```bash
    ./swap list-sellers --rendezvous-point /dnsaddr/swap.sethforprivacy.com/p2p/12D3KooWCULyZKuV9YEkb6BX8FuwajdvktSzmMg4U5ZX2uYZjHeu
    ```

    其他推荐的汇合节点：

    ```bash
    /dns4/rendezvous.xmr.radio/tcp/8888/p2p/12D3KooWN3n2MioS515ek6LoUBNwFKxtG2ribRpFkVwJufSr7ro7
    ```

    此命令将列出可用的卖家、他们的最小/最大交易金额以及当前提供的交易价格

    {{< figure src="/bitcoin-monero-atomic-swaps/list-sellers.png" align="center" style="border-radius: 8px;" >}}

2. 开始交换（这里使用我的对等方，根据需要替换为首选对等方）

    ```bash
    ./swap buy-xmr --receive-address <你的门罗币地址> --change-address <你的比特币找零地址> --seller <卖家地址>
    ```

    ***不要忘记将门罗币和比特币地址占位符替换为你自己的！***

    {{< figure src="/bitcoin-monero-atomic-swaps/buy-xmr.png" align="center" style="border-radius: 8px;" >}}

3. 向提供的地址存入比特币，确保发送的金额超过最小值以覆盖交换交易的手续费
4. 观察交换过程

    日志将显示正在进行的步骤，耐心等待，因为比特币和门罗币都需要确认，这可能需要一些时间

5. 获利

# 进行交换（网页界面）

1. 前往 <https://unstoppableswap.net/>
2. 选择首选的交换提供商
3. 输入你希望在交换提供商设定的最小和最大范围内交换的比特币或门罗币数量

    {{< figure src="/bitcoin-monero-atomic-swaps/swap_web.png" align="center" style="border-radius: 8px;" >}}

4. 输入你控制的门罗币和比特币地址

    {{< figure src="/bitcoin-monero-atomic-swaps/addresses_web.png" align="center" style="border-radius: 8px;" >}}

5. 按照指示打开终端

    {{< figure src="/bitcoin-monero-atomic-swaps/instructions_web.png" align="center" style="border-radius: 8px;" >}}

6. 将提供的命令复制粘贴到你的终端并运行该命令

    {{< figure src="/bitcoin-monero-atomic-swaps/cli_web.png" align="center" style="border-radius: 8px;" >}}

7. 向提供的地址存入比特币，确保发送的金额超过最小值以覆盖手续费
8. 观察交换过程
    日志将显示正在进行的步骤，耐心等待，因为比特币和门罗币都需要确认，这可能需要一些时间

    {{< figure src="/bitcoin-monero-atomic-swaps/cli_swap_web.png" align="center" style="border-radius: 8px;" >}}

9. 获利

    {{< figure src="/bitcoin-monero-atomic-swaps/success_web.png" align="center" style="border-radius: 8px;" >}}

# 如果交换失败怎么办

重要的是要意识到，如果在发送比特币后交换失败，你需要及时执行以下两个步骤：

1. 尝试恢复交换

    ```bash
    ./swap resume --swap-id <交换ID>
    ```

2. 如果恢复失败，等待比特币存款交易的72个确认
3. 在比特币存款交易的72个确认后取消交换

    ```bash
    ./swap cancel --swap-id <交换ID>
    ```

4. 在发布取消交易后立即退款，并在比特币取消交易的72个确认之前

    ```bash
    ./swap refund --swap-id <交换ID>
    ```

如果上述步骤未能正确取消和退款交易，并且你确定你等待了所需的时间，请在[Github上提交问题](https://github.com/comit-network/xmr-btc-swap/issues)或在Matrix上联系(`#comit-monero:matrix.org`)寻求帮助*尽快*。

更多关于交换协议步骤的详细信息，请参见<https://comit.network/blog/2020/10/06/monero-bitcoin/#long-story-short>。

***注意：如果在执行取消后72个确认通过，ASB可以选择惩罚未能正确完成的交换，允许他们拿走比特币作为对你未能正确响应的惩罚。确保在发起取消后的72个确认期间内执行退款步骤。***

# 需要知道的事情

- 价格自动从Kraken获取并定期更新，ASB运行者会在市场价格基础上增加一个点差。
- 你提供的比特币找零地址应该是*未使用*的地址，出于隐私考虑。
- 比特币找零地址将在交换失败的情况下用于将资金返还到你自己的钱包。
- 门罗币接收地址应该是每个交换对等方（或每个交换）的子地址，理想情况下。
- 比特币侧需要2个确认，门罗币侧需要10个确认，所以在交换过程中要有耐心，让`swap`工具完成它的工作。如果你在交换过程中需要停止它，可以使用`./swap resume`功能，但理想的做法是让工具一直打开直到交换完成。
- 更多关于交换协议和涉及步骤的信息，请参见<https://comit.network/blog/2020/10/06/monero-bitcoin/>。
- 如果你在交换过程中遇到问题，请在[Github上提交问题](https://github.com/comit-network/xmr-btc-swap/issues)或在Matrix上联系(`#comit-monero:matrix.org`)寻求帮助。

# 免责声明

虽然这个软件对我来说运行良好，并且已经准备好用于主网，但它当然不是万无一失的，仍然在积极开发中。交换应该总是以完全交换或双方都取回资金结束，但要注意可能存在的错误，你*非常*早于一般的原子交换。

我对因交换过程中涉及的比特币/门罗币处理而可能丢失的任何资金或问题不承担责任，但如果遇到问题，我会尽力帮助。

# 结论

希望这是一个简单易懂的指南，帮助你开始通过原子交换无信任地交换比特币和门罗币！原子交换是消除交易所信任和未来潜在监管权力的重要工具，我对它们终于成为可能并且运行良好感到非常兴奋。

如果你有具体问题或需要帮助，请通过[Signal, Matrix, Threema, 或电子邮件]({{< ref "/content/about.md#how-to-contact-me" >}})联系。
