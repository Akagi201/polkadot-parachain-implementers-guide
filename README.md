# Polkadot 平行链实现指南

波卡官方 `The Polkadot Parachain Host Implementers' Guide` 中文版

英文原版地址: [repo]([Polkadot](https://github.com/paritytech/polkadot/tree/master/roadmap/implementers-guide)) rev: 55bf03157ff14dfabd6f63c5d65aa108ce42d83e

## Build the book

```sh
brew install graphviz # for macOS
sudp pacman -S graphviz # for Arch Linux
sudo apt-get install graphviz # for Ubuntu/Debian
cargo install mdbook mdbook-linkcheck mdbook-graphviz mdbook-mermaid
mdbook serve roadmap/implementers-guide
open http://localhost:3000
```

## mdbook translate plugins

* <https://lib.rs/crates/mdbook-translation>