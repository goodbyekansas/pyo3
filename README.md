# PyO3

[![Build Status](https://travis-ci.org/PyO3/pyo3.svg?branch=master)](https://travis-ci.org/PyO3/pyo3)
[![Build Status](https://ci.appveyor.com/api/projects/status/github/PyO3/pyo3?branch=master&svg=true)](https://ci.appveyor.com/project/fafhrd91/pyo3)
[![codecov](https://codecov.io/gh/PyO3/pyo3/branch/master/graph/badge.svg)](https://codecov.io/gh/PyO3/pyo3)
[![crates.io](http://meritbadge.herokuapp.com/pyo3)](https://crates.io/crates/pyo3)
[![Join the dev chat](https://img.shields.io/gitter/room/nwjs/nw.js.svg)](https://gitter.im/PyO3/Lobby)

[Rust](http://www.rust-lang.org/) bindings for [Python](https://www.python.org/). This includes running and interacting with python code from a rust binaries as well as writing native python modules.

* User Guide: [stable](https://pyo3.rs) | [master](https://pyo3.rs/master)
* [API Documentation](https://docs.rs/crate/pyo3/)

A comparison with rust-cpython can be found [in the guide](https://pyo3.rs/master/rust-cpython.html).

## Usage

Pyo3 supports python 2.7 as well as python 3.5 and up. The minimum required rust version is 1.29.0-nightly 2018-07-16.

You can either write a native python module in rust or use python from a rust binary.

### Using rust from python

Pyo3 can be used to generate a native python module.

**`Cargo.toml`:**

```toml
[package]
name = "rust-py"
version = "0.1.0"

[lib]
name = "rust_py"
crate-type = ["cdylib"]

[dependencies.pyo3]
version = "0.3"
features = ["extension-module"]
```

**`src/lib.rs`**

```rust
#![feature(use_extern_macros, specialization)]

#[macro_use]
extern crate pyo3;

use pyo3::prelude::*;

#[pyfunction]
/// Formats the sum of two numbers as string
fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
    Ok((a + b).to_string())
}

/// This module is a python moudle implemented in Rust.
#[pymodinit]
fn rust_py(py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_function!(sum_as_string))?;

    Ok(())
}
```

On windows and linux, you can build normally with `cargo build --release`. On Mac Os, you need to set additional linker arguments. One option is to compile with `cargo rustc --release -- -C link-arg=-undefined -C link-arg=dynamic_lookup`, the other is to create a `.cargo/config` with the following content: 

```toml
[target.x86_64-apple-darwin]
rustflags = [
  "-C", "link-arg=-undefined",
  "-C", "link-arg=dynamic_lookup",
]
```

Also on macOS, you will need to rename the output from \*.dylib to \*.so. On Windows, you will need to rename the output from \*.dll to \*.pyd.

[`setuptools-rust`](https://github.com/PyO3/setuptools-rust) can be used to generate a python package and includes the commands above by default. See [examples/word-count](examples/word-count) and the associated setup.py.

### Using python from rust

Add `pyo3` this to your `Cargo.toml`:

```toml
[dependencies]
pyo3 = "0.3"
```

Example program displaying the value of `sys.version`:

```rust
#![feature(use_extern_macros, specialization)]

extern crate pyo3;

use pyo3::prelude::*;

fn main() -> PyResult<()> {
    let gil = Python::acquire_gil();
    let py = gil.python();
    let sys = py.import("sys")?;
    let version: String = sys.get("version")?.extract()?;

    let locals = PyDict::new(py);
    locals.set_item("os", py.import("os")?)?;
    let user: String = py.eval("os.getenv('USER') or os.getenv('USERNAME')", None, Some(&locals))?.extract()?;

    println!("Hello {}, I'm Python {}", user, version);
    Ok(())
}
```

## Examples and tooling

 * [examples/word-count](examples/word-count) _Counting the occurences of a word in a text file_
 * [hyperjson](https://github.com/mre/hyperjson) _A hyper-fast Python module for reading/writing JSON data using Rust's serde-json_
 * [rust-numpy](https://github.com/rust-numpy/rust-numpy) _Rust binding of NumPy C-API_
 * [pyo3-built](https://github.com/PyO3/pyo3-built) _Simple macro to expose metadata obtained with the [`built`](https://crates.io/crates/built) crate as a [`PyDict`](https://pyo3.github.io/pyo3/pyo3/struct.PyDict.html)_

## License

PyO3 is licensed under the [Apache-2.0 license](http://opensource.org/licenses/APACHE-2.0).
Python is licensed under the [Python License](https://docs.python.org/2/license.html).
