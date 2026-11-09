---
name: melanoma-cancer-vs-normal-cell-entropy-contrast
description: Use when a single-cell RNA-Seq count matrix has malignant vs non-malignant
  (and cell-type) annotations and you need to test whether cancer cells show higher
  differentiation potency/signalling entropy than microenvironment cells, without
  feature selection or training.
when_to_use_negative:
- When the analysis goal requires ordering cells along a trajectory in pseudo-time
  using pseudotime algorithms such as Monocle or MPath — the article notes this potency
  insight is not readily obtainable with those tools, but they serve a different purpose
- When only transcriptomic entropy without network information is acceptable (e.g.
  StemID- or SLICE-style measures) — this skill specifically requires the interaction-network-context
  SR computation
- When the input lacks malignant/non-malignant or cell-type annotations, since the
  contrast and potency-graded ordering cannot be defined
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SCENT
  role: Provides the SignallingEntropy routine that computes per-cell signalling entropy
    (SR) in the context of a protein interaction network, without feature selection
    or training
  repo: https://github.com/aet21/SCENT
- name: R
  role: Language in which the Tirosh count matrix is loaded, SCENT is run, the Wilcoxon
    rank-sum test is performed, and tirosh_sr_scores.csv is written
- name: BiocStyle
  role: Document style used for the SCENT tutorial vignette covering the potency estimation
    workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:8b034805eacdeb04b161d98a358d9302603fae648cef11dbbfec788f8fd54e55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# melanoma-cancer-vs-normal-cell-entropy-contrast

## Summary

Computes per-cell signalling entropy (SR) with the SCENT SignallingEntropy routine on the Tirosh melanoma scRNA-Seq data (3,256 non-malignant and 1,257 cancer cells across 12 patients) and contrasts cancer-cell entropy against each non-malignant tumour-microenvironment cell type using a Wilcoxon rank-sum test.

## When to use

Use when a single-cell RNA-Seq count matrix has malignant vs non-malignant (and cell-type) annotations and you need to test whether cancer cells show higher differentiation potency/signalling entropy than microenvironment cells, without feature selection or training.

## When NOT to use

- When the analysis goal requires ordering cells along a trajectory in pseudo-time using pseudotime algorithms such as Monocle or MPath — the article notes this potency insight is not readily obtainable with those tools, but they serve a different purpose
- When only transcriptomic entropy without network information is acceptable (e.g. StemID- or SLICE-style measures) — this skill specifically requires the interaction-network-context SR computation
- When the input lacks malignant/non-malignant or cell-type annotations, since the contrast and potency-graded ordering cannot be defined

## Inputs

- Tirosh melanoma scRNA-Seq count matrix with 4,513 cells (3,256 non-malignant and 1,257 cancer) and malignant/non-malignant plus cell-type annotations
- Bundled protein interaction network shipped with the SCENT package

## Outputs

- Per-cell SR (signalling entropy) scores for all 4,513 cells
- Per-cell-type mean SR summary table with cancer-vs-non-malignant Wilcoxon rank-sum P-values (expected P<1e-500 for cancer vs non-malignant, Fig. 5b)
- tirosh_sr_scores.csv containing per-cell SR scores and the per-cell-type summary

## How to apply

Load the Tirosh melanoma scRNA-Seq count matrix with malignant/non-malignant and cell-type annotations into R, keeping all 4,513 cells (3,256 non-malignant, 1,257 cancer). Run the SCENT SignallingEntropy routine on the expression matrix in the context of the bundled protein interaction network to compute the SR score per cell, deliberately without feature selection or training, since signalling entropy can discriminate pluripotent from multipotent and differentiated cells in an unbiased manner. Summarize mean SR per cell type (ECs, CAFs, T cells, B cells, macrophages, NK cells, cancer cells) and rank non-malignant types by mean SR to verify the potency-graded ordering with ECs and CAFs at the top. Contrast cancer cells against each non-malignant cell type with a Wilcoxon rank-sum test on the pooled per-cell SR scores across all 12 patients; the expected result is a dramatic entropy increase in cancer cells with P<1e-500. Write per-cell SR scores and per-cell-type summary tables to tirosh_sr_scores.csv.

## Related tools

- **SCENT** (Provides the SignallingEntropy routine that computes per-cell signalling entropy (SR) in the context of a protein interaction network, without feature selection or training) — https://github.com/aet21/SCENT
- **R** (Language in which the Tirosh count matrix is loaded, SCENT is run, the Wilcoxon rank-sum test is performed, and tirosh_sr_scores.csv is written)
- **BiocStyle** (Document style used for the SCENT tutorial vignette covering the potency estimation workflow)

## Examples

```
devtools::install_github("aet21/SCENT"); SR <- SignallingEntropy(expData = tirosh_counts, ppi = SCENT_ppi); wilcox.test(SR[cancer_cells], SR[nonmalignant_cells])
```

## Evaluation signals

- Wilcoxon rank-sum test comparing pooled cancer-cell vs non-malignant-cell SR scores across all 12 melanoma patients yields P<1e-500 (matches Fig. 5b)
- Mean SR is higher for cancer cells than for every non-malignant cell type (ECs, CAFs, T cells, B cells, macrophages, NK cells)
- Non-malignant cell types ordered by mean SR reproduce the potency-graded ordering with ECs and CAFs at the top
- SR scores are computed for all 4,513 cells (3,256 non-malignant + 1,257 cancer) with no feature selection or training step applied
- tirosh_sr_scores.csv exists and contains one SR score per cell plus a per-cell-type summary table

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- Criticality hallmarks such as an SI peak before a cell-fate transition remain untested, pending high-resolution time-course and genome-wide scRNA-Seq data
- In scenarios with high dropout rate a marker gene may preclude direct use, which is precisely why network-based signalling entropy is used instead of marker-based potency assignment

## Evidence

- [results] Pooling single-cell signalling entropy across all 12 melanoma patients shows a dramatic increase in entropy for cancer cells versus non-malignant cells (Wilcoxon P<1e-500): [evidence span withheld — anchor status: not_found]
- [discussion] signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells without feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [readme] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge such as marker expression"
- [readme] devtools::install_github("aet21/SCENT"): "devtools::install_github("aet21/SCENT")"
- [abstract] differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need"
