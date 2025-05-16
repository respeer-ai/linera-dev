# 搭建前端开发环境​​

## 支持的浏览器​​

截至本文撰写时（2023年基准），Linera客户端库已兼容主流浏览器。但其确实依赖部分较新的浏览器特性，若您的浏览器版本过低，可能难以正常使用本教程。具体要求如下：

- [import maps](https://caniuse.com/import-maps)
- [`SharedArrayBuffer`](https://caniuse.com/sharedarraybuffer)
- [WebAssembly threading primitives](https://caniuse.com/wasm-threads)
- [top-level `await`](https://caniuse.com/mdn-javascript_operators_await_top_level)
  — 此写法是为教程简洁性而设计，但若您的浏览器不支持相关特性，可轻松将其抽离并替换为替代方案。

## ​创建基础HTML页面

首先，我们创建一个简单的HTML用户界面。此页面暂时不会连接到Linera，但我们可以将其作为脚手架来搭建开发环境。我们将这个页面命名为`index.html`，并且它将是我们构建前端时唯一需要编辑的文件。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Counter</title>
  </head>
  <body>
    <p>Chain: <span id="chain-id">requesting chain…</span></p>
    <p>Clicks: <span id="count">0</span></p>
    <button id="increment">Click me!</button>
  </body>
</html>
```

## 运行前端服务

为了使用 JavaScript 客户端 API，我们需要一个 Web 服务器。由于我们使用 [`SharedArrayBuffer`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) 在 WebAssembly 线程之间共享内存，通过 `file://` URI 从磁盘运行前端将无法正常工作，因为 `SharedArrayBuffer` 需要跨源隔离[cross-origin isolation](https://developer.mozilla.org/en-US/docs/Web/API/Window/crossOriginIsolated)来保证安全性。

在本教程中，我们将使用 [`http-server`](https://github.com/http-party/http-server)，但只要服务器能够设置 `Cross-Origin-Opener-Policy` 和 `Cross-Origin-Embedder-Policy` 头部，任何服务器都可以使用。

要使用`http-server`，首先请确保已安装Node.js。在Ubuntu系统上，可以通过以下命令完成安装：

```shellsession
sudo apt install nodejs
```

接下来，输入命令

```shellsession
npx http-party/http-server \
  --header Cross-Origin-Embedder-Policy:require-corp \
  --header Cross-Origin-Opener-Policy:same-origin
```

可以用于在`localhost`上托管我们的HTML页面。

```admonish info
Note that we use `http-party/http-server` here to use `http-server`
from GitHub.  Writing just `http-server` will pull the version from
npm, which at the time of writing is very old and doesn't support
custom headers.
```

## 获取客户端库

整个Linera客户端（包括WebAssembly及其所有内容）已发布到Node包仓库，包名为[`@linera/client`](https://www.npmjs.com/package/@linera/client)。我们将通过以下方式将其引入`node_modules`：


```shellsession
npm install @linera/client@0.14.0
```

````admonish warning title="A note on bundlers"
We're serving our `node_modules` here, so no bundling step is
required.  However, if you do choose to bundle your frontend, it is
important that both the Web worker entry point and the
`@linera/client` library itself remain in separate files, with their
signatures intact, in order for the Web worker to be able to refer to
them.  For example, if using Vite, make sure to define an extra
entrypoint for `@linera/client`, preserve its signature, and exclude
it from dependency optimization:

``` typescript
export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        index: 'index.html',
        linera: '@linera/client',
      },
      preserveEntrySignatures: 'strict',
    },
  },
  optimizeDeps: {
    exclude: [
      '@linera/client',
    ],
  },
})
```
````
