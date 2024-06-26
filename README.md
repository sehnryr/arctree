# arctree
![Build Status](https://github.com/sehnryr/arctree/workflows/arctree/badge.svg)
[![Crates.io](https://img.shields.io/crates/v/arctree.svg)](https://crates.io/crates/arctree)
[![Documentation](https://docs.rs/arctree/badge.svg)](https://docs.rs/arctree)
[![Rust 1.49.0](https://img.shields.io/badge/rust-1.49.0-orange.svg)](https://releases.rs/docs/1.49.0/)
![](https://img.shields.io/badge/unsafe-forbidden-brightgreen.svg)

*arctree* is a "DOM-like" tree implemented using atomic reference counting.

## Origin

This is a fork of [rctree](https://github.com/RazrFalcon/rctree).

## Details

"DOM-like" here means that data structures can be used to represent
the parsed content of an HTML or XML document,
like [*the* DOM](https://dom.spec.whatwg.org/) does,
but don't necessarily have the exact same API as the DOM.
That is:

* A tree is made up of nodes.
* Each node has zero or more *child* nodes, which are ordered.
* Each node has a no more than one *parent*, the node that it is a *child* of.
* A node without a *parent* is called a *root*.
* As a consequence, each node may also have *siblings*: its *parent*'s other *children*, if any.
* From any given node, access to its
  parent, previous sibling, next sibling, first child, and last child (if any)
  can take no more than *O(1)* time.
* Each node also has data associated to it,
  which for the purpose of this project is purely generic.
  For an HTML document, the data would be either the text of a text node,
  or the name and attributes of an element node.
* The tree is mutable:
  nodes (with their sub-trees) can be inserted or removed anywhere in the tree.

The lifetime of nodes is managed through *atomic reference counting*.
To avoid reference cycles which would cause memory leaks, the tree is *asymmetric*:
each node holds optional *strong references* to its next sibling and first child,
but only optional *weak references* to its parent, previous sibling, and last child.

Nodes are destroyed as soon as there is no strong reference left to them.
The structure is such that holding a reference to the root
is sufficient to keep the entire tree alive.
However, if for example the only reference that exists from outside the tree
is one that you use to traverse it,
you will not be able to go back "up" the tree to ancestors and previous siblings after going "down",
as those nodes will have been destroyed.

Weak references to destroyed nodes are treated as if they were not set at all.
(E.g. a node can become a root when its parent is destroyed.)

Since nodes are *aliased* (have multiple references to them),
[`RwLock`](https://docs.rs/parking_lot/0.12.1/parking_lot/rwlock/type.RwLock.html)
is used for interior mutability.

Advantages:

* A single `Node` user-visible type to manipulate the tree, with methods.
* Memory is freed as soon as it becomes unused (if parts of the tree are removed).

Disadvantages:

* Any tree manipulation, including read-only traversals,
  requires incrementing and decrementing reference counts,
  which causes run-time overhead.
* Nodes are allocated individually, which may cause memory fragmentation and hurt performance.

## Minimum Supported Rust Version (MSRV)

The current MSRV is 1.49.0 due to the `parking_lot` dependency.

## License

*arctree* is licensed under the **MIT License**.
