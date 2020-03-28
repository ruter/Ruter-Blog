---
layout: post
title: 基于星云链开发智能合约和DApp
categories: [Blockchain]
description: 基于星云链开发智能合约和DApp过程中的要点和注意事项
keywords: DApp, 区块链, 智能合约
---

最近一段时间，除了上班加班之外，基本上都在忙着开发 DApp，也就是所谓的去中心化应用（Decentralized Applications）啦，为什么突然就搞起这个了呢？事情是这样的……

就在前不久，很偶然地了解到了 Loom Network 这家公司和 DApp 这个概念，然后花了两天业余时间（合着也就几个小时吧）跟着 Loom 的基础教程「[Learn to Code Ethereum DApps By Building Your Own Game](https://cryptozombies.io/)」学习了如何编写以太坊的智能合约，觉得十分有趣，就开始关注起了 DApp 和区块链智能合约开发相关的信息。

刚好又过了一段时间，看到了星云（Nebulas）发布的「[星云激励计划第一季](https://blog.nebulas.io/2018/04/29/implementation-details-of-nebulas-incentive-program-season-1/)」活动，就是鼓励开发者们基于星云链主网开发去中心化应用（DApp）的活动，里面也包含了不小的奖励。一开始我并没有打算参与的，然后有一点没一点地看了些官方文档什么的，然后发现好像难度并不高，而且刚好有了 idea 可以实现，于是乎就从画脑图开始了第一次基于区块链的去中心化应用开发之旅。

在两周不到的时间里，我成功开发并提交了两个 DApp，分别是「[星云宠物卡](https://petcard.ruterly.com/)」和「[星云打卡](https://punch.ruterly.com/)」这两款小应用，这两个 DApp 的合约以及前端代码均已开源在了我的 GitHub 仓库中，希望可以给各位一点帮助，虽然代码写得并不漂亮。你可以发现在「星云宠物卡」和「星云打卡」这两个项目里，相同的功能逻辑会有些不一样的处理方式，后者应该是优于前者的。

两个 DApp 的仓库传送门：

- [星云宠物卡](https://github.com/ruter/nebulas-pet-card)
- [星云打卡](https://github.com/ruter/nebulas-punch-in)

如果这两个小项目能给到你一点启发，请在我开发的两个 DApp 中使用试试，同时给我的两个项目 Star 一个🌟

在这篇文章里我不会手把手地写一篇如何基于星云链开发 DApp 的教程，我会把一些有用的资料以及踩到的坑进行记录和整理，以方便读者们查阅和学习。如果你对这个开发活动感兴趣，欢迎使用我的邀请链接进行注册「[点我注册](https://incentive.nebulas.io/cn/signup.html?invite=8Kkw7)」，这样当你的应用通过官方审核之后，我会获得 40 NAS 的奖励，而你除了获得审核通过之后的 100 NAS 奖励外还会额外获得 10 NAS（仅通过我的邀请链接注册且通过应用审核）的奖励。

# 在开发之前

在你即将开始开发之前，我的建议是先看一下官方博客发布的几篇文章，这些文章我会在后文中的「开发教程」下列出。除了看官方教程学习之外，你还应该准备好开发相关的工具，其中最重要的就是「[星云 Web 钱包](https://github.com/nebulasio/web-wallet)」了，因为它是用来创建钱包、部署合约、执行合约函数进行调试的工具。

# 开始开发

开发的基本流程可以大致分为「编写合约」——「部署合约」——「测试合约」——「编写前端页面」——「修改合约」这五个步骤。

在编写合约之前，请大致过一遍星云的「[智能合约](https://github.com/nebulasio/wiki/blob/master/smart_contract_ch.md)」相关 API，以便更好地熟悉相关接口进行合约的编写，合约的编写难度不高，需要注意的是，在合约中和转账数值相关的，都是使用的基本单位 Wei，并且都是整数，很多人会以 NAS 作为单位编写相关逻辑导致出现一些错误。

合约编写完成之后，使用 Web 钱包，将右上角的「Mainnet」改为「Testnet」，然后选择「合约」——「部署」将你的合约部署到测试网中进行测试，部署合约需要花费一定的 Gas，你可以在官网[领取](https://testnet.nebulas.io/claim/)测试用的 NAS 到你的钱包中用于部署测试。

在你要编写前端页面和你部署的合约进行交互前，你可能会用到以下两个库：

- [neb.js](https://github.com/nebulasio/neb.js)
- [nebPay.js](https://github.com/nebulasio/nebPay)

这两个库分别是对星云 API 的封装以及专门用于发起交易的库，其中后者可以和浏览器钱包插件进行交互，可以唤起钱包发起交易等。

关于 nebPay.js 和浏览器钱包插件的交互，这里给大家划一个重点，要好好看一下浏览器钱包插件的例子，特别是「[这一个例子](https://github.com/ChengOrangeJu/WebExtensionWallet/blob/master/example/TestPage_old.html)」里的第 85 ~ 97 行，`getAccount` 这个方法是用于直接从钱包插件中获取当前用户的，因为当时没有好好看例子，不知道有这个方法，于是乎在我开发的第一个应用时就用了个蹩脚的实现方式，就是在合约中编写一个 `getAccount` 的方法，然后在需要直接获取用户当前钱包插件中解锁的钱包地址时，通过 nebPay.js 的 `simulateCall` 方法模拟调用，从而获取到当前用户的地址。这个教训告诉我们，**要好好地看文档和例子！**

# 合约开发中的注意点

如果你的合约中有一些管理员专属的管理用接口，建议不要将管理员的地址硬编码在合约中，这是为了应对钱包私钥泄露的意外情况下在出问题前将管理员地址及时修改，保证合约和相关内容的安全，推荐的做法是定义一个管理员地址的属性 `superuserAddress` 在合约中，然后再定义一个修改管理员地址的函数 `setSuperuser`：

    var YourContract = function () {
        LocalContractStorage.defineProperties(this, {
            superuserAddress: null,
            otherProperty: null
        });
    };
    
    YourContract.prototype = {
    
        init: function () {
            this.superuserAddress = Blockchain.transaction.from;
            this.otherProperty = 'xxx';
        },
    
        setSuperuser: function(address) {
            if (Blockchain.transaction.from === this.superuserAddress) {
                this.superuserAddress = address;
            } else {
                throw new Error("Permission Denied");
            }
        }
    };
    
    module.exports = YourContract;

合约开发中难免会遇上需要转入和转出代币的需求，在这种时候尤其要注意做好相关的记录和判断，一个用户往合约中转入多少代币，满足何种条件后他可以从中取走多少代币，在合约中都应该有相关的属性去记录，以及在转出时判断转出的金额是否是其所应得的，否则会出现用户盗刷合约中的代币的情况，请牢牢地记住一条编码原则：永远不要相信用户的输入。

另外要提醒大家，如果你的应用有转账到合约中去的相关功能，请务必一定要留一个转出的方法，防止出现转入的代币无法取出的情况出现。

这里暂时先写这么多，如果你有什么疑惑，欢迎留言，我将会倾囊相授，知无不言。最后再多说一句，请一定好仔细翻看官方文档和例子，希望各位能够开发出优秀的应用拿到大奖😆

# 活动细则

- [星云激励计划第一季细则](https://blog.nebulas.io/2018/04/29/implementation-details-of-nebulas-incentive-program-season-1/)
- [星云激励计划第一季实施细则官方解读](https://blog.nebulas.io/2018/05/02/official-interpretation-of-nebulas-incentive-program-implementation-details-season-1/)
- [星云激励计划 Q&A](https://blog.nebulas.io/2018/05/03/nebulas-incentive-program-qa/)

# 开发资源

- [星云官方 Wiki](https://github.com/nebulasio/wiki/wiki)
- [Chrome 浏览器钱包插件](https://github.com/ChengOrangeJu/WebExtensionWallet)
- [星云 Web 钱包](https://github.com/nebulasio/web-wallet)
- [neb.js](https://github.com/nebulasio/neb.js)
- [nebPay.js](https://github.com/nebulasio/nebPay)
- [智能合约](https://github.com/nebulasio/wiki/blob/master/smart_contract_ch.md)

# 开发教程

- [手把手教你一小时开发DApp](https://blog.nebulas.io/2018/04/28/nebulas-incentive-program%E2%80%8A-%E2%80%8A-demo/)
- [手把手教你星云DApp开发（第一部分）](https://blog.nebulas.io/2018/05/04/how-to-build-a-dapp-on-nebulas-part-1/)
- [手把手教你星云DApp开发（第二部分)](https://blog.nebulas.io/2018/05/05/how-to-build-a-dapp-on-nebulas-part-2/)
- [如何在Dapp中使用NebPay SDK](https://blog.nebulas.io/2018/05/09/how-to-use-nebpay-in-your-dapp/)

# 钱包知识

- [星云Web钱包教程1：创建NAS钱包](https://blog.nebulas.io/2018/04/12/creating-a-nas-wallet/)
- [星云Web钱包教程2：发起转账](https://blog.nebulas.io/2018/04/17/sending-nas-from-your-wallet/)
- [星云Web钱包教程3：离线签名交易](https://blog.nebulas.io/2018/04/18/signing-a-transaction-offline/)
- [星云Web钱包教程4：查看钱包信息](https://blog.nebulas.io/2018/04/19/view-wallet-information/)
- [星云Web钱包教程5：查看交易状态](https://blog.nebulas.io/2018/04/28/check-tx-status/)
- [星云Web钱包教程6：部署智能合约](https://blog.nebulas.io/2018/04/28/deploy-a-smart-contract/)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2018/05/15/Develop-Nebulas-DApps/
