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
