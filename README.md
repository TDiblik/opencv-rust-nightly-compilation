# ! DO NOT USE !

As of 04.05.2024 <sup>(dd.MM.yyyy)</sup>, the compilation errors which caused opencv to not compile on nightly, [have been resolved](https://github.com/twistedfall/opencv-rust/issues/548). If you're using this repository in your `Cargo.toml`, please migrate over to (at least) `opencv = "0.91.3"`.

# Here's the original README:

## ! Temporary repository !

At the time of writing, 12.04.2024 <sup>(dd.MM.yyyy)</sup>, you cannot compile [opencv-rust](twistedfall/opencv-rust/) (in dev mode), using the latest Rust nightly compiler (rustc 1.79.0-nightly), because of the new [unsafe_precondition enforcment instead of UB](https://github.com/rust-lang/rust/pull/120594).

Basically, if you try to compile atm using nightly, it will fail with the following error:

```
thread 'main' panicked at library\core\src\panicking.rs:155:5:
unsafe precondition(s) violated: slice::from_raw_parts requires the pointer to be aligned and non-null, and the total size of the slice not to exceed `isize::MAX`
```

After some research, I found out that, this is caused by (as mentioned before) [Rust's #120594 PR](https://github.com/rust-lang/rust/pull/120594).
However, the error is NOT produced by opencv-rust, instead it's produced by [clang-rs](https://github.com/KyleMayes/clang-rs) by the `tokenize` function which is called from [field.rs at line 153](https://github.com/twistedfall/opencv-rust/blob/985af46844b74581d38325561b5baa240292c797/binding-generator/src/field.rs#L153).
There is [a PR opened in clang-rs](https://github.com/KyleMayes/clang-rs/pull/58) addressing this issue. For the opencv-rust to once again compile with nightly: clang-rs needs to merge the PR -> clang-rs has to release a new version -> (potentionaly) opencv-rust must bump up to the new version -> (potentionaly) opencv-rust has to release new version. I don't have that much time.

## Fix 1

You can use this repository as a drop-in replacement for the `opencv = "0.89.0"` inside of your `Cargo.toml` until the clang-rs fix is not available.

```toml
[dependencies]
opencv = { git = "https://github.com/TDiblik/opencv-rust-nightly-compilation.git" }
```

Once you've done that, you'll be able to compile your Rust project the same way you could with the stable compiler version.

## Fix 2

Since the new checks are only applied in debug mode, compilation with nightly using the release mode will work. HOWEVER that results in insane compile times, especially in larger codebases.

## Fix 3

If you want to "fix" the "bug" inside `opencv-rust` directly, you could theoretically implement some checks before the code that calls the `tokenize` function (linked before), however I find that to be kinda pointless, since the fix to the `clang-rs` will get merged and shipped sooner or later.

# What next?

This repository only "merges" and "includes" the clang-rs fix. Once the clang-rs fix gets merged and shipped, and opencv-rust will be compileable with nightly once again, I will archive this fork.

If you already see this fork as archived, it means that the bug I was facing is fixed in upstream and you SHOULD NOT use this fork in your Rust project.

I chose to "archive" (instead of "delete") once it's fixed, because there could be some "bright mind" that decides to use this in production forever...
