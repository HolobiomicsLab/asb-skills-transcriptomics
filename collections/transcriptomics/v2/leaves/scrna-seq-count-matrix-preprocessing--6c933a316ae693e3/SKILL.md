---
name: scrna-seq-count-matrix-preprocessing
description: Use when raw single-cell RNA-seq count matrices must be quality-controlled
  and prepared as anndata objects before mapping new cells (e.g. Gfi1−/−, Irf8−/−,
  double-KO GMPs) onto an existing STREAM reference graph.
when_to_use_negative:
- The new cells have totally different fates compared to those present in the reference
  structure — the article warns the mapping procedure may not be optimal in this case
- Input is already a normalized/feature-selected expression table or a non-transcriptomic
  modality requiring a different pipeline branch (scATAC-seq peaks/k-mers/motifs need
  stream_atac)
- The goal is to infer a trajectory de novo from a single pooled dataset — use STREAM's
  reconstruction tutorials instead of the mapping/preprocessing path
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: STREAM
  role: Pipeline used for QC of scRNA-seq counts and for the MLLE-based mapping of
    new cells onto a fixed reference trajectory
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Data container class in which STREAM expects the scRNA-seq count matrices
- name: ElPiGraph
  role: Core algorithm that learned the elastic-principal-graph reference onto which
    preprocessed cells are mapped
  repo: sysbio-curie/ElPiGraph.R
- name: conda
  role: Environment management for installing stream=1.0 with Python 3.7
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:fea4c23e2c1f72a95f76fdc5176927b9b15c6671f5420306abdee3efa31fa36d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# scrna-seq-count-matrix-preprocessing

## Summary

Preprocess raw scRNA-seq count matrices (loading into anndata, QC metrics and plots, feature selection) so that genetically perturbed populations can be projected onto a precomputed STREAM elastic-principal-graph reference trajectory without re-fitting it.

## When to use

Use when raw single-cell RNA-seq count matrices must be quality-controlled and prepared as anndata objects before mapping new cells (e.g. Gfi1−/−, Irf8−/−, double-KO GMPs) onto an existing STREAM reference graph.

## When NOT to use

- The new cells have totally different fates compared to those present in the reference structure — the article warns the mapping procedure may not be optimal in this case
- Input is already a normalized/feature-selected expression table or a non-transcriptomic modality requiring a different pipeline branch (scATAC-seq peaks/k-mers/motifs need stream_atac)
- The goal is to infer a trajectory de novo from a single pooled dataset — use STREAM's reconstruction tutorials instead of the mapping/preprocessing path

## Inputs

- Raw scRNA-seq count matrices for perturbed GMP populations (Gfi1−/−, Irf8−/−, Gfi1−/−Irf8−/−)
- Wild-type reference scRNA-seq dataset with precomputed STREAM elastic-principal-graph trajectory (ElPiGraph core)
- Gene expression matrix in anndata format

## Outputs

- QC-filtered anndata object ready for trajectory mapping
- Per-branch allocation table with fractions of each genotype on each reference branch (fractions summing to 1)
- Visualization of mapped mutant cells on the reference branch structure (Fig. 2f-g style)

## How to apply

Install STREAM from Bioconda (stream=1.0 for RNA-seq; add stream_atac only for scATAC-seq) inside a Python 3.7 conda environment named env_stream, or use the pinellolab/stream Docker image. Load the raw scRNA-seq count matrices into an anndata object, since STREAM is written using the class anndata; use the QC metrics and plots added in STREAM v1.0 to filter cells and genes before analysis. Keep the wild-type reference trajectory fixed (learned with the ElPiGraph core) and never pool mutant cells with reference cells when mapping: STREAM's mapping feature projects new populations using MLLE-based neighborhood weights instead of re-computing the trajectory. For the GMP perturbation workflow, load the wild-type reference plus Gfi1−/−, Irf8−/− and Gfi1−/−Irf8−/− GMP counts, map each genotype onto the reference graph, tabulate per-branch fractions, and visualize mapped cells to reproduce the Fig. 2f-g pattern.

## Related tools

- **STREAM** (Pipeline used for QC of scRNA-seq counts and for the MLLE-based mapping of new cells onto a fixed reference trajectory) — https://github.com/pinellolab/STREAM
- **anndata** (Data container class in which STREAM expects the scRNA-seq count matrices)
- **ElPiGraph** (Core algorithm that learned the elastic-principal-graph reference onto which preprocessed cells are mapped) — sysbio-curie/ElPiGraph.R
- **conda** (Environment management for installing stream=1.0 with Python 3.7)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook
```

## Evaluation signals

- The anndata object is correctly populated with the scRNA-seq count matrix and passes STREAM QC metrics/plots
- Per-branch allocation table contains all three genotypes with per-branch fractions summing to 1
- Expert/LLM review confirms the expected pattern: Gfi1−/− biased to the monocyte branch, Irf8−/− to the granulocyte branch, double KO balanced
- No cells were pooled with the reference and the reference trajectory was not re-fitted during mapping
- Mapped-cell visualization reproduces the Fig. 2f-g branch-diversion pattern

## Limitations

- The mapping procedure may not be optimal if the new cells have totally different fates compared to the reference structure
- STREAM does not yet adapt to cyclic or disconnected graph topologies, and intuitive visualization for complex ElPiGraph topologies is limited
- Scalability to millions of cells is not established for STREAM and many other trajectory inference methods
- STREAM v1.0 removed support for the command-line interface; analysis is notebook/website based

## Evidence

- [readme] STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [readme] 1) added QC metrics and plots: "1) added QC metrics and plots"
- [intro] no trajectory inference method provides the possibility to map new cells to previously obtained reference trajectories without pooling cells and re-computing trajectories: "no trajectory inference method provides the possibility to map new cells to previously obtained reference trajectories without pooling cells and re-computing trajectories."
- [methods] conda create -n env_stream python=3.7 stream=1.0 jupyter: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [discussion] the mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure: "However, the mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure"
