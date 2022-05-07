> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/37653878)

本文作者对比了当前主流的包管理工具 npm、yarn、pnpm 之间的区别，并提出了合适的使用建议，以下为译文：

NPM
---

npm 是 Node.js 能够如此成功的主要原因之一。npm 团队做了很多的工作，以确保 npm 保持向后兼容，并在不同的环境中保持一致。

npm 是围绕着[语义版本控制（semver）](https://link.zhihu.com/?target=http%3A//semver.org/)的思想而设计的，下面是从他们的网站摘抄过来的：

_给定一个版本号：主版本号. 次版本号. 补丁版本号， 以下这三种情况需要增加相应的版本号:_

*   _主版本号： 当 API 发生改变，并与之前的版本不兼容的时候_
*   _次版本号： 当增加了功能，但是向后兼容的时候_
*   _补丁版本号： 当做了向后兼容的缺陷修复的时候_

npm 使用一个名为 package.json 的文件，用户可以通过 npm install --save 命令把项目里所有的依赖项保存在这个文件里。

例如，运行 npm install --save lodash 会将以下几行添加到 package.json 文件中。

```
"dependencies": {
    "lodash": "^4.17.4"
}
```

请注意，在版本号 lodash 之前有个 ^ 字符。这个字符告诉 npm，安装主版本等于 4 的任意一个版本即可。所以如果我现在运行 npm 进行安装，npm 将安装 lodash 的主版本为 4 的最新版，可能是 lodash@4.25.5（@是 npm 约定用来确定包名的指定版本的）。你可以在此处查看所有支持的字符：[https://docs.npmjs.com/misc/semver](https://link.zhihu.com/?target=https%3A//docs.npmjs.com/misc/semver)。

理论上，次版本号的变化并不会影响向后兼容性。因此，安装最新版的依赖库应该是能正常工作的，而且能引入自 4.17.4 版本以后的重要错误和安全方面的修复。

但是，另一方面，即使不同的开发人员使用了相同的 package.json 文件，在他们自己的机器上也可能会安装同一个库的不同种版本，这样就会存在潜在的难以调试的错误和 “在我的电脑上…” 的情形。

大多数 npm 库都严重依赖于其他 npm 库，这会导致嵌套依赖关系，并增加无法匹配相应版本的几率。

虽然可以通过 npm config set save-exact true 命令关闭在版本号前面使用的默认行为，但这个只会影响顶级依赖关系。由于每个依赖的库都有自己的 package.json 文件，而在它们自己的依赖关系前面可能会有符号，所以无法通过 package.json 文件为嵌套依赖的内容提供保证。

为了解决这个问题，npm 提供了 [shrinkwrap](https://link.zhihu.com/?target=https%3A//docs.npmjs.com/cli/shrinkwrap) 命令。此命令将生成一个 npm-shrinkwrap.json 文件，为所有库和所有嵌套依赖的库记录确切的版本。

然而，即使存在 npm-shrinkwrap.json 这个文件，npm 也只会锁定库的版本，而不是库的内容。即便 npm 现在也能阻止用户多次重复发布库的同一版本，但是 npm 管理员仍然具有强制更新某些库的权力。

这是引用自 shrinkwrap 文档的内容：

_如果你希望锁定包中的特定字节，比如是为了保证能正确地重新部署或构建，那么你应该在源代码控制中检查依赖关系，或者采取一些其他的机制来校验内容，而不是靠校验版本。_

npm 2 会安装每一个包所依赖的所有依赖项。如果我们有这么一个项目，它依赖项目 A，项目 A 依赖项目 B，项目 B 依赖项目 C，那么依赖树将如下所示：

```
node_modules
- package-A
-- node_modules
--- package-B
----- node_modules
------ package-C
-------- some-really-really-really-long-file-name-in-package-c.js
```

这个结构可能会很长。这对于基于 Unix 的操作系统来说只不过是一个小烦恼，但对于 Windows 来说却是个破坏性的东西，因为有很多程序无法处理超过 260 个字符的文件路径名。

npm 3 采用了扁平依赖关系树来解决这个问题，所以我们的 3 个项目结构现在看起来如下所示：

```
node_modules
- package-A
- package-B
- package-C
-- some-file-name-in-package-c.js
```

这样，一个原来很长的文件路径名就从./node_modules/package-A/node_modules/package-B/node-modules/some-file-name-in-package-c.js 变成了 / node_modules/some-file-name-in-package-c.js。

你可以在[这里](https://link.zhihu.com/?target=https%3A//docs.npmjs.com/how-npm-works/npm3)阅读到更多有关 NPM 3 依赖解析的工作原理。

这种方法的缺点是，npm 必须首先遍历所有的项目依赖关系，然后再决定如何生成扁平的 node_modules 目录结构。npm 必须为所有使用到的模块构建一个完整的依赖关系树，这是一个耗时的操作，是 [npm 安装速度慢的一个很重要的原因](https://link.zhihu.com/?target=https%3A//github.com/npm/npm/issues/8826)。

由于我没有详细了解 npm 的变化，所以我想当然的以为每次运行 npm install 命令时，NPM 都得从互联网上下载所有内容。

但是，我错了，npm 是有本地缓存的，它保存了已经下载的每个版本的压缩包。本地缓存的内容可以通过 npm cache ls 命令进行查看。本地缓存的设计有助于减少安装时间。

总而言之，npm 是一个成熟、稳定、并且有趣的包管理器。

Yarn
----

Yarn 发布于 2016 年 10 月，并在 Github 上迅速拥有了 2.4 万个 Star。而 npm 只有 1.2 万个 Star。这个项目由一些高级开发人员维护，包括了 Sebastian McKenzie（[Babel.js](https://link.zhihu.com/?target=https%3A//babeljs.io/)）和 Yehuda Katz（[Ember.js](https://link.zhihu.com/?target=https%3A//www.emberjs.com/)、[Rust](https://link.zhihu.com/?target=https%3A//www.rust-lang.org/en-US/)、[Bundler](https://link.zhihu.com/?target=http%3A//bundler.io/) 等）。

从我搜集到的情况来看，Yarn 一开始的主要目标是解决上一节中描述的由于语义版本控制而导致的 npm 安装的不确定性问题。虽然可以使用 npm shrinkwrap 来实现可预测的依赖关系树，但它并不是默认选项，而是取决于所有的开发人员知道并且启用这个选项。

Yarn 采取了不同的做法。每个 yarn 安装都会生成一个类似于 npm-shrinkwrap.json 的 yarn.lock 文件，而且它是默认创建的。除了常规信息之外，yarn.lock 文件还包含要安装的内容的校验和，以确保使用的库的版本相同。

由于 yarn 是崭新的经过重新设计的 npm 客户端，它能让开发人员并行化处理所有必须的操作，并添加了一些其他改进，这使得运行速度得到了显著的提升，整个安装时间也变得更少。我估计，速度提升是 yarn 受欢迎的主要原因。

像 npm 一样，yarn 使用本地缓存。与 npm 不同的是，yarn 无需互联网连接就能安装本地缓存的依赖项，它提供了离线模式。这个功能在 2012 年的 npm 项目中就被提出来过，但一直没有实现。

yarn 还提供了一些其他改进，例如，它允许合并项目中使用到的所有的包的许可证，这一点让人很高兴。

一个有趣的事情是，yarn 文档的态度开始针对 npm 发生改变，因为 yarn 项目变得流行起来。

最开始的 yarn 公告是这么介绍 yarn 的安装的：

* 最简单的入门方法是运行：

```
npm install -g yarn 
yarn*
```

现在的 yarn 安装页面是这么说的：

_注意：通常情况下不建议通过 npm 进行安装。npm 安装是非确定性的，程序包没有签名，并且 npm 除了做了基本的 SHA1 哈希之外不执行任何完整性检查，这给安装系统程序带来了安全风险。_

_基于这些原因，强烈建议你通过最适合于你的操作系统的安装方法来安装 yarn。_

以这种速度发展下去的话，如果 yarn 要宣布他们自己的 registry，让开发者慢慢淘汰 npm 的话，我们一点都不会感到惊讶。

看起来似乎要感谢 yarn，npm 终于意识到他们需要更加关注一些大家强烈要求的问题了。当我在审核我之前提到的强烈要求的 “离线” 功能时，我注意到这个需求正在被积极地修复之中。

pnpm
----

正如我所提到的，在 [pnpm](https://link.zhihu.com/?target=https%3A//github.com/pnpm/pnpm) 的作者 Zoltan Kochan 发表了 “[为什么要用 pnpm？](https://link.zhihu.com/?target=https%3A//www.kochan.io/nodejs/why-should-we-use-pnpm.html)” 之后，我才知道 pnpm。

我不会介绍太多的细节（因为这篇文章已经发布很久了），但是你可以查看我的[最初的帖子](https://link.zhihu.com/?target=https%3A//medium.com/%40akras14/omg-npm-clone-that-finally-makes-sense-3478588879)来寻找更多的内容，同时[在 Twitter 上加入讨论](https://link.zhihu.com/?target=https%3A//twitter.com/akras14/status/855474658832900096)。

**但是**

我想指出的是，pnpm 运行起来非常的快，甚至[超过了 npm 和 yarn](https://link.zhihu.com/?target=https%3A//github.com/pnpm/node-package-manager-benchmark)。

为什么这么快呢？ 因为它采用了一种巧妙的方法，利用硬链接和符号链接来避免复制所有本地缓存源文件，这是 yarn 的最大的性能弱点之一。

使用链接并不容易，会带来一堆问题需要考虑。

正如 Sebastian 在 [Twitter](https://link.zhihu.com/?target=https%3A//twitter.com/sebmck/status/855553631680069637) 上指出的那样，他最初是打算在 yarn 中使用符号链接的，但是由于其他[一些原因](https://link.zhihu.com/?target=https%3A//github.com/yarnpkg/yarn/issues/1761%23issuecomment-259706202)放弃了它。

同时，正如在 Github 上拥有 2000 多个 Star 那样，pnpm 能够为许多人所用。

此外，截至 2017 年 3 月，它继承了 yarn 的所有优点，包括离线模式和确定性安装。

总结
--

我认为 yarn 和 pnpm 的开发人员做了一个惊人的工作。我个人喜欢的是确定性安装，因为我喜欢控制，我不喜欢惊喜。

无论这场竞争的结果是什么，我很感谢 yarn 在 npm 的脚下点了一把火，提供了另外一个选择。

我确信 yarn 是一个更安全的选择，但是 pnpm 可能是一些测试用例的更好的选择。例如，它可以在运行大量集成测试并希望尽可能快地安装依赖关系的中小型团队中发挥作用。

最后，我认为，npm 仍然提供了一个非常有用的解决方案，支持大量的测试用例。大多数开发人员使用原始 npm 客户端仍然可以做得很好。

更多 RN 参考
--------

[react-native 技术的优劣](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzI3OTU0MzI4MQ%3D%3D%26mid%3D2247485690%26idx%3D1%26sn%3D44537ca3fcfb5347df3dde1a388cc4dc%26chksm%3Deb476464dc30ed72a0a9f1cabd86375a0a18bd1478e8ca7e17bb7bcc81bc9ebc553b5f24c1f5%26scene%3D21%23wechat_redirect)

[学习 React Native 必看的几个开源项目](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzI3OTU0MzI4MQ%3D%3D%26mid%3D2247485812%26idx%3D1%26sn%3D4214cefd6a686c614eac9a1e56ce7290%26chksm%3Deb4765eadc30ecfca3c965f2ff8792b6d5f4ac017c7a90cf23ca86d2ec7761f6c624259ffcf8%26scene%3D21%23wechat_redirect)

[手把手教你 React Native 实战之开山篇《一》](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzI3OTU0MzI4MQ%3D%3D%26mid%3D2247485769%26idx%3D1%26sn%3D5b2307bdda6fd3e399731b31757583e2%26chksm%3Deb4765d7dc30ecc1eb68b0260c85bafcbab9dab4d9682d22f30fc06e8f7c85b34348e15d1894%26scene%3D21%23wechat_redirect)

​[手把手教你 React Native 实战从 React 到 Rn《二》](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzI3OTU0MzI4MQ%3D%3D%26mid%3D2247485835%26idx%3D1%26sn%3D8c91c8da4747657ab71b1edd104ff521%26chksm%3Deb476515dc30ec036fabde3a7bfc5a00208991fcc59f9f7984c9152e9f489fee1ce7bd787030%26scene%3D21%23wechat_redirect)