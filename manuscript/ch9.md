# 后记

在这本书的后记中，我想回顾一下我们所经历的旅程，这是一个从构建一个简单的静态页面开始，逐步引入各种工具和技术，到最后创建一个完全自动化的，可以持续集成和交付的全栈应用的旅程。

我们的旅程从一个简单的静态页面开始。静态页面的优点在于其简洁性和易于理解，但随着应用的增长，静态页面的限制也开始显现。于是，我们引入了`package.json`和`scripts`，使得我们的构建过程更具可配置性和灵活性。这也让我们开始接触了自动化的概念，通过运行简单的脚本，我们可以执行一系列的任务，从而节省了手动操作的时间。

然后，我们引入了静态服务器。通过静态服务器，我们可以更好地模拟实际的服务器环境，帮助我们提前发现和解决在生产环境中可能遇到的问题。同时，静态服务器也为我们提供了一个开发环境，我们可以在此环境中测试我们的应用，而无需每次都将其部署到生产环境。

接下来，我们引入了React和打包器（bundler）。React作为目前最受欢迎的前端框架，它提供了一种模块化的方式来构建复杂的用户界面，而打包器则帮助我们将多个文件和模块打包成一个或多个优化过的文件，使得我们的应用能够在不同的环境中运行。

为了确保我们的代码质量，我们接下来引入了静态检查工具。通过静态检查，我们可以在代码还没有运行之前发现可能的错误和问题，这对于大型项目来说是非常重要的。

我们继续通过引入Cypress来进行验收测试。验收测试是一个很好的自动化测试手段，可以帮助我们确保应用的各个部分都按照预期运行。并且，我们还使用了文件监视（watch）功能，这让我们在修改代码后，可以立即看到结果，提高了我们的开发效率。

然后，我们开始引入后台API的交互，这是我们的应用从静态页面变成一个真正的动态应用的关键步骤。为了让测试更稳定，我们引入了stub数据，模拟了API的响应。这样，我们的测试就不再依赖于实际的后端服务器，使得测试结果更可靠，更具有可预测性。

在我们的旅程的最后阶段，我们引入了持续集成和持续交付，以及GitHub Actions。通过持续集成和持续交付，我们可以在代码提交后立即运行测试，然后将代码部署到生产环境，这使得我们的开发流程更为自动化，也更加高效。而GitHub Actions则是一个强大的工具，让我们可以在GitHub仓库中直接设置和运行自动化工作流程。

最后，我们将应用部署到了GitHub Pages上。GitHub Pages是一个免费的静态站点托管服务，我们可以很容易地将我们的应用发布到公网上，使得其他人也可以访问和使用我们的应用。

在这个旅程中，我们经历了从一个简单的静态页面到一个复杂的全栈应用的转变，我们引入并学习了许多工具和技术，包括React，bundler，静态检查，自动化测试，文件监视，后端API交互，stub数据，持续集成和交付，GitHub Actions，以及应用部署。每一步都让我们的应用更加完善，也使得我们的开发过程更加自动化和高效。

这就是我们的旅程，一次从零到一，从简单到复杂，从手动到自动化的旅程。这个过程既有挑战，也有成就，但最重要的是，我们通过这个过程，深入了解了现代前端开发的各个方面，从而成为了更好的开发者。希望这本书能够帮助你在你的开发旅程中，无论你是初学者还是有经验的开发者，都能有所收获。