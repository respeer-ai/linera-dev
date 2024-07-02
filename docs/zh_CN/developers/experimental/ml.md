# 在 Linera 上的机器学习

Linera 应用程序的合同/服务分离允许在边缘安全高效地运行机器学习模型。

应用程序的合同通过共识算法强制执行所有正确性保证，从而检索正确的模型，而客户端则在无计量服务的链下执行推理。由于服务运行在用户自己的硬件上，因此可以被隐式信任。

## 指南

现有示例使用 [Hugging Face](https://huggingface.co/) 提供的 [`candle`](https://github.com/huggingface/candle) 框架作为底层机器学习框架。

`candle` 是一个以性能和可用性为重点的 Rust 最小化机器学习框架。它还可以编译为 Wasm，在浏览器内外都有很好的 Wasm 支持。查看 candle 的 [示例](https://github.com/huggingface/candle/tree/main/candle-wasm-examples) 来获得支持的模型类型的灵感。

### 入门

要向现有的 Linera 项目添加机器学习功能，您需要将以下依赖项添加到 Linera 项目的 `Cargo.toml` 文件中：

```toml
candle-core = "0.4.1"
getrandom = { version = "0.2.12", default-features = false, features = ["custom"] }
rand = "0.8.5"
```

如果需要运行大型语言模型，还需要以下 crate：

```toml
candle-transformers = "0.4.1"
tokenizers = { git = "https://github.com/christos-h/tokenizers", default-features = false, features = ["unstable_wasm"] }
```

### 提供随机性

机器学习框架使用随机数进行推理。Linera 服务运行在 Wasm VM 中，无法访问操作系统的随机数生成器。因此，我们需要手动种子化 `candle` 使用的随机数生成器。

创建一个文件 `src/random.rs` 并添加以下内容：

```rust,ignore
use std::sync::{Mutex, OnceLock};

use rand::{rngs::StdRng, Rng, SeedableRng};

static RNG: OnceLock<Mutex<StdRng>> = OnceLock::new();

fn custom_getrandom(buf: &mut [u8]) -> Result<(), getrandom::Error> {
    let seed = [0u8; 32];
    RNG.get_or_init(|| Mutex::new(StdRng::from_seed(seed)))
        .lock()
        .expect("failed to get RNG lock")
        .fill(buf);
    Ok(())
}

getrandom::register_custom_getrandom!(custom_getrandom);
```

这将使得 `candle` 和依赖于 `getrandom` 的其他 crate 能够访问确定性的随机数生成器。如果不需要确定性行为，也可以使用系统 API 从时间戳种子化随机数生成器。

### 加载模型到服务中

目前无法将模型保存在链上，请参阅下面的 `限制` 部分。

要执行模型推理，必须将模型加载到服务中。为此，我们将在服务收到查询时使用 `fetch_url` API：

```rust,ignore
impl Service for MyService {
    async fn handle_query(&self, request: Request) -> Response {
        // do some stuff here
        let raw_weights = self.runtime.fetch_url("https://my-model-provider.com/model.bin");
        // do more stuff here
    }
}
```

模型可以从本地 web 服务器提供或直接从模型提供者（如 Hugging Face）获取。

此时，我们获得了与模型和分词器对应的原始字节。`candle` 支持多种模型权重存储格式，包括量化和非量化的 (`gguf`, `ggml`, `safetensors` 等)。

根据您使用的模型格式，`candle` 提供了便利函数将字节转换为类型化的 `struct`，用于执行推理。以下是非量化 Llama 2 模型的示例：

```rust,ignore
    fn load_llama_model(cursor: &mut Cursor<Vec<u8>>) -> Result<(Llama, Cache), candle_core::Error> {
        let config = llama2_c::Config::from_reader(cursor)?;
        let weights =
            llama2_c_weights::TransformerWeights::from_reader(cursor, &config, &Device::Cpu)?;
        let vb = weights.var_builder(&config, &Device::Cpu)?;
        let cache = llama2_c::Cache::new(true, &config, vb.pp("rot"))?;
        let llama = Llama::load(vb, config.clone())?;
        Ok((llama, cache))
    }
```

### 推理

使用 `candle` 执行推理并不是一种“一刀切”的过程。不同的模型需要不同的逻辑来执行推理，因此如何执行推理的具体细节超出了本文档的范围。

幸运的是，有多个示例可以作为在 Wasm 中执行推理的指南：

- [Llm Stories](https://github.com/linera-io/linera-protocol/tree/main/examples/llm)
- [生成式 NFTs](https://github.com/linera-io/linera-protocol/tree/main/examples/gen-nft)
- [Candle Wasm 示例](https://github.com/huggingface/candle/tree/main/candle-wasm-examples)

## 限制

### 硬件加速

虽然服务运行时支持 SIMD 指令，但通用 GPU 硬件加速目前不受支持。因此，对于较大模型，本地模型推理的性能会有所降低。

### 在链模型

由于区块大小的限制，模型需要存储在链下，直到引入 [Blob API](https://github.com/linera-io/linera-protocol/issues/1981)。Blob API 将允许将大型二进制数据块存储在链上，并由验证者保证其正确性和可用性。

### 最大模型大小

目前，可以加载到应用程序服务中的模型的最大大小受以下限制：

1. 服务的 Wasm 运行时的可寻址内存为 4 GiB。
2. 无法直接加载模型到 GPU 中。

建议在当前开发阶段使用较小的模型（50 Mb - 100 Mb）。