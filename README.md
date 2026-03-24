# water-level-est-v1
A vision system that can accurately estimate water level data from camera images of rivers, streams, and lakes.

## Setup

Install the local `pynims` package in editable mode so it is importable from notebooks and scripts:

```sh
python -m venv .venv
source .venv/bin/activate  # on Windows: .venv\Scripts\activate
pip install -e packages/pynims
```

Editable mode means changes to the source under `packages/pynims/` are reflected immediately without reinstalling.
