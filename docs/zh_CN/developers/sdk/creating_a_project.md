# 创建一个 Linera 项目

要创建你的 Linera 项目，请使用 `linera project new` 命令。该命令应在 `linera-protocol` 文件夹外执行。它会设置项目的基础结构和必要的文件：

```bash
linera project new my-counter
```

- `linera project new` 将通过创建以下关键文件来启动你的项目：
  - `Cargo.toml`：项目的清单文件，包含创建应用所需的必要依赖项；
  - `src/lib.rs`：应用的 ABI（应用二进制接口）定义；
  - `src/state.rs`：应用的状态；
  - `src/contract.rs`：应用的合约，以及合约字节码的二进制目标；
  - `src/service.rs`：应用的服务，以及服务字节码的二进制目标。
