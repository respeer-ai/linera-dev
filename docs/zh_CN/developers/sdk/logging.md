# 在应用中打印日志

应用可以使用[`log` crate](https://crates.io/crates/log)根据重要性打印不同级别的日志。对于开发者和终端用户来说，日志都是有用的。`linera service`命令默认将会打印"info"(`log::info!`, `log::warn!` and `log::error!`)级别以上的应用日志。

开发阶段，我们可以通过设置`linera service`的`RUST_LOG`环境变量打开更低日志级别(例如`log::debug!` and `log::trace!`)。下面的示例展示了为应用启用trace级别日志，并将系统其余部分日志设置为warn级别：

```ignore
export RUST_LOG="warn,linera_execution::wasm=trace"
```
