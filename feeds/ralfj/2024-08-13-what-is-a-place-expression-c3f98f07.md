---
title: What is a place expression?
url: https://www.ralfj.de/blog/2024/08/14/places.html
published: "2024-08-13T22:00:00Z"
feed: ralfj
guid: https://www.ralfj.de/blog/2024/08/14/places.html
---

# What is a place expression?

One of the more subtle aspects of the Rust language is the fact that there are actually two kinds of expressions: *value expressions* and *place expressions*. Most of the time, programmers do not have to think much about that distinction, as Rust will helpfully insert automatic conversions when one kind of expression is encountered but the other was expected. However, when it comes to unsafe code, a proper understanding of this dichotomy of expressions can be required. Consider the following [example](https://play.rust-lang.org/?version=nightly&mode=debug&edition=2021&gist=9a8802d20da16d6569510124c5827794):

```
// As a "packed" struct, this type has alignment 1.
#[repr(packed)]
struct MyStruct {
  field: i32
}

let x = MyStruct { field: 42 };
let ptr = &raw const x.field;
// This line is fine.
let ptr_copy = &raw const *ptr;
// But this line has UB!
// `ptr` is a pointer to `i32` and thus requires 4-byte alignment on
// memory accesses, but `x` is just 1-aligned.
let val = *ptr;

```

Here I am using the unstable but [soon-to-be-stabilized](https://github.com/rust-lang/rust/pull/127679) “raw borrow” operator, `&raw const`. You may know it in its stable form as a macro, `ptr::addr_of!`, but the `&` syntax makes the interplay of places and values more explicit so we will use it here.

The last line has Undefined Behavior (UB) because `ptr` points to a field of a packed struct, which is not sufficiently aligned. But how can it be the case that evaluating `*ptr` is UB, but evaluating `&raw const *ptr` is fine? Evaluating an expression should proceed by first evaluating the sub-expressions and then doing something with the result. However, `*ptr` is a sub-expression of `&raw const *ptr`, and we just said that `*ptr` is UB, so shouldn’t `&raw const *ptr` also be UB? That is the topic of this post.

(You might have already encountered the distinction of place expressions and value expressions in C and C++, where they are called lvalue expressions and rvalue expressions, respectively. While the basic syntactic concept is the same as in Rust, the exact cases that are UB are different, so we will focus entirely on Rust here.)

### Making the implicit explicit

The main reason why this dichotomy of place expressions and value expressions is so elusive is that it is entirely implicit. Therefore, to understand what actually happens in code like the above, the first step is to add some new syntax that lets us make this implicit distinction explicit in the code.

Normally, we may think of (a fragment of) the grammar of Rust expressions roughly as follows:

> *Expr* ::=
>
> *Literal* \| *LocalVar* \| *Expr* `+` *Expr* \| `&` *BorMod* *Expr* \| `*` *Expr* \|
>
> *Expr* `.` *Field* \| *Expr* `=` *Expr* \| …
>
> *BorMod* ::= `​` \| `mut` \| `raw` `const` \| `raw` `mut`
>
> *Statement* ::=
>
> `let` *LocalVar* `=` *Expr* `;` \| …

This directly explains why we can write expressions like `*ptr = *other_ptr + my_var`.

However, to understand places and values, it is instructive to consider a different grammar that explicitly has two kinds of expressions. I will first give the grammar, and then explain it with some examples:

> *ValueExpr* ::=
>
> *Literal* \| *ValueExpr* `+` *ValueExpr* \| `&` *BorMod* *PlaceExpr* \|
>
> *PlaceExpr* `=` *ValueExpr* \| `load` *PlaceExpr*
>
> *PlaceExpr* ::=
>
> *LocalVar* \| `*` *ValueExpr* \| *PlaceExpr* `.` *Field*
>
> *Statement* ::=
>
> `let` *LocalVar* `=` *ValueExpr* `;` \| …

*Value expressions* are those expressions that compute a value: literals like `5`, computations like `5 + 7`, but also expressions that compute values of pointer type like `&my_var`. However, the expression `my_var` (referencing a local variable), according to this grammar, is *not* a value expression, it is a *place expression*. This is because `my_var` actually denotes a place in memory, and there’s multiple things one can do with a place: one can load the contents of the place from memory (which produces a value), one can create a pointer to the place (which also produces a value, but does not access memory at all), or one can store a value into this place (which in Rust produces the `()` value, but the side-effect of changing the contents of memory is more relevant). Besides local variables, the other main example of a place expression is the result of the `*` operator, which takes a *value* (of pointer type) and turns it into a place.[1](#fn:deref) Furthermore, given a place of struct type, we can use a field projection to obtain a place just for that field.

This may sound odd, because it means that `let new_var = my_var;` is not actually a valid statement in our grammar! To accept this code, the Rust compiler will automatically convert this statement into a form that fits the grammar by adding `load` whenever needed.[2](#fn:desugar) `load` takes a place and, as the name indicates, performs a load from memory to obtain the value currently stored in this place. The desugared form of the statement therefore is `let new_var = load my_var;`.

To consider a more complicated example, the assignment expression `*ptr = *other_ptr + my_var` mentioned above desugars to `*(load ptr) = load *(load other_ptr) + load my_var`. That’s a lot of `load` expressions! It is instructive to convince yourself that every one of them is necessary to make this term fit the grammar. In particular, `*` works on a value expression (so we need `load other_ptr` to obtain the value stored in this place) and produces a place expression (so we need to `load` again to obtain a value expression that we can use with `+`). However, the left-hand side of `=` is a place expression, so we do not `load` the result of the `*` there.

Since the `load` operator is introduced implicitly, it is sometimes referred to a “place-to-value coercion”. Understanding where place-to-value coercions or `load` expressions are introduced is the key to understanding the example at the top of this blog post. So let us write the relevant part of that example again, using our more explicit grammar:

```
let ptr = &raw const x.field;
// This line is fine.
let ptr_copy = &raw const *(load ptr);
// But this line has UB!
let val = load *(load ptr);

```

Suddenly, it makes perfect sense why the last line has UB but the previous one does not! The expression `&raw const *(load ptr)` merely computes the place `*(load ptr)` *without ever loading from it*, and then uses `&raw const` to turn that place into a value. This is worth repeating: the `*` operator, usually referred to as “dereferencing a pointer”, *does not access memory in any way*. All it does is take a value of pointer type, and convert it into a place. This is a pure operation that can never fail. In the last line, there is an extra `load` applied to the result of the `*`, and *that* is where a memory access happens—and in this case, UB occurs since the place is not sufficiently aligned.

It is completely legal to evaluate a place expression that produces an unaligned place, and it is also legal to then turn that unaligned place into a raw pointer value. Generally, in terms of UB, you should think of places as being pretty much like raw pointers: there is no requirement that they point to valid values, or even to existing memory.[3](#fn:field) However, it is *not* legal to load from (or store to) an unaligned place, which is why `load *(load ptr)` is UB.

In other words, when `*ptr` is used as a value expression (as it is in our example), then it is *not* a sub-expression of `&raw const *ptr` because the implicit place-to-value coercion adds an extra `load` around `*ptr` that is not added in `&raw const *ptr`.

### Other examples of place expression surprises

The other main example where place expressions can lead to surprising behavior is in combination with the `_` pattern. For instance:

```
let ptr = std::ptr::null::<i32>();
let _ = *ptr; // This is fine!
let _val = *ptr; // This is UB.

```

Note that the grammar above cannot represent this program: in the full grammar of Rust, the `let` syntax is something like “ `let` *Pattern* `=` *PlaceExpr* `;`”, and then pattern desugaring decides what to do with that place expression. If the pattern is a binder (the common case), a `load` gets inserted to compute the initial value for the local variable that this binder refers to. However, if the pattern is `_`, then the place expression still gets evaluated—but the result of that evaluation is simply discarded. MIR uses a `PlaceMention` statement to indicate these semantics.

In particular, this means that the `_` pattern does *not* incur a place-to-value coercion! The desugared form of the relevant part of this code is:

```
PlaceMention(*(load ptr)); // This is fine!
let _val = load *(load ptr); // This is UB.

```

As you can see, the first line does not actually load from the pointer (the only `load` is there to load the pointer itself from the local variable that stores it). No value is ever constructed when a place expression is used with the `_` pattern. In contrast, the last line actually creates a new local variable, and therefore a place-to-value coercion is inserted to compute the initial value for that variable.

The same also happens with `match` statements:

```
let ptr = std::ptr::null::<i32>();
match *ptr { _ => "happy" } // This is fine!
match *ptr { _val => "not happy" } // This is UB.

```

The scrutinee of a `match` expression is a place expression, and if the pattern is `_` then a value is never constructed. However, when an actual binder is present, this introduces a local variable and a place-to-value coercion is inserted to compute the value that will be stored in that local variable.

**Note on `unsafe` blocks.** Note that wrapping an expression in a block forces it to be a value expression. This means that `unsafe { *ptr }` always loads from the pointer! In other words:

```
let ptr = std::ptr::null::<i32>();
let _ = *ptr; // This is fine!
let _ = unsafe { *ptr }; // This is UB.

```

The fact that braces force a value expression can occasionally be useful, but the fact that `unsafe` blocks do that is definitely quite unfortunate.

### Are there also value-to-place coercions?

So far, we have discussed what happens when a place expression is encountered in a spot where a value expression was expected. But what about the opposite case? Consider:

```
let x = &mut 15;

```

According to our grammar, `&` (in this case with the `mut` modifier) needs a place expression, but `15` is a value expression. How can the Rust compiler accept such code?

In this case, the desugaring involves introducing new “temporary” local variables:

```
let mut _tmp = 15;
let x = &mut _tmp;

```

The exact scope in which this temporary is introduced is defined by [non-trivial rules](https://github.com/rust-lang/lang-team/blob/master/design-meeting-minutes/2023-03-15-temporary-lifetimes.md) that are outside the scope of this blog post; the key point is that this transformation again makes the program valid according to the more explicit grammar.

There is one exception to this rule, which is the left-hand side of an assignment operator: if you write something like `15 = 12 + 19`, the value `15` is not turned into a temporary place, and instead the program is rejected. Introducing temporaries here is very unlikely to produce a meaningful result, so there’s no good reason to accept such code.

### Conclusion

Whenever we are using a place expression where a value is expected, or a value expression where a place is expected, the Rust compiler implicitly transforms our program into a form that matches the grammar given above. If you are only writing safe code, you can almost always entirely forget about this transformation. However, if you are writing unsafe code and want to understand why some programs have UB and others do not, it can be crucial to understand what exactly happens. If you only remember one thing from this blog post, then remember that `*` dereferences a pointer but *does not load from memory*; instead, all it does is turn the pointer into a place—it is the subsequent implicit place-to-value conversion that performs the actual load. I hope that giving a name to this implicit `load` operator can help demystify the topic of places and values. :)

1. **Update (2025-12-26):** If you check the [Rust reference](https://doc.rust-lang.org/reference/expressions.html#r-expr.place-value.place-context), you may notice that it actually says that `*` takes a *place* expression. This is a somewhat peculiar Rust design choice related to custom smart pointers implementing the `Deref` trait, and related to borrow checking. That turns out to be easier if you only dereference places. However, when it comes to the operational semantics, the overall picture is much cleaner if we say that `*` works on values. [↩](#fnref:deref)

2. The Rust compiler does not actually explicitly do such a desugaring, but this happens implicitly as part of compiling the program into MIR form. [↩](#fnref:desugar)

3. One subtlety, however, is that the *PlaceExpr* `.` *Field* expression performs *in-bounds* pointer arithmetic using the rules of the [`offset` method](https://doc.rust-lang.org/nightly/std/primitive.pointer.html#method.offset). This is the one case where a place expression does care about pointing to existing memory. This is unfortunate, but optimizations greatly benefit from this rule and since the introduction of the `offset_of!` macro, it should be extremely rare that unsafe code would want to do a field projection on a dangling pointer. [↩](#fnref:field)
