# 和后端API通信

在这一章，我们来学习如何进一步扩展今日箴言，使其能够从远程的服务器中获取箴言列表，而不是我们手工的写在源码中。

通过网络请求获取数据是大部分应用程序都会采用的模式。前端通过HTTP协议，主动发起请求，后端应用程序经过分析和处理（确定请求的格式，以及前端希望的格式）之后，返回对应的数据。常见的基于HTTP的API有RESTful API和GraphQL等。

## Quotable API

我们将使用Q[uotable](https://github.com/lukePeavey/quotable#list-quotes)作为后端的API。Quotable API是一个免费的RESTful API，它提供了大量的名言和格言。每个名言都包含作者、内容以及它们所属的标签或分类。

以下是其基本功能：

- 随机获取名言：您可以调用API随机获取一条名言，包括名言的内容和作者等信息。
- 获取特定作者的名言：您可以根据作者的名字，获取该作者的所有名言。
- 搜索名言：API提供搜索功能，您可以根据关键词，搜索相关的名言。
- 根据标签获取名言：名言都会有相应的标签，例如“爱情”、“生活”、“友谊”等，您可以根据标签获取相关的名言。

Quotable API以JSON格式返回数据，易于程序处理。通过此API，开发者可以在自己的项目中使用丰富的名言资源，且不需要关心数据存储和管理的问题。

我们可以通过命令行接口来访问Quotable API，来看看数据格式：

```bash
curl https://api.quotable.io/quotes/random
```

这条命令使用了`curl`，以Quotable的url为参数以获取一条随机的名言。服务器返回的格式如下：

```json
[
  {
    "_id": "N43bqeXRBeqI",
    "content": "Just trust yourself, then you will know how to live.",
    "author": "Johann Wolfgang von Goethe",
    "tags": [
      "Famous Quotes"
    ],
    "authorSlug": "johann-wolfgang-von-goethe",
    "length": 52,
    "dateAdded": "2021-05-07",
    "dateModified": "2023-04-14"
  }
]
```

可以看到，服务器返回了一个只有一个元素的数组，数据包含了我们关心的content和author字段，以及一些元数据信息，比如长度，收录时间，标记等。

Quotable还支持按照标记或者作者进行检索，比如我们可以获取所有与快乐相关的名言：

```bash
curl https://api.quotable.io/quotes?tags=happiness&limit=3
```

注意上面的命令将结果限制在3条，这样我们就无须频繁调用后端API。Qutoable还支持根据对结果排序，分页以及关键字搜索等功能。我们在本书中仅需要随机名言这个接口。

## 前端代码

在前端，我们可以使用`fetch`发送请求，当服务器返回数据之后，和之前那样渲染该数据即可。我们需要在App.jsx中使用`useEffect`：

```jsx
const App = () => {
  const [quotes, setQuotes] = useState([]);

  useEffect(() => {
    const fetchQuotes = async () => {
      fetch("https://api.quotable.io/quotes/random?limit=3")
        .then((r) => r.json())
        .then(quotes => setQuotes(quotes))
    }

    fetchQuotes();
  }, []);

  return (
		//...
  );
};
```

首先，我们取代了hard code的固定的quotes定义。其实，在useEfeect中，我们使用fetch来获取三个随机的名言。最后当fetch到数据之后，页面会使用这三条数据。

当我们刷新页面后，会惊喜的发现页面变成了一片空白。打开浏览器的调试器（inspect页面）会发现控制台的很多错误。

![console error](ch6/console.png)

这是因为，fetch是一个异步操作，即发送请求时，quotes数组还是空的，但是由于我们使用了`quotes[index].content`这样的表达式来渲染，因此会得到一个访问undefined对象的content属性的错误。

我们可以为这个表达式加上一个读保护来解决这个问题：

```jsx
return (
  <>
    <div>
      <p className="content">{quotes[index] && quotes[index].content}</p>
      <p>
        <span className="author">&mdash; {quotes[index] && quotes[index].author}</span>
      </p>
    </div>
    <button onClick={clickHandler}>next</button>
  </>
);
```

上面的代码首先确保`quotes[index]`是存在的（不为undefined），然后在这个前提下访问其content和author属性。这样页面就又可以正常工作了。

## 修复测试

如果这时候我们运行测试`npm run e2e`，会发现所有测试都失败了。这是因为，测试期望的数据依然是我们之前hard code到源代码中的数据。🤔️，这就有些麻烦了。一方面，我们希望这些名人名言是随机的，另一方面，我们在测试中需要测试页面上的名言。当运行测试时，测试代码怎么知道此时后台会返回哪些“随机”数据呢？

这就要引入stub的概念了。在测试中我们可以通过某项机制拦截网络请求，然后假扮成服务器返回一段我们预先定义好的数据，这样测试就可以通过。在实际应用程序的执行中，即在最终用户的浏览器上，由于不存在这段拦截代码，因此用户看到的又是真实的随机数据。

在Cypress中，我们可以使用`cy.intercept()`方法拦截网络请求并返回stub的数据。这样可以使我们的前端测试不再依赖后端API的实际返回，而是可以根据我们自己设定的数据来验证我们的代码。

例如：

```js
cy.intercept('GET', '/quotes/1', {
  statusCode: 200,
  body: {
    id: 1,
    quote: "Truth can only be found in one place: the code.",
    author: "Robert C. Martin"
  }
});
```

上面的代码中，`cy.intercept()`方法拦截了所有向`/quotes/1`的GET请求，并返回了我们预设的JSON数据。在此之后，所有向`/quotes/1`的GET请求都会返回我们预设的数据，而不是实际的后端API返回。

对于我们的测试代码，我们需要在`quote-of-the-day.spec.cy.js`中做如下修改：

```js
describe("quote of the day spec", () => {
  beforeEach(() => {
    cy.intercept('GET', 'https://api.quotable.io/quotes/random*', {
      statusCode: 200,
      body: quotes
    });
  });

  it("displays a quote", () => {});

  it("clicks next button", () => {});
});
```

在上边的测试代码中我们使用了`beforeEach`：这是一个Cypress的钩子函数，它会在每一个测试用例执行前运行。在这个函数内，我们使用`cy.intercept()`方法拦截了所有指向`https://api.quotable.io/quotes/random*`的GET请求，并且使它们返回预设的`quotes`数据。

这样，我们就可以在一个稳定的环境中测试我们的代码，而不必担心API返回的数据可能带来的不确定性。

## 重构代码

我们都希望编写高质量的代码，代码应该容易理解，容易修改。当看到不合理的结构，不够优雅的写法时，作为职业程序员，我们都会忍不住去修改使其变得更好。但是很多时候我们即使有这种美好的想法，却很难实施。究其原因，很大程度上是这样做的成本太高。

所谓成本高可以描述为这样几个部分：首先，理解已有的写的比较糟糕的代码本身就需要很多时间和精力。其实，如果在重构过程中出错了，我们又需要花费额外的时间和精力去重新测试。所以很多时候程序员选择延缓重构。这样又会导致以后的理解和改写更加困难，花费更多的时间和精力等。

所以从根本上来说，我们应该建立一种机制，这个机制可以在一开始就保护我们用较少的时间来验证我们的修改，而且无需担心破坏已有的功能。这个机制就是自动化测试以及自动化的构建脚本。

有了测试的保护，我们就可以对目前的代码做一些结构性的调整，而无需担心最终程序被破坏。这一点在编程中十分重要。

我们可以将网络数据获取的部分移动到一个单独自定义的hook，然后存入一个新文件`useFetchQuotes.js`中：

```js
import { useEffect, useState } from "react";

const useFetchQuotes = () => {
  const [quotes, setQuotes] = useState([]);
  const [loading, setLoading] = useState(false);

  const fetchQuotes = () => {
    setLoading(true);
    fetch("https://api.quotable.io/quotes/random?limit=3")
      .then((r) => r.json())
      .then((data) => {
        setLoading(false);
        setQuotes(data);
      })
      .catch((e) => {
        setLoading(false);
      });
  };

  useEffect(() => {
    fetchQuotes();
  }, []);

  return { quotes, loading, fetchQuotes };
};

export { useFetchQuotes };
```

这段代码定义了一个名为`useFetchQuotes`的自定义React Hook，它的作用是从Quotable API中获取三个随机引用。以下是对这段代码的详细解释：

- `useState`用于在函数组件中定义状态。`quotes`是一个状态变量，用于存储从API获取的引语，`setQuotes`是对应的设置器函数。`loading`是一个状态变量，用于标记数据是否正在被加载，`setLoading`是对应的设置器函数。
- `fetchQuotes`函数用于从Quotable API获取随机引语。在开始获取数据时，它会将`loading`设置为`true`。当数据被成功获取时，它将引用存储在`quotes`状态变量中，并将`loading`设置为`false`。如果获取数据的过程中出现错误，它将仅仅将`loading`设置为`false`。
- `useEffect`钩子在组件首次渲染时调用`fetchQuotes`函数，因此引用在组件首次渲染时被获取。`[]`作为`useEffect`的依赖数组，这意味着`fetchQuotes`函数仅在组件首次渲染时调用，而非每次渲染都调用。
- `useFetchQuotes`最终返回一个对象，这个对象包含`quotes`和`loading`两个状态变量，分别表示已获取的引用和加载状态。这使得在使用这个自定义Hook的组件中，可以通过解构得到这两个状态变量，从而在组件中使用它们。

然后在App.jsx中，我们可以直接使用这个hook提供的loading状态和quotes数组。对于JSX部分，我们还可以抽取一个新的组件`Quote`，从而使得代码更加清晰紧凑：

```jsx
import React from "react";

const Quote = ({ quote }) => {
  return (
    <div>
      <p className="content">{quote && quote.content}</p>
      <p>
        <span className="author">&mdash; {quote && quote.author}</span>
      </p>
    </div>
  );
};

export { Quote };
```

这样，修改后的App.jsx就会被精简为：

```jsx
const App = () => {
  const [index, setIndex] = useState(0);
  const { loading, quotes } = useFetchQuotes();

  const clickHandler = () => {
    setIndex((index) => (index + 1) % 3);
  };

  return (
    <>
      {loading && <div>loading...</div>}
      {!loading && <Quote quote={quotes[index]} />}
      <button onClick={clickHandler}>next</button>
    </>
  );
};
```

这样每个部分的指责更加清晰，网络由自定义hook负责，具体的展现由Quote组件完成，而App则负责具体的协调和调用。更重要的是，在获得这些更整洁的代码的同时，我们的自动化测试依然是通过状态。

![end to end](ch6/e2e.png)

也就是说，我们无需担心交付的软件被不小心破坏。

## 小结

本章节我们探讨了使用React和API构建应用的主题。我们介绍了如何使用Quotable API获取数据以及如何获取数据并展现。在引入网络请求之后，学习了如何使用Cypress提供的`cy.intercept()`方法来拦截并模拟API请求。

修复测试之后，我们对当前的代码进行了重构，抽取出了自定义Hook `useFetchQuotes`，以及一个用户展示的组件Quote，从而使得代码更加整洁易读。通过一个功能的开发，我们可以明确的体会到自动化测试和构建脚本在软件开发中重要的作用。