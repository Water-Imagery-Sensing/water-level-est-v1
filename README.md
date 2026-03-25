# water-level-est-v1

A vision system that can accurately estimate water level data from camera images of rivers, streams, and lakes.

## Setup

### 1. Create a virtual environment

```sh
python -m venv .venv
source .venv/bin/activate  # on Windows: .venv\Scripts\activate
```

### 2. Install PyTorch

PyTorch must be installed before other dependencies. Choose the command that matches your hardware — see https://pytorch.org/get-started/locally/ for the latest options.

**NVIDIA GPU (CUDA 12.8):**
```sh
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu128
```

**CPU only (Linux / Windows):**
```sh
pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu
```

**macOS (Apple Silicon / MPS):**
```sh
pip install torch torchvision
```
PyTorch wheels on PyPI include MPS support for macOS natively — no special index URL needed.

### 3. Install SAM 2

```sh
cd packages
git clone https://github.com/facebookresearch/sam2.git
pip install -e sam2
```

### 4. Install project dependencies

```sh
pip install -r requirements.txt
pip install -e packages/pynims
```

`pynims` is installed in editable mode so changes to its source under `packages/pynims/` are reflected immediately without reinstalling.

## SAM 2 Model Checkpoint

Notebook `02_create-masks.ipynb` requires a SAM 2 model checkpoint. Checkpoints are not included in this repository due to file size — download them from the [SAM 2 GitHub repository](https://github.com/facebookresearch/sam2) and place them in the `checkpoints/` directory.

The notebook is configured to use `sam2.1_hiera_tiny.pt` by default:

```
checkpoints/
└── sam2.1_hiera_tiny.pt
```

If you switch to a different model variant, update the `sam2_checkpoint` and `model_cfg` variables in the notebook's configuration cell accordingly. Config files are resolved from the installed `sam2` package and don't need to be downloaded separately.
