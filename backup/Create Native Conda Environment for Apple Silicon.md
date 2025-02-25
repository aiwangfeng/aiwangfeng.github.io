### Command

```
CONDA_SUBDIR=osx-arm64 conda create -n native -c conda-forge python=3.11
```

### Make it permanent

```
conda activate native
conda config --env --set subdir osx-arm64
```

### Install some packeges

```
conda install -c apple tensorflow-deps
pip install tensorflow-macos tensorflow-metal
```