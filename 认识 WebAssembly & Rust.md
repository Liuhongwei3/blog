## 参考文章

> [为什么说 WASM 是 Web 的未来？](https://juejin.cn/post/7035991254257106958)

> https://www.zhihu.com/question/41432444/answer/3054624918

> https://juejin.cn/post/7051391151735046152

> https://developer.mozilla.org/zh-CN/docs/WebAssembly

> [据说Rust和WASM可以让Javascript变得更强，有值得推荐的项目吗？](https://www.zhihu.com/question/512116956)

> [Yew 使用](https://zhuanlan.zhihu.com/p/297920205)


建议按照上述顺序阅读。首先可以先认识一下 wasm 整体概念，接着去 MDN 上看具体描述，然后结合实际应用进一步了解，查看其特性以及应用状况，最后看看现有框架使用。

## 结论

wasm 的优势就是轻量、性能更好（在一些需要高性能领域），更接近底层等，同时可以使得 c/c++、rust、go 等编写的程序转化为 wasm 并结合 js 胶水代码在浏览器端运行（跨平台），不断衍生出一些工具和开发框架，可以探索一下，但是离具体前端场景应用可能还需要进一步深入了解，个人觉得应该是在构建打包、大型（高性能）应用（使用 js 性能过低场景）等方面应用。

### 2023年8月3日19:43:22 更新

探索了一下 assemblyscript 的使用，上手还是较快，可以直接使用类似 ts 的语法编写，然后使用 asc 编译后就会自动生成 js 胶水代码（加载和执行wasm）、wasm 文件、d.ts 类型代码，整体使用还行，至于在生产环境中的表现以及相关生态需要进一步验证

而 napi-rs 是一个用于构建 Node.js 插件的 Rust 库。它提供了一组 Rust 绑定，使得开发者可以使用 Rust 编写高性能的 Node.js 模块。napi-rs 提供了一种简单的方式来与 JavaScript 进行交互，并且具有良好的性能。

在进一步探索中未发现在现有生产环境中的落地处，暂时当做进一步学习和了解
