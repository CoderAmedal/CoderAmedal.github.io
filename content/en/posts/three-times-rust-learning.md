+++
date = '2026-04-14'
title = 'Rust Unlimited Intro - Three Times Failed'
translationKey = 'three-times-rust-learning'
categories = ['Notes']
tags = ['Rust', 'Programming']
+++

# Rust Unlimited Intro - Three Times Failed

## Introduction

Rust has topped Stack Overflow's "most loved language" survey for years, but its learning curve is equally famous. I started and quit learning Rust three times since 2023 before finally getting past the initial barrier. This is the story of that journey.

## Why Learn Rust?

- **Performance & Safety**: Memory safety without garbage collection — no more GC pause nightmares or segfaults
- **Modern Toolchain**: Cargo, rustfmt, and Clippy create a seamless build/test/lint experience that makes C++ tooling feel archaic
- **Type System That Catches Bugs Early**: The compiler catches an incredible amount of errors before runtime — fewer 3 AM on-call wake-ups
- **Growing Industry Adoption**: From Linux kernel modules to cloud infrastructure (AWS, Cloudflare), Rust is becoming the standard for systems-level work

## Three Attempts, Three Failures

### Attempt 1: Ownership sent me running

The first three chapters of The Rust Book felt comfortable — modern syntax, immutability by default, elegant pattern matching. Then Chapter 4 on Ownership hit:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1; // s1 is *moved* here
    // println!("{}", s1); // Compile error — s1 is no longer valid!
}
```

I genuinely couldn't understand why a simple assignment would invalidate the original variable. I didn't yet grasp that Rust was enforcing single-ownership at compile time to eliminate use-after-free. I closed the book.

### Attempt 2: Lifetimes broke me

A few months later I tried again, grinding through ownership. Then lifetimes appeared:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

The `'a` syntax felt like fighting the compiler rather than writing code. In hindsight, the borrow checker simply needs explicit annotations when it can't infer which input the return value ties to — but at the time, it just felt hostile.

### Attempt 3: What finally worked

On the third attempt, I changed strategy: **stop trying to understand everything upfront. Just write code that works.**

- Use `.clone()` liberally to get past ownership errors — ignore the "you're wasting memory" guilt
- Start with small CLI tools, not performance-critical libraries
- Use owned types (`String` instead of `&str`) in structs to avoid lifetime headaches initially

After writing a few hundred lines of working Rust, I went back and re-read the ownership and borrowing chapters. Suddenly they made sense: **the compiler isn't punishing you — it's managing memory on your behalf at compile time.**

## The Three Core Rules

Once it clicked, Rust's constraints distill down to three simple rules:

1. **Each value has exactly one owner at a time**
2. **At any given moment, you can have either one mutable reference OR any number of immutable references — never both**
3. **References must always be valid (enforced by lifetimes)**

Master these three and most compiler errors become self-explanatory.

## Recommended Learning Path

If you're starting out:

1. **[The Rust Book](https://doc.rust-lang.org/book/)** — Chapters 1-6 are essential; the interactive code snippets in the browser are excellent
2. **[Rustlings](https://github.com/rust-lang/rustlings)** — Make tests pass by fixing code; the most hands-on way to learn
3. **[Rust By Example](https://doc.rust-lang.org/stable/rust-by-example/)** — Learn syntax and standard library through annotated examples
4. **[Exercism Rust Track](https://exercism.org/tracks/rust)** — Community-reviewed exercises with mentor feedback

## Will This Time Stick?

I'm setting a concrete goal: finish the entire Rust Book and build at least two small tools I'd actually use. The learning curve is real, but crossing it gives you something rare — genuine confidence that your code won't blow up in production due to memory bugs.

---

*Learning in progress — more notes to come...*
