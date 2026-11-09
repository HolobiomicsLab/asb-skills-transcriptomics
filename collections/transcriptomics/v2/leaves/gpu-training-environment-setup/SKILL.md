---
name: gpu-training-environment-setup
description: Use when a deep-learning model such as mvTCR must be trained on GPU,
  and no environment with the pinned Python, pip package, and matching CUDA PyTorch
  version exists yet; scale RAM to dataset size.
when_to_use_negative:
- Running on a Mac with M1 chip — several dependencies are unavailable per the README
- Machine has no GPU/CUDA support — mvTCR training requires it and the setup would
  not be usable
- An environment with the required pinned versions (e.g., mvtcr==0.1.3 + PyTorch 1.8.0
  for reproducibility) already exists
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: mvTCR
  role: Target deep-learning package installed into the conda environment via pip;
    requires GPU-enabled PyTorch to train
  repo: https://github.com/SchubertLab/mvTCR
- name: PyTorch
  role: Backend framework installed with a CUDA-matched version after pip install
    mvtcr
- name: conda
  role: Creates and isolates the Python environment with the required Python version
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:9db32d377fd8fc0704a13d71c165c09d93e594b1b46c9b9005e9c340b5166fd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - CoNGA
  - PyTorch
  - conda
  - mvTCR
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gpu-training-environment-setup

## Summary

Set up an isolated conda environment with mvTCR installed via pip and a CUDA-enabled PyTorch build, sized with enough RAM/GPU memory for the target dataset, so deep-learning training runs (e.g., mvTCR benchmarks against tessa and CoNGA) are reproducible and hardware-feasible.

## When to use

Use when a deep-learning model such as mvTCR must be trained on GPU, and no environment with the pinned Python, pip package, and matching CUDA PyTorch version exists yet; scale RAM to dataset size.

## When NOT to use

- Running on a Mac with M1 chip — several dependencies are unavailable per the README
- Machine has no GPU/CUDA support — mvTCR training requires it and the setup would not be usable
- An environment with the required pinned versions (e.g., mvtcr==0.1.3 + PyTorch 1.8.0 for reproducibility) already exists

## Inputs

- Linux or Windows 10 machine with NVIDIA GPU and CUDA driver support
- conda package manager
- Target dataset size estimate (cell count) to dimension RAM/GPU memory

## Outputs

- Activated conda environment 'mvTCR' with pip-installed mvtcr and nb_conda_kernels
- CUDA-matched PyTorch installation (e.g., v2.1.2, or v1.8.0 for paper reproduction)
- Environment capable of running mvTCR training and tutorial notebooks

## How to apply

Create a dedicated conda environment (mvTCR's current install uses Python 3.10; the reproducibility paper used Python 3.8.8 with mvtcr==0.1.3 and PyTorch 1.8.0) with 'conda create --name mvTCR python=3.10 -y', activate it, then 'pip install mvtcr' followed by 'conda install nb_conda_kernels -y'. Install a PyTorch version with a CUDA build matching the machine's driver using the pytorch.org previous-versions instructions (e.g., PyTorch 2.1.2 for current mvTCR, 1.8.0 for paper reproduction). A GPU- and CUDA-capable machine is heavily encouraged; MAC with M1 chip is not supported due to missing dependencies. Size memory to the workload: 8 GB RAM sufficed for training on a preprocessed 10x dataset with >60k cells, while hyperparameter optimization was run on a single 32 GB GPU machine or a 4-GPU node with 512 GB; scale memory allocation with dataset size when subsampling to atlas scale (e.g., 100,000 TIL cells). Installation takes about 10 minutes and has been tested on Windows 10 and Linux 4.18.0.

## Related tools

- **mvTCR** (Target deep-learning package installed into the conda environment via pip; requires GPU-enabled PyTorch to train) — https://github.com/SchubertLab/mvTCR
- **PyTorch** (Backend framework installed with a CUDA-matched version after pip install mvtcr)
- **conda** (Creates and isolates the Python environment with the required Python version)

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr && conda install nb_conda_kernels -y
```

## Evaluation signals

- 'conda activate mvTCR' succeeds and 'pip show mvtcr' reports the intended version (e.g., 0.1.3 for reproducibility)
- torch.cuda.is_available() returns True after installing the CUDA-matched PyTorch build
- mvTCR tutorial notebooks execute within the documented ~15 minutes on the subsampled dataset with expected outputs produced
- Training on the preprocessed 10x dataset (>60k cells) completes within the 8 GB RAM reference footprint, confirming memory sizing is adequate
- nvidia-smi/CUDA version reported by PyTorch matches the driver's supported CUDA version

## Limitations

- MAC with M1 chip unsupported: several dependencies are currently unavailable
- Installation requires roughly 10 minutes and was only tested on Windows 10 and Linux 4.18.0
- Different compute-cluster runtimes, hardware components, and CUDA versions can cause trained models to diverge from the published ones, so paper results additionally rely on models hosted on Zenodo
- Memory must scale with dataset size; 8 GB is only a reference for >60k cells and larger subsamples (e.g., 100k cells) need more

## Evidence

- [methods] conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr: "conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr"
- [readme] The installation procedure will take approx. 10 minutes and has been tested on Windows 10 and Linux 4.18.0. For MAC with M1 chip, there are currently several dependencies not available. A machine with GPU- and CUDA-support is heavily encouraged.: "The installation procedure will take approx. 10 minutes and has been tested on Windows 10 and Linux 4.18.0. For MAC with M1 chip, there are"
- [readme] The required memory scales with the dataset size. As a reference, 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells).: "The required memory scales with the dataset size. As a reference, 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k"
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
