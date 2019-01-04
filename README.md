# Datatest: data-driven tests in Rust

Crate for supporting data-driven tests.

Data-driven tests are tests where individual cases are defined via data rather than in code.
This crate implements a custom test runner that adds support for additional test types.

## Files-driven test

First type of data-driven tests are "file-driven" tests. These tests define a directory to
scan for test data, a pattern (a regular expression) to match and, optionally, a set of
templates to derive other file paths based on the matched file name. For each matched file,
a new test instance is created, with test function arguments derived based on the specified
mappings.

Each argument of the test function must be mapped either to the pattern or to the template.
See the example below for the syntax.

The following argument types are supported:
* `&str`, `String`: capture file contents as string and pass it to the test function
* `&[u8]`, `Vec<u8>`: capture file contents and pass it to the test function
* `&Path`: pass file path as-is

#### Note

Each test could also be marked with `#[test]` attribute, to allow running test from IDEs which
have built-in support for `#[test]` tests. However, if such attribute is used, it should go
after `#[datatest::files]` attribute, so `datatest` attribute is handled earlier and `#[test]`
attribute is removed.

### Example

```rust
#![feature(custom_test_frameworks)]
#![test_runner(datatest::runner)]

#[datatest::files("tests/test-cases", {
  input in r"^(.*).input\.txt",
  output = r"${1}.output.txt",
})]
fn sample_test(input: &str, output: &str) {
  assert_eq!(format!("Hello, {}!", input), output);
}
```

#### Ignoring individual tests

Individual tests could be ignored by specifying a function of signature
`fn(&std::path::Path) -> bool` using the following syntax on the pattern (`if !<func_name>`):

```rust
#![feature(custom_test_frameworks)]
#![test_runner(datatest::runner)]

fn is_ignore(path: &std::path::Path) -> bool {
  true // some condition
}

#[datatest::files("tests/test-cases", {
  input in r"^(.*).input\.txt" if !is_ignore,
  output = r"${1}.output.txt",
})]
fn sample_test(input: &str, output: &str) {
  assert_eq!(format!("Hello, {}!", input), output);
}
```

## Data-driven tests

Second type of tests supported by this crate are "data-driven" tests. These tests define a
YAML file with a list of test cases (via `#[datatest::data(..)]` attribute, see example below).
Each test case in this file (the file contents must be an array) is deserialized into the
argument type of the test function and a separate test instance is created for it.

Test function must take exactly one argument and the typoe of this argument must implement
[`serde::Deserialize`]. Optionally, if this implements [`ToString`] (or [`std::fmt::Display`]),
it's [`ToString::to_string`] result is used to generate test name.

#### `#[test]` attribute

Each test could also be marked with `#[test]` attribute, to allow running test from IDEs which
have built-in support for `#[test]` tests. However, if such attribute is used, it should go
after `#[datatest::files]` attribute, so `datatest` attribute is handled earlier and `#[test]`
attribute is removed.

### Example

```rust
#![feature(custom_test_frameworks)]
#![test_runner(datatest::runner)]

use serde::Deserialize;

#[derive(Deserialize)]
struct TestCase {
  name: String,
  expected: String,
}

#[datatest::data("tests/tests.yaml")]
fn sample_test(case: TestCase) {
  assert_eq!(case.expected, format!("Hi, {}!", case.name));
}

```

### More examples

For more examples, check the [tests](https://github.com/commure/datatest/blob/master/tests/datatest.rs).

## License

Licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any
additional terms or conditions.
