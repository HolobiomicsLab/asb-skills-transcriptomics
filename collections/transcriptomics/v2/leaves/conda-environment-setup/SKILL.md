---
name: conda-environment-setup
description: Use when installing STREAM (or similar Bioconda-packaged single-cell
  tools) to reconstruct branching trajectories and need an isolated Python 3.7 environment
  with pinned versions before running the STREAM tutorials in Jupyter.
when_to_use_negative:
- 'Analysis requires the STREAM command-line interface: support for the CLI was removed
  in STREAM v1.0, which provides only the interactive notebook workflow'
- Installation via Bioconda is blocked in the working environment — use the Docker
  route ('docker pull pinellolab/stream') instead rather than hand-installing dependencies
- The target analysis is generic single-cell processing unrelated to STREAM; this
  skill only guarantees the dependency stack (anndata, plotting libraries) pinned
  for STREAM v1.0
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: conda
  role: Package manager used to add Bioconda channels and create/activate the env_stream
    environment with python=3.7, stream=1.0, jupyter
- name: STREAM
  role: The installed trajectory-inference pipeline (via Bioconda stream=1.0) whose
    tutorials this environment enables; written using the anndata class
  repo: https://github.com/pinellolab/STREAM
- name: Anaconda
  role: Base Python 3 Anaconda distribution that must be present before the Bioconda
    channels and env_stream environment can be configured
- name: anndata
  role: Data-class dependency automatically satisfied by the stream package; the class
    STREAM is written with, used to load count matrices
  repo: https://github.com/pinellolab/STREAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:d92e712e3ad98475b0203aa5bec2e6faf4ac517b06a3a558869b61b2c3400cd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - Anaconda
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  - conda
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# conda-environment-setup

## Summary

Provision a reproducible Python 3.7 environment for STREAM trajectory analysis by installing the stream v1.0 package from Bioconda inside a dedicated conda environment (env_stream) together with Jupyter, so the scRNA-seq/scATAC-seq bifurcation tutorials run as published.

## When to use

Use when installing STREAM (or similar Bioconda-packaged single-cell tools) to reconstruct branching trajectories and need an isolated Python 3.7 environment with pinned versions before running the STREAM tutorials in Jupyter.

## When NOT to use

- Analysis requires the STREAM command-line interface: support for the CLI was removed in STREAM v1.0, which provides only the interactive notebook workflow
- Installation via Bioconda is blocked in the working environment — use the Docker route ('docker pull pinellolab/stream') instead rather than hand-installing dependencies
- The target analysis is generic single-cell processing unrelated to STREAM; this skill only guarantees the dependency stack (anndata, plotting libraries) pinned for STREAM v1.0

## Inputs

- No data inputs; requires an installed conda (Anaconda/Miniconda) with Python 3
- Optional: scRNA-seq count matrix (e.g., Olsson et al. wild-type LSK/CMP/GMP/LKCD34+ subpopulations) to be loaded into an anndata object once the environment is active

## Outputs

- An activated conda environment 'env_stream' with python 3.7, stream 1.0, and jupyter installed
- A running Jupyter Notebook session from which STREAM tutorials (e.g., 1.1-STREAM_scRNA-seq (Bifurcation).ipynb) can be executed

## How to apply

First add the required channels in order ('defaults', 'bioconda', 'conda-forge') so Bioconda resolves the stream recipe. Then create an environment named env_stream with 'conda create -n env_stream python=3.7 stream=1.0 jupyter' for scRNA-seq analysis; add the stream_atac dependency if scATAC-seq features are needed. Activate the environment with 'conda activate env_stream' and launch 'jupyter notebook' inside it, since the class STREAM is written against (anndata, Wolf et al. 2018) and the plotting stack (pandas, matplotlib, seaborn) are resolved by this pinned recipe — v1.0 fixed incompatibilities with recent pandas/matplotlib/seaborn. Run the tutorial notebook 1.1-STREAM_scRNA-seq (Bifurcation).ipynb from within this environment. As an alternative that avoids dependency configuration entirely, pull the Docker image 'pinellolab/stream' and run Jupyter inside the container with the data folder mounted at /data.

## Related tools

- **conda** (Package manager used to add Bioconda channels and create/activate the env_stream environment with python=3.7, stream=1.0, jupyter)
- **STREAM** (The installed trajectory-inference pipeline (via Bioconda stream=1.0) whose tutorials this environment enables; written using the anndata class) — https://github.com/pinellolab/STREAM
- **Anaconda** (Base Python 3 Anaconda distribution that must be present before the Bioconda channels and env_stream environment can be configured)
- **anndata** (Data-class dependency automatically satisfied by the stream package; the class STREAM is written with, used to load count matrices) — https://github.com/pinellolab/STREAM

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream
```

## Evaluation signals

- All three channels added in order: 'conda config --add channels defaults/bioconda/conda-forge' completes without conflicts
- 'conda create -n env_stream python=3.7 stream=1.0 jupyter' solves successfully and 'conda activate env_stream' switches the prompt to env_stream
- 'stream' is importable at version 1.0 and 'jupyter notebook' opens in the browser as described in the README
- The environment resolves the anndata dependency (Wolf et al. 2018) and plotting stack pinned to versions compatible with STREAM v1.0 (no pandas/matplotlib/seaborn incompatibility errors from the Dec 2021 v1.1 fix list appear)
- The 1.1-STREAM_scRNA-seq (Bifurcation) tutorial notebook runs end-to-end without import errors

## Limitations

- The recipe pins python=3.7 and stream=1.0; later stream versions (e.g., v1.1, Dec 2021) fixed incompatible pandas/matplotlib/seaborn behavior, so the environment may need upgrading if plotting issues arise
- scATAC-seq analysis requires the extended recipe 'conda create -n env_stream python=3.7 stream=1.0 stream_atac jupyter'; the plain scRNA-seq recipe is insufficient for ATAC features
- The bioconda route assumes conda/Anaconda is installed with Python 3; if dependency resolution fails, the Docker alternative ('docker pull pinellolab/stream') is the supported fallback

## Evidence

- [methods] conda create recipe: "Create an environment named `env_stream` , install **stream**, **jupyter**, and activate it"
- [readme] conda command: "conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [methods] anndata note: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [readme] docker alternative: "With Docker no installation is required, the only dependence is Docker itself. Users will completely get rid of all the installation and configuration issues."
- [readme] conda activate note: "$ conda activate env_stream"
