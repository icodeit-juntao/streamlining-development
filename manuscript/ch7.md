# 静态检查

在上一章结束的时候，我们的今日箴言已经比最开始的静态页面复杂了许多。我们的工程中有了负责网络请求的自定义hook，有负责展现的Quote组件，有专门的CSS，还有一些测试代码。随着更多的代码的加入（以及更多的集成工作），我们需要考虑如何确保代码的内置质量。

在代码开发中，要完成某项任务往往有多种做法，比如可以把所有代码写到同一个组件中，也可以拆分为多个独立组件；可以将样式内联到JSX中，也可以定义单独的文件；可以使用箭头函数，也可以使用function关键字等等。

在工程中，特别是有多人协作的工程中，我们需要使用一些工具来确保大家的代码风格一致。这样在理解和修改时才可以更加高效。这类工具就是linter，即检查器工具。

在JavaScript工程中，我们需要使用检查器（比如[ESlint](https://eslint.org/)）来自动检查源代码中潜在问题。这些问题可以包括语法错误、代码风格不一致、未使用的变量、可能的逻辑错误等。ESLint检查起，可以帮助开发人员在代码还未执行或部署之前就发现并解决这些问题。

使用 ESLint 工具在工程中的主要原因有：

1. **保持代码风格一致**：ESLint可以帮助开发者遵循一致的编码风格，使得代码库更易于阅读和维护。开发团队可以定义自己的编码风格规则，然后通过 ESLint 自动强制实施这些规则。
2. **提高代码质量**：通过检查常见的编程和逻辑错误，ESLint可以帮助开发者提高代码质量，减少因为一些低级错误而浪费的调试时间。
3. **促进最佳实践**：ESLint包括了许多关于最佳实践的规则，例如禁止使用已废弃的语言特性，或者推荐使用更加有效的解决方案。这可以帮助开发者更好地了解并遵循最新和最佳的开发实践。
4. **集成到构建流程中**：ESLint可以作为构建流程的一部分来执行，例如在每次提交代码之前运行Linting，或者在持续集成（CI）流程中运行。这样可以确保只有满足规定编码风格和质量的代码才能够被提交或部署。

## 配置ESlint

首先我们需要安装和配置ESlint工具，它内置了一个配置脚本，可以方便开发者很快的完成设置。只需要在工程的根目录中执行：

```bash
npm init @eslint/config
```

配置脚本会提问一些问题，我们根据今日箴言的设置配置如下：

![config eslint](ch7/init-eslint.png)

然后配置脚本会根据我们的选择来决定下载哪些依赖，完成安装之后，我们本地就可以使用eslint进行检查了。

```bash
node_modules/.bin/eslint src/
```

这条命令调用eslint来检查`src`目录，由于我们在上一步采用了airbnb的代码风格（一种比较广泛被使用的规则），因此检查出了这样一些问题。

![eslint](ch7/eslint.png)

这些错误分别表示在文件`quote-of-the-day/src/useFetchQuotes.js` 中：

1. `1:37 error Strings must use singlequote quotes`：在代码中，字符串使用了双引号，但 ESLint 的规则要求必须使用单引号。需要将这个字符串的双引号改为单引号。
2. `9:11 error Strings must use singlequote quotes`：同上。
3. `15:15 error 'e' is defined but never used no-unused-vars`：在代码中，变量 `e` 已经定义，但在后续的代码中没有被使用到。这可能意味着在某处遗漏了对这个变量的使用，或者这个变量是多余的，可以移除。
4. `27:10 error Prefer default export on a file with single export import/prefer-default-export`：这条规则要求，当一个文件中只有一个export的时候，使用default export（默认导出）而不是named export（命名导出）。我们需要将代码中的 named export 改为 default export。

此外，ESlint提示我们，可以通过`—fix`选项来尝试自动消除这些错误。如果我们用`node_modules/.bin/eslint src —fix`来再次执行。

ESlint自动将双引号改成了单引号，但是对于未使用的变量`e`和默认导出的情况，它不知道如何处理，因此需要程序员来消除该问题。

和其他的构建任务一样，我们可以将lint加入到package.json中，用来简化我们的工作。这样我们的scripts节就更新成了：

```json
"scripts": {
  "e2e": "cypress run --spec cypress/e2e/*.spec.cy.js",
  "build": "esbuild src/index.jsx --bundle --outfile=dist/main.js",
  "start": "http-server dist -p 3000",
  "watch": "esbuild --bundle src/index.jsx --outfile=dist/main.js --watch",
  "lint": "eslint src/"
},
```

我们已经有了静态检查，自动化测试，构建，本地服务等任务，这些任务在我们本地开发时会提供很多便利。比如在提交代码之前，我们可以运行lint，如果一切正常，紧接着可以执行自动化测试，最终打包，然后再将本地的提交推送到远程服务器。

不过这里的一些动作依然需要手动执行，如果我们（或者其他开发人员）忘记执行linting，则签入之后的代码就混入了不规范的写法。如果我们可以在事先就杜绝这种失误的发生就好了。事实上，我们可以使用Git Hooks来自动执行这些任务。

## Git Hooks

Git Hooks 是一种强大的特性，它允许你在 Git 的重要工作流程中触发自定义脚本。在 Git 仓库中有一个名为 `.git/hooks` 的目录，其中包含一些示例脚本。这些脚本可以在你执行诸如 commit、push 或 checkout 等操作时被触发。这些脚本是服务器和客户端都可以运行的，有助于实现自动化工作流。

Git Hooks 可以分为两大类：

1. 客户端Hooks：这些钩子被触发在你在本地操作时，例如提交(commit)和合并(merge)。这些钩子可以帮你在提交时进行代码审查、代码风格检查等工作，以确保代码质量。
2. 服务器端Hooks：这些钩子在推送到服务器后被触发。例如，你可以设置一个hook在代码被推送到主分支后自动部署你的应用。

你可以使用任何可以执行的文件编写 Git Hook 脚本，例如 shell 脚本、Python 脚本等。在脚本中，你可以访问到 Git 环境中的变量和参数，从而实现各种自定义功能。

例如，`pre-commit` hook可以用来在提交前运行 linter 或者单元测试，如果检查或测试失败，那么提交就会被中止。这样可以保证不会有低质量的代码被提交。

我们来一起定义一个客户端的hook，每次我们生成commit时，我们希望进行一次静态检查，并报告语法错误或者不符合代码风格的问题等。

首先我们需要创建一个新的文件 `pre-commit` 在 `.git/hooks`目录，内容如下：

```bash
#!/bin/sh

npm run lint
```

这样，当我们在命令行运行`git commit`时，这个hook就会被自动触发。

![git commit](ch7/pre-commit.png)

当运行成功后，commit创建成功，否则会报错并失败。我们需要修复代码，然后再次提交。我们用同样的方法可以定义`pre-push`脚本，并在push之前执行`npm run e2e`来保证功能的正确。

不过就像上面提到的，我们这里定义的Git Hooks只能在本地使用。如果团队里的其他成员想要使用同样的脚本，他们不得不自己重新写一遍。因此我们需要使用husky工具来将这些脚本统一管理起来，这样所有人的环境就会统一起来。

## 使用Husky

Husky提供了配置脚本，我们只需要执行：

```bash
npx husky-init && npm install
```

即可完成初始化和安装。配置完成之后，本地目录会生成一个 `.husky` 的目录，其中就是我们放置自定义hook的位置。此外，配置脚本还会将package.json修改，添加一个`prepare`的任务：

```bash
"prepare": "husky install"
```

这样当我们（以及团队里的其他程序员）执行`npm install`时，husky就会被安装。默认地，husky会在`.husky`目录中生成一个`pre-commit`文件，内容如下：

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm test
```

我们需要将`npm test`修改为`npm run lint`：

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint
```

然后将整个`.husky`目录都签入到Git中管理，这样团队里的其他人都可以获得相同的环境。

## 小结

这一章，我们引入了代码静态检查工具ESlint，并且定义了新的任务`lint`。此外，我们描述了如何配置Git Hooks以及Husky来帮我们在合适的时机自动化执行预定义的脚本，这样不但可以放置开发人员的疏忽，而且可以做到整个团队使用同样的配置，以最大限度的降低发生错误的概率。

在下一章，我们将一起学习如何配置远程服务器，使得我们可以确保程序不但可以在本地工作，而且在服务器端也可以正常工作。