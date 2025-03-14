[![INFORMS Journal on Computing Logo](https://INFORMSJoC.github.io/logos/INFORMS_Journal_on_Computing_Header.jpg)](https://pubsonline.informs.org/journal/ijoc)

# Sempervirens: A Fast Reconstruction Algorithm for Noisy and Incomplete Binary Matrix Representations of Trees

This archive is distributed in association with the [INFORMS Journal on
Computing](https://pubsonline.informs.org/journal/ijoc) under the [MIT License](LICENSE).

The software and data in this repository are a snapshot of the software and data
that were used in the research reported on in the paper 
[Sempervirens: A Fast Reconstruction Algorithm for Noisy and Incomplete Binary Matrix Representations of Trees](https://doi.org/10.1287/ijoc.2019.0000). 
The snapshot is based on 
[this SHA](https://github.com/nevenag/sempervirens/commit/825cd71d70fcd864c4976160e4d9fc00b2e45e6b)
in the development repository. 

**Important: This code is being developed on an on-going basis at
https://github.com/nevenag/sempervirens. Please go there if you would like to
get a more recent version or would like support.

## Cite

To cite the contents of this repository, please cite both the paper and this repo, using their respective DOIs.

https://doi.org/10.1287/ijoc.2023.0373

https://doi.org/10.1287/ijoc.2023.0373.cd

Below is the BibTex for citing this snapshot of the repository.

```
@misc{Sempervirens
  author =        {N. Junnarkar, C. Kizilkale, N. Golubovic, M. Arcak, A. Buluc},
  publisher =     {INFORMS Journal on Computing},
  title =         {{Sempervirens: A Fast Reconstruction Algorithm for Noisy and Incomplete Binary Matrix Representations of Trees}},
  year =          {2023},
  doi =           {10.1287/ijoc.2023.0373.cd},
  url =           {https://github.com/INFORMSJoC/2023.0373},
  note =          {Available for download at https://github.com/INFORMSJoC/2023.0373},
}  
```

## Table of Contents

1. [Quick Start](#QS)
2. [Installation](#Installation)
3. [Commandline Usage](#CLI)
4. [Library Usage](#LIB)
5. [Input Output Format](#IOFormat)
6. [Using the Rust Implementation](#Rust)
7. [Files and Directories](#Files)
8. [Contact Information](#Contact)

## Quick Start <a name="QS"></a>

Assuming Python and dependencies are present you can run:

```bash
git clone git@github.com:nevenag/sempervirens.git
cd sempervirens
python sempervirens/reconstructor.py sample_data/noisy_data.SC 0.001 0.2 0.05 -o reconstructed_data.SC.CFMatrix
```

The file `reconstructor.py` is a standalone implementation; the file can be moved to wherever necessary.

## Installation <a name="Installation"></a>

Sempervirens requires Python and only two Python packages: NumPy and Pandas. Pandas can be omitted if only using Sempervirens as a library.

Python dependencies can be installed as follows:

```bash
brew install python@3.10
brew install pip
pip install numpy
pip install pandas
```

Follow the steps from [Quick Start](#QS) or below to continue.

All code has been tested with Python 3.10.

## Commandline Usage <a name="CLI"></a>

The following reconstructs the noisy matrix in `noisy_matrix_filename` given false positive probability `fpp`, false negative probability `fnp`, and missing entry probability `mep`. The reconstructed matrix is written to `noisy_matrix_filename.CFMatrix`.

```bash
python reconstructor.py noisy_matrix_filename fpp fnp mep
```

The output file can be specified with the optional `-o` flag:

```bash
python reconstructor.py noisy_matrix_filename fpp fnp mep -o output_filename
```

Help information can be found by running `python reconstructor.py --help`.

## Library Usage <a name="LIB"></a>

The `reconstructor.py` file can be imported and used as a library in other Python code.
Example usage is as follows.

```python
from reconstructor import reconstruct
...
reconstruction = reconstruct(noisy_mat, fpp, fnp, mep)
...
```

An example of using Pandas to read and write matrices of files can be seen at the bottom of `reconstructor.py`.

## Input/Output Format <a name="IOFormat"></a>

The input and output to the `reconstruct` function is a NumPy matrix with `n` rows and `m` columns. This describes `n` objects in terms of `m` characters (for example, cells in terms of mutations). Element `(i, j)` of the matrix can be either `0`, `1`, or `3`. If it is `0`, then object `i` does not have character `j`. If it is `1`, then object `i` has character `j`. If it is `3`, then it is unknown whether object `i` has character `j` (the output of `reconstruct` will not have any `3`s).

Below is an example file used for input. The input file must be a tab separated value (TSV) file. The first row and column of the file are used as labels of the rows and columns respectively. The rest of the TSV must represent a matrix with the format described above. The output file will be of the same format as the input file, reusing the input file's row and column labels.

```text
cellID/mutID  mut0  mut1  mut2  mut3  mut4  mut5  mut6  mut7
cell0         0     0     3     0     0     0     0     0
cell1         0     3     1     0     0     0     1     1
cell2         0     0     1     0     0     0     1     1
cell3         1     1     0     0     0     0     0     0
cell4         0     0     1     0     3     0     0     0
cell5         1     0     0     0     0     0     0     0
cell6         0     0     1     0     0     0     1     1
cell7         0     0     1     0     0     0     0     0
cell8         3     0     0     0     3     0     3     1
cell9         0     1     0     0     0     0     0     0
```

## Using the Rust Implementation <a name="Rust"></a>

A faster implementation is provided in `sempervirens-rs`. This implementation must be compiled from source.
Once compiled, it is used similarly to the commandline Python implementation. Help information can be found by running `./sempevirens-rs --help`.

### Compiling Instructions

First make sure [Rust](https://www.rust-lang.org/) is installed. Then, in the `sempervirens-rs` directory, run the following. The binary will be `target/release/sempervirens-rs` and can be moved to wherever needed.

```bash
cargo build --release
```

### Basic Linear Algebra Subprograms (BLAS)

By default, a version of OpenBLAS is compiled into `sempervirens-rs`.
For best performance, the Basic Linear Algebra Subprograms (BLAS) installation should be configured.
Configuring BLAS requires changing the following lines in `Cargo.toml`:

```toml
blas-src = { version = "0.9", default-features = false, features = ["openblas"] }
openblas-src = { version = "0.10", default-features = false, features = ["cblas", "static"] }
```

as well as the following line at the top of `sempervirens-rs/src/lib.rs`:

```rust
extern crate openblas_src;
```

Information on this can be found in [blas-src](https://docs.rs/blas-src/latest/blas_src/).
Examples for using system OpenBLAS and Intel MKL are below.

#### OpenBLAS

To use the system OpenBLAS installation (if available), change the lines in `Cargo.toml` to:

```toml
blas-src = { version = "0.9", default-features = false, features = ["openblas"] }
openblas-src = { version = "0.10", default-features = false, features = ["cblas", "system"] }
```

and the line in `lib.rs` to:

```rust
extern crate openblas_src;
```

#### Intel MKL

To use the system Intel Math Kernel Library (Intel MKL) installation (if available), change the lines in `Cargo.toml` to:

```toml
blas-src = { version = "*", default-features = false, features = ["intel-mkl"] }
intel-mkl-src = { version = "0.8.1", features =  ["ENTER_INSTALLATION_HERE"] }
```

and the line in `lib.rs` to:

```rust
extern crate intel_mkl_src;
```

The text "ENTER_INSTALLATION_HERE" should be configured according to [intel-mkl-src](https://github.com/rust-math/intel-mkl-src).

## Files and Directories <a name="Files"></a>

`sempervirens`:

* `reconstructor.py`: The reconstruction code and the commandline interface code.
* `metrics.py`: Functions for metrics; e.g. ancestor-descendant score.

`sempervirens-rs`: The Rust implementation.

* `src/lib.rs`: The reconstruction code.
* `src/main.rs`: The commandline interface code.

`tests`: Code for evaluating algorithms on datasets and comparing them.

`data`: Datasets for testing algorithms.

* Get input data from [`data.zip`](https://github.com/nevenag/sempervirens/blob/main/data.zip).
* `noisy_data.SC` sample input file.

`metrics`: Results of running algorithms on various datasets.

## Contact Information <a name="Contact"></a>

For questions, please contact [Neelay Junnarkar](mailto:neelay.junnarkar@berkeley.edu) and [Can Kızılkale](mailto:cankizilkale@berkeley.edu).
