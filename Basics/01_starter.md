# Starter

## Installation

We’ll download Rust through `rustup`, a command line tool for managing Rust versions and associated tools.

```shell
rustup update

rustup self uninstall

rustup --version
```

## New Project

`Cargo` is Rust’s build system and package manager. 

```shell
cargo --version

cargo new hello_cargo
cd hello_cargo

cargo build     # built a project

cargo run       # compile the code and then run the resulting executable all in one command

cargo check     # quickly checks your code to make sure it compiles but doesn’t produce an executable
```

`Cargo.toml` is in the TOML (Tom’s Obvious, Minimal Language) format, which is Cargo’s configuration format.

## Summary

- Install the latest stable version of Rust using rustup
- Update to a newer Rust version
- Open locally installed documentation
- Write and run a “Hello, world!” program using rustc directly
- Create and run a new project using the conventions of Cargo