# ​应用程序日志输出规范​​

应用程序可通过 [`log` crate](https://crates.io/crates/log) 输出不同优先级的日志信息。日志信息在开发阶段具有重要作用，同时也可为终端用户提供有效支持。默认情况下，`linera service` 命令会记录应用程序中 `info` 及以上级别的日志（即 `log::info!`、`log::warn!` 和 `log::error!`）。

在开发过程中，记录低优先级日志（如 `log::debug!` 和 `log::trace!`）具有重要调试价值。需在运行 `linera service` 前设置 `RUST_LOG` 环境变量以启用此类日志。下面的示例启用应用程序的trace级别日志和`linera`二进制程序其他模块的警告级别日志:

```ignore
export RUST_LOG="warn,linera_execution::wasm=trace"
```
