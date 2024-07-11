# More about Cargo and Crates.io

## Customizing Builds with Release Profiles

Cargo has two main profiles: the `dev` profile Cargo uses when you run `cargo build` and the `release` profile Cargo uses when you run `cargo build --release`. The `dev` profile is defined with good defaults for development, and the `release` profile has good defaults for release builds.

Cargo has default settings for each of the profiles that apply when you haven't explicitly added any [profile.*] sections in the project’s Cargo.toml file. By adding [profile.*] sections for any profile you want to customize, you override any subset of the default settings. For example, here are the default values for the opt-level setting for the dev and release profiles:

```rust
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

## Publishing a Crate to Crates.io

### Document with comments

### Publishing to Crates.io

1. Setup a Crates.io account

Before you can publish any crates, you need to create an account on crates.io and get an API token. To do so, visit the home page at crates.io and log in via a GitHub account. (The GitHub account is currently a requirement, but the site might support other ways of creating an account in the future.) Once you’re logged in, visit your account settings at https://crates.io/me/ and retrieve your API key. Then run the cargo login command with your API key, like this:

```shell
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

2. Add Metadata to a New Crate

Your crate will need a unique name. While you’re working on a crate locally, you can name a crate whatever you’d like. However, crate names on crates.io are allocated on a first-come, first-served basis. Once a crate name is taken, no one else can publish a crate with that name. Before attempting to publish a crate, search for the name you want to use. If the name has been used, you will need to find another name and edit the name field in the Cargo.toml file under the [package] section to use the new name for publishing, like so:

```rust
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT"

[dependencies]
```

3. Publishing

```shell
$ cargo publish
```

4. New Version

change the version value specified in your Cargo.toml file and republish.

5. Deprecating Versions from Crates.io with `cargo yank`

To yank a version of a crate, in the directory of the crate that you’ve previously published, run cargo yank and specify which version you want to yank. For example, if we've published a crate named guessing_game version 1.0.1 and we want to yank it, in the project directory for guessing_game we'd run:

```shell
$ cargo yank --vers 1.0.1
```

By adding --undo to the command, you can also undo a yank and allow projects to start depending on a version again:

```shell
$ cargo yank --vers 1.0.1 --undo
```