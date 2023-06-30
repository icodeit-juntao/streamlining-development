# React应用

这一章，我们继续上一章的页面，继续今日箴言应用的开发。我们这次增加了一些更加复杂的内容。首先我们要引入React来方便编写页面。对于我们这个规模的程序，使用React有种杀鸡用牛刀的感觉。这里引入React有两点考虑，其一是React是一个非常流行的前端UI库，读者可以比较容易在学习之后直接应用；其二是React的JSX需要被编译成JavaScript，这需要我们自然引入bundle的概念，方便读者在工作中对照参考。

除了React之外，我们将使用`esbuild`来编译代码使其成为浏览器可以理解的格式。在具体的功能上，我们将在页面上加入一个按钮，当用户点击该按钮时，刷新一条新的名言。

## 安装和配置React

我们首先需要安装React和react-dom这两个库到工程：

```sh
npm install react react-dom --save
```

然后在`src`目录中创建一个`app.jsx`的文件，将其内容修改为：

```jsx
import React from 'react';

const App = () => {
  return (<div>
    <p className="content">If you want to lift yourself up, lift up someone else.</p>
    <p><span className="author">&mdash; Booker T. Washington</span></p>
  </div>)
}

export default App;
```

这段代码定义了一个React组件，名为"App"。这个函数组件不接收任何参数，并返回一个JSX元素，即它的视图表示。此组件渲染的 JSX 包含一个 `div` 标签，这个 `div` 标签下有两个 `p`（段落）标签。第一个 `p` 标签有一个类名（className）"content"，内容为名言本身。第二个 `p` 标签包含一个 `span` 标签，`span` 标签有一个类名“author”，内容为名言的作者。

要使用这个组件，我们还需要一个`index.jsx`，内容如下：

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import App from './app';

const domNode = document.getElementById('root');
const root = createRoot(domNode);

root.render(<App />);
```

对于这段代码，以下是各部分的详细解释：

- `import React from 'react';` 引入了React库。React 是一个用于构建用户界面的JavaScript库。
- `import { createRoot } from 'react-dom/client';` 引入了 `createRoot` 函数，这是一个用于开启并创建并发模式（Concurrent Mode）的新API。
- `import App from './app';` 引入了名为 App 的React组件。假定 `App` 组件在同一目录的 `app.js`x 文件中定义。
- `const domNode = document.getElementById('root');` 这一行代码在整个 HTML 文档中查找 id 为 'root' 的元素，并将其保存在 `domNode` 中。这就是我们要渲染React应用的HTML节点。
- `const root = createRoot(domNode);` 使用 `createRoot` 函数创建一个并发模式（Concurrent Mode）的根。并发模式是一个新的React模式，允许React在渲染过程中进行中断和恢复操作，从而实现更平滑的用户体验。
- `root.render(<App />);` 将 `App` 组件渲染到前面定义的 `root` 中。这一行代码启动了整个React应用的渲染过程。

这时候，我们不再需要HTML文件中的hard code，而只需要定义一个id为root的元素即可：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Quote of the day</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div id="root" />
  <script src="main.js"></script>
</body>
</html>
```

在这段HTML代码中，**`<script src="main.js"></script>`**所引入的**`main.js`**，就是我们应用程序的JavaScript代码，它通过构建工具编译而成。

## 打包工具

现在我们遇到了一个难题：我们编写的，浏览器并不能认识这些容易被开发者理解和维护的JSX代码。浏览器可以识别并执行JavaScript代码，通过DOM API还可以操作页面元素，但是对于诸如`<App />` 这样的代码则无法识别。因此我们需要一个工具，将我们编写的JSX***翻译***成浏览器可以认识的格式。

此外，由于我们使用了`import`这样的前端语句，即在一个JavaScript文件中使用另外一个JavaScript文件，浏览器需要一种更好的方式来识别这些依赖。我们需要的这个工具就是Bundler，即打包工具。

**Bundler程序（或模块打包工具）**，顾名思义，就是将多个模块打包成一个或多个文件的工具，让你能在客户端（如浏览器）中运行模块化的代码。这些打包工具通常还有其他功能，如编译转码、压缩代码、处理资源文件（如图片和样式表）等。

Bundler的主要作用包括：

1. **依赖管理**：Bundle工具可以跟踪代码之间的依赖关系，并确保它们按照正确的顺序加载。
2. **代码优化**：Bundle工具可以通过压缩和树摇（tree-shaking）等技术优化代码，从而减小输出文件的大小，提高代码运行效率。
3. **转译代码**：Bundle工具可以通过Babel等工具将新版JavaScript（或TypeScript等其他语言）转译为旧版JavaScript，使得新的语言特性能在更多环境中运行。

常见的JavaScript打包工具包括Webpack、Rollup、Parcel等，各有各的特点和优势。例如，Webpack功能强大、配置灵活，是目前最流行的打包工具；Rollup更适合于库的打包，支持tree-shaking；Parcel无需配置即可使用，适合快速开发。

**ESBuild** 是一个较新的打包工具，其主要特点是极快的速度。ESBuild使用Go编写，并采用并行化、缓存等技术，使得其在处理大型代码库时，其速度比大多数JavaScript写的打包工具快了几个数量级。同时，ESBuild也支持常见的打包功能，如加载器（loader）、插件系统等，让你能灵活地处理各种文件和代码转译需求。

首先我们需要安装`esbuild`这个工具：

```bash
npm install --save-exact esbuild
```

使用`esbuild`非常简单，我们需要指定入口文件 — 即我们应用程序从哪里开始，然后它会自动分析该文件所需要的依赖，并会根据扩展名使用合适的翻译程序（比如将JSX翻译成JavaScript的程序），最终生成一个结果文件。

我们在Shell中执行下面的命令来编译我们的应用程序：

```bash
./node_modules/.bin/esbuild src/index.jsx --bundle --outfile=dist/main.js

  main.js  1.0mb ⚠️

⚡ Done in 41ms
```

这是一个在命令行界面执行的命令，用于调用`esbuild`打包工具，对源代码进行打包。让我们分解这个命令，看看每个部分的含义：

- `./node_modules/.bin/esbuild`：这是`esbuild`可执行文件在项目中的位置。当你在项目中安装了`esbuild`时，其可执行文件会被放置在`node_modules/.bin`目录下。这段路径就是从当前目录到`esbuild`可执行文件的相对路径。
- `src/index.jsx`：这是`esbuild`需要打包的入口文件。`esbuild`会从这个文件开始，找到所有的依赖，然后把它们一起打包。
- `-bundle`：这是一个命令行选项，告诉`esbuild`需要把所有的模块捆绑到一起。在没有这个选项的情况下，`esbuild`只会转译代码，而不会进行模块捆绑。
- `-outfile=dist/main.js`：这个命令行选项指定了输出文件的位置和文件名。`esbuild`会把打包的结果输出到这个文件。

现在在文件`dist/main.js`中，所有的代码都是浏览器可以认识的JavaScript。当然我们并不希望每次都手动的在命令行中输入上边这条命令，我们可以修改package.json中的build任务。

```json
"scripts": {
  "build": "esbuild src/index.jsx --bundle --outfile=dist/main.js",
  "start": "http-server dist -p 3000"
},
```

`"build"`: 这个脚本命令用于构建项目。它使用 `esbuild` 工具，将 `src/index.jsx` 文件进行打包，并将打包后的结果输出到 `dist/main.js` 文件中。我们可以在命令行中使用 `npm run build` 命令来执行构建操作，生成项目的打包文件。

接下来我们可以将`index.html`移动到`dist`目录中。

```bash
mv src/index.html dist/
```

让我们来运行一下目前的构建脚本来看看效果吧：

```bash
npm run build
npm start
```

呃，功能好像是没有问题，不过我们的名人名言好像没有居中对齐：

![missed css](ch4/missed-css.png)

这是因为我们在打包过程中丢失了CSS文件。在新的代码结构中，我们并没有任何地方提及CSS文件，因此esbuild自然无法知道如何打包。我们可以在`index.jsx`中包含`style.css`：

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import App from './app';

import './style.css';

const domNode = document.getElementById('root');
const root = createRoot(domNode);

root.render(<App />);
```

再次执行`npm run build`，我们可以看到一个名为`main.css`的文件被生成到了`dist`目录。我们只需要在`index.html`中引用这个CSS文件即可。

![npm build](ch4/npm-build.png)

修改后的`index.html`文件内容变成了：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Quote of the day</title>
  <link rel="stylesheet" href="main.css">
</head>
<body>
  <div id="root" />
  <script src="main.js"></script>
</body>
</html>
```

重构构建，然后执行`npm start`，这时候我们在浏览器中就可以看到正确的结果了。

这里我们可以小节一下，我们现在完成了将页面从HTML到React的改造，我们引入了esbuild来编译React代码，生成浏览器可以识别的最终JavaScript代码。我们修改了在package.json中的`build`任务，这样我们就无须手工调用esbuild。

不过现在流程上有一个小小的问题：如果我们修改了组件代码，我们需要手动的终止http-server（通过CTRL+C），然后重新手工执行`npm run build`，然后再次`npm start`。有没有什么办法可以自动的探测我们的修改，然后自动编译main.js呢？

答案是可以的，我们需要使用esbuild的`—watch`选项来动态监测文件修改。我们需要在pacakge.json中定义一个新的任务：

```json
"watch": "esbuild --bundle src/index.jsx --outfile=dist/main.js --watch"
```

这样我们就无须停止`http-server`服务，只需要在另外一个Terminal窗口中执行`npm run watch`即可。

![npm watch](ch4/npm-watch.png)

## 实现“下一条”

有了这些自动化脚本作为基础设施，接下来我们就可以开始“下一条”功能的开发了。我们首先需要定义一个数组，然后需要在App组件中添加一个按钮。当用户点击按钮时，我们更新数组的index，获取下一条名言并展示。

```js
const quotes = [
  {
    content:
      "Any fool can write code that a computer can understand. Good programmers write code that humans can understand.",
    author: "Martin Fowler",
  },
  {
    content: "Truth can only be found in one place: the code.",
    author: "Robert C. Martin",
  },
  {
    content:
      "Optimism is an occupational hazard of programming: feedback is the treatment.",
    author: "Kent Beck",
  },
];
```

首先定义一个名言的数组，每个元素包含一个content字段和一个author字段。然后修改`App.jsx`为一下内容：

```jsx
const App = () => {
  const [index, setIndex] = useState(0);

  const clickHandler = () => {
    setIndex((index) => (index + 1) % 3);
  };

  return (
    <>
      <div>
        <p className="content">{quotes[index].content}</p>
        <p>
          <span className="author">&mdash; {quotes[index].author}</span>
        </p>
      </div>
      <button onClick={clickHandler}>next</button>
    </>
  );
};
```

在`App`中，我们使用了React的内置`useState`函数来管理一个状态变量`index`，并提供一个更新该状态的函数`setIndex`。`useState`的初始值为0，也就是说，`index`最初被设置为0。

然后我们定义了一个名为`clickHandler`的函数，该函数通过调用`setIndex`来更新`index`的值。新的`index`值取决于当前`index`值，将其加1然后取模3，所以`index`的值在0，1，2之间循环。

在组件的返回值中，使用了一组`p`标签来显示`quotes[index].content`和`quotes[index].author`的值，`quotes`可能是一个包含引用的数组，并且`index`用来选择显示哪一个引用。

同时，在返回的JSX中还包含了一个按钮，其`onClick`属性绑定了`clickHandler`函数。所以每当用户点击这个按钮时，`clickHandler`函数就会被调用，进而更新`index`的值。

![localhost](ch4/fixed.png)

## 小结

这一章我们介绍了如何将我们的HTML静态页面转换为React应用，并使用`esbuild`来进行构建。我们还为应用添加了一个功能：点击按钮来获得下一条名言。在这个过程中，我们学习了进一步扩展构建脚本，实现本地编译的自动化。