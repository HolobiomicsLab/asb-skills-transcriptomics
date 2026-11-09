---
name: module-trait-correlation-analysis
description: Use when a processed scRNA-seq dataset has per-cell trait labels (outcome,
  condition, cell type) and you have detected co-expression modules and need to know
  which modules associate with which trait.
when_to_use_negative:
- The input is bulk RNA-seq or already aggregated expression profiles that lack single-cell
  metadata — SCANet's pseudo-bulking of neighboring cells and trait-association workflow
  assumes per-cell trait labels.
- 'The goal is causal inference of regulatory relationships: GCNs rely on correlation,
  not causation, so module–trait correlations can yield false positives and should
  not be read as mechanistic direction without downstream GRN validation (e.g. GRNBoost2
  + RcisTarget).'
- Trait labels are missing or ambiguous — the module–trait correlation step requires
  cell-level metadata (outcome, condition, cell type) to correlate against.
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCANet
  role: Python package orchestrating module inference and module–trait/cell-type association
    analysis; installable via pip and imported as scanet.
  repo: https://github.com/oubounyt/SCANet
- name: WGCNA
  role: Modified weighted gene co-expression network analysis used inside SCANet for
    network inference and co-expression module detection before trait correlation.
- name: BioNERO
  role: Framework through which SCANet constructs the gene–gene correlation matrix
    and infers the signed GCN from pseudo-bulk representative cells.
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:aecb69b8aa535473b93c9b0a9810084589a453b05e166a16e0edb65f7ca0604d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - BioNERO
  - GRNBoost2
  - Python
  - RcisTarget
  - SCANet
  - Scanpy
  - WGCNA
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# module-trait-correlation-analysis

## Summary

Correlates single-cell-derived gene co-expression modules (inferred with SCANet's modified WGCNA/BioNERO framework) against cell-level trait metadata such as patient outcome or diet condition, to identify modules tracking a phenotype (e.g. M11 with deceased patients, M21 with survivors).

## When to use

Use when a processed scRNA-seq dataset has per-cell trait labels (outcome, condition, cell type) and you have detected co-expression modules and need to know which modules associate with which trait.

## When NOT to use

- The input is bulk RNA-seq or already aggregated expression profiles that lack single-cell metadata — SCANet's pseudo-bulking of neighboring cells and trait-association workflow assumes per-cell trait labels.
- The goal is causal inference of regulatory relationships: GCNs rely on correlation, not causation, so module–trait correlations can yield false positives and should not be read as mechanistic direction without downstream GRN validation (e.g. GRNBoost2 + RcisTarget).
- Trait labels are missing or ambiguous — the module–trait correlation step requires cell-level metadata (outcome, condition, cell type) to correlate against.

## Inputs

- Preprocessed scRNA-seq expression matrix with cell-level metadata (e.g. Cillo et al. 2021 GSE180578 PBMC data; Aliluev et al. 2021 enteroendocrine H5ad/HDF5/loom loaded via Scanpy)
- Cell type sub-population selection (e.g. classical monocytes from PBMCs of deceased/surviving patients and healthy controls)
- Trait metadata per cell or patient (patient outcome: healthy, deceased, surviving; diet condition: HFHSD vs control)
- Detected co-expression modules with gene memberships (e.g. 27 modules of 30–195 genes)

## Outputs

- Module–trait correlation table (modules × traits, e.g. M11 vs deceased, M21 vs surviving)
- Ranked list of modules associated with each trait (e.g. M4 most correlated with ISCs in HFHSD mice)
- Module gene lists and per-module statistics (gene counts, internal correlation structure)

## How to apply

First subset the relevant cell population (e.g. classical monocytes from GSE180578 PBMCs, or enteroendocrine cells from Aliluev et al. 2021) and divide its cells into N sub-clusters, averaging gene expression within each sub-cluster to build representative pseudo-bulk cells, down-sampling so every cell type is represented equally. Infer a signed gene co-expression network with the modified WGCNA in BioNERO via SCANet, using Pearson's r, Spearman's, or biweight midcorrelation. Detect co-expression modules from the network (in the Cillo et al. case, 27 modules of 30–195 genes each) and correlate each module against the trait metadata (healthy/deceased/surviving, or HFHSD vs control) to produce the module–trait correlation table. Select modules following the 'Modules selection guidelines' in Supplementary File 1, and check that each module graph fits the scale-free topology characteristic of real-world biological networks.

## Related tools

- **SCANet** (Python package orchestrating module inference and module–trait/cell-type association analysis; installable via pip and imported as scanet.) — https://github.com/oubounyt/SCANet
- **WGCNA** (Modified weighted gene co-expression network analysis used inside SCANet for network inference and co-expression module detection before trait correlation.)
- **BioNERO** (Framework through which SCANet constructs the gene–gene correlation matrix and infers the signed GCN from pseudo-bulk representative cells.)

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- Module count and sizes match expected results: e.g. 27 co-expression modules of classical monocytes with 30–195 genes per module (Cillo et al. case); 25 modules with M4 top ISC-associated module (Aliluev et al. case).
- Trait associations reproduce the reported pattern: M11 shows the highest positive correlation with deceased patients while M21 correlates with surviving patients.
- Each module graph fits the scale-free topology, a characteristic of real-world biological networks.
- Hub genes are identified as the intersection of the top 10% most connected genes within the module (e.g. over half of M11 genes/hubs code for heat shock proteins such as HSPB1, DNAJA1, CRYAB).
- Module selection followed the documented 'Modules selection guidelines' (Supplementary File 1).

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis.
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected.
- Gene–drug databases used downstream (Drugst.One) update weekly, impacting reproducibility: running identical analyses and parameters on newer datasets may yield different results.
- No version control system for Drugst.One is yet available to ensure consistent results across SCANet versions.

## Evidence

- [intro] SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection: "SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection"
- [results] 27 modules finding: "We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module (M) (Fig. 2C)"
- [results] M11/M21 trait correlation finding: "In particular, M11 had the highest positive correlation with deceased patients, while M21 had a high correlation with surviving patients"
- [methods] module selection guidelines: "The “Modules selection guidelines” section in Supplementary File 1 offers a comprehensive description of recommended guidelines for the selection of modules"
- [readme] README usage note: "A full detailed example of `SCANet` : Refer to this [Jupyter Notebook](https://github.com/oubounyt/SCAn/blob/main/docs/full-example.ipynb)."
