---
name: pearson-correlation-significance-testing
description: Use when you need to quantify and significance-test the association between
  two per-cell score vectors (e.g., AXL-high vs MITF-high program scores in scRNA-seq)
  and compare that association before vs after confounder correction.
when_to_use_negative:
- The score vectors are not paired per cell (correlation requires matched cells across
  the two vectors)
- You need monotonic-but-nonlinear association — use Spearman/Kendall rather than
  Pearson, which the paper's P-value is not defined for
- Input matrices are not on the recommended scale (unscaled/uncentered/linear-transformed),
  which changes average-expression program scores and hence R
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Produces the corrected expression matrix (projects gene expression to a circular
    manifold and removes cell-cycle effects) whose program-score correlation is compared
    against the uncorrected baseline
  repo: https://github.com/KChen-lab/cyclum
- name: ccRemover
  role: Alternative linear cell-cycle-correction method used to generate comparison
    score vectors for the Supplementary Fig. 7 contrast
- name: Seurat
  role: Alternative cell-cycle-correction method used to generate comparison score
    vectors for the Supplementary Fig. 7 contrast
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:609fab1c07d62e3f1f62e87dd5d50cb5abed242424bc80608366d20c24ced778
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/cyclum
  tools:
  - Cyclum
  - Seurat
  - ccRemover
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pearson-correlation-significance-testing

## Summary

Compute and significance-test Pearson correlations between per-cell gene-program scores (e.g., AXL and MITF in Mel78 melanoma cells) on uncorrected and cell-cycle-corrected expression matrices, to quantify how a confounder correction (Cyclum) changes the relationship between programs.

## When to use

Use when you need to quantify and significance-test the association between two per-cell score vectors (e.g., AXL-high vs MITF-high program scores in scRNA-seq) and compare that association before vs after confounder correction.

## When NOT to use

- The score vectors are not paired per cell (correlation requires matched cells across the two vectors)
- You need monotonic-but-nonlinear association — use Spearman/Kendall rather than Pearson, which the paper's P-value is not defined for
- Input matrices are not on the recommended scale (unscaled/uncentered/linear-transformed), which changes average-expression program scores and hence R

## Inputs

- Per-cell program score vectors (one per gene set, e.g., AXL and MITF average-expression scores) from uncorrected Mel78 scRNA-seq matrix
- Matching score vectors computed on Cyclum/ccRemover/Seurat cell-cycle-corrected matrices
- Per-condition condition labels (uncorrected, Cyclum, ccRemover, Seurat)

## Outputs

- Per-condition Pearson R values and P-values (e.g., CSV such as mel78_axl_mitf_correlations.csv)
- A before/after-correction contrast table showing the correlation shift induced by cell-cycle removal

## How to apply

Compute per-cell program scores as the average log2(x+1) expression of each published gene set (AXL, MITF) on the preprocessed matrix — the article recommends scaled (variance = 1), centered (mean = 1), log2(x+1)-transformed data. Compute Pearson R and its P-value between the two score vectors for each condition: uncorrected, Cyclum-corrected (circular-manifold projection removes cell-cycle effects), and optionally ccRemover- and Seurat-corrected for comparison. Rationale: the correction reveals the expected mutual exclusivity that cell-cycle-driven co-variation masks, so the uncorrected correlation (Mel78: R = -0.02, P = 0.81) versus the Cyclum-corrected correlation (R = -0.43, P = 9e-7) is the key contrast to report. Write per-condition R and P values to a single table (e.g., mel78_axl_mitf_correlations.csv) so the correction effect is directly auditable.

## Related tools

- **Cyclum** (Produces the corrected expression matrix (projects gene expression to a circular manifold and removes cell-cycle effects) whose program-score correlation is compared against the uncorrected baseline) — https://github.com/KChen-lab/cyclum
- **ccRemover** (Alternative linear cell-cycle-correction method used to generate comparison score vectors for the Supplementary Fig. 7 contrast)
- **Seurat** (Alternative cell-cycle-correction method used to generate comparison score vectors for the Supplementary Fig. 7 contrast)

## Examples

```
from scipy.stats import pearsonr; R, P = pearsonr(axl_scores_corrected, mitf_scores_corrected)  # expect R=-0.43, P=9e-7 for Cyclum-corrected Mel78
```

## Evaluation signals

- Uncorrected Pearson correlation reproduces the reported Mel78 value R = -0.02 with P = 0.81 within numerical tolerance
- Cyclum-corrected correlation reproduces R = -0.43 with P = 9e-7, a clearly negative (mutually exclusive) association
- P-values are significant only for the corrected conditions, consistent with Supplementary Fig. 7 (uncorrected null, Cyclum significant)
- Output table contains exactly one R and one P per condition (uncorrected, Cyclum, ccRemover, Seurat) with no missing cells
- Program scores are computed as average expression of the published gene sets on the log2(x+1), centered, variance-1-scaled matrix — recompute scores if the transform differs

## Limitations

- The observed shift in R depends on the correction method — Cyclum models circular manifolds, whereas linear-representation corrections (scLVM, Seurat, ccRemover) may leave residual periodic effects, so R values are not directly interchangeable across methods
- Pearson correlation is sensitive to the data transform; the paper specifies scaled (variance = 1), centered (mean = 1), log2(x+1) preprocessing as the precondition for these values
- Cyclum is designed for periodic-process effects; applying it (and hence this correlation contrast) to non-periodic confounders is outside the validated scope

## Evidence

- [methods] Preprocessing recommendation: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [methods] Cyclum correction scope: "It provides methods to recover cell cycle information and remove cell cycle effects from the scRNA-seq data."
- [intro] Linear-correction alternatives: "The cell cycle regression approaches implemented in scLVM17, Seurat18, and ccRemover19 are based on linear representations"
- [methods] Circular manifold rationale: "The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes."
- [discussion] Implementation note: "The Cyclum package is efficiently implemented in Python using Keras with TensorFlow and has been comprehensively tested."
