# Linera 上的机器学习

Linera应用的合约/服务分离设计允许在边缘节点安全高效地运行机器学习模型。

链上共识算法确保应用程序合约检索到正确的模型，客户端应用在无计量的链下服务执行推理。由于服务时运行在用户所有的硬件上，因此是可信任的。

## 指南

实例应用使用 [Hugging Face](https://huggingface.co/) 提供的 [`candle`](https://github.com/huggingface/candle) 框架作为底层机器学习框架。

`candle` 是专注于性能和可用性的最小化机器学习框架，使用Rust实现，可以编译为Wasm，在浏览器内外都有很好的Wasm支持。可以从candle的[示例](https://github.com/huggingface/candle/tree/main/candle-wasm-examples)了解该框架支持哪些模型。

### 开始

为了支持机器学习能力，我们需要将下列依赖项添加到Linera应用的`Cargo.toml`文件中：

```terminal
candle-core = "0.4.1"
getrandom = { version = "0.2.12", default-features = false, features = ["custom"] }
rand = "0.8.5"
```

如果需要运行大型语言模型，还需要添加下列crate：

```terminal
candle-transformers = "0.4.1"
tokenizers = { git = "https://github.com/christos-h/tokenizers", default-features = false, features = ["unstable_wasm"] }
```

### 随机源

机器学习框架使用随机数进行推理。Linera服务运行在Wasm虚拟机中，无法访问操作系统的随机数生成器。因此，我们需要为`candle`使用的随机数生成器设置种子。

创建一个文件 `src/random.rs` 并添加以下内容：

```terminal
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

这样，`candle`和其他依赖于`getrandom`的crate就能够访问该随机数生成器了。如果不需要确定性随机数，也可以使用系统时间戳作为随机数生成器的种子。

### 在服务中加载模型

目前无法将模型存储在链上，请参阅下面的`局限性`章节获得更多细节。

执行推理前，需要通过`fetch_url`加载模型：

```terminal
impl Service for MyService {
    async fn handle_query(&self, request: Request) -> Response {
        // do some stuff here
        let raw_weights = self.runtime.fetch_url("https://my-model-provider.com/model.bin");
        // do more stuff here
    }
}
```

模型可以从本地web服务器加载或直接从模型提供者(如Hugging Face)拉取。

至此，我们在服务内存储了模型和标注。`candle`支持多种量化和非量化的模型权重存储格式(`gguf`, `ggml`, `safetensors`等)。

我们可以通过`candle`提供的函数方便地将字节转换为类型化的`struct`，用于执行推理。以下是非量化 Llama 2 模型的示例：

```terminal
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

使用`candle`执行推理并不是一种“一刀切”的过程，不同的模型需要不同的逻辑来执行推理，如何执行推理的具体细节不在本文档讨论范围。

幸运的是，下列示例可以作为使用Wasm执行推理的指南：

- [Llm Stories](https://github.com/linera-io/linera-protocol/tree/main/examples/llm)
- [生成式 NFTs](https://github.com/linera-io/linera-protocol/tree/main/examples/gen-nft)
- [Candle Wasm 示例](https://github.com/huggingface/candle/tree/main/candle-wasm-examples)

## 限制

### 硬件加速

虽然服务支持SIMD指令，但当前并不支持通用GPU硬件加速。因此，较大的模型执行本地推理的性能会有所降低。

### 链上模型

由于区块大小的限制，模型需要存储在链下。计划引入的[Blob API](https://github.com/linera-io/linera-protocol/issues/1981)允许将大型二进制数据块存储在链上，并由验证者保证其正确性和可用性。

### 最大模型大小

目前，可以加载到应用程序服务中的模型的最大大小受以下限制：

1. 服务的 Wasm 运行时的可寻址内存为 4 GiB。
2. 无法直接加载模型到 GPU 中。

建议在当前开发阶段使用较小的模型（50 Mb - 100 Mb）。
