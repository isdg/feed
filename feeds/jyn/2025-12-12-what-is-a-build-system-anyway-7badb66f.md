---
title: what is a build system, anyway?
url: https://jyn.dev/what-is-a-build-system-anyway/
published: "2025-12-12T00:00:00Z"
feed: jyn
guid: https://jyn.dev/what-is-a-build-system-anyway/
---

# what is a build system, anyway?

Andrew Nesbitt recently wrote a post titled [What is a Package Manager](https://nesbitt.io/2025/12/02/what-is-a-package-manager.html)? This post attempts to do the same for build systems.

## big picture [Anchor link for: big-picture](https://jyn.dev/what-is-a-build-system-anyway/\#big-picture)

At a high level, **build systems** are tools or libraries that provide a way to **define** and **execute** a series of transformations from **input** data to **output** data that are **memoized** by **caching** them in an **object store**.

Transformations are called **steps** or **rules** [1](https://jyn.dev/what-is-a-build-system-anyway/#fn-14) and define how to execute a **task** that generates zero or more outputs from zero or more inputs. A rule is usually the **unit of caching**; i.e. the **cache points** are the outputs of a rule, and **cache invalidations** must happen on the inputs of a rule. Rules can have **dependencies** on previous outputs, forming a directed graph called a **dependency graph**. Dependencies that form a cyclic graph are called **circular dependencies** and are usually banned.[2](https://jyn.dev/what-is-a-build-system-anyway/#fn-5)

Outputs that are only used by other rules, but not “interesting” to the end-user, are called **intermediate outputs**.

A output is **outdated**, **dirty**, or **stale** if one of its dependencies is modified, or, **transitively**, if one of its dependencies is outdated. Stale outputs invalidate the cache and require the outputs to be **rebuilt**. An output that is cached and not dirty is **up-to-date**. Rules are outdated if any of their outputs are outdated. If a rule has no outputs, it is always outdated.

Each invocation of the build tool is called a **build**. A **full build** or **clean build** occurs when the cache is empty and all transformations are executed as a **batch job**. A cache is **full** if all its rules are up-to-date. An **incremental build** occurs when the cache is partially full but some outputs are outdated and need to be rebuilt. Deleting the cache is called **cleaning**.

A build is **correct** or **sound** if all possible incremental builds have the same result as a full build.[3](https://jyn.dev/what-is-a-build-system-anyway/#fn-1) A build is **minimal** (occasionally **optimal**) if rules are rerun at most once per build, and only run if necessary for soundness ( [Build Systems à la Carte](https://www.microsoft.com/en-us/research/wp-content/uploads/2018/03/build-systems.pdf), [Pluto](https://www.mathematik.uni-marburg.de/~seba/publications/pluto-incremental-build.pdf)).

In order for a build to be sound, all possible cache invalidations must be **tracked** as dependencies.

A build system without caching is called a **task runner** or **batch compiler**. Note that task runners still often support dependencies even if they don't support caching. Build systems with caching can emulate a task runner by only defining tasks with zero outputs, but they are usually not designed for this use case. [4](https://jyn.dev/what-is-a-build-system-anyway/#fn-7)

Some examples of build systems: `make`, `docker build`, rustc. Some examples of task runners: [`just`](https://just.systems/man/en/), shell scripts, [gcc](https://fabiensanglard.net/dc/driver.php).

## specifying dependencies [Anchor link for: specifying-dependencies](https://jyn.dev/what-is-a-build-system-anyway/\#specifying-dependencies)

A build can be either **inter-process**, in which case the task is usually a single **process execution** and its input and output files, or **intra-process**, in which case a task is usually a single function call and its arguments and return values.

In order to track dependencies, either all inputs and outputs must be **declared** in source code ahead of time, or it must be possible to **infer** them from the execution of a task.

Build systems that track changes to a rule definition are called **self-tracking**. Past versions of the rule are called its **history** ( [Build Systems à la Carte](https://www.microsoft.com/en-us/research/wp-content/uploads/2018/03/build-systems.pdf)).

The act of inferring dependencies from runtime behavior is called **tracing**. If a traced rule depends on a dependency that hasn’t been built yet, the build system may either error, **suspend** the task and **resume** it later once the dependency is built, or **abort** the task and **restart** it later once the dependency is built ( [Build Systems à la Carte](https://www.microsoft.com/en-us/research/wp-content/uploads/2018/03/build-systems.pdf)).

Inter-process builds often declare their inputs and outputs, and intra-process builds often infer them, but this is not inherent to the definition. [5](https://jyn.dev/what-is-a-build-system-anyway/#fn-8)

Some example of intra-process builds include spreadsheets, the [wild](https://davidlattimore.github.io/posts/2024/11/19/designing-wilds-incremental-linking.html) linker, and memoization libraries such as python’s [`functools.cache`](https://docs.python.org/3/library/functools.html#functools.cache).

### applicative and monadic structure [Anchor link for: applicative-and-monadic-structure](https://jyn.dev/what-is-a-build-system-anyway/\#applicative-and-monadic-structure)

A build graph is **applicative** if all inputs, outputs, and rules are declared ahead of time. We say in this case the graph is **statically known**. Very few build systems are purely applicative, almost all have an escape hatch.

The graph is **monadic** if not all outputs are known ahead of time, or if rules can generate other rules dynamically at runtime. Inputs that aren’t known ahead of time are called **dynamic dependencies**. Dynamic dependencies are weaker than a fully monadic build system, in the sense that they can [express](https://buttondown.com/hillelwayne/archive/the-capability-tractability-tradeoff/) fewer build graphs. [6](https://jyn.dev/what-is-a-build-system-anyway/#fn-15)

Build systems that do not require declaring build rules are always monadic.

Some examples of monadic build systems include [Shake](https://shakebuild.com/), ninja [`dyndeps`](https://ninja-build.org/manual.html#ref_dyndep), and Cargo build scripts.

Some examples of applicative build systems include `make` (with [recursive make](https://accu.org/journals/overload/14/71/miller_2004/) and [self-rebuilding Makefiles](https://www.gnu.org/software/make/manual/html_node/Remaking-Makefiles.html) disallowed), Bazel (excluding native rules), and map/reduce libraries with memoization, such as [this unison program](https://www.unison-lang.org/articles/distributed-datasets/incremental-evaluation/).

### early cutoff [Anchor link for: early-cutoff](https://jyn.dev/what-is-a-build-system-anyway/\#early-cutoff)

If a dirty rule R has an outdated output, reruns, and creates a new output that matches the old one, the build system has an opportunity to avoid running later rules that depend on R. Taking advantage of that opportunity is called **early cutoff**.

See [the rustc-dev-guide](https://rustc-dev-guide.rust-lang.org/queries/incremental-compilation-in-detail.html) for much more information about early cutoff. [7](https://jyn.dev/what-is-a-build-system-anyway/#fn-9)

### rebuild detection [Anchor link for: rebuild-detection](https://jyn.dev/what-is-a-build-system-anyway/\#rebuild-detection)

In unsound build systems, it’s possible that the build system does not accurately **detect** that it needs to rebuild. Such systems sometimes offer a way to **force-rerun** a target: keeping the existing cache, but rerunning a single rule. For inter-process build systems, this often involves `touch` ing a file to set its modification date to the current time.

## the executor [Anchor link for: the-executor](https://jyn.dev/what-is-a-build-system-anyway/\#the-executor)

A **build executor** runs tasks and is responsible for **scheduling** tasks in an order that respects all dependencies, often using heuristics such as dependency depth or the time taken by the task on the last run. They also detect whether rule inputs have been modified, making the rule outdated; this is called **rebuild detection**. The build executor is responsible for restarting or suspending tasks in build systems that support it.

Executors usually schedule many tasks in parallel, but this is not inherent to the definition.

Executors often provide **progress reporting**, and sometimes allow **querying** the dependency graph. Occasionally they trace the inputs used by the task to enforce they match the declared dependencies, or to automatically add them to an internal dependency graph.

## inter-process builds [Anchor link for: inter-process-builds](https://jyn.dev/what-is-a-build-system-anyway/\#inter-process-builds)

In the context of inter-process builds, an **artifact** is an output file generated by a rule.[8](https://jyn.dev/what-is-a-build-system-anyway/#fn-6) A **source file** is an input file that is specific to the current **project** [9](https://jyn.dev/what-is-a-build-system-anyway/#fn-11) (sometimes **repository** or **workspace**) as opposed to a **system dependency** that is reused across multiple projects. A project is loosely defined but generally refers to the set of all input and output files that the build system knows about, usually contained in a single directory. Source files can be **generated**, which means they are an output of a previous rule.

**Build files** contain rule definitions, including (but not limited to) task definitions, input and output declarations, and metadata such as a human-readable description of the rule. Inputs are usually split into **explicit inputs** passed to the spawned process, **implicit inputs** that are tracked by the build system but not used in the task definition, and **order-only inputs** that must exist before the rule can execute, but do not invalidate the cache when modified.

Process executions have more inputs than just files, such as the rule itself, environment variables, the current time, the current working directory, and occasionally network services or local daemons [10](https://jyn.dev/what-is-a-build-system-anyway/#fn-2).

The set of all inputs that are not source files or command line arguments is called the **environment**. Processes can be **sandboxed** to prevent them from depending on the network, a daemon, or occasionally system dependencies; this is sometimes called a **sandboxed environment** or **isolated environment**.

System dependencies are more expansive than I think they are often understood to be. They include compilers, linkers, programming language libraries [11](https://jyn.dev/what-is-a-build-system-anyway/#fn-3), and [static and dynamically linked object files](https://jyn.dev/build-system-tradeoffs/#dynamic-linking-and-platform-maintainers), but also the dynamic loader, language runtime, and various system configuration files. The subset of these dependencies needed for building a minimal program in a given language, along with various tools for inspecting and modifying the outputs at runtime, are called a **toolchain**. Toolchains are inherently specific to a given language, but sometimes (e.g. in GCC) a single compiler will support multiple languages as inputs.

A build is **hermetic** (rarely, **self-contained** or **isolated** [12](https://jyn.dev/what-is-a-build-system-anyway/#fn-16)) if it uses no system dependencies and instead defines all its dependencies in the project ( [Bazel](https://bazel.build/basics/hermeticity)). Sandboxing and hermeticity are orthogonal axes; neither one implies the other. For example, docker builds are sandboxed but not hermetic, and nix shells are hermetic but not sandboxed.

Compiler or linkers sometimes have their own **incremental caches**. Reusing the cache requires you to **trust** the compiler to be sound when incrementally rebuilding. This is usually implicit, but hermetic or sandboxed builds require an opt-in to reuse the cache. Bazel calls this kind of reuse a [**persistent worker**](https://bazel.build/versions/7.0.0/remote/persistent).

### determinism [Anchor link for: determinism](https://jyn.dev/what-is-a-build-system-anyway/\#determinism)

A build is **deterministic** if it creates the same output every time in some specific environment. A build is **reproducible** if it is deterministic and also has the same output in [any environment](https://reproducible-builds.org/docs/commandments/), as long as the system dependencies remain the same.

### remote caching [Anchor link for: remote-caching](https://jyn.dev/what-is-a-build-system-anyway/\#remote-caching)

Caching can be remote or local. **Remote caching** is almost always unsound unless the build is both hermetic and reproducible (i.e. its only environment dependencies are controlled by the build system).

Downloading files from the remote cache is called **materializing** them. Most build systems with remote caching **defer materialization** as long as possible, since in large build graphs the cache is often too large to fit on disk. Builds where the cache is never fully materialized are called **shallow builds** ( [Build Systems à la Carte](https://www.microsoft.com/en-us/research/wp-content/uploads/2018/03/build-systems.pdf)).

Remote caching usually, but not necessarily, uses **content addressed hashing** in a **key-value store** to identify which artifact to download.

Some example build systems that use remote caching: Bazel, Buck2, nix, `docker build`.

### interface [Anchor link for: interface](https://jyn.dev/what-is-a-build-system-anyway/\#interface)

Build systems usually have a way to run a subset of the build. The identifier used to specify which part of the build you want to run is called a **target**.[13](https://jyn.dev/what-is-a-build-system-anyway/#fn-4) Targets are usually the filenames of an artifact, but can also be abstract names of one or more rules. Bazel-descended build systems call these names [**labels**](https://bazel.build/concepts/labels). Make-descended build systems call these [**phony targets**](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html). Some build systems, such as cargo, do not use target identifiers but instead only have subcommands with arguments; the combination of arguments together specifies a set of targets.

Some example targets:

- `make all`
- `cargo build --test http_integration`
- `buck2 build :main`

### meta-build systems [Anchor link for: meta-build-systems](https://jyn.dev/what-is-a-build-system-anyway/\#meta-build-systems)

Inter-process build systems are often divided into a **configuration** step and a **build** step. A build system that only runs the configuration step, and requires another tool for the build step, is called a **meta-build system**.

Usually this meta-build system **discovers** the rules that need to be executed (often through file globbing or some other programmatic way to describe dependencies), then **serializes** these rules into an [**action graph**](https://jyn.dev/i-want-a-better-action-graph-serialization/), which can be stored either in-memory or on-disk. On-disk serialized action graphs are usually themselves build files, in the sense that you can write them by hand but you wouldn't want to.

Configuration steps usually allow the developer to choose a set of **configuration flags** (occasionally, **build flags**) that affect the generated rules.

Some build systems also integrate directly with the [**package manager**](https://nesbitt.io/2025/12/02/what-is-a-package-manager.html), but this is uncommon, and usually the build system expects all packages to be pre-downloaded into a known location.

Some examples of meta-build systems are CMake, meson, and autotools.

### VFS [Anchor link for: vfs](https://jyn.dev/what-is-a-build-system-anyway/\#vfs)

Advanced build systems can integrate with a **virtual file system** (VFS) to check-out source control files on-demand, rather than eagerly ( [EdenFS](https://github.com/facebook/sapling?tab=readme-ov-file#ejdenfs)). A VFS can also persistently store **content hashes** and provide efficient **change detection** for files, avoiding the need for file watching or constant re-hashing.

## intra-process builds [Anchor link for: intra-process-builds](https://jyn.dev/what-is-a-build-system-anyway/\#intra-process-builds)

The equivalent of system dependencies within a process is **non-local state**, including environment variables, globals, thread-locals, and class member fields (for languages where `this` is passed implicitly). Especially tricky are function calls that do **inter-process communication** (IPC), which are basically never sound to cache. Tracing intra-process builds is very very hard since it’s easy to call a function that depends on global state without you knowing. [14](https://jyn.dev/what-is-a-build-system-anyway/#fn-19)

In this intra-process context, most object stores are **in-memory caches**. A build system that supports saving ( **persisting**) the cache to disk is said to have [**persistence**](https://jyn.dev/complected-and-orthogonal-persistence/). The system for persisting the cache is sometimes called a **database**, even if it is not a general-purpose database in the sense the term is normally used ( [Salsa](https://salsa-rs.netlify.app/tutorial/db)).

### tracing [Anchor link for: tracing](https://jyn.dev/what-is-a-build-system-anyway/\#tracing)

Tracing intra-process build systems are sometimes called a **query system**. [15](https://jyn.dev/what-is-a-build-system-anyway/#fn-18) They work similarly to their inter-process equivalents: the interface looks like normal function calls, and the build system tracks which functions call which other functions, so it knows which to rerun later.

Some examples of tools with tracing intra-process build systems: [salsa](https://docs.rs/salsa/latest/salsa/), the [rustc query system](https://rustc-dev-guide.rust-lang.org/query.html).

### FRP [Anchor link for: frp](https://jyn.dev/what-is-a-build-system-anyway/\#frp)

Intra-process build systems that allow you to explicitly declare dependencies usually come from the background of [**functional reactive programming**](https://blog.janestreet.com/breaking-down-frp/) (FRP). FRP is most often used in UI and frontend design, but many of the ideas are the same as the build systems used for compiling programs.

Unlike any of the build systems we've talked about so far, FRP libraries let you look at *past versions* of your outputs, which is sometimes called **remembering** state ( [React](https://react.dev/learn/state-a-components-memory)). To make this easier to reason about, rules can be written as **event handlers**.

Some examples of libraries with dependency declarations: [React](https://react.dev/learn/adding-interactivity).

## so, what counts as a build system? [Anchor link for: so-what-counts-as-a-build-system](https://jyn.dev/what-is-a-build-system-anyway/\#so-what-counts-as-a-build-system)

A build system is pretty much anything that lets you specify dependencies on a previous artifact 😄 Some more weird examples of build systems:

- Github Actions (jobs and workflows)
- Static site generators
- Docker-compose files
- Systemd unit files
- Excel

Hopefully this post has given you both a vocabulary to talk about build systems and a context to compare them!

## bibliography [Anchor link for: bibliography](https://jyn.dev/what-is-a-build-system-anyway/\#bibliography)

- [Andrew Nesbitt, “What is a Package Manager?”](https://nesbitt.io/2025/12/02/what-is-a-package-manager.html)
- [jyn, “build system tradeoffs”](https://jyn.dev/build-system-tradeoffs/)
- [Jade Lovelace, “The postmodern build system”](https://jade.fyi/blog/the-postmodern-build-system/)
- [Casey Rodarmor, “Just Programmer's Manual”](https://just.systems/man/en/)
- [Fabien Sanglard, “Driving Compilers”](https://fabiensanglard.net/dc/driver.php)
- [The Rust Project Contributors, “Incremental compilation in detail”](https://rustc-dev-guide.rust-lang.org/queries/incremental-compilation-in-detail.html)
- [The Rust Project Contributors, “Queries: demand-driven compilation”](https://rustc-dev-guide.rust-lang.org/query.html)
- [“functools — Higher-order functions and operations on callable objects — Python 3.14.2 documentation”](https://docs.python.org/3/library/functools.html)
- [Hillel Wayne, “The Capability-Tractability Tradeoff”](https://buttondown.com/hillelwayne/archive/the-capability-tractability-tradeoff/)
- [Neil Mitchell, “Shake Build System”](https://shakebuild.com/)
- [“The Ninja build system”](https://ninja-build.org/manual.html)
- [Peter Miller, “Recursive Make Considered Harmful”](https://accu.org/journals/overload/14/71/miller_2004/)
- [Rebecca Mark and Paul Chiusano, “Incremental evaluation via memoization · Unison programming language”](https://www.unison-lang.org/articles/distributed-datasets/incremental-evaluation/)
- [“Hermeticity  \|  Bazel”](https://bazel.build/basics/hermeticity)
- [“Persistent Workers  \|  Bazel”](https://bazel.build/versions/7.0.0/remote/persistent)
- [“Labels  \|  Bazel”](https://bazel.build/concepts/labels)
- [“Commandments of reproducible builds”](https://reproducible-builds.org/docs/commandments/)
- [Mokhov et. al., Build Systems à la Carte](https://www.microsoft.com/en-us/research/wp-content/uploads/2018/03/build-systems.pdf)
- [“Phony Targets (GNU make)”](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html)
- [Facebook, “Sapling: A Scalable, User-Friendly Source Control System.”](https://github.com/facebook/sapling?tab=readme-ov-file)
- [Erdweg et. al., "A Sound and Optimal Incremental Build System with Dynamic Dependencies"](https://www.mathematik.uni-marburg.de/~seba/publications/pluto-incremental-build.pdf)
- [David Lattimore, “Designing Wild's incremental linking”](https://davidlattimore.github.io/posts/2024/11/19/designing-wilds-incremental-linking.html)
- [Bo Lord, “How to Recalculate a Spreadsheet”](https://lord.io/spreadsheets/)
- [“ninja— `depfile_parser`”](https://github.com/ninja-build/ninja/blob/4b72b15aac766afe6a9a2c0dad535b0c2035a550/src/depfile_parser.in.cc)
- [“salsa - A generic framework for on-demand, incrementalized computation”](https://salsa-rs.netlify.app/)
- [“Defining the database struct - Salsa”](https://salsa-rs.netlify.app/tutorial/db)
- [Felix Klock and Mark Rousskov on behalf of the Rust compiler team, “Announcing Rust 1.52.1”](https://blog.rust-lang.org/2021/05/10/Rust-1.52.1/)
- [Yaron Minsky, “Jane Street Blog - Breaking down FRP ”](https://blog.janestreet.com/breaking-down-frp/)
- [“Adding Interactivity – React”](https://react.dev/learn/adding-interactivity)
- [“State: A Component's Memory – React”](https://react.dev/learn/state-a-components-memory)

1. Nearly all build systems are inconsistent about whether a rule refers to an *abstract description* of how to build an output (i.e., can be reused for multiple sets of inputs and outputs), or a *concrete instantiation* of that description for a specific set of inputs and outputs. We have to live with the ambiguity, unfortunately. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-14-1)

2. Weird things can happen here though; for example early cutoff can allow circular dependencies. This sometimes comes up for generated build.ninja files. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-5-1)

3. The [pluto paper](https://www.mathematik.uni-marburg.de/~seba/publications/pluto-incremental-build.pdf) defines this as “after a build, generated files consistently reflect the latest source files”. Neither my definition nor pluto's definition are particularly well-defined if the build is non-deterministic. Defining this formally would probably require constructing an isomorphism between all programs with the same runtime behavior; but “runtime behavior” is not well-defined for a general-purpose build system that can output artifacts that are not programs. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-1-1)

4. As we'll see later, the reverse is also true: a common design for build systems is to automatically inject cache points into an existing task runner, or to design the rule file to look as similar to a shell script or function call as possible. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-7-1)

5. In particular, nearly all modern inter-process build systems have a limited form of tracing where they ask the compiler to generate "dep-info" files [16](https://jyn.dev/what-is-a-build-system-anyway/#fn-13) that show which files were used (usually through imports) by a given source file. Note that this dep-info is not available until after the first time a build has run, and that this only works if the compiler supports it. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-8-1)

6. For more information about the spectrum of designs between applicative and monadic, see [the post-modern build system](https://jade.fyi/blog/the-postmodern-build-system/). [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-15-1)

7. Note that the dev-guide assumes that tasks are expensive relative to the cost of constructing the graph. This is true in the context of rustc, where LLVM codegen [17](https://jyn.dev/what-is-a-build-system-anyway/#fn-10) normally dominates compilation time, but it isn't true for e.g. [spreadsheets](https://lord.io/spreadsheets/). [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-9-1)

8. It's possible for tasks to create files that aren't tracked by the build system, but these aren't called artifacts. I don't know a good word for these; "byproducts" is the closest but some build systems use that to mean *any* intermediate artifacts. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-6-1)

9. I'm not super happy with this definition because it conflicts with how compilers use the term, but I do think it describes how most build systems think about files. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-11-1)

10. Poorly written rules can also depend on which other rules are executing at the same time, which is called a **race condition**. Note this does not require the rule to be unsound, only for it to use intermediate files the build system doesn’t know about. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-2-1)

11. for C, header files; for other languages, usually source files or intermediate representations. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-3-1)

12. Yes, this overlaps with the term for sandboxing. Try to avoid the word "isolated" if possible. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-16-1)

13. This has no relation to a **target platfom**, which is related to cross-compiling. I wish we had better names for these things. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-4-1)

14. I would actually describe this as much harder than tracing an inter-process build system, since [there aren't very good systems for tracking memory access](https://jyn.dev/i-want-a-better-build-executor/#environment-variables). See [this post about unstable fingerprints](https://blog.rust-lang.org/2021/05/10/Rust-1.52.1/) for an idea of what bugs this causes in practice. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-19-1)

15. This actually has very strong analogies to the way "query" is used in a database context: just like a tracing query system, a database has to be able to restart a query's transaction if the data it's trying to access has been modified. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-18-1)

16. What is a dep-info file? Good question! It's a makefile. It's literally a makefile. Don't you just love [proving backslashes by induction](https://github.com/ninja-build/ninja/blob/4b72b15aac766afe6a9a2c0dad535b0c2035a550/src/depfile_parser.in.cc#L27-L47)? [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-13-1)

17. Or, more rarely, type-checking, borrow-checking, or coherence checking. [↩](https://jyn.dev/what-is-a-build-system-anyway/#fr-10-1)
