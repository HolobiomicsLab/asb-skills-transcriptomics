---
name: anndata-single-cell-data-handling
description: Use when a single-cell dataset (qPCR, scRNA-seq, or scATAC-seq matrix
  plus optional graph/pseudotime) must be wrapped in a cells × features AnnData object
  before running STREAM or a stream-plot reconstruction.
when_to_use_negative:
- Input is already an .h5ad or in-memory AnnData object consistent with STREAM's expectations
- Data are bulk transcriptomic or non-single-cell, where cells × features structure
  and per-cell labels do not apply
- The goal is trajectory inference with tools that do not consume the anndata class
  (e.g., pure R Monocle2/ElPiGraph.R pipelines that use their own containers)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STREAM
  role: Consumes and operates on the anndata container for trajectory reconstruction,
    exploration, and mapping
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: The data class STREAM is written with; holds cells × features matrices plus
    metadata
  repo: http://anndata.rtfd.io
- name: conda
  role: Installs stream (v1.0) and jupyter into env_stream for working with AnnData
    in Python 3
provenance:
  source_task_ids:
  - task_004
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:955246099504d45e32c440ff217bfd1568643e1090384dc6c6adc308e24bdddf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Monocle2
  - Python 3
  - STREAM
  - anndata
  - conda
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# anndata-single-cell-data-handling

## Summary

Load and structure single-cell expression matrices (cells × features), inferred branching graphs, pseudotime, and cell-type labels into the anndata class so that STREAM and downstream trajectory tasks operate on a consistent data container.

## When to use

Use when a single-cell dataset (qPCR, scRNA-seq, or scATAC-seq matrix plus optional graph/pseudotime) must be wrapped in a cells × features AnnData object before running STREAM or a stream-plot reconstruction.

## When NOT to use

- Input is already an .h5ad or in-memory AnnData object consistent with STREAM's expectations
- Data are bulk transcriptomic or non-single-cell, where cells × features structure and per-cell labels do not apply
- The goal is trajectory inference with tools that do not consume the anndata class (e.g., pure R Monocle2/ElPiGraph.R pipelines that use their own containers)

## Inputs

- Single-cell expression matrix (cells × features) from qPCR, scRNA-seq, or scATAC-seq (peaks/k-mers/motifs)
- Sorted population / cell-type labels (e.g., six Nestorowa-sorted populations)
- Inferred branching graph and pseudotime values (for stream-plot reconstruction)

## Outputs

- anndata AnnData object with expression matrix, observation metadata (cell labels), and optional graph/pseudotime fields
- AnnData ready as direct input to STREAM trajectory fitting and plotting functions

## How to apply

Construct an anndata object holding the expression matrix with cells as observations and features (genes, transcription factors, or ATAC peaks/k-mers/motifs) as variables, mirroring the container STREAM is written with. For the Nestorowa mouse hematopoiesis scRNA-seq dataset (1656 sorted cells) or the ~270-cell qPCR dataset (~170 transcription-factor features across six sorted populations), place the sorted-population labels in the observation metadata so they can serve as ground truth for precision/recall evaluation or cell-type-labeled trajectory plots. When reconstructing a stream plot, also attach the inferred branching graph and pseudotime to the object so per-branch sliding-window cell counts and gene-expression signals can be extracted along each branch. Stream v1.0 is installed via Bioconda (`conda create -n env_stream python=3.7 stream=1.0 jupyter`) and analyzed in Jupyter following the STREAM scRNA-seq bifurcation tutorial.

## Related tools

- **STREAM** (Consumes and operates on the anndata container for trajectory reconstruction, exploration, and mapping) — https://github.com/pinellolab/STREAM
- **anndata** (The data class STREAM is written with; holds cells × features matrices plus metadata) — http://anndata.rtfd.io
- **conda** (Installs stream (v1.0) and jupyter into env_stream for working with AnnData in Python 3)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- AnnData object exposes an observations × features matrix with correct dimensions (e.g., 1656 cells for the Nestorowa dataset; ~270 cells × ~170 TFs for the qPCR dataset)
- Sorted-population labels are recoverable from observation metadata and match the number of distinct classes
- Loading the object into STREAM without errors in the env_stream conda environment (python=3.7, stream=1.0)
- Downstream STREAM runs produce the expected diverging/transition gene detections (e.g., Car2, Epx, Coro1a, Blvrb) confirming the object fed the analysis correctly
- Attached graph/pseudotime fields align with cell indices when stream-plot width polygons are computed

## Limitations

- STREAM v1.0 removed support for the command-line interface, so data handling is Notebook/Jupyter-centric (or via the Docker image pinellolab/stream:1.0)
- Version 1.1 fixed incompatibilities with newer pandas/matplotlib/seaborn; older AnnData-loading scripts may break on unpinned dependency versions
- STREAM does not natively support cyclic or disconnected graph topologies, so AnnData-encoded trajectory graphs must be tree-like for faithful handling

## Evidence

- [methods] STREAM is written using the class `anndata`: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [other] Load the ~270-cell qPCR dataset in anndata: [evidence span withheld — anchor status: not_found]
- [methods] conda env creation: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [other] Load dataset with anndata for stream plot: [evidence span withheld — anchor status: not_found]
- [readme] Stream readme anndata reference: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)](http://anndata.rtfd.io) and available as user-friendly open source software"
