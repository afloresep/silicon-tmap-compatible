# tmap (Python 3.9+ & Apple Silicon Compatible)

This is a fork of [tmap](https://github.com/reymond-group/tmap) that is compatible with **Python 3.9-3.12** and **Apple Silicon (arm64)**.

tmap is a very fast visualization library for large, high-dimensional data sets. The graph layouts are based on the [OGDF](https://ogdf.uos.de/) library.

## Installation

### Option 1: Install from GitHub (Recommended)

```bash
pip install git+https://github.com/afloresep/silicon-tmap-compatible.git
```

### Option 2: Install from pre-built wheels

Check the [Releases](https://github.com/afloresep/silicon-tmap-compatible/releases) page for pre-built wheels for your platform.

### Option 3: Build from source

For Apple Silicon Macs or if you need to build from source:

```bash
# Clone the repository
git clone https://github.com/afloresep/silicon-tmap-compatible.git
cd silicon-tmap-compatible

# Create conda environment with build dependencies
conda create -n tmap python=3.12 -y
conda activate tmap
conda install -c conda-forge cmake ninja llvm-openmp numpy scipy matplotlib annoy pybind11 -y

# Build OGDF
cd ogdf-conda/src
mkdir -p build installed
cd build
cmake .. \
  -DCMAKE_INSTALL_PREFIX="$PWD/../installed" \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_OSX_ARCHITECTURES=arm64  # Only for Apple Silicon
cmake --build . --parallel
cmake --install .
cd ../../..

# Install tmap
export LIBOGDF_INSTALL_PATH="$PWD/ogdf-conda/src/installed"
pip install . --no-build-isolation
```

## Visualization

We recommend using [faerun](https://github.com/reymond-group/faerun-python) to plot the data laid out by tmap:

```bash
pip install faerun
```

## Quick Example

```python
import tmap as tm
import numpy as np

# Generate random data
data = np.random.rand(1000, 100)

# Create MinHash fingerprints
mh = tm.Minhash(128)
fps = [tm.VectorFloat(row) for row in data]
lf = tm.LSHForest(128)
lf.batch_add(mh.batch_from_weight_array(fps))
lf.index()

# Layout
cfg = tm.LayoutConfiguration()
cfg.k = 20
x, y, s, t, _ = tm.layout_from_lsh_forest(lf, cfg)

# Plot with faerun or matplotlib
import matplotlib.pyplot as plt
plt.scatter(x, y, s=1)
plt.show()
```

## Tutorial and Documentation

See the original documentation at [http://tmap.gdb.tools](http://tmap.gdb.tools)

## Examples

| Name | Description |
| ---- | ----------- |
| [MNIST](http://tmap.gdb.tools/src/mnist/mnist.html) | Visualization of the MNIST data set |
| [Drugbank](http://tmap.gdb.tools/src/drugbank/drugbank.html) | All drugs registered in Drugbank |
| [Project Gutenberg](http://tmap.gdb.tools/src/gutenberg/gutenberg.html) | Linguistic relationships between books |

## Compatibility

| Python | Linux | macOS Intel | macOS Apple Silicon | Windows |
|--------|-------|-------------|---------------------|---------|
| 3.9    | ✅    | ✅          | ✅                  | ✅      |
| 3.10   | ✅    | ✅          | ✅                  | ✅      |
| 3.11   | ✅    | ✅          | ✅                  | ✅      |
| 3.12   | ✅    | ✅          | ✅                  | ✅      |

## Changes from Original

- Fixed compatibility with Python 3.9-3.12
- Fixed Apple Silicon (arm64) builds
- Updated deprecated `distutils` imports
- Added proper `pyproject.toml` build configuration

## License

BSD 3-Clause License - see [LICENSE.md](LICENSE.md)

## Credits

Original tmap by [Daniel Probst](https://github.com/daenuprobst) and the [Reymond Group](https://github.com/reymond-group).
