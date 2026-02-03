[![PyPI - Downloads](https://static.pepy.tech/badge/tmap-silicon)](https://pepy.tech/project/tmap-silicon)
[![PyPI - Monthly](https://static.pepy.tech/badge/tmap-silicon/month)](https://pepy.tech/project/tmap-silicon)


# tmap (Python 3.9+ & Apple Silicon Compatible)

This is a fork of [tmap](https://github.com/reymond-group/tmap) that is compatible with **Python 3.9-3.12** and **Apple Silicon (arm64)**.

tmap is a very fast visualization library for large, high-dimensional data sets. The graph layouts are based on the [OGDF](https://ogdf.uos.de/) library.

## Installation

### Option 1: Install from pip

```bash
pip install tmap-silicon
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

