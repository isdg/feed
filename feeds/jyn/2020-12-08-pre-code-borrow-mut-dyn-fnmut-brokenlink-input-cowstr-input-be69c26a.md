---
title: <pre><code>&'borrow mut dyn FnMut(BrokenLink<'input>) -> CowStr<'input></code></pre> and other valid rust programs
url: https://jyn.dev/borrow-mut-dyn-fnmut-brokenlink-cowstring-and-other-valid-rust-programs/
published: "2020-12-08T00:00:00Z"
feed: jyn
guid: https://jyn.dev/borrow-mut-dyn-fnmut-brokenlink-cowstring-and-other-valid-rust-programs/
---

# <pre><code>&'borrow mut dyn FnMut(BrokenLink<'input>) -> CowStr<'input></code></pre> and other valid rust programs

This is a story about type signatures, Higher Ranked Trait Bounds ( [HRTB](https://doc.rust-lang.org/nomicon/hrtb.html)), and the most confusing diagnostics bug I've seen in the Rust compiler. Along the way we'll learn how [`pulldown-cmark`](https://github.com/raphlinus/pulldown-cmark) has been trying to fix the same API for 3 different releases, and discover that some bugs only appear at the compile time of downstream crates.

If you don't know what some of the words mean, there is [an appendix](https://jyn.dev/borrow-mut-dyn-fnmut-brokenlink-cowstring-and-other-valid-rust-programs/#appendix). Unfortunately, this article needs too much background knowledge to fit all of it in a blog post.

## Background [Anchor link for: background](https://jyn.dev/borrow-mut-dyn-fnmut-brokenlink-cowstring-and-other-valid-rust-programs/\#background)

Our story starts with a simple change to rustdoc: It wants to [hide the `[]`](https://github.com/rust-lang/rust/pull/79781) around [intra-doc links](https://doc.rust-lang.org/rustdoc/linking-to-items-by-name.html) in search results. (For context, search results have links stripped since otherwise the relative links would be broken, which means rustdoc needs to do some preprocessing of the markdown.) That would turn screenshots like this:

![Before](https://user-images.githubusercontent.com/37223377/101309556-92908680-3801-11eb-8420-0609e7af4e92.png)

into this:

![After](https://user-images.githubusercontent.com/37223377/101309591-a0dea280-3801-11eb-85e1-620549f64bf6.png)

Rustdoc does this in several [different](https://github.com/rust-lang/rust/blob/d32c80467db39672fa612e1519564ad5fd473e91/src/librustdoc/html/markdown.rs#L1066) [places](https://github.com/rust-lang/rust/blob/d32c80467db39672fa612e1519564ad5fd473e91/src/librustdoc/html/markdown.rs#L1151), so naturally this should be pulled out into a single function that can be reused. This is a callback to pulldown's markdown [`Parser`](https://docs.rs/pulldown-cmark/0.8.0/pulldown_cmark/struct.Parser.html).

```rust
// Omitted: in real life, this would only mark intra-doc links as valid, not all broken links.
fn summary_broken_link_callback<'a>(link: BrokenLink<'a>) -> Option<(CowStr<'a>, CowStr<'a>)> {
    Some(("#".into(), link.reference.to_owned().into()))
}

```

Except that doesn't work:

```
error[E0621]: explicit lifetime required in the type of `md`
    --> src/librustdoc/html/markdown.rs:1065:9
     |
1050 | fn markdown_summary_with_limit(md: &str, length_limit: usize) -> (String, bool) {
     |                                    ---- help: add explicit lifetime `'static` to the type of `md`: `&'static str`
...
1065 |         Parser::new_with_broken_link_callback(md, summary_opts(), Some(&mut summary_broken_link_callback))
     |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ lifetime `'static` required

error: higher-ranked subtype error
    --> src/librustdoc/html/markdown.rs:1065:72
     |
1065 |         Parser::new_with_broken_link_callback(md, summary_opts(), Some(&mut summary_broken_link_callback))
     |                                                                        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

error[E0716]: temporary value dropped while borrowed
    --> src/librustdoc/html/markdown.rs:1065:77
     |
1065 |         Parser::new_with_broken_link_callback(md, summary_opts(), Some(&mut summary_broken_link_callback))
     |         --------------------------------------------------------------------^^^^^^^^^^^^^^^^^^^^^^^^^^^^--
     |         |                                                                   |                            |
     |         |                                                                   |                            temporary value is freed at the end of this statement
     |         |                                                                   creates a temporary which is freed while still in use
     |         argument requires that borrow lasts for `'static`

```

## Lifetime annotations: action at a distance [Anchor link for: lifetime-annotations-action-at-a-distance](https://jyn.dev/borrow-mut-dyn-fnmut-brokenlink-cowstring-and-other-valid-rust-programs/\#lifetime-annotations-action-at-a-distance)

What on earth is going on here? We have a simple function that takes a link and returns it, right? What does that have to do with "higher-ranked subtypes" and `'static`? To find out, we have to look at the definition of [`new_with_broken_link_callback()`](https://docs.rs/pulldown-cmark/0.8.0/pulldown_cmark/struct.Parser.html#method.new_with_broken_link_callback).

```rust
pub fn new_with_broken_link_callback(
    text: &'a str,
    options: Options,
    broken_link_callback: Option<&'a mut dyn FnMut(BrokenLink<'_>) -> Option<(CowStr<'a>, CowStr<'a>)>>
) -> Parser<'a> { /* ... */ }

```

Wow! There's a lot going on here, especially in `broken_link_callback`. Let's take that type apart a little:

- `Option` means the callback is optional; if you pass in `None` it will just do nothing.
- `&'a mut` is a mutable reference.
- `FnMut(BrokenLink<'_>) -> Option<(CowStr<'a>, CowStr<'a>)>` is a trait for 'functions taking `BrokenLink` and returning a tuple of `CowStr`'.
- Because `FnMut` is a *trait* and not a type, `dyn` turns it into a type (i.e. this is [runtime polymorphism](https://stackoverflow.com/questions/28961957/example-of-runtime-polymorphism-in-java)).

But there's something funny here - the *same* lifetime `'a` is used for both the reference to the function, and the types it outputs. That's what made our function break: since `summary_broken_link_callback` is a function, any reference to it has a `'static` lifetime. Fortunately, Rust will implicitly [reborrow](https://doc.rust-lang.org/nomicon/lifetimes.html#the-area-covered-by-a-lifetime) the reference if it lives too long. Unfortunately, we don't know how long it *should* live. That's why the error is mentioning `'static`; since it can't figure out the right lifetime, it falls back to the longest possible. By looking at the definition of [`Parser`](https://docs.rs/pulldown-cmark/0.8.0/src/pulldown_cmark/parse.rs.html#2051-2062), we can see why that gives an error about `md`:

```rust
pub struct Parser<'a> {
    md: &'a str,
    broken_link_callback: BrokenLinkCallback<'a>,
    // ... some fields omitted
}

pub type BrokenLinkCallback<'a> =
    Option<&'a mut dyn FnMut(BrokenLink) -> Option<(CowStr<'a>, CowStr<'a>)>>;

```

So there's *many* lifetimes that have been tied together here:

1. The lifetime of the input, `md`
2. The lifetime of the borrow, `&mut broken_link_callback`
3. The lifetime of the function *outputs*, which are unnamed here (pulldown [calls them](https://docs.rs/pulldown-cmark/0.8.0/src/pulldown_cmark/parse.rs.html#2380) `url` and `title`).

And that explains all the errors:

1. `md` is a temporary, but `&mut broken_link_callback` is `'static`
2. `&mut broken_link_callback` can't find an appropriate lifetime for the borrow, so it gives up altogether with "higher-ranked subtype error"
3. The arguments are temporaries, so the outputs are temporaries too (because we declared `broken_link_callback` as `fn summary_broken_link_callback<'a>(link: BrokenLink<'a>) -> Option<(CowStr<'a>, CowStr<'a>)>`), but `&mut broken_link_callback` is `'static`

## [How do you solve a problem like a lifetime?](https://www.youtube.com/watch?v=s-VRyQprlu8) [Anchor link for: how-do-you-solve-a-problem-like-a-lifetime](https://jyn.dev/borrow-mut-dyn-fnmut-brokenlink-cowstring-and-other-valid-rust-programs/\#how-do-you-solve-a-problem-like-a-lifetime)

Notice that the bug here is *not* in `broken_link_callback` \- it's in the API itself, which is tying the lifetimes together unnecessarily. If you write `broken_link_callback` on its own, it will compile and run just fine, it's only the way that it interacts with `Parser` that breaks things. The fix is not to tie the lifetimes together:

```rust
pub struct Parser<'input, 'callback> {
    text: &'input str,
    broken_link_callback: BrokenLinkCallback<'input, 'callback>,
}

pub type BrokenLinkCallback<'input, 'borrow> =
    Option<&'borrow mut dyn FnMut(BrokenLink<'input>) -> Option<(CowStr<'input>, CowStr<'input>)>>;

```

Notice that 1. and 3. are still tied together; this is because pulldown has an API that yields [events with the lifetime of the input](https://docs.rs/pulldown-cmark/0.8.0/pulldown_cmark/struct.Parser.html#associatedtype.Item). To use the output of the callback in the `Iterator` implementation, the outputs have to live as long as the full markdown input, not just the current link.

An interesting note is that `broken_link_callback` had a similar issue over overspecifying lifetimes: if you change the type signature to

```rust
fn summary_broken_link_callback<'a>(
	link: BrokenLink<'_>
) -> Option<(CowStr<'a>, CowStr<'a>)> {
  Some(("#".into(), link.reference.to_owned().into()))
}

```

it will suddenly compile. In particular, the lifetime of the input is no longer tied to the lifetime of the output, which works because `link.reference` is copied, not borrowed.

For more information about this problem, including strange and disconcerting errors that show up if you give lifetimes the wrong annotations, see [the issue](https://github.com/raphlinus/pulldown-cmark/issues/509) against pulldown, as well as [the
PR](https://github.com/raphlinus/pulldown-cmark/pull/510) I made fixing it.

## Bonus: Wait, where's my diagnostics bug? I was promised a diagnostics bug! [Anchor link for: bonus-wait-where-s-my-diagnostics-bug-i-was-promised-a-diagnostics-bug](https://jyn.dev/borrow-mut-dyn-fnmut-brokenlink-cowstring-and-other-valid-rust-programs/\#bonus-wait-where-s-my-diagnostics-bug-i-was-promised-a-diagnostics-bug)

For some reason, the error in a stand-alone program is different from the one when you compile `broken_link_callback` as part of rustdoc:

```
error: implementation of `FnOnce` is not general enough
   --> src/lib.rs:8:80
    |
8   |       for _ in Parser::new_with_broken_link_callback(txt, Options::empty(), Some(&mut callback)) {
    |                                                                                  ^^^^^^^^^^^^^ implementation of `FnOnce` is not general enough
    |
   ::: /home/jyn/.local/lib/rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ops/function.rs:219:1
    |
219 | / pub trait FnOnce<Args> {
220 | |     /// The returned type after the call operator is used.
221 | |     #[lang = "fn_once_output"]
222 | |     #[stable(feature = "fn_once_output", since = "1.12.0")]
...   |
227 | |     extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
228 | | }
    | |_- trait `FnOnce` defined here
    |
    = note: `for<'a> fn(pulldown_cmark::BrokenLink<'a>) -> Option<(pulldown_cmark::CowStr<'a>, pulldown_cmark::CowStr<'a>)> {callback}` must implement `FnOnce<(pulldown_cmark::BrokenLink<'_>,)>`
    = note: ...but `FnOnce<(pulldown_cmark::BrokenLink<'_>,)>` is actually implemented for the type `for<'a> fn(pulldown_cmark::BrokenLink<'a>) -> Option<(pulldown_cmark::CowStr<'a>, pulldown_cmark::CowStr<'a>)> {callback}`

```

In particular, the `note:` makes no sense: it shows the same types above and below! This is a long-lived [diagnostics issue](https://github.com/rust-lang/rust/issues/79643) with the compiler itself, going back [at least until 2017](https://github.com/rust-lang/rust/issues/41078).

## Appendix [Anchor link for: appendix](https://jyn.dev/borrow-mut-dyn-fnmut-brokenlink-cowstring-and-other-valid-rust-programs/\#appendix)

- [HRTB](https://doc.rust-lang.org/nomicon/hrtb.html)
- [Rustdoc](https://doc.rust-lang.org/rustdoc/)
- [intra-doc links](https://doc.rust-lang.org/rustdoc/linking-to-items-by-name.html)
- [reborrowing](https://doc.rust-lang.org/nomicon/lifetimes.html#the-area-covered-by-a-lifetime)
- I wrote more about [runtime polymorphism](https://stackoverflow.com/questions/28961957/example-of-runtime-polymorphism-in-java) (which is like `virtual` in C++ or any method call in Python) in an [ACM talk](https://acm.cse.sc.edu/assets/2020-09-09).
