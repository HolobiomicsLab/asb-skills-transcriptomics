---
name: cell-type-potency-ranking-analysis
description: Use when a scRNA-Seq count matrix with malignant/non-malignant or cell-type
  annotations is available and the goal is to estimate and rank differentiation potency
  of cell types without marker genes, feature selection, or training.
when_to_use_negative:
- The task requires reconstructing developmental trajectories or lineage ordering
  by expression dynamics — the article notes this insight is not readily obtainable
  using other algorithms such as Monocle or MPath, but SCENT provides potency ordering,
  not Monocle-style trajectory fitting.
- Only network-free transcriptomic entropy is desired (e.g. StemID/SLICE-style measures)
  — signalling entropy's advantage is driven by a positive correlation between the
  transcriptome and the connectome, so it requires an interaction network.
- Marker-gene-based cell identification is sufficient and dropout is not a concern
  — SCENT's value is potency estimation without prior biological knowledge such as
  marker expression.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCENT
  role: Core package; its SignallingEntropy routine computes per-cell SR scores against
    an interaction network to estimate differentiation potency
  repo: https://github.com/aet21/SCENT
- name: R
  role: Runtime environment for loading count matrices and running SCENT workflows
- name: BiocStyle
  role: Bioconductor style used in the SCENT/CCAT tutorial vignette documenting the
    workflow
- name: CCAT
  role: Companion tool presented in the SCENT tutorial for estimating differentiation
    potency of single cells
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:a57b9e033951978a307b692ff687c9d38a2fd53369495a45f1765550fd29e6a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cell-type-potency-ranking-analysis

## Summary

Compute per-cell signalling entropy (SR) with SCENT's SignallingEntropy routine over a protein interaction network to rank cell types by differentiation potency, e.g. reproducing the Tirosh melanoma result where cancer cells show dramatically higher SR than non-malignant tumour-microenvironment cells (Wilcoxon rank-sum P<1e-500).

## When to use

Use when a scRNA-Seq count matrix with malignant/non-malignant or cell-type annotations is available and the goal is to estimate and rank differentiation potency of cell types without marker genes, feature selection, or training.

## When NOT to use

- The task requires reconstructing developmental trajectories or lineage ordering by expression dynamics — the article notes this insight is not readily obtainable using other algorithms such as Monocle or MPath, but SCENT provides potency ordering, not Monocle-style trajectory fitting.
- Only network-free transcriptomic entropy is desired (e.g. StemID/SLICE-style measures) — signalling entropy's advantage is driven by a positive correlation between the transcriptome and the connectome, so it requires an interaction network.
- Marker-gene-based cell identification is sufficient and dropout is not a concern — SCENT's value is potency estimation without prior biological knowledge such as marker expression.

## Inputs

- scRNA-Seq count matrix with per-cell annotations (malignant/non-malignant label and cell type), e.g. Tirosh melanoma data (3,256 non-malignant + 1,257 cancer cells)
- bundled protein interaction network shipped with the SCENT package

## Outputs

- per-cell SR (signalling entropy) score table, e.g. tirosh_sr_scores.csv
- per-cell-type summary of mean SR with cancer-vs-non-malignant contrasts and Wilcoxon rank-sum P-values

## How to apply

Load the scRNA-Seq count matrix (e.g. Tirosh melanoma: 3,256 non-malignant and 1,257 cancer cells from 12 patients) with malignant/non-malignant and cell-type annotations into R. Install SCENT from GitHub (aet21/SCENT) and run the SignallingEntropy routine on the full expression matrix in the context of the bundled protein interaction network, computing an SR score for each cell (4,513 cells in the melanoma case) without feature selection or training — the method's rationale is that potency is approximated by signalling promiscuity (entropy) of a cell's transcriptome against the network. Summarize mean SR per cell type, contrast cancer cells against each non-malignant type (ECs, CAFs, T cells, B cells, macrophages, NK cells) using a Wilcoxon rank-sum test, and verify the potency-graded ordering (ECs and CAFs at the top). Export per-cell SR scores and a per-cell-type summary table (e.g. tirosh_sr_scores.csv).

## Related tools

- **SCENT** (Core package; its SignallingEntropy routine computes per-cell SR scores against an interaction network to estimate differentiation potency) — https://github.com/aet21/SCENT
- **R** (Runtime environment for loading count matrices and running SCENT workflows)
- **BiocStyle** (Bioconductor style used in the SCENT/CCAT tutorial vignette documenting the workflow)
- **CCAT** (Companion tool presented in the SCENT tutorial for estimating differentiation potency of single cells)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT"); sr <- SCENT::SignallingEntropy(expData = tirosh_counts, ppi = SCENT_ppi); write.csv(sr, "tirosh_sr_scores.csv")
```

## Evaluation signals

- Cancer cells show dramatically higher mean SR than every non-malignant cell type, with Wilcoxon rank-sum P<1e-500 (Tirosh melanoma benchmark).
- Non-malignant cell types rank in a potency-graded order with ECs and CAFs at the top of mean SR.
- SR scores are produced for all cells (4,513 in the melanoma case) with no feature selection or training step applied.
- Per-cell SR scores and per-cell-type summary are written to tirosh_sr_scores.csv with matching cell counts (3,256 non-malignant, 1,257 cancer).

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established.
- Criticality hallmarks, such as an SI peak before a cell-fate transition, remain untested pending high-resolution time-course and genome-wide scRNA-Seq data.
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy.

## Evidence

- [abstract] differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network"
- [discussion] signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [methods] A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT: "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
- [results] Pooling single-cell signalling entropy across all 12 melanoma patients shows a dramatic increase in entropy for cancer cells versus non-malignant cells, which is highly statistically significant (Wilcoxon rank-sum test P<1e−500: [evidence span withheld — anchor status: not_found]
- [readme] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
