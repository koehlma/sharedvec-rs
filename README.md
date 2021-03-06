# SharedVec    [![Build Status]][actions] [![Latest Version]][crates.io]

[Build Status]: https://img.shields.io/github/workflow/status/koehlma/sharedvec-rs/Pipeline/main?label=tests
[actions]: https://github.com/koehlma/sharedvec-rs/actions
[Latest Version]: https://img.shields.io/crates/v/sharedvec.svg
[crates.io]: https://crates.io/crates/sharedvec


`SharedVec` is a **fast but limited ordered collection** for storing values of a single
type.


## What is a `SharedVec`?

`SharedVec` is a fast and ordered collection, similar to `Vec`, onto which values
can be pushed. In contrast to a `Vec`, a `SharedVec` allows pushing values through
a shared reference. Pushing values is an *O(1)* operation and will never relocate
previously pushed values, i.e., previous values remain at a stable address in memory.
This enables safe pushing through a shared reference.

When pushing a value, a `SharedVec` returns a shared reference to the value in
addition to a *key*. This key does *not* borrow from the `SharedVec` and can be
used to retrieve the value in *O(1)*. In addition, given an exclusive reference to
the `SharedVec`, the key can be used to obtain an exclusive reference to the value
in *O(1)*. Every key corresponds to an *index* indicating the position of the value
in the `SharedVec`. Values can also be accessed by their index in *O(log n)*.
Iterating over a `SharedVec` or converting it to a `Vec` will also preserve the
order in which values have been pushed onto the `SharedVec`.

Here is a list of similar data structures and their differences:

- A [`TypedArena`](https://docs.rs/typed-arena/) does not provide a key and
  returns an exclusive reference to a value inserted through a shared reference. A
  key is useful because it exists independently of the `SharedVec` (it does not
  borrow). It can thus be passed around more freely than a reference and
  can also be meaningfully serialized (for details see below).
- A [`Slab`](https://docs.rs/slab) and a [`SlotMap`](https://docs.rs/slotmap) cannot
  be mutated trough a shared reference. If mutation through a shared reference is
  not required, you may want to consider those as they are generally much more
  flexible.


## Serialization

Using the `serde` feature flag, a `SharedVec` and its keys can be serialized with
[Serde](https://docs.rs/serde).

A `SharedVec` storing values of type `T` is serialized as a sequence of type `T`,
just as a `Vec` is, and keys are serialized as an index into this sequence. This
enables external tools to simply treat keys
as indices into the serialized sequence. Using a previously serialized and then
deserialized key for accessing a value without also serializing and then deserializing
the corresponding `SharedVec` is an *O(log n)* operation (just as accessing by index).

This exact serialization behavior is considered part of the stability guarantees.


## Example

```rust
let vegetables = SharedVec::<&'static str>::new();

let (cucumber_key, cucumber) = vegetables.push("Cucumber");
let (paprika_key, paprika) = vegetables.push("Paprika");

assert_eq!(vegetables[cucumber_key], "Cucumber");

assert_eq!(Vec::from(vegetables), vec!["Cucumber", "Paprika"]);
```