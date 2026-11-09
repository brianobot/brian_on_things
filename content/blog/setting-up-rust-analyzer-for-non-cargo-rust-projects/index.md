+++
title = "Setting Up Rust Analyzer to work for Non-Cargo-Based Rust Projects"
date = 2025-12-16
description = "Getting autocomplete, type hints, and navigation in single-file Rust programs—without Cargo."
draft = false

[extra]
toc = true
og_image = "cover.jpg"
+++

![](cover.jpg)

## Introduction

While working on the [Advent of Code](https://adventofcode.com/) Challenge for this year, I planned to solve the challenges in Python and the Rust Programming Language, and after solving the first 2 days in Python, I decided to solve those in Rust too, it didn’t take long before I realized how difficult I found it working with Rust programs without the benefits of a Language server like Rust Analyzer.

## The Problem

Rust Analyzer is built around Cargo. When you run:

```bash
cargo new my_project
```

Cargo quietly generates the metadata Rust Analyzer relies on to understand your code: crate boundaries, dependencies, edition, and access to the standard library.

But when you’re working with:

- a single `.rs` file
- a mixed-language repository
- small, disposable programs (like Advent of Code)

You lose all of that context. Rust Analyzer has nothing to latch onto, and the editor experience degrades sharply.

Unlike Python, where tooling scales gracefully from one file to one thousand, Rust’s tooling assumes Cargo by default.

## Motivation

Conventionally, when you start a new Rust project with `cargo new <project_name>`, cargo generates some configuration about your project which your Rust analyzer picks and uses to provide useful autocompletion features, go to definition support, and inline type hinting.

But when working with simpler setups like a single Rust file, you lose out on the benefits that come with the Rust analyzer and the Cargo project configuration. Unlike Programming languages like Python, which can work straightforwardly as single files and as complex project structures. \
Of course, with the benefits of the autocompletion and the rest, you can still very well write and compile single file rust programs.

After stumbling into a bug while implementing a solution for the challenges in Advent of Code day 2 in Rust, and I realized how much harder it was to walkthrough the types in my program visually and get support for autocompletion, method names and signatures, I decided to do an internet search on how to configure Rust Analyzer to work for single file rust Project without Cargo.

While searching, I came across many such advice;

![](image-2.png)

People kept generally hinting at using cargo, but, like in my case, it was not reasonable to set up a cargo project for every single day’s challenge, and it was a multi-language environment, and not a Rust-specific project.

In my case:

- Each Advent of Code solution was a standalone program
- The repository contained multiple languages
- Creating and maintaining a Cargo project per day felt like overkill

I didn’t want a build system.\
I wanted my editor to understand Rust.

## Solution

Rust Analyzer has support for non-cargo based project, just like the use case I wished to use it for above, but to give Rust Analyzer details on your setup and directory to find standard Rust libraries, you have to provide these details in a rust-project.json file in the root directory containing the single Rust files you wish to work on without Cargo. See [details here](https://rust-analyzer.github.io/book/non_cargo_based_projects.html).

With a small amount of explicit configuration, you can give Rust Analyzer everything it needs:

- where Rust is installed
- which files are crates
- which edition to use

No Cargo required.

```json
# rust-project.json

{
  "sysroot": "/Users/Apple/.rustup/toolchains/stable-aarch64-apple-darwin",
  "crates": [
    {
      "display_name": "core",
      "root_module": "/Users/Apple/.rustup/toolchains/stable-aarch64-apple-darwin/lib/rustlib/src/rust/library/core/src/lib.rs",
      "edition": "2024",
      "deps": [],
      "cfg": [
        "target_arch=\"aarch64\"",
        "target_os=\"macos\"",
        "unix"
      ],
      "env": {},
      "is_proc_macro": false
    },
    {
      "display_name": "alloc",
      "root_module": "/Users/Apple/.rustup/toolchains/stable-aarch64-apple-darwin/lib/rustlib/src/rust/library/alloc/src/lib.rs",
      "edition": "2024",
      "deps": [
        { "crate": 0, "name": "core" }
      ],
      "cfg": [
        "target_arch=\"aarch64\"",
        "target_os=\"macos\"",
        "unix"
      ],
      "env": {},
      "is_proc_macro": false
    },
    {
      "display_name": "std",
      "root_module": "/Users/Apple/.rustup/toolchains/stable-aarch64-apple-darwin/lib/rustlib/src/rust/library/std/src/lib.rs",
      "edition": "2024",
      "deps": [
        { "crate": 0, "name": "core" },
        { "crate": 1, "name": "alloc" }
      ],
      "cfg": [
        "target_arch=\"aarch64\"",
        "target_os=\"macos\"",
        "unix"
      ],
      "env": {},
      "is_proc_macro": false
    },
    {
      "display_name": "day_4",
      "root_module": "/Users/Apple/Documents/advent-of-codes/2025/day_4.rs",
      "edition": "2024",
      "deps": [{ "crate": 2, "name": "std" }],
      "cfg": [
        "target_arch=\"aarch64\"",
        "target_os=\"macos\"",
        "unix"
      ],
      "env": {},
      "is_proc_macro": false
    },
    {
      "display_name": "day_5",
      "root_module": "/Users/Apple/Documents/advent-of-codes/2025/day_5.rs",
      "edition": "2024",
      "deps": [{ "crate": 2, "name": "std" }],
      "cfg": [
        "target_arch=\"aarch64\"",
        "target_os=\"macos\"",
        "unix"
      ],
      "env": {},
      "is_proc_macro": false
    }
  ]
}
```

## Explanation:

- **sysroot**: Tells Rust Analyzer where your Rust toolchain lives—the standard library, core, alloc, and compiler support crates.\
  You can get this value by entering the following command

```bash
rustc --print sysroot
```

- **crates**: An array describing each logical Rust crate in the workspace.\
  For simple setups, this often maps** one-to-one with a single **`.rs`** file**, but it can also represent larger module trees.\
  Take note to pass the Absolute path of the `root_module` attribute

Notice that the first 3 crates are standard crates that Rust Analyzer needs to annotate builtin traits properly, ensure that you own defined crates (single Rust files) that depend on those traits to gain the complete type annotation benefits.

After creating and saving this file to your working directory, you should reload your editor workspace to get all the great benefits of Rust Analyzer working on your single-file Rust Programs (non-Cargo projects)

## Conclusion

Rust doesn’t stop you from writing small, single-file programs, but without proper tooling, it certainly makes you *feel* the absence. Rust Analyzer, by default, expects Cargo to be the narrator of your project’s story, and when Cargo is missing, the editor is left guessing in the dark.

The good news is that this limitation is not a dead end. By explicitly describing your setup using a `rust-project.json` file, you can restore nearly all the ergonomics you’d expect from a full Cargo-based project: autocomplete, type hints, navigation, and a clearer understanding of your code’s structure.

For lightweight experiments, coding challenges, or mixed-language repositories, this approach strikes a practical balance. You keep the simplicity of single-file Rust while reclaiming the insight and confidence that come from a well-informed editor.

Sometimes, the compiler already knows enough. You just need to teach your tools to listen.
