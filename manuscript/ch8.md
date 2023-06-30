# 实现持续交付

在上一章结束的时候，我们本地开发中已经有了很多自动化的机制来保证代码的正确性。而且由于这些配置都是保存在代码库中的，所有的团队成员都有这几乎一样的配置，因而我们的开发和打包方式几乎完美了。

不过这里有一个不容易看出来的问题，而且是一个重大的问题。那就是，我们在本地可以正常运行的程序，在生产环境还可以正常运行吗？我们经常会遇到的一个问题是：我们明明看到测试通过，而且手工测试也没有问题，但是一上环境就出了问题。

这是因为，我们本地的配置可能与生产环境或者测试环境不尽相同。而我们的应用程序又依赖于这些配置。这就会导致“在我电脑上可以工作”(It works on my machine)的问题。

要解决这个问题，我们需要引入持续集成和持续交付的实践。好消息是，基于我们已有的构建脚本，以及GitHub Actions，实现持续集成，持续交付非常容易。

## 持续集成

持续集成（Continuous Integration，简称CI）是一种开发实践，开发人员会频繁地（一天多次）将代码集成到主分支。每次集成都通过自动化的构建（包括编译、发布、自动化测试）来验证，可以尽早地发现并修复集成错误。这让团队可以尽快地发现问题，并让开发者更频繁地提交更小的变更，这减少了找出并解决问题的时间。

1. **Agent**：在 CI 系统中，Agent 指的是执行构建任务的服务器。它接收构建任务，执行构建脚本，然后返回结果。Agent 可以是物理机器，也可以是虚拟机，还可以是容器。
2. **Task**：任务是指定的一系列操作，这些操作会在指定的 Agent 上执行。例如编译代码，运行测试，打包应用等。
3. **构建脚本**：构建脚本是指导 Agent 如何执行构建任务的脚本。构建脚本通常会在源代码中和项目一起维护。
4. **工作原理**：当开发者提交代码到版本控制系统（例如 Git）时，CI 服务会被触发，然后它会在 Agent 上运行构建任务。首先，Agent 会从版本控制系统中获取最新的代码，然后执行构建脚本。构建脚本通常包含编译代码，运行测试，打包应用等步骤。当所有步骤成功完成后，构建任务就会成功。如果有任何步骤失败，构建任务就会失败。

持续集成的主要目标是提供快速反馈，如果构建失败，开发者应该立刻获取通知，以便尽快修复问题。此外，CI 也能提供一个稳定的环境，用来运行各种测试，确保代码质量。为了更好地实现持续集成，团队需要遵循一些最佳实践，例如保持构建快速，避免 "broken builds"，频繁集成代码等。

在本章，我们将探讨如何使用Github Actions来实现持续集成，以及最终的持续交付。

## Github Actions简介

GitHub Actions 是 GitHub 提供的一个自动化工具，让你可以在 GitHub 仓库中直接编写和执行 CI/CD（持续集成/持续交付）工作流。这意味着你可以自动化任何你的软件开发工作流程，包括构建、测试和部署应用程序，甚至更多。

每个 GitHub Action 工作流都由一个或多个 jobs 组成，而每个 job 又由一系列的步骤组成。这些步骤可以是执行命令、运行脚本，或者使用 GitHub 社区分享的 actions。这里的Runner就是传统CI服务器中的Agent，用于具体的执行任务。

![Github Actions结构](ch8/github-actions.png)

这些工作流可以被各种 GitHub 事件触发，例如推送（push）、拉取请求（pull request）或者发布（release）。你也可以调度工作流在特定时间或间隔运行，就像 cron job 一样。

下面是一个简单的 GitHub Actions 工作流的例子，它在每次推送代码到主分支时，都会运行单元测试：

```yml
name: Run tests

on:
  push:
    branches:
      - master

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Check out repository
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v1
      with:
        node-version: 14

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test
```

这个配置文件定义的作业大致过程是：首先，在 Ubuntu 虚拟环境上检出你的代码，然后设置 Node.js 环境，安装依赖项，最后运行测试。让我们逐行详细解释：

- `jobs:`：这是定义工作流作业的开始。在这个级别下，你可以定义一个或多个作业，每个作业都是并行运行的，除非你明确指定它们的依赖关系。
- `test:`：这是作业的名字，可以是任何你选择的名字。在这种情况下，这个作业的目标是运行测试，所以叫做 "test"。
- `runs-on: ubuntu-latest`：这告诉 GitHub Actions 这个作业应该在最新版本的 Ubuntu 虚拟环境上运行。
- `steps:`：这是定义作业中执行的步骤的地方。
- `name: Check out repository`：这是步骤的名字。这个步骤的目的是检出仓库的代码。
- `uses: actions/checkout@v2`：这告诉 GitHub Actions 使用 "actions/checkout@v2" 这个 action。这个 action 会检出你的代码仓库到 runner，所以后续的步骤可以访问它。
- `name: Set up Node.js`：这个步骤设置 Node.js 环境。
- `uses: actions/setup-node@v1`：这个 action 设置了 Node.js 环境。
- `with:`：这是用来传递参数给 action 的。
- `node-version: 14`：这告诉 setup-node action 我们想要设置的 Node.js 的版本。
- `name: Install dependencies`：这个步骤安装项目的依赖项。
- `run: npm ci`：这执行 "npm ci" 命令，它将根据 package-lock.json 文件精确地安装项目的依赖项。
- `name: Run tests`：这个步骤运行测试。
- `run: npm test`：这执行 "npm test" 命令，它将运行定义在 package.json 文件中的测试。

这种直接在代码仓库中定义和运行 CI/CD 工作流的方式，使得开发团队可以更容易地跟踪和管理自动化过程，并使其与代码更紧密地集成。

## 创建工作流

我们现在开始在今日箴言中定义Github Actions工作流。首先创建工作流所需的目录`.github/workflows`，然后创建一个`build.yml`文件：

```bash
mkdir -p .github/workflows
touch .github/workflows/build.yml
```

在`.github/workflows/build.yml`中，我们先定义一个名为`build`的job，其中包含4个步骤：签出代码，配置node.js环境，安装依赖，运行`npm run build`。

```yml
name: Build Quote Application

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup node
        uses: actions/setup-node@v3
        with:
          node-version: 18.14.2
          cache: npm

      - name: Install
        run: npm ci

      - name: Test
        run: npm run lint
```

我们将整个`.github`目录提交，然后推送到Github。这将会触发Github Actions执行我们定义好的工作流。

![Github Actions界面](ch8/build-workflow.png)

可以看到，我们的工作流成功运行。我们可以在这个基础上扩展`test`，加入更多的步骤。一个常见的任务是打包，即将最终构建出来的文件压缩打包，以供部署时使用。

## 生成软件包

我们可以对`build.yml`做一些简单的修改，使得其在完成测试之后，执行`npm run build`来编译源码，并生成一个包。

```yml
name: Build Quote Application

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    name: Build Frontend Package
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup node
        uses: actions/setup-node@v3
        with:
          node-version: 18.14.2
          cache: npm

      - name: Install
        run: npm ci

      - name: Test
        run: npm run lint

      - name: Build
        run: npm run build

      - name: Packaging
        uses: actions/upload-artifact@v3
        with:
          name: quote-of-the-day
          path: dist/
```

我们加入了`build`和`packaging`两个新的步骤，即首先编译应用程序的源码到`dist`目录，然后用upload-artifact将结果打包并压缩到quote-of-the-day中。

![上传生产包](ch8/upload.png)

非常好，这样我们的软件就不再依赖于任何一个开发者的本地环境了。每个开发可能使用Mac，或者Linux甚至是Windows，只要在CI服务器上（即上面的`ubuntu-latest`）使用的环境与生产环境一致，那么这个包就是就绪的。

## 执行验收测试

除此之外，我们还希望在CI环境中执行自动化验收测试。这样当功能被破坏时我们就可以第一时间知道并修复。同时，这些任务需要在打包完成之后执行。如果静态检查失败，或者编译失败，我们就无需执行验收测试。也就是说，运行验收测试的步骤依赖于静态检查和编译过程。

本来运行验收测试是比较复杂的过程：我们需要在合适的时机启动http-server，然后运行测试，然后在测试运行结束之后关闭http-server。不过`cypress`提供了包装好的Github Actions，我们只需调用即可。

我们需要在`build.yml`中添加一个新的任务：

```yml
cypress:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Cypress run
        uses: cypress-io/github-action@v5
        with:
          build: npm run build
          start: npm start
          command: npm run e2e
```

我们定义了一个名为 `cypress` 的新的 GitHub Actions 任务，它是在持续集成过程中运行端到端测试的作业。这里面有一些需要注意的点：

- `needs: build`：这表示该作业依赖于另一个名为 "build" 的作业。只有在 "build" 作业成功完成后，"cypress" 作业才会开始运行。
- `uses: cypress-io/github-action@v5`：这使用了 Cypress 官方提供的 GitHub action。这个 action 设置并运行 Cypress 测试。
- `build: npm run build`：在运行测试之前，这个参数告诉 Cypress action 需要执行 "npm run build" 命令来构建项目。
- `start: npm start`：这个参数告诉 Cypress action 在运行测试之前需要启动应用，这是通过执行 "npm start" 命令完成的。
- `command: npm run e2e`：这个参数定义了实际运行的命令，即 "npm run e2e"，它启动 Cypress 并运行定义在 "e2e" script 中的测试。

非常好！在这些自动化构建脚本和工作流的的支持下，我们的每一次推送（push）都会触发该工作流，在工作流中，静态检查和自动化测试都会执行，以确保每一次的代码改动都可以生成一个可以部署的**就绪软件**。

那么，我们可不可以更进一步，如果每次的所有测试都通过，而且验收测试也通过的话，我们就发布应用到生产环境。这样我们的新功能就可以被真实的用户使用！

## 持续交付

持续交付（Continuous Delivery）是软件开发中的一种策略，该策略强调每一次更改都能被频繁地、可靠地，并在需要时随时部署到生产环境。在持续交付中，软件在整个生命周期中都是可以部署的。开发人员不断将代码更改提交给版本控制系统，并通过自动化的构建和测试流程来验证这些更改。然后这些更改被部署到类生产环境进行更深入的测试。如果一切顺利，这些更改就可以部署到生产环境。这样，持续交付让软件处于始终可以发布的状态。

持续交付的好处：

1. **减少风险**：持续交付能够确保每次部署的变更小，出问题的可能性降低，同时也更容易排查和修复问题。
2. **提高质量**：通过自动化的测试和部署，持续交付有助于降低人为错误，提高软件质量。
3. **更快的上市时间**：持续交付能够让你更快地、更频繁地发布新功能和改进，从而更快地满足用户需求，提高用户满意度。

使用Github Actions，我们可以很容易的做到持续交付。不论是前端应用还是后端应用，通过Github Actions提供的封装好的actions，以及其他第三方平台提供的actions，我们可以很容易的将一个就绪软件部署到合适的环境。

由于我们的应用是一个纯前端的程序，无需自己部署后端服务，因此我们可以将其部署到Github Pages上。GitHub Pages是GitHub提供的一种静态网站托管服务。它能够直接从GitHub上的仓库中获取HTML、CSS和JavaScript文件，然后生成一个直接对外公开的网站。这使得用户可以非常方便地建立和维护个人网站、项目网站，甚至是文档库。

我们希望当验收测试通过后完成一次发布，我们需要定义一个新的deploy的任务如下：

```yml
deploy:
    name: Deploy Application
    runs-on: ubuntu-latest
    needs: cypress

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

这段配置定义了一个名为"deploy"的工作流任务，该任务的目的是部署应用。它需要在 "ubuntu-latest" 的环境下运行，并且依赖于之前定义的 "cypress" 任务。如果 "cypress" 任务成功完成，"deploy" 任务才会开始运行。

`environment` 字段定义了任务运行的环境名称和URL。这里的环境被命名为 "github-pages"，并且URL由部署步骤的输出参数 "page_url" 决定。

在步骤部分，定义了一个名为 "Deploy to GitHub Pages" 的步骤，它使用了 "actions/deploy-pages@v2" 这个 GitHub Actions。这个 Action 的作用是将你的应用部署到 GitHub Pages。

`id: deployment` 是给该步骤一个唯一的ID，后续的步骤或任务可以通过这个ID获取到这个步骤的输出内容，比如这里的 `${{ steps.deployment.outputs.page_url }}` 就是获取了这个步骤的 "page_url" 输出。

同时，在`build`一步中，除了使用`actions/upload-artifact@v3`之外，我们还需要使用一个特殊的action来为Github Pages打包：

```yml
- name: Upload pages artifact
  uses: actions/upload-pages-artifact@v1
  with:
    name: github-pages
    path: dist/
```

注意此处需要定义最终的文件名为`github-pages`，这样Github Pages才能正确识别。如果这时候提交代码并推送，在Github的Actions页签上会看到一个错误：

![运行失败](ch8/failed-last.png)

查看日志可以知道，Github默认地并没有启用“使用Actions进行部署”的功能，因此我们需要在项目的设置中开启这个功能。

```bash
Error: Error: Failed to create deployment (status: 404) with build version 1da0b6ac55db2504119ce931701b4b667bd34efe. 
Ensure GitHub Pages has been enabled: https://github.com/icodeit-juntao/quote-of-the-day/settings/pages
```

因此我们需要开启标记为`beta`的Github Actions选项：

![启用Actions](ch8/enable-actions.png)

除此之外，由于部署需要修改我们的代码库，因此需要为`build.yml`指定写权限。

```yml
permissions:
  contents: read
  pages: write
  id-token: write
```

这样我们的应用就终于可以部署成功了。注意看这里任务的依赖，部署任务依赖于验收测试，验收测试依赖于构建。这样以来，如果我们的测试足够细致，那么发布到生产环境的应用一定是可以工作的。

![成功发布](ch8/success.png)

最终的`build.yml`的代码是这样的：

```yml
name: Build Quote Application

on:
  push:
    branches: [ "main" ]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    name: Build Frontend Package
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup node
        uses: actions/setup-node@v3
        with:
          node-version: 18.14.2
          cache: npm

      - name: Install
        run: npm ci

      - name: Test
        run: npm run lint

      - name: Build
        run: npm run build

      - name: Upload pages artifact
        uses: actions/upload-pages-artifact@v1
        with:
          name: github-pages
          path: dist/

      - name: Packaging
        uses: actions/upload-artifact@v3
        with:
          name: quote-of-the-day
          path: dist/

  cypress:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Cypress run
        uses: cypress-io/github-action@v5
        with:
          build: npm run build
          start: npm start
          command: npm run e2e

  deploy:
    name: Deploy Application
    runs-on: ubuntu-latest
    needs: cypress

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

在我们对于持续部署的工作流中，我们定义了自动化项目的构建、测试和部署过程。它定义了在主分支(main branch)有新的提交时(pull request)，就触发下面定义的三个任务：

1. **build**任务：在一个最新版的Ubuntu环境下执行，主要完成代码的拉取(checkout)，设置Node环境，安装依赖，执行测试（linting），并且构建应用。构建完成后，会上传构建的产品(dist文件夹)作为artifact。
2. **cypress**任务：这个任务依赖于build任务，也就是说只有当build任务成功完成后，才会执行cypress任务。它也在一个最新版的Ubuntu环境下执行，同样会先拉取代码，然后运行Cypress完成端到端的测试。
3. **deploy**任务：这个任务依赖于cypress任务，也就是说只有当cypress任务成功完成后，才会执行deploy任务。这个任务负责将构建的产品部署到GitHub Pages上。任务成功后，会把部署的页面URL作为输出。

## 小结

在本章中，我们探讨了如何使用GitHub Actions将我们的开发流程自动化，从而实现持续集成和持续交付（CI/CD）。首先，我们介绍了GitHub Actions，这是一个可以在GitHub仓库中直接执行软件工作流程的工具。

我们在工作流程中设置了三个主要任务：构建前端包，运行端到端测试，以及将应用部署到GitHub Pages。在构建任务中，我们使用了GitHub Actions来执行诸如安装依赖、运行lint检查和编译应用等操作。然后，我们设置了一个依赖于构建任务的端到端测试任务，用于在构建后自动运行Cypress测试。

最后，我们配置了一个部署任务，这个任务依赖于Cypress测试任务，只有当所有测试通过后，才会执行。在部署任务中，我们使用了GitHub Actions的部署页功能，将编译后的应用部署到GitHub Pages上。

通过这种方式，我们实现了从代码提交到应用部署的全自动化流程。这不仅提升了开发效率，也确保了代码质量和应用的稳定性。