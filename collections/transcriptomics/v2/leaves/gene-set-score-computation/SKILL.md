---
name: gene-set-score-computation
description: 'Use when building or applying a ProjectSVR reference model: you need
  a gene-set (signature) score matrix — e.g., from cNMF-derived or overexpression-tested
  gene sets, top 25 genes per set — as regression input features.'
when_to_use_negative:
- The downstream task requires differential expression or marker testing itself rather
  than per-cell signature scoring — use overexpression testing only to define the
  gene sets, not as the scoring method
- The analysis goal is unsupervised dimension reduction/clustering rather than supervised
  reference mapping against a pre-built embedding
- Only a small number of highly variable genes (not curated gene sets) are intended
  as model features — ProjectSVR is designed around gene signature score matrices
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ProjectSVR
  role: Provides ComputeModuleScore wrapper (method='AUCell') that computes the gene-set
    score matrix used as SVR regression features
  repo: https://github.com/JarningGau/ProjectSVR
- name: AUCell
  role: Rank-based gene activity scoring engine producing 0–1 scores robust to normalization
- name: cNMF
  role: Derives continuous-state gene programs used as the gene sets to score
  repo: https://github.com/dylkot/cNMF
- name: UCell
  role: Alternative signature score calculation package listed in ProjectSVR dependencies
- name: Seurat
  role: Hosts the reference atlas object from which gene set scores are computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:3b1073c417f20d23fddc653619e8ca8669dbb82573c68567c7897383fb80632a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/JarningGau/ProjectSVR
  tools:
  - AUCell
  - ProjectSVR
  - Seurat
  - UCell
  - cNMF
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gene-set-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-cell gene signature/activity scores (e.g., with AUCell rank-based scoring) from a gene set matrix so that reference mapping models like ProjectSVR can learn the relationship between signature scores and reference embedding coordinates.

## When to use

Use when building or applying a ProjectSVR reference model: you need a gene-set (signature) score matrix — e.g., from cNMF-derived or overexpression-tested gene sets, top 25 genes per set — as regression input features.

## When NOT to use

- The downstream task requires differential expression or marker testing itself rather than per-cell signature scoring — use overexpression testing only to define the gene sets, not as the scoring method
- The analysis goal is unsupervised dimension reduction/clustering rather than supervised reference mapping against a pre-built embedding
- Only a small number of highly variable genes (not curated gene sets) are intended as model features — ProjectSVR is designed around gene signature score matrices

## Inputs

- Single-cell expression matrix (e.g., Seurat object or count matrix, raw or normalized)
- Gene sets derived from overexpression testing (e.g., top 25 genes per cell type, ribosomal/mitochondrial genes removed) or cNMF programs
- Reference atlas object (e.g., mTCA.seurat.slim.qs) whose cells are to be scored

## Outputs

- Cells × gene-sets gene signature score matrix (AUCell scores in 0–1)
- Score matrix formatted as regression features for ProjectSVR model fitting or projection

## How to apply

Derive cell-type/State-specific gene sets from overexpression testing (discrete types, filtering e.g. p_val_adj < 1e-6 and avg_log2FC > log2(1.5), dropping ribosomal and mitochondrial genes) or from cNMF for continuous states; use the top 25 genes per set by default, since results were consistent across gene set sizes. Score every cell with AUCell (ProjectSVR::ComputeModuleScore with method='AUCell'), which ranks expression so scores (0–1) are robust to normalization and work without raw counts; UCell is an acceptable alternative per the README dependencies. The resulting cells × gene-sets score matrix is the feature input for FitEnsembleSVM / pre-built ProjectSVR models, paired with UMAP_1/2 reference embeddings as regression targets.

## Related tools

- **ProjectSVR** (Provides ComputeModuleScore wrapper (method='AUCell') that computes the gene-set score matrix used as SVR regression features) — https://github.com/JarningGau/ProjectSVR
- **AUCell** (Rank-based gene activity scoring engine producing 0–1 scores robust to normalization)
- **cNMF** (Derives continuous-state gene programs used as the gene sets to score) — https://github.com/dylkot/cNMF
- **UCell** (Alternative signature score calculation package listed in ProjectSVR dependencies)
- **Seurat** (Hosts the reference atlas object from which gene set scores are computed)

## Examples

```
seu.q <- ProjectSVR::ComputeModuleScore(seu.ref, features = gene.sets, method = 'AUCell')
```

## Evaluation signals

- All scores fall in the 0–1 range expected of AUCell rank-based activity scores
- Score matrix dimensions equal (number of reference cells) × (number of gene sets), with one column per signature (top 25 genes per set by default)
- Downstream check: ensemble SVR (t=10 models, m=8000 subsample) trained on these scores predicts held-out reference UMAP_1/2 coordinates with accuracy consistent with comparable performance to Seurat, scArches, Symphony, and ProjecTILs
- Sensitivity check: results remain consistent when gene set size (top-N genes) is varied, since the paper reports robustness across settings
- Scores are computable on rank-normalized data, i.e., identical inputs after different normalization schemes should yield comparable rankings

## Limitations

- Gene set size is a hyperparameter; although results were consistent across settings, the default top 25 genes may not be optimal for every atlas
- AUCell scores rely on gene sets derived from the reference; novel query states absent from the reference may produce misleading projections
- Performance declines when projecting across platforms (e.g., Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth, which can distort signature scores

## Evidence

- [methods] Score matrix feeding the ensemble fit: [evidence span withheld — anchor status: not_found]
- [readme] Installation requirement for scoring backends: [evidence span withheld — anchor status: external_doc]
