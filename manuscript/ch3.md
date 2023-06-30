# 静态页面实现

在这一章，我们将开发今日箴言的第一个版本。这个版本非常简单，仅仅实现在页面上显示一个静态的名人名言，并在浏览器中运行即可。

我们可以创建一个名为 `quote-of-the-day` 的目录：

```bash
mkdir -p quote-of-the-day
cd quote-of-the-day
```

通过这两行命令：

1. `mkdir -p quote-of-the-day`：创建一个名为 "quote-of-the-day" 的目录。其中，`mkdir` 是一个命令用于创建目录，`p` 是一个选项，表示如果目录已经存在，则不会报错并继续执行。这条命令会在当前工作目录下创建一个名为 "quote-of-the-day" 的目录。
2. `cd quote-of-the-day`：进入到 "quote-of-the-day" 目录中。`cd` 是一个命令用于改变当前工作目录，后面的 "quote-of-the-day" 是目标目录的名称。这条命令会将当前工作目录切换到 "quote-of-the-day" 目录中，以便在该目录下执行其他操作。

在这个空的目录中，我们可以执行 `npm init` 来将其初始化为一个Node.js的工程：

```bash
$ npm init -y
```

此处的 `-y` 选项告诉 `npm` 命令，跳过交互式的界面，按照默认值生成Node.js工程即可。完成之后，npm会在当前目录写入一个package.json文件。这个文件就是我们工程的描述文件，我们后续的所有构建脚本将会围绕着这个文件展开。

```bash
Wrote to /Users/juntao/icodeit/ideas/quote-of-the-day/package.json:

{
  "name": "quote-of-the-day",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

这时候，如果我们在命令行执行 `npm test`就会得到这样的输出：

```bash
> quote-of-the-day@1.0.0 test
> echo "Error: no test specified" && exit 1

Error: no test specified
```

这是因为，在package.json中的scripts中，我们定义了一个命令：当执行 `test`时，这个命令会被执行：

```bash
echo "Error: no test specified" && exit 1
```

这个命令实际上是两个独立的命令，**`echo "Error: no test specified"`**和**`exit 1`**，它们通过逻辑运算符**`&&`**连接在一起。和编程语言中的效果一样，第二个命令（exit 1）只有在第一个命令（echo）成功执行后才会被执行。如果我们将package.json中的test修改为：

```bash
"scripts": {
  "test": "echo \"⛔️ I don't know what to run 🤷\" && exit 1"
},
```

当我们再次运行 `npm test` 时，命令行的输出就会变为：

![npm test](ch3/npm-test.png)

此外，我们还可以在scripts中定义新的任务。比如，我们可以定义一个名为 `start` 的任务，当执行时我们可以打印当前时间：

```bash
"scripts": {
  "test": "echo \"⛔️ I don't know what to run 🤷\" && exit 1",
  "start": "echo 🕞: `date`"
},
```

当执行 `npm start` 的时候，命令行中会显示：

![npm start](ch3/npm-start.png)

好了，现在我们对package.json中的scripts节有了一个大致的了解，现在可以开始**今日箴言**的开发了。

## 初始化工程

首先，我们需要定义两个目录，一个用来存储源代码的 `src` 和一个用来当作服务器根目录的 `dist`。由于我们的应用程序会运行在本地的一个端口上，因此我们需要一个HTTP服务器，这样当用户通过浏览器访问这个端口时，访问到的就是 `dist` 目录中的内容了。

```bash
mkdir -p src dist
touch src/index.html src/style.css
```

通过这两行命令：

1. `mkdir -p src dist`：创建两个目录，分别为 "src" 和 "dist"。`mkdir` 是一个命令用于创建目录，`p` 是一个选项，表示如果目录已经存在，则不会报错并继续执行。这条命令会在当前工作目录下创建 "src" 和 "dist" 两个目录。
2. `touch src/index.html src/style.css`：创建两个文件，分别为 "src/index.html" 和 "src/style.css"。`touch` 是一个命令用于创建文件，后面跟着文件的路径和名称。这条命令会在 "src" 目录下创建一个名为 "index.html" 的文件和一个名为 "style.css" 的文件。

第一个版本里，在index.html中，我们只需要显示一条静态的名人名言即可：

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
<div>
  <p class="content">If you want to lift yourself up, lift up someone else.</p>
  <p><span class="author">&mdash; Booker T. Washington</span></p>
</div>
</body>
</html>
```

在对应的CSS文件中，我们定义了一个文本居中的样式：

```css
body {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
}
```

这段代码的作用是将 `<body>` 元素的内容居中显示在视口中，并且使得整个页面的高度占满整个视口高度。这在很多网页布局中常见，可以使页面的内容居中对齐并充满屏幕。

我们已经完成了第一个版本所需要的代码修改，接下来我们需要安装一个HTTP服务器，这样就可以通过浏览器来访问我们的应用了。

## 静态服务器：http-server

Node.js中的http-server是一个简单的命令行工具，用于在本地创建一个基本的HTTP服务器。它允许您在开发过程中快速启动一个静态文件服务器，用于测试和调试网页、应用程序或其他静态资源。

http-server提供了一个轻量级的开发服务器，可以在本地主机上运行，并监听指定的端口。您可以使用http-server命令行工具来启动服务器，并指定要提供的文件目录和端口号。一旦服务器启动，您可以通过在浏览器中访问相应的URL来查看和访问您的文件。

http-server是一个流行的开发工具，因为它简单易用，不需要额外的配置，并提供了基本的HTTP服务器功能，如静态文件访问、缓存控制和跨域支持。它是Node.js生态系统中许多开发者在本地开发过程中的首选工具之一。

我们可以通过在Terminal中执行下列命令来安装这个工具：

```sh
npm install http-server --save-dev
```

安装了 `http-server` 之后，我们就可以在Shell中使用 `http-server` 命令了。通常我们需要指定一个目录为HTTP服务器的根目录，然后指定一个端口：

```sh
http-server dist -p 3000
```

这个命令的作用是在当前工作目录下的 **`dist`** 文件夹中启动一个 HTTP 服务器，并将其监听在端口号为 3000 的地址上。这样，我们就可以通过在浏览器中访问 **`http://localhost:3000`** 来查看和访问 **`dist`** 文件夹中的静态文件。

不过现在的问题是， `dist` 目录中没有任何的内容。我们需要将 `src` 中的HTML和CSS文件拷贝到 `dist` 中。这一步目前来看有点多余，这样做的原因是为了将源代码和产品代码隔离。在我们的第一个版本中，源代码比较简单，我们无须做转换或者复杂的开发，因此源代码和产品代码是一样的。在实际场景中，源代码往往是便于开发者编写和调试的格式，而生产代码则无须考虑可读性等，而且往往会被压缩或者混淆（以防止反向工程），因此两者往往并不等同。我们将在下一章详细讨论。

我们可以在package.json中定义一个新的任务 `build` , 该任务的内容就是将 `src` 中的HTML和CSS拷贝到 `dist`中：

```json
"scripts": {
  "build": "cp src/* dist/"
},
```

最后，我们需要将 `start` 任务修改为：

```json
"scripts": {
  "build": "cp src/* dist/",
  "start": "http-server dist -p 3000"
},
```

我们需要先执行 `npm build`完成源文件到产品代码的拷贝，然后当执行 `npm start`时， `http-server` 就会在3000端口上运行。

```sh
npm run build
npm start
```

在浏览器中输入 `[http://localhost:3000](http://localhost:3000)` 就可以看到我们第一个版本的**今日箴言**了！

![running in localhost](ch3/localhost.png)

在本章中，我们首先介绍了 Shell和命令行接口的使用，包括如何在命令行中输入命令，如何查看命令的输出，以及如何利用命令行来快速高效地进行开发工作。

接着，我们花了一部分篇幅，解读了 Node.js 的 `package.json` 文件中的 `scripts` 节。我们了解了如何定义自己的任务，如何使用 `npm run` 命令来运行这些任务，以及这些任务在项目开发和构建过程中的重要性。

此外，我们还详细介绍了 `http-server` 的使用。我们讲解了如何使用这个简单的 HTTP 服务器来预览和测试我们的项目，让大家对这个常用的开发工具有了更深入的了解。

最后，我们用所学知识成功构建并运行了 "今日箴言" 页面的第一个版本。通过这个实战项目，读者能够更好地理解和掌握上述的理论知识，而且我们也演示了如何在本地环境中运行 HTTP 服务器，并在服务器中部署并预览我们的项目。

下一章我们将使用React来扩展第一章中的内容，并进一步学习构建脚本的知识。