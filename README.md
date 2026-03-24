# water-level-est-v1
A vision system that can accurately estimate water level data from camera images of rivers, streams, and lakes.

## Setup

Create a virtual environment, then install the notebook dependencies and the local `pynims` package:

```sh
python -m venv .venv
source .venv/bin/activate  # on Windows: .venv\Scripts\activate
pip install -r requirements.txt
pip install -e packages/pynims
```

If you use `uv`:

```sh
uv venv
uv pip install -r requirements.txt
uv pip install -e packages/pynims
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
