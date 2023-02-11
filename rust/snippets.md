# docs: rust/snippets
#rust #snippets 

## install rust
See instructions on https://www.rust-lang.org/tools/install

add `${HOME}/.cargo/bin`  to `${PATH}`:

## update rust

```bash
rustup update
```

## cargo
```bash
# build
cargo build

# clean
cargo build

# build only one binary (file must exist: src/bin/paths.rs)
cargo build --bin paths
```