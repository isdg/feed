---
title: Fast Tensor Canonicalization in Rust
url: https://www.stephendiehl.com/posts/tensor_canonicalization_rust/
published: "2025-06-29T01:00:00Z"
feed: diehl
guid: https://www.stephendiehl.com/posts/tensor_canonicalization_rust/
---

# Fast Tensor Canonicalization in Rust

I recently had a need to do fast tensor canonicalization, but the library ecosystem for this was lacking. In physics and differential geometry, manipulating tensors and especially bringing them into canonical form under their index symmetries is a recurring and computationally intensive task. Every undergrad who takes their first relativity class understands the tensor gymnastics required can often be daunting, and in real-life problems they get even more gnarly.

- [Github Source Code](https://github.com/sdiehl/butler-portugal)
- `git clone https://github.com/sdiehl/butler-portugal.git`

While existing systems like Mathematica offer some support, and the Xperm.c library provides a high-performance C implementation, there was no, let's just say "modern" and open source solution. So I built a new [`butler-portugal` library](https://docs.rs/butler-portugal/latest/butler_portugal/), a pure Rust implementation of the Butler-Portugal algorithm for tensor canonicalization.

The canonical algorithm for tensor canonicalization (har har) is the little known **Butler-Portugal algorithm**, which is a method for bringing tensors into a unique, minimal form under their symmetries. The algorithm approaches tensor canonicalization by recasting the problem in the language of permutation group theory, specifically as the search for a minimal representative in a double coset of the symmetric group. Given a tensor with a set of slot symmetries \\( S \\) (permutations of indices that leave the tensor invariant up to sign) and dummy symmetries \\( D \\) (arising from contracted or repeated indices), the algorithm considers the action of these groups on the set of all possible index arrangements. The set of all tensors equivalent under these symmetries forms the double coset \\( DgS \\), where \\( g \\) is the initial arrangement of indices. The canonicalization task is to find the lexicographically minimal element in this double coset, which serves as the unique canonical form. This is achieved by systematically generating the orbit of \\( g \\) under the action of \\( S \\), then, for each representative, applying the dummy symmetries \\( D \\) to further minimize the arrangement. To do this we use the **Schreier-Sims algorithm** to efficiently enumerate the relevant group elements and orbits, avoiding combinatorial explosion. By using the structure of the double coset and the efficient traversal of orbits, the Butler-Portugal algorithm ensures that all symmetry relations are respected and that the resulting canonical form is both unique and minimal, even for tensors with intricate and overlapping symmetry properties.

Let’s step back and consider what symbolic tensor manipulation actually means. Unlike in machine learning, in physics tensors aren’t just multidimensional arrays; they’re objects with named indices, each carrying a variance (covariant and contravariant, or downstairs and upstairs respectively), and they transform in specific ways under coordinate changes. Their symmetries, like being symmetric or antisymmetric under index exchange or more elaborate patterns as in the Riemann tensor, are essential algebraic properties that encode physical laws.

Take general relativity as a motivating example. The Einstein field equations,

$$

G\_{\\mu\\nu} = 8\\pi T\_{\\mu\\nu}

$$

relate the Einstein tensor \\( G\_{\\mu\\nu} \\), which encodes spacetime curvature, to the stress-energy tensor \\( T\_{\\mu\\nu} \\). Both are symmetric rank-2 tensors meaning we can permute their indices freely in our computations:

$$

G\_{\\mu\\nu} = G\_{\\nu\\mu}, \\qquad T\_{\\mu\\nu} = T\_{\\nu\\mu}

$$

When working with these objects symbolically, you want to be able to manipulate expressions, contract indices, and simplify results, all while respecting these symmetries. For instance, the Riemann curvature tensor \\( R\_{\\mu\\nu\\rho\\sigma} \\) is a rank-4 object with a rich set of symmetries:

$$

R\_{\\mu\\nu\\rho\\sigma} = -R\_{\\nu\\mu\\rho\\sigma} = -R\_{\\mu\\nu\\sigma\\rho} = R\_{\\rho\\sigma\\mu\\nu}

$$

and it satisfies the so-called [first Bianchi identity](https://en.wikipedia.org/wiki/Curvature_form#Bianchi_identities),

$$

R\_{\\mu\[\\nu\\rho\\sigma\]} = 0

$$

where the brackets denote antisymmetrization over the enclosed indices. These symmetry identities are crucial for simplifying calculations and for ensuring that derived quantities, like the Ricci tensor or scalar, have the right properties.

Contraction is another fundamental operation. It reduces the rank of a tensor by summing over a pair of indices—one upper and one lower:

$$

T^\\mu\_{\ \\mu} = \\sum\_{\\mu} T^\\mu\_{\ \\mu}

$$

This is how you build invariants, like the Ricci scalar which is a single real number assigned to each point on a Riemannian manifold that measures the degree to which the geometry near that point deviates from flatness:

$$

R = g^{\\mu\\nu} R\_{\\mu\\nu}

$$

And it’s essential that any symbolic manipulation system handles these operations correctly, respecting all symmetries and producing results in a canonical, unambiguous form. To make this concrete, here’s a minimal Rust example using `butler-portugal` to canonicalize a symmetric tensor:

```rust
use butler_portugal::{Tensor, TensorIndex, Symmetry, canonicalize};

fn main() -> butler_portugal::Result<()> {
    // Construct the metric tensor g_{μν} with indices in reverse order
    let mut g = Tensor::new(
        "g",
        vec![TensorIndex::new("nu", 0), TensorIndex::new("mu", 1)],
    );
    // Declare symmetry: g_{μν} = g_{νμ}
    g.add_symmetry(Symmetry::symmetric(vec![0, 1]));

    println!("Original tensor: {g}");

    let canonical = canonicalize(&g)?;
    println!("Canonical form: {canonical}");
    println!("Coefficient: {}", canonical.coefficient());
    Ok(())
}

```

This code constructs a symmetric metric tensor with indices in non-canonical order, declares its symmetry, and invokes the canonicalization algorithm. The output is the unique, minimal representative of the tensor under its symmetry group, along with the appropriate sign or coefficient.

At the core of my library is a representation of tensors as objects with named indices, each carrying variance and position. Symmetries are encoded as group actions, with support for symmetric, antisymmetric, cyclic, and custom symmetries. The canonicalization process proceeds by generating all valid permutations of indices under the specified symmetries to enumerate the relevant subgroup of the symmetric group. For each permutation, the algorithm computes the resulting tensor and its sign, ultimately selecting the lexicographically minimal form, and then giving that back to the user.

The symmetries of a tensor are encoded as group actions on its indices. The most common types are:

- **Symmetric**: invariant under exchange of specified indices,

  $$

  T\_{ab} = T\_{ba}

  $$
- **Antisymmetric**: changes sign under exchange,

  $$

  F\_{ab} = -F\_{ba}

  $$
- **Cyclic**: invariant under cyclic permutations,

  $$

  C\_{abc} = C\_{bca} = C\_{cab}

  $$
- **Pair symmetry**: exchange of index pairs, as in the Riemann tensor,

  $$

  R\_{abcd} = R\_{cdab}

  $$

The core challenge in the algorithm is the efficient enumeration of group elements. Naive approaches quickly become intractable as the number of indices grows, the worst-case behavior remains \\( O(n!) \\) time complexity. However we can use Schreier-Sims algorithm, a staple of computational group theory, provides a scalable solution by constructing a [**base and strong generating set**](https://en.wikipedia.org/wiki/Strong_generating_set) (or BSGS for short) for the permutation group generated by the tensor’s symmetries. This allows the algorithm to enumerate group elements without redundancy and with minimal overhead. Alternatively the library also supports projection onto irreducible representations specified by [Young tableaux](https://en.wikipedia.org/wiki/Young_tableau), enabling the decomposition of tensors into symmetry types corresponding to the representations of the symmetric group. I won't explain this here, but it's well-documented elsewhere and I put the lecture note links in the repository

My implementation draws inspiration from the [Xperm.c library](https://xact.es/), which is widely regarded as the gold standard for tensor canonicalization in symbolic computation. However, porting the algorithm to Rust was quite a challenge as the C and Mathematica and code is quite dated and not particularly well-documented. So hopefully if you're a practitioner in the field, you can look at my implementation, compare it with the reference and hopefully this makes it easier for some poor future physics grad student who doesn't need to build the same thing again to do their research.

So that's `butler-portugal`, which hopefully brings high-performance tensor canonicalization to the Rust ecosystem. Can install it with:

```bash
cargo add butler-portugal

```

The library is quite efficient, MIT-licensed and hopefully useful in any domain where the manipulation of tensors with complex symmetries is required, and hopefuly useful for at least one other person besides myself!
