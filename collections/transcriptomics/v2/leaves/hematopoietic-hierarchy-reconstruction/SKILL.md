---
name: hematopoietic-hierarchy-reconstruction
description: Use when input is a raw single-cell chromatin-accessibility (scATAC-seq)
  k-mer or peak count matrix and the goal is to infer a branching developmental trajectory
  with pseudotime and identify transcription-factor regulators along branches.
when_to_use_negative:
- The data are already a curated feature table (e.g. precomputed chromVAR motif deviations
  or gene expression alone) — use the corresponding STREAM mode (motifs, peaks, or
  scRNA-seq) rather than the k-mer-based epigenomic workflow
- The expected trajectory is cyclic or a disconnected graph — the paper notes further
  improvement is necessary to adapt STREAM to these more general structures
- Only transcriptomic scRNA-seq data are available and epigenomic regulation is not
  of interest — a transcriptomics-specific trajectory tool may suffice
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
tools:
- name: STREAM
  role: End-to-end trajectory inference pipeline for scATAC-seq k-mer/chromVAR features;
    provides pseudotime, branch assignment, and branch-specific k-mer ranking
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core principal-graph algorithm behind STREAM that learns the branching trajectory
    graph
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Data-class format in which STREAM represents the scATAC-seq count matrix and
    annotations
- name: conda / Bioconda
  role: Installation channel used to create the env_stream environment with the stream
    and stream_atac packages
  repo: https://github.com/pinellolab/STREAM
- name: chromVARmotifs
  role: Curated human/mouse motif PWM collections (cisBP, HOMER, ENCODE) for linking
    branch-ranked k-mers to transcription factors
  repo: https://github.com/pinellolab/STREAM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:248dff82350d11d30c72d0210d4fb563d5041961f052bc809f41776bd05b32e7
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
  - conda / Bioconda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hematopoietic-hierarchy-reconstruction

## Summary

Reconstruct the human hematopoietic developmental hierarchy (HSC → MPP → erythroid, lymphoid, myeloid branches) directly from raw scATAC-seq chromatin-accessibility data by running the STREAM/ElPiGraph trajectory pipeline on k-mer/chromVAR features, then ranking branch-specific k-mers and mapping them to master-regulator transcription factors such as GATA1 (erythroid) and CEBPA (myeloid).

## When to use

Use when input is a raw single-cell chromatin-accessibility (scATAC-seq) k-mer or peak count matrix and the goal is to infer a branching developmental trajectory with pseudotime and identify transcription-factor regulators along branches.

## When NOT to use

- The data are already a curated feature table (e.g. precomputed chromVAR motif deviations or gene expression alone) — use the corresponding STREAM mode (motifs, peaks, or scRNA-seq) rather than the k-mer-based epigenomic workflow
- The expected trajectory is cyclic or a disconnected graph — the paper notes further improvement is necessary to adapt STREAM to these more general structures
- Only transcriptomic scRNA-seq data are available and epigenomic regulation is not of interest — a transcriptomics-specific trajectory tool may suffice

## Inputs

- Raw scATAC-seq k-mer count matrix for QC-passed cells (e.g. 2034 human bone-marrow cells), loaded into an anndata object
- Alternative STREAM inputs: scATAC-seq peak or motif feature matrices, or scRNA-seq expression matrices
- Transcription-factor motif PWM collections (e.g. chromVARmotifs human_pwms_v1/v2) for k-mer-to-TF mapping

## Outputs

- Trajectory figure of the hematopoietic hierarchy (HSC → MPP → erythroid, lymphoid, myeloid branches) with pseudotime
- Branch assignment of each cell to a trajectory branch with pseudotime values
- Ranked k-mer list per branch with mapped transcription factors (e.g. GATA1 for erythroid, CEBPA for myeloid)

## How to apply

Install the scATAC-seq-enabled STREAM environment with Bioconda (`conda create -n env_stream python=3.7 stream=1.0 stream_atac jupyter`) and load the raw k-mer count matrix (2034 QC-passed human bone-marrow cells in the demonstrated case) into an anndata object. Compute k-mer accessibility deviations as z-scores and apply PCA-based feature selection to retain informative k-mers before trajectory inference, since raw accessibility counts are sparse and high-dimensional. Embed the selected features with MLLE and run ElPiGraph — the core algorithm behind STREAM — to learn the branching principal graph and assign cells to branches with pseudotime; STREAM requires no prior knowledge of time points, start cell, or the number of branching events. Rank k-mers along each detected branch and map the top k-mers to their associated transcription factors (motif collections such as buenrostrolab/chromVARmotifs supply the PWMs). Save the trajectory figure showing the HSC-to-lineage hierarchy and the ranked k-mer-to-TF table as outputs.

## Related tools

- **STREAM** (End-to-end trajectory inference pipeline for scATAC-seq k-mer/chromVAR features; provides pseudotime, branch assignment, and branch-specific k-mer ranking) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core principal-graph algorithm behind STREAM that learns the branching trajectory graph) — https://github.com/pinellolab/STREAM
- **anndata** (Data-class format in which STREAM represents the scATAC-seq count matrix and annotations)
- **conda / Bioconda** (Installation channel used to create the env_stream environment with the stream and stream_atac packages) — https://github.com/pinellolab/STREAM
- **chromVARmotifs** (Curated human/mouse motif PWM collections (cisBP, HOMER, ENCODE) for linking branch-ranked k-mers to transcription factors) — https://github.com/pinellolab/STREAM

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 stream_atac jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- The reconstructed graph reproduces the known hematopoietic topology: HSC → MPP splitting into erythroid, lymphoid, and myeloid branches
- Top branch-ranked k-mers map to expected master regulators — GATA1 for the erythroid branch and CEBPA for the myeloid branch
- PCA-based k-mer feature selection retains only informative k-mers; check that the feature matrix is reduced from the full k-mer set before MLLE/ElPiGraph
- Pseudotime increases monotonically from HSC toward the terminal lineage branches on each branch
- The conda environment resolves with the stream_atac add-on installed (python=3.7, stream=1.0)

## Limitations

- STREAM is not yet adapted to cyclic or disconnected graph topologies; further improvement is necessary for these more general structures
- ElPiGraph supports learning complex graphs but more work is necessary to provide intuitive visualizations for sophisticated topologies
- The mapping procedure may not be optimal for new cells whose fates are totally different from those in the reference structure
- Scalability to millions of cells is not established for many trajectory inference methods including STREAM

## Evidence

- [abstract] STREAM pipeline description: "Here we present STREAM, an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data."
- [methods] anndata dependency: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] No prior knowledge required: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [discussion] ElPiGraph role and visualization caveat: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [readme] scATAC-seq env creation command: "$ conda create -n env_stream python=3.7 stream=1.0 stream_atac jupyter"
