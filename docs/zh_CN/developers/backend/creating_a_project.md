# 创建项目

使用 `linera project new` 命令初始化项目。需在 `linera-protocol` 文件夹外执行该命令，其会自动构建项目脚手架并生成必需文件：

```bash
linera project new my-counter
```

通过执行 `linera project new` 命令初始化项目时，系统将自动生成以下核心文件：

- `Cargo.toml`: y项目清单文件，包含构建应用所需的依赖项配置;
- `src/lib.rs`: 定义应用程序的二进制接口（ABI）;
- `src/state.rs`: 应用程序状态管理模块;
- `src/contract.rs`: 合约逻辑实现文件，包含合约字节码的二进制目标配置;
- `src/service.rs`: 服务端逻辑实现文件，包含服务字节码的二进制目标配置。

> 在编写Linera应用程序时，约定使用应用程序名称作为`trait`、`struct`等标识符的前缀。因此，在本手册后续内容中，我们将采用类似`CounterContract`（计数器合约）、`CounterService`（计数器服务）的命名方式。
