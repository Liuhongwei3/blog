## 概述



## WebAssembly

### 简介

> [wasm](https://webassembly.org/): WebAssembly (abbreviated *Wasm*) is a binary instruction format for a stack-based virtual machine. Wasm is designed as a portable compilation target for programming languages, enabling deployment on the web for client and server applications.
>
> WebAssembly 是基于栈式虚拟机的二进制指令集，可以作为编程语言的编译目标，能够部署在 web 客户端和服务端的应用中。

### 特点

- 高效快捷：它是体积小且加载快的二进制格式，充分发挥硬件能力使其能够达到原生语言的执行速度
- 安全：它运行在一个内存安全的沙箱执行环境，可以在现有的 js 虚拟机中实现；嵌入到 web 中时会强制执行浏览器的同源和权限安全策略
- 开放且可调试：WebAssembly被设计为以[文本格式](https://webassembly.org/docs/text-format/)精美打印，用于调试，测试，实验，优化，学习，教学和手动编写程序。在网络上查看 Wasm 模块[的源代码](https://webassembly.org/docs/faq/#will-webassembly-support-view-source-on-the-web)时将使用文本格式
- 开放式 web 平台的一部分：WebAssembly 旨在维护 [Web 的](https://webassembly.org/docs/web/)无版本、功能测试和向后兼容的性质；可以被 JavaScript 调用，进入 JavaScript 上下文，也可以像 Web API 一样调用浏览器的相关功能；不仅可以运行在浏览器上，也可以运行在非 Web 环境下（如 Node.js、物联网设备等）

> MDN 解释：https://developer.mozilla.org/zh-CN/docs/WebAssembly/Concepts

总结：体积小、加载快、安全、支持在浏览器和非 node 环境运行，从而使得非 js 语言（比如 c/c++)底层语言也能在浏览器中运行。

### 历史

2013 年，asm.js 问世，算是 wasm 的前身（具体细节见：[了解 asm.js](https://baijiahao.baidu.com/s?id=1621291166647607978&wfr=spider&for=pc))，主要针对数值进行类型设定使其应用场景主要为数值计算密集型场景，由于一些问题和局限性使其发展并未如同预期。

2017 年，上述四大主要浏览器厂商达成共识，开始支持 wasm 并不断进行改进完善。

2019 年，wasm 正式成为第四门 web 语言并发布 1.0 版本。

2022 年，wasm 2.0 草案发布，最新介绍见：https://webassembly.github.io/spec/core/intro/introduction.html。

目前已被主要浏览器引擎支持：Firefox、Chrome、Safari、Edge

[<img src="https://pic.imgdb.cn/item/64cc99d61ddac507cc389f59.jpg" alt="wasm-roadmap"/>](https://pic.imgdb.cn/item/64cc99d61ddac507cc389f59.jpg)

补充：[FAQ - WebAssembly](https://webassembly.org/docs/faq/) （其中还提到了 LLVM，比较核心不过笔者生疏，有兴趣者可以深入一下）

### 原理

首先抛出一个疑问：js 和 V8 已经足够强大了，为什么还是需要 wasm 呢？到底能带来更多的什么收益？

回答这个问题前先来看一下各自的运行机制：

#### js

![js-running](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/114c4cc2304f470783a2a7dfe71c4988~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

总结版：

![](https://mmbiz.qpic.cn/mmbiz_jpg/33P2FdAnju9qIZLt4XPwrceNLxdOMMlrZ8LYqUlFNXCxPv7oAiaPW87Giax61TARmKfLdmBtCOg86OAxHzrp2QBg/640?wx_fmt=jpeg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

- 加载源代码
- parser 将代码解析成 AST 抽象语法树
- 解释器进行解释然后编译再执行
- 最后再进行垃圾回收

可以发现引擎每次都需要解析和不断优化。监视器会持续监视代码的运行，若发现执行频次较高的代码则会标记为热点代码并进一步优化，优化后的代码会比之前执行的快得多，不过这个优化动作也会消耗一些时间，不过这些优化都是需要基于类型推断所做的，恰好 js 是动态类型的，就会使得引擎很可能推断类型失败，那么之前优化的代码就需要删掉重来一次，进一步使其多消耗一些时间；然后垃圾回收时是删除内存中所有的非活动对象（具体见：[V8 垃圾回收](https://blog.csdn.net/weixin_43832981/article/details/128803266))。

#### wasm

![](https://mmbiz.qpic.cn/mmbiz_jpg/33P2FdAnju9qIZLt4XPwrceNLxdOMMlrqeJ9XkRl18CzdxZDFuwucZdW6FLXncmRgBDIFFOghBLYPzowdpbYpg/640?wx_fmt=jpeg&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

- 加载 wasm（已经是被编译和优化过的二进制）
- 同样解析成 AST (如果不在 web 运行，可能不会有这一步？待验证)
- 编译为机器码，不过由于其已被提前编译和优化过，所以该阶段会很快
- 模块实例化并执行，在实例化的时候，JS 引擎会实例化状态和执行栈，最后再执行模块。

总结：wasm 执行步骤明显少于 js，并且编译前 wasm 已经被提前编译优化，进一步缩短执行前的准备时间，加上其流式编译就更上一层楼。

从理论上来说确实执行会很快，不过并非始终比原生 js 快，比如一些简单的计算场景，可能两者整体上差距不大，但是 wasm 会多一步与 js 引擎的交互过程，也需要耗费时间，所以正如上述补充的官方 faq 中提到的：wasm 不是替代 js，而是补充 js，而是在一些高计算需要的场景使用，例如，构建、图形、模拟、图像/声音/视频处理、可视化、 动画、压缩等。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a942ef894b484f9893496fcc014bd3e7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 应用

综上所述，wasm 作为一个拥有上述四大特点的二进制中间产物，那么不同的开发者都可以很好的应用。比如对 js 开发者来说，为了享受高性能，则可以使用 rust/assemblyScript 生成 wasm 来实现；而 c/c++/rust 开发者也可以利用自身语言优势生成 wasm 提供给 js 开发者使用或者在 web 使用。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2456bc5625a4867abe881de5dc37029~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

- Google Earth
- Figma - 画板
- Bilibili - 投稿

### 语言

> 抛出一个小疑问，为什么 wasm 是使用 rust/C/C++/assemblyScript 这些语言，而不是 java/C# ？感兴趣同学可业余探索一下~

![](https://mmbiz.qpic.cn/mmbiz_png/lP9iauFI73zibNGBUCqjMsywO8cicPV6cgOC6KlbSQtxARncr5icrKral1ALMG5OA2aJ8iaicAro09qGscDFlM1p2vrQ/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

#### Rust

> [rust-lang](https://www.rust-lang.org/)：A language empowering everyone to build reliable and efficient software.

##### 环境搭建

点击官网的安装菜单，如果是 windows 直接下载对应 exe 程序并运行即可。

```shell
C:\Users\admin>rustup -V
rustup 1.26.0 (5af9b9484 2023-04-05)
info: This is the version for the rustup toolchain manager, not the rustc compiler.
info: The currently active `rustc` version is `rustc 1.71.0 (8ede3aae2 2023-07-12)`

C:\Users\admin>rustc -V
rustc 1.71.0 (8ede3aae2 2023-07-12)

C:\Users\admin>cargo -V
cargo 1.71.0 (cfd3bbd8f 2023-06-08)
```

##### 示例

> https://doc.rust-lang.org/rust-by-example/hello.html

由于笔者暂未深入了解，所以此处暂不深入展开，后续文章中可按需产出。（< ^_^ >）

#### AssemblyScript

> [AssemblyScript](https://www.assemblyscript.org/getting-started.html#setting-up-a-new-project)：A TypeScript-like language for WebAssembly.

##### 初始化小例子

```shell
cd assemblyScript-demo
npm init
npm install --save-dev assemblyscript
npx asinit .
npm run asbuild
npm test
npm start
```

```typescript
// index.ts
export function add(a: i32, b: i32): i32 {
  return a + b;
}
```

```html
<script type="module">
import { add } from "./build/release.js";
document.body.innerText = add(1, 2);
</script>
```

```javascript
// 胶水代码
async function instantiate(module, imports = {}) {
  const { exports } = await WebAssembly.instantiate(module, imports);
  return exports;
}
export const {
  memory,
  add,
} = await (async url => instantiate(
  await (async () => {
    try { return await globalThis.WebAssembly.compileStreaming(globalThis.fetch(url)); }
    catch { return globalThis.WebAssembly.compile(await (await import("node:fs/promises")).readFile(url)); }
  })(), {
  }
))(new URL("release.wasm", import.meta.url));
```

build 生成的文件：

- release.d.ts：类型声明
- release.js：加载执行 wasm 的胶水代码
- release.wasm：打包后的 wasm 文件

使用 asc 打包，可以在项目中使用 `ascconfig.json` 进行打包配置

```json
{
  "targets": {
    "debug": {
      "outFile": "build/debug.wasm",
      "textFile": "build/debug.wat",
      "sourceMap": true,
      "debug": true
    },
    "release": {
      "outFile": "build/release.wasm",
      "textFile": "build/release.wat",
      "sourceMap": true,
      "optimizeLevel": 3,
      "shrinkLevel": 0,
      "converge": false,
      "noAssert": false
    }
  },
  "options": {
    "bindings": "esm"
  }
}
```

打包命令：

```shell
npx asc assembly/index.ts --target release
```

打开 `index.html` 即可查看结果。

##### 更多例子（官方）

> https://www.assemblyscript.org/examples/game-of-life.html

###### 核心代码

```javascript
// Compute the size of and instantiate the module's memory
var memory = new WebAssembly.Memory({
  initial: ((byteSize + 0xffff) & ~0xffff) >>> 16
});

// Fetch and instantiate the module
fetch("build/release.wasm")
.then(response => response.arrayBuffer())
.then(buffer => WebAssembly.instantiate(buffer, {
  env: {
    memory,
    abort() {},
    "Math.random": Math.random
  },
  config: {
    BIT_ROT,
    BGR_ALIVE: rgb2bgr(RGB_ALIVE) | 1, // little endian, LSB must be set
    BGR_DEAD:  rgb2bgr(RGB_DEAD) & ~1, // little endian, LSB must not be set
  },
}))
.then(module => {
    var exports = module.instance.exports;

  // Initialize the module with the universe's width and height
  exports.init(width, height);
}).catch(err => {
  alert("Failed to load WASM: " + err.message + " (ad blocker, maybe?)");
  console.log(err.stack);
});
```

###### 预览

![](https://pic.imgdb.cn/item/64ccd4901ddac507ccc751bd.jpg)

### napi-rs

> https://napi.rs/cn：是一个使用 **Rust** 构建预编译 **Node.js** 原生扩展的框架，提供了一组 Rust 绑定，使得开发者可以使用 Rust 编写高性能的 Node.js 模块。
>
> https://juejin.cn/post/7243413934765408315

### yew

> https://yew.rs/zh-Hans/：A framework for creating reliable and efficient web applications.

## 更多阅读

- [MDN-WASM](https://developer.mozilla.org/zh-CN/docs/WebAssembly)
- [十分钟搞懂 WebAssembly](https://mp.weixin.qq.com/s/hE8TMmSXKcylXY9pXmlw9Q)
- [认识 WebAssembly 与 Rust 实践](https://mp.weixin.qq.com/s/NA3lXimLOzPe_C91KicysQ)
- [走进 WebAssembly 的世界](https://juejin.cn/column/7210666370487681082) (系列文章，讲述了从认识到实战，非常硬核)
- [为什么说 WASM 是 Web 的未来？](https://juejin.cn/post/7035991254257106958)
- [据说Rust和WASM可以让Javascript变得更强，有值得推荐的项目吗？](https://www.zhihu.com/question/512116956)

