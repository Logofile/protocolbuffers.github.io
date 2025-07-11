+++
title = "Building Rust Protos"
weight = 784
linkTitle = "Building Rust Protos"
description = "Describes how to build Rust protos using Cargo or Bazel."
type = "docs"
+++

## Cargo

See the
[protobuf-example](https://docs.rs/crate/protobuf-example/latest/source/) crate
for an example of how to set up your build.

## Bazel

The process of building a Rust library for a Protobuf definition is similar to
other programming languages:

1.  Use the language-agnostic `proto_library` rule:

    ```build
    proto_library(
        name = "person_proto",
        srcs = ["person.proto"],
    )
    ```

2.  Create a Rust library:

    ```build {highlight="lines:1,8-11"}
    load("//third_party/protobuf/rust:defs.bzl", "rust_proto_library")

    proto_library(
        name = "person_proto",
        srcs = ["person.proto"],
    )

    rust_proto_library(
        name = "person_rust_proto",
        deps = [":person_proto"],
    )
    ```

3.  Use the library by including it in a Rust binary:

    ```build {highlight="lines:1,14-20"}
    load("//third_party/bazel_rules/rules_rust/rust:defs.bzl", "rust_binary")
    load("//third_party/protobuf/rust:defs.bzl", "rust_proto_library")

    proto_library(
        name = "person_proto",
        srcs = ["person.proto"],
    )

    rust_proto_library(
        name = "person_rust_proto",
        deps = [":person_proto"],
    )

    rust_binary(
        name = "greet",
        srcs = ["greet.rs"],
        deps = [
            ":person_rust_proto",
        ],
    )
    ```

{{% alert title="Note" color="note" %}} Don't use
`rust_upb_proto_library` or `rust_cc_proto_library` directly.
`rust_proto_library` checks the global build flag to choose the appropriate
backend for you. {{% /alert %}}
