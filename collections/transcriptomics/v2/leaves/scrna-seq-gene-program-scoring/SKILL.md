---
name: scrna-seq-gene-program-scoring
description: Use when a preprocessed single-cell expression matrix must be reduced
  to per-cell scores for named gene sets (e.g. AXL-high vs MITF-high resistance programs)
  and correlated to test program relationships.
when_to_use_negative:
- The count matrix has not yet been preprocessed (scaled, centered, log2(x+1)) — scores
  computed on raw counts are not comparable to published program scores
- The analysis goal is cell-cycle phase inference or removal rather than downstream
  program-level phenotyping — use Cyclum's pseudotime/correction functionality directly
  instead
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Cell-cycle correction tool; produces the corrected matrix on which program
    scores are recomputed to reveal program relationships (e.g. AXL-MITF mutual exclusivity)
  repo: https://github.com/KChen-lab/Cyclum
- name: ccRemover
  role: Linear cell-cycle regression baseline used to produce a comparison corrected
    matrix for the Supplementary Fig. 7 program-score correlation comparison
- name: Seurat
  role: Linear cell-cycle regression baseline used to produce an additional corrected
    matrix for the program-score correlation comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:4ac7b3f73f8a36d4b142b75b8517bbd51d997cef5e7fb50fd0a5c9a96f64d652
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - Seurat
  - ccRemover
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# scrna-seq-gene-program-scoring

## Summary

Computes per-cell gene program scores (average expression of a published gene set, e.g. the AXL and MITF drug-resistance signatures) on a preprocessed scRNA-seq matrix, then tests Pearson correlation between program scores to quantify relationships such as mutual exclusivity between cellular states.

## When to use

Use when a preprocessed single-cell expression matrix must be reduced to per-cell scores for named gene sets (e.g. AXL-high vs MITF-high resistance programs) and correlated to test program relationships.

## When NOT to use

- The count matrix has not yet been preprocessed (scaled, centered, log2(x+1)) — scores computed on raw counts are not comparable to published program scores
- The analysis goal is cell-cycle phase inference or removal rather than downstream program-level phenotyping — use Cyclum's pseudotime/correction functionality directly instead

## Inputs

- Preprocessed scRNA-seq expression matrix (scaled variance = 1, centered, log2(x+1) transformed), e.g. from the proliferating melanoma sample Mel78
- Published gene sets defining the programs, e.g. the AXL and MITF gene sets
- Optionally, cell-cycle-corrected versions of the matrix (Cyclum, ccRemover, or Seurat outputs)

## Outputs

- Per-cell program scores for each gene set (e.g. AXL score and MITF score per cell)
- Per-condition Pearson correlation R and P values between program scores (e.g. R = -0.43, P = 9e-7 after Cyclum correction)
- Summary table such as mel78_axl_mitf_correlations.csv with all correlations and p-values

## How to apply

Preprocess the scRNA-seq count matrix using the transform the article recommends: scaled to variance = 1, centered, log2(x+1). For each cell, compute each program score as the average expression of the genes in the published signature (e.g. the AXL gene set and the MITF gene set), repeating the computation on each matrix variant being compared (uncorrected, Cyclum-corrected, ccRemover-corrected, Seurat-corrected). Cyclum correction is appropriate here because it projects gene expression onto a circular manifold to remove cell-cycle effects, unlike linear methods. Compute the Pearson correlation between the two program-score vectors per condition and test its significance, writing all R and P values per condition to a CSV such as mel78_axl_mitf_correlations.csv. Judge correctness by reproducing the reported values: uncorrected R = -0.02 (P = 0.81) and Cyclum-corrected R = -0.43 (P = 9e-7) within numerical tolerance.

## Related tools

- **Cyclum** (Cell-cycle correction tool; produces the corrected matrix on which program scores are recomputed to reveal program relationships (e.g. AXL-MITF mutual exclusivity)) — https://github.com/KChen-lab/Cyclum
- **ccRemover** (Linear cell-cycle regression baseline used to produce a comparison corrected matrix for the Supplementary Fig. 7 program-score correlation comparison)
- **Seurat** (Linear cell-cycle regression baseline used to produce an additional corrected matrix for the program-score correlation comparison)

## Examples

```
import scipy.stats as st; r, p = st.pearsonr(axl_program_scores, mitf_program_scores)
```

## Evaluation signals

- Uncorrected Mel78 matrix reproduces AXL-MITF Pearson R = -0.02 (P = 0.81)
- Cyclum-corrected Mel78 matrix reproduces AXL-MITF Pearson R = -0.43 (P = 9e-7)
- R and P values match the reported Supplementary Fig. 7 values within numerical tolerance
- All per-condition correlations and p-values are present in the output CSV (e.g. mel78_axl_mitf_correlations.csv) for every matrix variant analyzed
- Program scores are computed as average expression of the published gene sets on identically preprocessed matrices across conditions

## Limitations

- Cell-cycle confounding can mask program relationships: on the uncorrected Mel78 matrix the AXL-MITF correlation is near zero (R = -0.02), so skipping correction yields a false negative for mutual exclusivity
- Cyclum requires a specific preprocessing transform (scaled variance = 1, centered, log2(x+1)); other transforms may not be supported
- Cyclum's corrected output depends on autoencoder training and its dependencies (Python 3.7, Keras, TensorFlow 1.x); newer dependency versions may change results

## Evidence

- [methods] Preprocessing uses scaled, centered log transform: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [readme] Cyclum removes cell-cycle effects for downstream analysis: [evidence span withheld — anchor status: external_doc]
- [methods] Cyclum uses a circular manifold rather than marker genes: "The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes."
- [abstract] Cyclum improves subpopulation delineation after correction: "Applying Cyclum to removing cell-cycle effects substantially improves delineations of cell subpopulations"
