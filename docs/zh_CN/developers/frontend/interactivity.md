# 与Linera交互

在你的页面中添加一个 `<script type="module">` 标签。模块的位置无关紧要：模块脚本会在页面加载完成后延迟执行。这里将编写所有与 Linera 交互所需的 JavaScript 代码。

## 导入Linera客户端库

要将Linera客户端库添加到你的页面中，请将以下import map放入HTML的`<head>`标签内：

```html
<script type="importmap">
  {
    "imports": {
      "@linera/client": "./node_modules/@linera/client/dist/linera_web.js"
    }
  }
</script>
```

现在，`@linera/client` 模块已可在你的项目中导入：

```html
<script type="module">
  import * as linera from '@linera/client';
</script>
```

## 关于计数器应用

我们需要部署在所选网络上的计数器应用程序的应用程序ID。本教程使用Testnet Babbage网络，以下应用程序ID对应的是在该网络中发布的计数器应用：

```javascript
const COUNTER_APP_ID =
  '2b1a0df8868206a4b7d6c2fdda911e4355d6c0115b896d4947ef8e535ee3c6b8';
```

如果你希望使用不同的网络或自建后端，可能需要更改应用程序ID。只要它指向满足计数器ABI的应用程序，本教程的其余部分无需修改即可正常工作。

## 初始化

要与Linera客户端库进行交互，我们需要做的第一件事是初始化它。这将下载WebAssembly二进制文件，为其创建新内存，并完成内存的初始化工作。

```javascript
await linera.default();
```

## 获取钱包

如果你已有钱包文件，可以使用 `linera.Wallet.fromJson` 函数从文件创建钱包。不过在本教程中，我们将连接 Testnet Babbage 水龙头，创建一个带有全新链的新钱包，并为其分配一些代币。我们还会更新页面中的 `#chain-id` 元素，向用户展示其新链的 ID。

```javascript
const faucet = await new linera.Faucet(
  'https://faucet.testnet-babbage.linera.net',
);
const wallet = await faucet.createWallet();
const client = await new linera.Client(wallet);
document.getElementById('chain-id').innerText = await faucet.claimChain(client);
```

## 与应用程序交互

调用 `client.application(applicationId)` 方法将获得一个代表应用后端的对象。

```javascript
const backend = await client.frontend().application(COUNTER_APP_ID);
```

你可以使用 `query` 方法查询后端应用，该方法接受一个任意字符串作为请求发送给后端，并返回响应的 `Promise`。我们可以用此方法更新页面中的 `#count` 元素，显示计数器的当前值。

```javascript
async function updateCount() {
  const response = await backend.query('{ "query": "query { value }" }');
  document.getElementById('count').innerText = JSON.parse(response).data.value;
}

updateCount();
```

计数器应用程序使用GraphQL作为其请求语言。根据惯例，Linera应用程序接受以[Apollo Server POST格式](https://www.apollographql.com/docs/apollo-server/v2/requests)编写的JSON字符串形式的GraphQL请求，但您的应用程序可以自由采用任何所需格式。

GraphQL 的`查询`操作可用于检查应用状态，而`变更`操作会促使客户端提议包含所请求修改结果的新区块。让我们为按钮添加一个事件处理程序，用于提议增加计数器的值。

```javascript
document.getElementById('increment').addEventListener('click', () => {
  backend.query('{ "query": "mutation { increment(value: 1) }" }');
});
```

## 通知与响应性

当你点击按钮时，计数器的值会增加，但当前的UI元素不会随之更新。让我们来解决这个问题。

`Client` 对象还支持添加通知回调函数。这是 Linera 反应性机制的核心：当客户端的某个链发生状态变化时，该回调函数会立即被触发，并接收到一个包含事件详情的通知对象。

在这种情况下，我们唯一需要关注的更新是新区块的产生，因为新区块意味着计数器的值发生了变化。因此，每当我们检测到新区块时，就立即更新计数器。

```javascript
client.onNotification(notification => {
  if (notification.reason.NewBlock) updateCount();
});
```

我们作为这条链的唯一所有者，数值查询完全是本地的：除了我们之外，没有人能修改链的状态。不过在一般情况下，其他客户端用户可能通过新增区块或发送消息来更新链，而我们也会以同样的方式立即收到通知。

## 结论

大功告成！仅需几行代码，我们就实现了一个应用前端，既能与Linera测试网通信，又支持与应用进行双向通信——包括在链状态发生变化时实时更新界面。

本教程的代码在更完善的版本可以在 [`linera-web` 仓库](https://github.com/linera-io/linera-web)的 `examples/hosted-counter` 子目录中找到，旁边还有一些更复杂的示例。或者，你也可以[在线试运行](https://demos.linera.net/hosted/counter)。
