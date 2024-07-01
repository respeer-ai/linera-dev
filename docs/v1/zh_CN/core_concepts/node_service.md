# 2.4. 节点服务

到目前为止我们已经展示Linera客户端二进制文件在终端中的用法。除此之外，Linera客户端还可以作为节点运行。Linera节点执行下列功能：

1. 执行区块，
2. 提供与应用和系统交互的GraphQL API和IDE，
3. 监听来自验证器的通知，自动更新本地微链。

将`linera`运行在`service`模式，即可与节点服务交互：

```bash
linera service
```

上面的命令将在8080的默认端口运行一个节点服务，服务端口可以通过传递`--port`标志修改。

## [关于GraphQL](https://linera-dev.respeer.ai/#/v1/zh_CN/core_concepts/node_service?id=a-note-on-graphql)

Linera使用GraphQL作为与系统不同部分交互的查询语言。GraphQL语言让客户端可以精准地从服务端获取需要的东西。

GraphQL广泛应用于应用开发领域，例如当我们需要从前端请求应用程序状态时。

更多GraphQL资料参见[官方文档](https://graphql.org/learn/)。

## [GraphiQL IDE](https://linera-dev.respeer.ai/#/v1/zh_CN/core_concepts/node_service?id=graphiql-ide)

为了方便开发者，节点服务集成了一个GraphQL IDE(GraphiQL)。在运行节点服务后，开发者通过浏览器访问`http://localhost:8080`即可使用GraphiQL。

GraphiQL IDE左半部窗口是schema浏览器，其中可以输入GraphQL查询参数，点击`播放`按钮(或者`CTRL^F5`)可以执行查询，查询结果将显示在右半部窗口。`http://localhost:8080`链接可以查看系统状态和应用列表，如果需要查询应用状态，需要使用应用列表返回的`link`。

![graphiql.png](../../node_service.assets/graphiql.png)

## [GraphQL系统API](https://linera-dev.respeer.ai/#/v1/zh_CN/core_concepts/node_service?id=graphql-system-api)

节点服务也暴露一系列系统操作的GraphQL API，点击`MutationRoot`可以查看完整操作列表。

## [GraphQL应用API](https://linera-dev.respeer.ai/#/v1/zh_CN/core_concepts/node_service?id=graphql-application-api)

Linera运行在节点服务模式时，向客户端提供应用API，开发者可以访问`http://localhost:8080/chains/<chain-id>/applications/<application-id>`，执行GraphQL查询应用在钱包管理的微链上的状态。

访问上面的链接将会在浏览器打开GraphiQL用户界面，开发者可以方便地通过图形化方式查看应用状态。
