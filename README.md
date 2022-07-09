# Polkadot 平行链实现指南

[![docs lint](https://github.com/Akagi201/polkadot-parachain-implementers-guide/actions/workflows/linter.yml/badge.svg)](https://github.com/Akagi201/polkadot-parachain-implementers-guide/actions/workflows/linter.yml)

波卡官方 `The Polkadot Parachain Host Implementers' Guide` 中文版

英文原版地址：[repo](https://github.com/paritytech/polkadot/tree/master/roadmap/implementers-guide) rev: 64f094877a96c4e3c41b861642b290bbd20cbaff

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
