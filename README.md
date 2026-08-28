# Segment Geospatial (SAM 3) — TACC Jupyter Cookbook

An interactive **Jupyter Lab** cookbook for running [segment-geospatial](https://github.com/opengeos/segment-geospatial) with the **Meta SAM 3** backend on TACC Lonestar6 (`ls6`). Adapted from [`Cookbook-Conda-Template`](https://github.com/In-For-Disaster-Analytics/Cookbook-Conda-Template).

This is the deployment app registered in the Cookbook UI — it builds a GPU (or CPU) Docker image published to GitHub Container Registry (GHCR) and launches an interactive Jupyter session on Lonestar6.

## What's in this repo

| File | Purpose |
| --- | --- |
| `.binder/environment.yaml` | Conda environment `segment-geospatial` (Python 3.12 + `segment-geospatial[sam3]` + Jupyter/leafmap/localtileserver). |
| `run.sh` | TACC interactive launcher: clones this repo, installs miniconda, creates the conda env, loads CUDA, and starts Jupyter Lab with a TAP token + port forwarding. |
| `Dockerfile` / `Dockerfile.cpu` | GPU (`nvidia/cuda:12.3`) and CPU (`ubuntu:22.04`) base images. The image only provides the runtime + `run.sh`; the Python env is built at job time from `.binder/environment.yaml`. |
| `.github/workflows/.docker-gpu.yaml` / `.docker-cpu.yaml` | Build & push `ghcr.io/in-for-disaster-analytics/segment-geospatial-cookbook-gpu` / `-cpu` on every push. |
| `app.json` / `app-cpu.json` | Tapis application definitions (interactive Jupyter). `app.json` = GPU, `app-cpu.json` = CPU. |
| `notebook.ipynb` | Example SAM 3 text-prompt segmentation notebook. |

## How it runs on TACC

1. A push to this repo triggers the GitHub Action to build and publish the `ghcr.io/in-for-disaster-analytics/segment-geospatial-cookbook-gpu` image.
2. The Tapis `app.json` references that image (`containerImage`) and runs it on `ls6` via Singularity, mounting TAP functions and user directories and enabling `--nv` (GPU).
3. `run.sh` starts Jupyter Lab; the session URL (with token) is sent to the Cookbook UI webhook so you can open the notebook in a browser.

## Requirements

- A TACC account with an allocation (request one [here](https://portal.tacc.utexas.edu/allocation-request)).
- SAM 3 checkpoint access: `facebook/sam3` and `facebook/sam3.1` on Hugging Face. Accept the license and run `huggingface-cli login` (or set `HF_TOKEN`) before using `SamGeo3`.

## Notes

- The conda env installs `segment-geospatial` **from PyPI** (currently v1.4.2, which ships `SamGeo3`), keeping the app in sync with the upstream library.
- Use the **GPU** app for SAM 3 (CUDA required); the CPU app is for lightweight exploration.
- This repo is adapted from `Cookbook-Conda-Template`; the original batch-script template is read-only.
