# 打印应用程序日志

应用程序可以使用 [`log` crate](https://crates.io/crates/log) 来打印不同重要性级别的日志消息。日志消息在开发过程中非常有用，但对最终用户也可能很有帮助。默认情况下，如果日志消息的重要性级别达到 "info" 或更高（例如 `log::info!`, `log::warn!`, `log::error!`），`linera service` 命令将记录这些消息。

在开发过程中，通常希望记录更低重要性级别的消息（如 `log::debug!` 和 `log::trace!`）。要启用它们，必须在运行 `linera service` 命令之前设置 `RUST_LOG` 环境变量。下面的示例启用了来自应用程序的 trace 级别消息，并启用了来自 `linera` 二进制的 warning 级别消息：

```ignore
export RUST_LOG="warn,linera_execution::wasm=trace"
```
