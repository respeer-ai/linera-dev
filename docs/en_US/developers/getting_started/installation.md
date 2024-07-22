# Installation

Let's start with the installation of the Linera development tools.

## [Overview](en_US/developers/getting_started/installation.md#overview)

The Linera toolchain consist of two crates:

- `linera-sdk` is the main library to program Linera applications in Rust.

- `linera-service` defines a number of binaries, including:

  - `linera` -- the main client tool, used to operate development wallets,
  - `linera-proxy` -- the proxy service, acting as a public entrypoint for each
    validator,
  - `linera-server` -- the service run by each worker of a validator, hidden
    behind the proxy.

## [Requirements](en_US/developers/getting_started/installation.md#Requirements)

The operating systems currently supported by the Linera toolchain can be
summarized as follows:

| Linux x86 64-bit | Mac OS (M1 / M2) | Mac OS (x86) | Windows  |
| ---------------- | ---------------- | ------------ | -------- |
| ✓ Main platform  | ✓ Working        | ✓ Working    | Untested |

The main prerequisites to install the Linera toolchain are Rust, Wasm, and
Protoc. They can be installed as follows on Linux:

- Rust and Wasm

  - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
  - `rustup target add wasm32-unknown-unknown`

- Protoc

  - `curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v21.11/protoc-21.11-linux-x86_64.zip`
  - `unzip protoc-21.11-linux-x86_64.zip -d $HOME/.local`
  - If `~/.local` is not in your path, add it:
    `export PATH="$PATH:$HOME/.local/bin"`

- On certain Linux distributions, you may have to install development packages
  such as `g++`, `libclang-dev` and `libssl-dev`.

For MacOS support and for additional requirements needed to test the Linera
protocol itself, see the installation section on
[GitHub](https://github.com/linera-io/linera-protocol/blob/main/INSTALL.md).

This manual was tested with the following Rust toolchain:

```rust
[toolchain]
channel = "1.77.2"
components = [ "clippy", "rustfmt", "rust-src" ]
targets = [ "wasm32-unknown-unknown" ]
profile = "minimal"
```

## [Installing from crates.io](en_US/developers/getting_started/installation.md#Installing-from-cratesio)

You may install the Linera binaries with

```bash
cargo install --locked linera-service@0.11.3
```

and use `linera-sdk` as a library for Linera Wasm applications:

```bash
cargo add linera-sdk@0.11.3
```

The version number `0.11.3` corresponds to the
current Devnet of Linera and may change frequently.

## [Installing from GitHub](en_US/developers/getting_started/installation.md#Installing-from-GitHub)

Download the source from [GitHub](https://github.com/linera-io/linera-protocol):

```bash
git clone https://github.com/linera-io/linera-protocol.git
cd linera-protocol
git checkout -t origin/devnet_2024_05_07  # Current release branch
```

To install the Linera toolchain locally from source, you may run:

```bash
cargo install --locked --path linera-service
```

Alternatively, for developing and debugging, you may instead use the binaries
compiled in debug mode, e.g. using `export PATH="$PWD/target/debug:$PATH"`.

This manual was tested against the following commit of the
[repository](https://github.com/linera-io/linera-protocol):

```text
2ada2e77e6a2f3dfa3bd32f4dc609bdadd0fbf3a
```

## [Bash helper (optional)](en_US/developers/getting_started/installation.md#Bash-helper-optional)

Consider adding the output of `linera net helper` to your `~/.bash_profile` to
help with [automation](en_US/developers/core_concepts/wallets.md#automation-in-bash).

## [Getting help](en_US/developers/getting_started/installation.md#Getting-help)

If installation fails, reach out to the team (e.g. on
[Discord](https://discord.gg/linera)) to help troubleshoot your issue or
[create an issue](https://github.com/linera-io/linera-protocol/issues/new) on
GitHub.
