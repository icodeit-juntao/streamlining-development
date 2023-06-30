# 自动化测试

随着我们开始加入新的一些功能，每次对代码的修改就需要考虑这样一个问题，即如何保证我们的修改不会破坏已有的功能呢？如果整个应用只有一个人开发的话，这倒也不是一个大问题。不过随着功能越来越复杂，以及有了其他团队成员的加入，确保每次的修改都不会破坏整个应用就变成了一个需要提前规划的问题。

在实践中，我们需要引入自动化测试来解决这个问题。我们需要针对每次的修改执行测试，如果测试失败，则需要完全修复之后再提交，并和远程同步。当然，自动化测试可以分为很多个不同层级，比如用户验收测试，集成测试，性能测试，安全测试和单元测试等等。我们在这一章需要讨论用户验收测试和组件测试两类。

## 验收测试

**用户验收测试**（User Acceptance Testing, UAT）是软件开发生命周期中的最后一阶段的测试过程，这一阶段的测试通常由最终用户或客户进行，以确认系统或产品是否符合预定的业务需求。其主要目标是验证软件是否满足业务需求，并确保系统可以在真实的业务场景中运行。

用户验收测试的重要性在于它提供了一种机制，可以确保软件产品不仅按照技术规范进行了开发，而且确实满足用户的实际需求。也就是说，UAT提供了一个确认最终用户能够在实际环境中有效使用软件的机会。

**Cypress**是一个流行的端到端测试框架，专为现代Web应用程序设计。它使得开发者可以编写用于测试应用程序的各个方面的代码，从简单的用户交互（如按钮点击和表单提交）到页面导航和动态内容加载等更复杂的行为。

以下是Cypress的一些关键特性：

- **实时重载**：Cypress可以在你保存测试文件后立即重新运行测试，使得开发过程更加高效。
- **自动等待**：你不需要在测试中添加等待或休眠，Cypress自动等待命令和断言完成。
- **一流的错误消息**：当测试失败时，Cypress提供详细的错误消息以帮助你理解发生了什么问题。
- **网络流量控制**：Cypress使你可以对网络请求进行存根和拦截，以模拟服务器行为。

通过Cypress，你可以执行用户验收测试，模拟用户在真实浏览器环境中与你的Web应用程序的交互，并验证程序的功能和性能是否满足预期。

安装`cypress`和其他node.js包一样，只需要执行下面的命令即可。

```bash
npm install cypress --save-dev
```

安装完成之后，我们使用初始化cypress。cypress提供了一个wizard，需要运行：

```bash
./node_modules/.bin/cypress open
```

来启动配置助手程序：

![运行配置助手程序](ch5/cypress-init.png)

我们这里选择E2E Testing，其他暂时按照默认值即可。助手程序会生成一些配置文件，在最后一步，它会提示我们生成一个spec文件，spec即为cypress中的测试描述文件，我们输入一个文件名即可，比如：quote-of-the-day.spec.cy.ts。

这样我们的工程目录下会多出来一个`cypress`的目录：

```bash
cypress
├── downloads
├── e2e
│   └── quote-of-the-day.spec.cy.js
├── fixtures
│   └── example.json
└── support
    ├── commands.js
    └── e2e.js
```

要执行`quote-of-the-day.spec.cy.js`，我们需要在Terminal中输入下列命令：

```bash
node_modules/.bin/cypress run --spec cypress/e2e/quote-of-the-day.spec.cy.js
```

这条命令使用了`cypress`，通过`—spec`选项指定要运行的测试文件。由于这个测试是cypress自动生成的，因而可以得到一个验证通过的界面。

![执行验收测试](ch5/e2e.png)

如果我们查看`quote-of-the-day.spec.cy.js`的内容，可以看到如下代码：

```js
describe('template spec', () => {
  it('passes', () => {
    cy.visit('https://example.cypress.io')
  })
})
```

在这个测试用例中，**`cy.visit('https://example.cypress.io')`**表示访问**`https://example.cypress.io`**这个URL。**`cy.visit`**是一个Cypress提供的函数，用于在测试中模拟用户访问一个页面的行为。如果页面成功加载，测试就会通过。

## 测试今日箴言

下面我们就可以修改这个测试代码，让其为我所用。我们需要访问本地的`http://localhost:3000`端口，然后验证代码上是否存在预设的一句名人名言：

```js
describe("quote of the day spec", () => {
  it("displays a quote", () => {
    cy.visit("http://localhost:3000");
    cy.contains(
      "Any fool can write code that a computer can understand. Good programmers write code that humans can understand."
    );
    cy.contains("Martin Fowler");
  });
});
```

`cy.visit("<http://localhost:3000>")`模拟用户访问"localhost:3000"，也就是我们本地运行的应用。`cy.contains`函数用于检查页面上是否存在某个指定的文本内容。这里的两个`cy.contains`函数分别检查名言的内容和作者是否被正确显示。如果在页面上找到了这两段预期的文本，测试就会通过；否则，测试将失败。

需要注意的是，这个测试会通过浏览器访问本地的3000端口，所以我们首先需要使用`npm start`在一个Terminal窗口中启动我们的应用程序，然后在另一个Terminal中执行：

```bash
node_modules/.bin/cypress run --spec cypress/e2e/quote-of-the-day.spec.cy.js
```

当然，我们可以在package.json中定义一个新的任务`e2e`:

```json
"scripts": {
  "e2e": "cypress run --spec cypress/e2e/*.spec.cy.js",
  "build": "esbuild src/index.jsx --bundle --outfile=dist/main.js",
  "start": "http-server dist -p 3000",
  "watch": "esbuild --bundle src/index.jsx --outfile=dist/main.js --watch"
},
```

这样我们进需要输入`npm run e2e`即可自动完成功能验证。

## 测试用户交互

我们再来添加一个测试：模拟用户点击“next”按钮，然后验证页面上的名言变更为新的一条。

```js
it("clicks next button", () => {
  cy.visit("http://localhost:3000");
  cy.contains('button', 'next').click();
  cy.contains(
    "Truth can only be found in one place: the code."
  );
  cy.contains("Robert C. Martin");
});
```

这段测试代码模拟了用户在访问 `http://localhost:3000` 页面并与 "next" 按钮进行交互的过程。

- `cy.visit("http://localhost:3000");` 这行代码用来打开应用程序。
- `cy.contains('button', 'next').click();` 这行代码会在页面中查找含有 'next' 文本的按钮，并模拟点击操作。
- `cy.contains("Truth can only be found in one place: the code.");` 和 `cy.contains("Robert C. Martin");` 两行断言来检查页面中是否存在相应的文本内容。

现在我们的package.json中scripts已经有了4个不同的任务了：

```json
"scripts": {
  "e2e": "cypress run --spec cypress/e2e/*.spec.cy.js",
  "build": "esbuild src/index.jsx --bundle --outfile=dist/main.js",
  "start": "http-server dist -p 3000",
  "watch": "esbuild --bundle src/index.jsx --outfile=dist/main.js --watch"
}
```

我们的四个任务分别为：

- "e2e" 命令运行位于 "cypress/e2e" 目录下的所有 Cypress 端到端测试文件。
- "build" 命令使用 esbuild 工具将 "src/index.jsx" 文件打包，生成的结果保存在 "dist/main.js" 文件中。
- "start" 命令用于启动一个服务，通过 http-server 将 "dist" 目录作为静态资源服务器的根目录，服务运行在 3000 端口上。
- "watch" 命令也是使用 esbuild 对 "src/index.jsx" 文件进行打包，但加入了 "--watch" 选项，这意味着每当源文件发生变化时，esbuild 会自动重新执行打包操作。

## 小结

通常来说，我们会在React代码库中使用诸如jest和react-testing-library等库来编写单元测试，在这个应用程序，为了保持主线的清晰，我们刻意忽略了这一点。关于前端的测试驱动开发，可以参考我的免费课程：[Test-Driven Development With React](https://icodeit.thinkific.com/courses/test-driven-development-with-react)以及[同名书籍](https://www.amazon.com.au/Test-Driven-Development-React-Apply-Applications/dp/1484269713)。