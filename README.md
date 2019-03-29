# zkp: a toolkit for Schnorr proofs

This crate has a toolkit for Schnorr-style zero-knowledge proofs,
instantiated using the ristretto255 group.

It provides two levels of API:

* a higher-level, declarative API based around the `define_proof` macro,
  which provides an embedded DSL for specifying proof statements in
  Camenisch-Stadler-like notation:
  ```
  define_proof! {
    vrf_proof,   // Name of the module for generated implementation
    "VRF",       // Label for the proof statement
    (x),         // Secret variables
    (A, G, H),   // Public variables unique to each proof
    (B) :        // Public variables common between proofs
    A = (x * B), // Statements to prove
    G = (x * H) 
    }
  ```
  This expands into a module containing an implementation of proving,
  verification, and batch verification.  Proving uses constant-time
  implementations, and the proofs have a derived implementation of
  (memory-safe) serialization and deserialization via Serde.

* a lower-level, imperative API inspired by [Bellman][bellman], which
  provides a constraint system for Schnorr-style statements.  This
  allows programmable construction of proof statements at runtime.  The
  higher-level `define_proof` macro expands into an invocation of the
  lower-level API.
  The lower-level API is contained in the `toolbox` module.


# Use and features

To enable the `define_proof` macro, import the crate like so:
```
#[macro_use]
extern crate zkp;
```

#### Nightly features

The `nightly` feature enables nightly-specific features.  It is required
to build the documentation.

#### Backend selection

`zkp` provides the following pass-through features to select a
`curve25519-dalek` backend:

* `u32_backend`
* `u64_backend`
* `simd_backend`

#### Transcript debugging

The `debug-transcript` feature is for development and testing, and
prints a log of the data fed into the proof transcript.

#### Autogenerated benchmarks

The `define_proof` macro builds benchmarks for the generated proof
statements, but because these are generated in the client crate (where
the macro expansion happens), they need an extra step to be enabled.

**To enable generated benchmarks in your crate, do the following**:

* Add a `bench` feature to your crate's `Cargo.toml`;
* Add `#[cfg_attr(feature = "bench", feature(test))]` to your crate's
  `lib.rs` or `main.rs`, to enable Rust's nightly-only benchmark
  feature.

# Examples

Examples of how to use the API can be found in the library's `tests`
directory.

Currently, the examples include:

* Specification of an "anonymous credential presentation with 10 hidden
  attributes" proof from CMZ'13;

* A signature and VRF construction with an auto-generated
  implementation;

* An example of using the lower-level constraint system API.

# WARNING

**THIS IMPLEMENTATION IS NOT YET READY FOR PRODUCTION USE**

While I expect the 1.0 version to be largely unchanged from the current
code, for now there are no stability guarantees on the proofs, so they
should not yet be deployed.

[bellman]: https://github.com/zkcrypto/bellman
