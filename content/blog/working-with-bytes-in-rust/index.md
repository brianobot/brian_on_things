+++
title = "Introduction to Bytes in Rust. Part 1"
date = 2026-11-02
description = "A beginner-friendly introduction to bytes in Rust: bits, bytes and hexadecimal, why computers use binary, and how Rust represents byte collections and strings."
draft = false

[extra]
toc = true
og_image = "cover.webp"
+++

## Why you have probably never had to think about bytes

As a backend developer, most of my work sits at a high level of abstraction. I write handlers that take in structs, strings, vectors and the occasional custom enum, and return types that implement a particular trait.

Here is a handler from an Axum project:

```rust
#[derive(Serialize)]
struct Message {
    message: String,
}

async fn handler() -> Json<Message> {
    Json(Message {
        message: String::from("Hello World!"),
    })
}
```

No bytes in sight. You define a struct, derive a trait, return it, and you're done. Somewhere further down, that `Json<Message>` becomes a sequence of bytes on a network socket, but someone else already did that work for you.

That is true for most backend projects: the framework has built the plumbing, and your job is to combine those building blocks to solve business problems.

Things change when you go below that line and start building the blocks yourself: parsing a binary protocol, writing a file format, computing a checksum. There, understanding bytes stops being optional. I've been learning the fundamentals of bytes in Rust, and this post is my attempt to explain what I've understood to someone else. Explaining it is also the best test of whether I understand it.

## Bits, bytes and the rest of the gang

When you send a friend an AI-generated meme, then at the lowest level that picture is nothing but a stream of **bits** travelling over a wire, a radio wave or a fibre-optic cable.

"Bit" is short for **binary digit**, and binary is a way of counting that uses only two digits: `0` and `1`. In decimal, we count `0, 1, 2, ... 9` and then add a new digit to go higher (`10`). Binary works the same way, except it runs out of digits after `1`, so it adds a new digit much sooner:

| Decimal | Binary |
| ------- | ------ |
| 0       | 0      |
| 1       | 1      |
| 2       | 10     |
| 3       | 11     |
| 4       | 100    |
| 5       | 101    |
| 6       | 110    |
| 7       | 111    |
| 8       | 1000   |

Each position in a binary number is worth double the one to its right (1, 2, 4, 8, ...), just as decimal positions are worth 1, 10, 100, and so on. So `101` is `1×4 + 0×2 + 1×1 = 5`.

### Bytes

Think about how written language works. Letters are the basic symbols, but a single letter rarely means much on its own. Arrange them into words, then sentences, then paragraphs, and meaning appears.

Bits work similarly. A single bit carries very little, so we group them. A group of **8 bits** is called a **byte**, and the byte is the basic unit that computers read, write and address in memory.

Eight bits can form 2⁸ = 256 different patterns, so a byte can represent any number from `0` to `255`. (Not every piece of information is exactly 8 bits wide: a flag might use a single bit, and some formats pack fields across byte boundaries. But the byte is the unit everything is built on.)

### Hexadecimal

Writing `11111111` every time you mean 255 is tedious and easy to get wrong, which is why we use **hexadecimal**, or "hex".

Hex is a base-16 counting system. It needs 16 digits, so after `0-9` it borrows the letters `A-F` (where `A` is 10 and `F` is 15). The useful part is that **one hex digit corresponds to exactly 4 bits**, so a byte is always exactly **two hex digits**:

| Binary      | Hex    | Decimal |
| ----------- | ------ | ------- |
| 0000 0001   | `0x01` | 1       |
| 0100 0001   | `0x41` | 65      |
| 1111 1111   | `0xFF` | 255     |

The `0x` prefix is just a marker meaning "this number is in hex". You'll see these constantly in low-level code. For example, `0x41` is 65, which happens to be the code for the letter `A`.

## Why would anyone leave decimal for binary?

It does feel like unnecessary work. We already count comfortably in base 10, so why make computers use something harder to read?

The answer is that computers aren't built to be read by humans. They are built from electronic circuits, which work with physical signals. Telling apart **two** states, a high voltage and a low voltage, is easy and reliable. Telling apart ten distinct voltage levels is hard, because electrical noise is always present, and a small disturbance could turn a 7 into an 8.

Two states leave a wide margin for error, so circuits can be simple, fast and dependable. Switch between those two states billions of times a second, and you can express any information at all, as long as you agree on what the patterns mean. (Quantum computers work differently, but that's a story for another day.)

## Bringing it down to Rust

Rust is a strongly typed language: every value has a type, and the compiler knows it at compile time. Often you write the type yourself as an annotation, but the compiler is smart enough to infer it in many cases.

The type for a single byte is **`u8`**: an **u**nsigned **8**-bit integer, holding values from 0 to 255.

```rust
let a: u8 = 65;
let b: u8 = 0x41;         // hex literal
let c: u8 = 0b0100_0001;  // binary literal (underscores are just for readability)
let d: u8 = b'A';         // byte literal: the ASCII code of 'A'

assert_eq!(a, b);
assert_eq!(b, c);
assert_eq!(c, d);
```

All four are the same value, written four different ways. A `u8` can't hold 256 or more: `let x: u8 = 256;` fails to compile, and arithmetic that overflows (like `255u8 + 1`) panics in debug builds.

### Collections of bytes

You'll mostly work with bytes in groups, and Rust has three common shapes for that:

```rust
let fixed: [u8; 4] = [0x89, 0x50, 0x4E, 0x47]; // array: size fixed at compile time
let mut growable: Vec<u8> = Vec::new();         // Vec: grows as needed
growable.extend_from_slice(&fixed);

let view: &[u8] = &growable[1..3];              // slice: a borrowed view into bytes
```

That array is the start of every PNG file, the format your meme was probably sent in. The `0x50 0x4E 0x47` part spells `PNG` in ASCII.

### Strings are bytes too

A Rust `String` or `&str` is really a sequence of bytes holding UTF-8 text, and you can look at them directly:

```rust
let s = "Hi!";
let bytes: &[u8] = s.as_bytes();
println!("{:?}", bytes);   // [72, 105, 33]

println!("{}", "é".len()); // 2
```

Note the last line: `"é"` is one character but **two** bytes, because UTF-8 uses more than one byte for characters outside basic ASCII. That's why `len()` on a string counts bytes, not characters.

To print numbers in hex or binary, use format specifiers:

```rust
println!("{:#04x} {:08b}", 65u8, 65u8); // 0x41 01000001
```

### What's next

A `u8` is easy, but most real data is bigger than one byte. A `u32` takes four bytes, and then you face a question: which byte goes first?

```rust
let n: u32 = 0x1234_5678;
println!("{:02x?}", n.to_be_bytes()); // [12, 34, 56, 78]  big-endian
println!("{:02x?}", n.to_le_bytes()); // [78, 56, 34, 12]  little-endian
```

That's called **endianness**, and it's where I'll pick up in the next post.