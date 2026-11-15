+++
title = "Rust references, aliasing and what the borrow checker buys"
date = 2026-11-12
description = ""
draft = true

[extra]
seo_title = "Rust references and aliasing and it's use"
toc = true
+++

# Keys points
- because 2 mutable references can not exist at a point, rust can reorder mutable refernces for speed and can store them across function calls which improves speed as compared to always reading their current value as is done in C and C++
- Since Rust 2018's non-lexical lifetime, a borrow ends at it last use which makes codes like this compiles
  ```rust
  let mut v = vec![1, 2, 3];
  let first = &v[0];
  v.push(4); // you would expect that this would not compile, because there is an alive shared referenced to an v, but that shared
  // reference is technically only alive until it last usage, so this works
  println!("v = {v}");
  ```
- in order to avoid compilations error around borrowing here are some tips that do not use clones
  - E0502: mutable borrow while a shared one lives 
    Fix: Move the read after the mutation or copy the small value of the container
    ```rust
      let mut v = vec![1, 2, 3];
      // let first = &v[0];
      v.push(4);
      let first = &v[0]; // this way the mutable references dies before the shared reference and they do not overlap
      ```
      or 

      ```rust
      let mut v = vec![1, 2, 3];
      let first = v[0]; // this copies the value if its a Copy type instead of taking a refernce
      v.push(4);
      ```
  - E0499: Two mutable borrows of one value
    Fix: split_at_mut, iter_mut, destructure the struct, or std::mem::swap
    ```rust
    // wrong approach  
    // let mut v = vec![1, 2, 3];
    // let a = &mut v[0];
    // let b = &mut v[1];        // error[E0499]: cannot borrow `v` as mutable more than once
    // correct approach
    let (left, right) = v.split_at_mut(1);
    let a = &mut left[0];
    let b = &mut right[0];    // fine: two slices that cannot overlap
    ```
  - E0506: assigning to a borrowed value
    Fix: Shorten the borrow - it ends at it last use
    ```rust
    let mut v = 32;
    let r = &v;
    // ensure that the reference last use is shorted that the mutation
    v = 10
    // println!("{r}") // using the reference beyond the assignment would cause this to fail
    ```
  - E0515: returning a reference to a local
    Fix: return an owned value, a &'static or write into the caller buffer
    ```rust
    fn speak(val: &str) -> &'static str {
        return "some beautiful code" // the return value is different embedded in the binary as a static reference
    }
    
    fn main() {
        let mut local;
        {   
            let msg = "hello";
            local = speak(msg);
        }
        println!("{}", local);
    }
    ```


## 
- 