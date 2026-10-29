---
name: scrna-seq-expression-matrix-processing
description: Use when input is a raw scRNA-Seq count/expression matrix with cell-type
  or malignant/non-malignant annotations and the goal is per-cell signalling entropy
  estimation of differentiation potency or cancer stem-cell phenotypes.
when_to_use_negative:
- Input is already a feature table or reduced representation (e.g. precomputed embeddings
  or marker scores) rather than a gene-by-cell expression matrix — SCENT requires
  the transcriptome integrated with the PPI network
- The analysis goal requires prior biological knowledge such as marker gene expression
  or timepoint ordering — the article's rationale is to avoid feature selection and
  training; trajectory tools like Monocle, MPath, StemID or SLICE may be more appropriate
- Only transcriptomic entropy (as in StemID/SLICE) is desired without network information
  — signalling entropy specifically exploits a transcriptome-connectome correlation
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: SCENT
  role: R package run on the full expression matrix with the bundled PPI network to
    compute normalized signalling entropy rate (SR) per cell, without feature selection
    or training
  repo: https://github.com/aet21/SCENT
- name: CCAT
  role: Companion tool in the SCENT tutorial for estimating differentiation potency
    of single cells
  repo: https://github.com/aet21/SCENT
- name: R
  role: Environment for loading the expression matrix and annotations, installing
    SCENT via devtools, running SignallingEntropy, and exporting CSV outputs
- name: BiocStyle
  role: Bioconductor styling package used in the SCENT tutorial vignette
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:639c264fbef1c23c4bf67dfc0e5bba03ec86e52e7856d8f721b8f0752c5a69c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# scrna-seq-expression-matrix-processing

## Summary

Loading and preparing single-cell RNA-Seq expression matrices (e.g. Tirosh melanoma with 3,256 non-malignant and 1,257 cancer cells; Chu et al. 1,018 hESC/progenitor cells) in R so that SCENT can compute per-cell signalling entropy (SR) scores by integrating expression with a PPI connectome network, without feature selection.

## When to use

Use when input is a raw scRNA-Seq count/expression matrix with cell-type or malignant/non-malignant annotations and the goal is per-cell signalling entropy estimation of differentiation potency or cancer stem-cell phenotypes.

## When NOT to use

- Input is already a feature table or reduced representation (e.g. precomputed embeddings or marker scores) rather than a gene-by-cell expression matrix — SCENT requires the transcriptome integrated with the PPI network
- The analysis goal requires prior biological knowledge such as marker gene expression or timepoint ordering — the article's rationale is to avoid feature selection and training; trajectory tools like Monocle, MPath, StemID or SLICE may be more appropriate
- Only transcriptomic entropy (as in StemID/SLICE) is desired without network information — signalling entropy specifically exploits a transcriptome-connectome correlation

## Inputs

- scRNA-Seq count/expression matrix (genes x cells), e.g. Tirosh melanoma matrix (3,256 non-malignant + 1,257 cancer cells) or Chu et al. matrix (1,018 cells)
- cell annotation table with malignant/non-malignant status and cell-type labels (ECs, CAFs, T cells, B cells, macrophages, NK cells)
- human protein-protein interaction (connectome) network bundled with SCENT

## Outputs

- Per-cell signalling entropy (SR) score table, e.g. tirosh_sr_scores.csv or chu_scent_sr.csv with SR values and class/cell-type labels
- Per-cell-type summary table of mean SR values and group contrasts (cancer vs each non-malignant type; pluripotent vs progenitor)
- Statistical test results (Wilcoxon rank-sum P-values; ROC AUC; Spearman correlation with pluripotency score)

## How to apply

Load the scRNA-Seq count matrix (e.g. the Tirosh melanoma matrix of 4,513 cells annotated as malignant vs non-malignant with cell types ECs, CAFs, T cells, B cells, macrophages, NK cells; or the Chu et al. 1,018-cell hESC/progenitor matrix) into R together with the bundled human protein interaction (connectome) network. Run the SCENT SignallingEntropy routine on the full expression matrix in the context of the PPI network, explicitly without feature selection, marker genes, or prior biological knowledge, to obtain a normalized signalling entropy rate (SR) score for each cell. Decide the contrast structure up front: cancer cells vs each non-malignant cell type (melanoma), or pluripotent hESCs vs non-pluripotent progenitors (Chu), and summarize mean SR per cell type. Validate by ranking non-malignant cell types by mean SR (expect potency-graded ordering with ECs and CAFs at top) and by non-parametric statistics (Wilcoxon rank-sum). Write per-cell SR scores plus labels/summaries to CSV (e.g. tirosh_sr_scores.csv, chu_scent_sr.csv).

## Related tools

- **SCENT** (R package run on the full expression matrix with the bundled PPI network to compute normalized signalling entropy rate (SR) per cell, without feature selection or training) — https://github.com/aet21/SCENT
- **CCAT** (Companion tool in the SCENT tutorial for estimating differentiation potency of single cells) — https://github.com/aet21/SCENT
- **R** (Environment for loading the expression matrix and annotations, installing SCENT via devtools, running SignallingEntropy, and exporting CSV outputs)
- **BiocStyle** (Bioconductor styling package used in the SCENT tutorial vignette)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- Tirosh melanoma: Wilcoxon rank-sum test comparing cancer vs non-malignant cells yields P<1e-500 (Fig. 5b), matching the reported dramatic entropy increase in cancer cells
- Chu et al.: ROC AUC for pluripotent hESC vs non-pluripotent progenitor discrimination equals 0.96, with Spearman correlation of SR vs pluripotency score of 0.91 and Wilcoxon P<1e-300
- Ranking non-malignant melanoma cell types by mean SR reproduces a potency-graded ordering with ECs and CAFs at the top
- SR computation runs on the full expression matrix with the PPI network and no feature selection or training; per-cell SR values are written to the expected CSV (tirosh_sr_scores.csv / chu_scent_sr.csv) with one row per cell

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- SCENT requires integration with a protein interaction network; transcriptomic entropy measures that omit network information are less accurate, but a suitable connectome must be available
- Signalling entropy discriminates pluripotent from multipotent and differentiated cells, but unrelated algorithms such as Monocle or MPath are noted as better suited for other ordering insights
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy; criticality hallmarks (e.g. SI peak before cell-fate transition) remain untested pending high-resolution time-course scRNA-Seq data

## Evidence

- [abstract] Potency estimation without feature selection on full expression matrix: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network"
- [readme] SCENT purpose statement from README: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [discussion] Discrimination without feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [readme] Installation command grounding example invocation: "devtools::install_github("aet21/SCENT")"
- [abstract] Transcriptome-connectome correlation driver: [evidence span withheld — anchor status: not_found]
