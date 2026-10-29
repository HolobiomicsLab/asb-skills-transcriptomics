---
name: kmert-accessibility-feature-selection
description: Use when the input is a raw scATAC-seq k-mer or peak count matrix (e.g.
  2034 QC-passed human bone-marrow cells) and the goal is trajectory inference or
  regulator discovery without gene-level annotations.
when_to_use_negative:
- The data is single-cell transcriptomic (scRNA-seq); STREAM's gene-based workflow,
  not the k-mer/chromVAR feature path, applies.
- The scATAC-seq features are already aggregated to annotated motifs or peaks — use
  STREAM's peak- or motif-based tutorials (2.1-STREAM_scATAC-seq_peaks.ipynb) instead
  of k-mer feature selection.
- Trajectory structure is cyclic or totally disconnected from the reference; STREAM's
  ElPiGraph core and mapping procedure do not properly support these topologies.
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0085
tools:
- name: STREAM
  role: End-to-end pipeline performing k-mer deviation z-scoring, PCA feature selection,
    MLLE embedding, and ElPiGraph trajectory learning on scATAC-seq data
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core principal-graph algorithm inside STREAM that learns the branching trajectory
    and assigns cells to branches with pseudotime
  repo: sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data class in which the raw k-mer count matrix is loaded and passed through
    the feature-selection workflow
  repo: http://anndata.rtfd.io
- name: chromVARmotifs
  role: PWMatrixList motif collections (human_pwms_v1/v2, cisBP-derived) used to map
    top-ranked branch k-mers to transcription factors
  repo: buenrostrolab/chromVARmotifs
- name: conda
  role: Creates the env_stream environment with stream=1.0 and the stream_atac add-on
    required for scATAC-seq k-mer analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:b6641acf2acf3bf772037dec3aa4046198f5db674d5f4396cf25ef3fdecbac6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  - chromVARmotifs
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kmert-accessibility-feature-selection

## Summary

Select informative DNA k-mer accessibility features from raw single-cell ATAC-seq count matrices (z-score deviation computation plus PCA-based filtering) so that branching trajectories can be reconstructed and lineage-driving transcription factors identified via k-mer-to-motif mapping.

## When to use

Use when the input is a raw scATAC-seq k-mer or peak count matrix (e.g. 2034 QC-passed human bone-marrow cells) and the goal is trajectory inference or regulator discovery without gene-level annotations.

## When NOT to use

- The data is single-cell transcriptomic (scRNA-seq); STREAM's gene-based workflow, not the k-mer/chromVAR feature path, applies.
- The scATAC-seq features are already aggregated to annotated motifs or peaks — use STREAM's peak- or motif-based tutorials (2.1-STREAM_scATAC-seq_peaks.ipynb) instead of k-mer feature selection.
- Trajectory structure is cyclic or totally disconnected from the reference; STREAM's ElPiGraph core and mapping procedure do not properly support these topologies.

## Inputs

- Raw scATAC-seq k-mer count matrix (cells × k-mers) for 2034 QC-passed human bone-marrow cells
- anndata (.h5ad) object loaded with the k-mer counts
- PWMatrixList motif collections (e.g. human_pwms_v1/v2 from chromVARmotifs) for k-mer-to-transcription-factor mapping

## Outputs

- Z-scored, PCA-filtered k-mer feature matrix suitable for MLLE/ElPiGraph
- Branch-specific ranked k-mer table mapped to transcription factors (e.g. GATA1, CEBPA)
- Trajectory figure showing the reconstructed hematopoietic hierarchy with pseudotime

## How to apply

Load the raw human bone-marrow scATAC-seq k-mer count matrix into an anndata object inside the stream_atac-enabled conda environment, then compute k-mer accessibility deviations (z-scores) across cells so the feature space reflects cell-specific chromatin accessibility rather than raw counts. Apply PCA-based feature selection on the z-scored k-mers to retain only informative k-mers, discarding those contributing little variance — this is the critical filter step before trajectory learning. Embed the selected k-mer features with MLLE dimensionality reduction, then feed the reduced space into ElPiGraph to learn the branching graph (HSC through MPP into erythroid, lymphoid, and myeloid branches) with pseudotime. Rank k-mers along each detected branch and map top k-mers to their associated transcription factors using motif resources such as chromVARmotifs PWMs, expecting master regulators like GATA1 (erythroid) and CEBPA (myeloid) to surface. Success is judged by correct recovery of the known hematopoietic hierarchy and biologically coherent branch-specific k-mer rankings.

## Related tools

- **STREAM** (End-to-end pipeline performing k-mer deviation z-scoring, PCA feature selection, MLLE embedding, and ElPiGraph trajectory learning on scATAC-seq data) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core principal-graph algorithm inside STREAM that learns the branching trajectory and assigns cells to branches with pseudotime) — sysbio-curie/ElPiGraph.R
- **anndata** (Data class in which the raw k-mer count matrix is loaded and passed through the feature-selection workflow) — http://anndata.rtfd.io
- **chromVARmotifs** (PWMatrixList motif collections (human_pwms_v1/v2, cisBP-derived) used to map top-ranked branch k-mers to transcription factors) — buenrostrolab/chromVARmotifs
- **conda** (Creates the env_stream environment with stream=1.0 and the stream_atac add-on required for scATAC-seq k-mer analysis)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 stream_atac jupyter; conda activate env_stream
```

## Evaluation signals

- The reconstructed trajectory recovers the known hematopoietic hierarchy: HSC → MPP branching into erythroid, lymphoid, and myeloid branches.
- Top branch-ranked k-mers map to expected master regulators — GATA1 for the erythroid branch and CEBPA for the myeloid branch.
- The z-scored k-mer matrix is reduced by PCA feature selection before embedding, not passed to MLLE with unfiltered raw counts.
- The analysis runs inside the env_stream environment created with stream=1.0 and the stream_atac add-on package.

## Limitations

- Scalability to millions of cells is not established for STREAM's trajectory inference.
- The mapping procedure may not be optimal if new cells have totally different fates than cells in the reference structure.
- Further work is needed to adapt STREAM to cyclic or disconnected graph topologies, and ElPiGraph's complex-graph support lacks intuitive visualizations.

## Evidence

- [discussion] Monocle2 and STREAM applied to single-cell chromatin accessibility: "so far very few trajectory inference methods (Monocle2 and STREAM) have been applied to analyze single-cell chromatin accessibility"
- [intro] No end-to-end open-source solution for epigenomic trajectory analysis: "these methods have been designed for analyzing single-cell transcriptomic data only. On the other hand, no end-to-end and open-source software solution exists"
- [readme] Conda environment setup for scATAC-seq STREAM analysis: "$ conda create -n env_stream python=3.7 stream=1.0 stream_atac jupyter"
- [readme] scATAC-seq feature support added in v1.0: "added support of scATAC-seq analysis using peaks as features"
- [readme] anndata is the data class used by STREAM: "STREAM is written using the class `anndata`"
