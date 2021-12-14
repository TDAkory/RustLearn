# Packages, Crates, Modules

- Packages: A Cargo feature that lets you build, test, and share crates
- Crates: A tree of modules that produces a library or executable
- Modules and use: Let you control the organization, scope, and privacy of paths
- Paths: A way of naming an item, such as a struct, function, or module

## Packages and Crates

A crate is a binary or library. The crate root is a source file that the Rust compiler starts from and makes up the root module of your crate.

A package is one or more crates that provide a set of functionality. A package contains a Cargo.toml file that describes how to build those crates.

A package can contain at most one library crate. It can contain as many binary crates as you’d like, but it must contain at least one crate (either library or binary).

```zsh
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml      # package with Cargo.toml
src
$ ls my-project/src
main.rs         # src/main.rs is the crate root of a binary crate with the same name as the package.
```

Likewise, Cargo knows that if the package directory contains src/lib.rs, the package contains a library crate with the same name as the package, and src/lib.rs is its crate root.

A package can have multiple binary crates by placing files in the src/bin directory: each file will be a separate binary crate.

## Modules [[Better Understand]](https://tonydeng.github.io/2019/10/28/rust-mod/)

Modules let us organize code within a crate into groups for readability and easy reuse. Modules also control the privacy of items.

Create a new library named restaurant by running `cargo new --lib restaurant`; then put the code below into `src/lib.rs` to define some modules and function signatures.

```rust
// Filename: src/lib.rs
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```


## Better Understanding

- [项目管理](https://wiki.jikexueyuan.com/project/rust-primer/cargo-projects-manager/cargo-projects-manager.html)
- [包和模块](https://wiki.jikexueyuan.com/project/rust-primer/module/module.html)

## Summary

Rust lets you split a package into multiple crates and a crate into modules so you can refer to items defined in one module from another module. You can do this by specifying absolute or relative paths. These paths can be brought into scope with a use statement so you can use a shorter path for multiple uses of the item in that scope. Module code is private by default, but you can make definitions public by adding the pub keyword.
