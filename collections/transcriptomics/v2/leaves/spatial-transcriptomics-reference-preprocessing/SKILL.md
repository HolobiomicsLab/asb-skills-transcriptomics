---
name: spatial-transcriptomics-reference-preprocessing
description: Use when reconstructing pseudo-space from scRNA-seq with scSpace and
  the candidate spatial reference (Visium/STARmap/Slide-seq) may differ architecturally
  from the target tissue, e.g., DLPFC vs agranular dACC lacking cortical layer 4.
when_to_use_negative:
- The target tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC in the
  paper), where even a well-matched reference yields unreliable reconstruction
- The reference's spatial architecture demonstrably differs from the target (e.g.,
  using a granular DLPFC reference for agranular dACC that lacks cortical layer 4)
  and no matching reference is available
- The goal is spatial domain identification on ST data itself — SpaGCN, BayesSpace,
  or DR-SC target that task, whereas this preprocessing only supports scRNA-seq pseudo-space
  reconstruction
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scSpace
  role: Downstream integrative algorithm whose TCA/MLP pipeline consumes the preprocessed
    reference and target matrices; this skill prepares the reference so its reconstruction
    is valid
  repo: github.com/ZJUFanLab/scSpace
- name: Visium
  role: Platform providing the spatial transcriptomics reference slice (e.g., DLPFC
    slice 151674) whose coordinates and spots must be assessed for architectural match
- name: scanpy
  role: Used to normalize and preprocess both the ST reference and target expression
    matrices before scSpace ingestion
  repo: scverse/scanpy
- name: pandas
  role: Reads the five required formatted .csv input files (reference expression,
    target expression, coordinates)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:3f2ca79afdc6e75e1991c40736e5e2a02957f080738b682860070a6ab5fd9701
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Visium
  - pandas
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spatial-transcriptomics-reference-preprocessing

## Summary

Assess and pre-process the spatial transcriptomics reference (e.g., a human DLPFC 10X Visium slice such as 151674) with scanpy before running scSpace, so that the reference's spatial architecture (e.g., cortical layers) matches the target tissue and the pseudo-space reconstruction is valid.

## When to use

Use when reconstructing pseudo-space from scRNA-seq with scSpace and the candidate spatial reference (Visium/STARmap/Slide-seq) may differ architecturally from the target tissue, e.g., DLPFC vs agranular dACC lacking cortical layer 4.

## When NOT to use

- The target tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC in the paper), where even a well-matched reference yields unreliable reconstruction
- The reference's spatial architecture demonstrably differs from the target (e.g., using a granular DLPFC reference for agranular dACC that lacks cortical layer 4) and no matching reference is available
- The goal is spatial domain identification on ST data itself — SpaGCN, BayesSpace, or DR-SC target that task, whereas this preprocessing only supports scRNA-seq pseudo-space reconstruction

## Inputs

- Spatial reference expression matrix with coordinates from a 10X Visium ST slice (e.g., human DLPFC slice 151674), formatted as .csv
- Target scRNA-seq or tissue-section expression matrix with coordinates removed, formatted as .csv
- Expected tissue architecture metadata (e.g., cortical layer labels) for the appropriateness assessment

## Outputs

- Normalized, preprocessed expression matrices (reference and target) suitable for scSpace TCA input
- Appropriateness assessment decision: reference accepted, rejected, or replaced by a matching-region reference

## How to apply

Load the preprocessed spatial reference and target expression matrices as five formatted .csv files read in by pandas, following the DLPFC vignette (slice 151674) conventions. Normalize and preprocess both matrices with scanpy (scSpace pins scanpy 1.9.1, pandas 1.5.0, scipy 1.9.2, Python >=3.8). Before reconstruction, perform an appropriateness assessment of the reference: confirm its spatial domains (e.g., DLPFC cortical layers) correspond to the target tissue's expected hierarchy, since spatial architectures can differ across regions of the same tissue — dACC is agranular and lacks cortical layer 4, so a DLPFC reference would be inappropriate for dACC targets. If layers are mismatched, substitute a reference from a matching region or biological replicate (scSpace performance is conserved across different melanoma replicates). Only after the reference is judged appropriate should TCA batch-effect elimination and MLP pseudo-space projection proceed.

## Related tools

- **scSpace** (Downstream integrative algorithm whose TCA/MLP pipeline consumes the preprocessed reference and target matrices; this skill prepares the reference so its reconstruction is valid) — github.com/ZJUFanLab/scSpace
- **Visium** (Platform providing the spatial transcriptomics reference slice (e.g., DLPFC slice 151674) whose coordinates and spots must be assessed for architectural match)
- **scanpy** (Used to normalize and preprocess both the ST reference and target expression matrices before scSpace ingestion) — scverse/scanpy
- **pandas** (Reads the five required formatted .csv input files (reference expression, target expression, coordinates))

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; cd scSpace-master; pip install -r requirements.txt; python setup.py install
```

## Evaluation signals

- The reference's cortical layers (or equivalent spatial domains) map one-to-one onto the expected target tissue hierarchy after preprocessing, checked with pandas/scipy on layer labels
- Normalized reference and target matrices share a common gene space with no batch-driven separation, verifiable before TCA is run
- After downstream reconstruction, pairwise-distance correlation between pseudo-space and reference positions is significantly positive
- Reconstructed pseudo-space reproduces the expected layer ordering (e.g., DLPFC cortical layer sequence) as in Fig. 2a-d of the paper

## Limitations

- Choice of spatial reference directly influences the spatial reconstruction; an inappropriate reference cannot be fixed by downstream steps
- Intrinsic differences in spatial homogeneity between tissues may influence the accuracy of scSpace even with a well-preprocessed reference
- Appropriate spatial ATAC-seq reference data may be unavailable, forcing use of an ST reference with converted gene activity matrices (via Signac) for scATAC-seq targets

## Evidence

- [discussion] Appropriateness assessment is mandatory for architecturally heterogeneous tissues: "as dACC is agranular and lacks cortical layer 4), performing an appropriateness assessment or necessary pre-processing steps of the spatial reference should be considered"
- [discussion] Choice of reference drives reconstruction quality: "Since the choice of spatial reference would directly influence the spatial reconstruction"
- [readme] Reference and target preprocessing is done in scanpy with pinned versions: "pip install -r requirements.txt"
- [readme] scSpace input format requirement: "To use scSpace we require five formatted `.csv` files as input (i.e. read in by pandas)."
- [discussion] Reference transferability across biological replicates: "while reconstructing the pseudo-space of melanoma, spatial references are derived from different biological replicates, however, the performance of scSpace on spatial variability identification is"
