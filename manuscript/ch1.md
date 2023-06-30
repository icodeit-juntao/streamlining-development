# 前言

这是一本关于如何使用命令行来完成开发任务自动化的小书。这本书的主要目的是通过一个实际的例子，教会你如何定义构建脚本，如何在本地和持续集成环境中使用这些脚本，实现诸如转译（翻译+编译），静态检查，打包，自动化测试，自动化部署等常见的开发任务，从而实现持续交付，以提升效率，保证产品质量。

持续集成以及持续交付是非常大的体系，通过一本小书来介绍其各个方面是不现实的，也并非我的初衷。我希望通过循序渐进的方式，向读者介绍CI/CD背后的机制：自动化脚本在软件开发中的重要作用。我会通过实现一个不复杂，但是又涉及诸多元素的应用程序来完成讲述。

## 预先知识

在这本书中，我假设你对编程有一定的了解，你需要了解Node环境，并且你的本地已经安装了[Node.js](https://nodejs.org/en)的当前版本。如果你在本地的Terminal中（假设你在Mac OSX环境下）执行

```bash
npm --version
```

可以得到类似 `9.5.0` 这样的版本号，说明你已经具备了一切环境。除此之外，你需要了解一些基本的React知识。这个并非重点，你只需要学习过这个[快速指南](https://react.dev/learn)就足够继续阅读，毕竟本书的目的是命令行和持续集成环境。

最后，你需要在[Github](https://github.com/)的账号，并且正确配置了ssh的公私钥。为了确认你的本地 SSH key 可以连接到 GitHub，你可以按照以下步骤进行：

1. 在本地命令行中运行以下命令：
    
    ```bash
    ssh -T git@github.com
    ```
    
    该命令会尝试通过 SSH 连接到 GitHub。`-T` 选项的作用是禁止 ssh 从请求一个 shell，这样我们就可以简单地测试连接性而不是尝试创建一个 shell。
    
2. 在你首次连接到 GitHub 的时候，你可能会看到一个警告信息，像这样：
    
    ```
    The authenticity of host 'github.com (IP ADDRESS)' can't be established.
    RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
    Are you sure you want to continue connecting (yes/no)?
    ```
    
    在这种情况下，你应该输入 "yes"，然后按回车。
    
3. 如果你的 SSH key 设置正确，你应该会看到一个类似这样的消息：
    
    ```
    Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
    ```
    
    其中的 "USERNAME" 就是你的 GitHub 用户名。这意味着 SSH 已经成功地连接到了 GitHub。如果你看到了这个信息，就说明你的本地 SSH key 可以成功地连接到 GitHub。
    
4. 如果你的 SSH key 没有正确设置，你可能会看到 "Permission denied" 的消息。如果出现这种情况，你可能需要重新生成你的 SSH key，或者确认你的公钥是否已经添加到了你的 GitHub 账户上。你可以查看 GitHub 的帮助文档，以获取更多有关设置 SSH 的信息。

## 今日箴言应用

我们要实现的应用叫：*今日箴言*。这是一个很简单的应用程序，它会随机的选取一条**名人名言**，并显示在一张卡片上。卡片底部有两个按钮，一个是将卡片以图片的形式下载到本地，另一个是刷新卡片，随机的显示另一条名人名言。

![Quote of the day](ch1/quote-of-the-day.png)

在实现这个应用的过程中，我们会涵盖这样一些内容：

- 解开命令行工具和Shell脚本的奥秘：掌握命令行的基本操作，理解Shell脚本的功能和使用场景。
- 利用npm scripts自动化你的开发流程：详解如何利用npm scripts定义和执行构建脚本，让繁琐的开发任务得以自动化。
- 编写用户验收测试，保障产品质量：介绍如何编写用户验收测试，提前发现并修复问题，确保产品功能的正确性和稳定性。
- 将React代码编译为浏览器可以运行的JavaScript：讲解如何将React代码编译为JavaScript，进一步理解React应用的运行原理。
- 调用RESTful API，与后端数据无缝对接：教授如何通过调用RESTful API，实现前后端数据的交互和展示。
- 使用Github Actions打造自动化的持续集成/交付流程：详解如何利用Github Actions实现持续集成和持续交付，提升软件开发的效率和质量。

通过这本书，你不仅将掌握这些技能，还将理解它们如何在实际项目中发挥作用，以及如何将它们融合到你的开发流程中。