# 3.1. 创建项目

使用`linera project new`命令创建Linera项目，该命令应该在`linera-protocol`目录外执行，执行`linera project new`将会帮助开发者建立应用脚手架和必要文件：

```bash
linera project new my-counter
```

`linera project new`创建下列关键文件，作为项目的引导程序：

- `Cargo.toml`: 项目manifest，其中包含创建应用的必要依赖；
- `src/lib.rs`: 应用的ABI定义；
- `src/state.rs`: 应用状态；
- `src/contract.rs`: 应用合约，将被编译为合约字节码；
- `src/service.rs`: 应用服务，将被编译为服务字节码；
- `.cargo/config.toml`: 项目编译器配置，设定`cargo`默认编译`wasm32-unknown-unknown`架构目标文件。
