---
name: cross-metric-droplet-alignment
description: Use when you have per-droplet DIEM debris scores from an snRNA-seq run
  and a per-droplet contamination proxy (e.g., percent spliced reads) and need to
  verify the debris score tracks extranuclear RNA contamination.
when_to_use_negative:
- Data are single-cell RNA-seq (e.g., PBMC scRNA-seq), where the correlation between
  debris score and spliced-read percent was reported as less successful due to smaller
  differences between background and cell type profiles
- Combining debris scores across independent samples for a single pooled correlation,
  since debris scores cannot be assumed to be on the same scale across experiments
- The background distribution has few or no genes increased relative to cell types,
  making the debris score unreliable as the correlation input
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: diem
  role: R package whose estimate_dbr_score function produces the per-droplet debris
    scores being correlated against percent spliced reads
  repo: https://github.com/marcalva/diem
- name: R
  role: Environment used to align barcodes and compute the Pearson correlation between
    the two per-droplet metric vectors
- name: stats
  role: R stats package (cor function) used to compute the Pearson correlation coefficient
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:88807bfaa56c5e509723aff02990b3b9073310c7134ab6392b8bc7f301e9a30e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - R
  - diem
  - stats
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-metric-droplet-alignment

## Summary

Aligns per-droplet DIEM debris scores with an independent contamination metric (percent of spliced reads) on shared droplet barcodes and computes their Pearson correlation in R, validating that DIEM's debris score quantifies extranuclear RNA contamination.

## When to use

Use when you have per-droplet DIEM debris scores from an snRNA-seq run and a per-droplet contamination proxy (e.g., percent spliced reads) and need to verify the debris score tracks extranuclear RNA contamination.

## When NOT to use

- Data are single-cell RNA-seq (e.g., PBMC scRNA-seq), where the correlation between debris score and spliced-read percent was reported as less successful due to smaller differences between background and cell type profiles
- Combining debris scores across independent samples for a single pooled correlation, since debris scores cannot be assumed to be on the same scale across experiments
- The background distribution has few or no genes increased relative to cell types, making the debris score unreliable as the correlation input

## Inputs

- Per-droplet DIEM debris scores (output of estimate_dbr_score on the SCE object from the mouse brain DIEM run)
- Per-droplet percent of reads spliced (output of the contamination module, keyed by droplet barcode)
- Droplet barcode identifiers shared between both metrics for alignment

## Outputs

- Pearson correlation coefficient between per-droplet debris score and percent spliced reads (e.g., stored as debris_score_spliced_correlation)
- Barcode-aligned two-column table of (debris score, percent spliced) per droplet

## How to apply

Load the per-droplet debris scores produced by the DIEM run's `estimate_dbr_score` step (task_001 output) and the per-droplet percent-spliced values from the contamination module (task_004 output). Align the two vectors by droplet/barcode identifier so every droplet has both a debris score and a percent-spliced value, dropping barcodes present in only one metric. Compute the Pearson correlation coefficient between the paired vectors in R (e.g., `stats::cor(method = "pearson")`). Validate that the correlation falls in the expected high positive range (~0.7–0.99), consistent with the paper's reported mean Pearson R of 0.89 across 8 snRNA-seq experiments, and record it (e.g., write to debris_score_spliced_correlation). Rationale: DIEM scores droplets on expression of debris-enriched genes; spliced reads are a marker of ambient/extranuclear RNA, so high correlation demonstrates the debris score captures true contamination rather than count depth.

## Related tools

- **diem** (R package whose estimate_dbr_score function produces the per-droplet debris scores being correlated against percent spliced reads) — https://github.com/marcalva/diem
- **R** (Environment used to align barcodes and compute the Pearson correlation between the two per-droplet metric vectors)
- **stats** (R stats package (cor function) used to compute the Pearson correlation coefficient)

## Examples

```
cor(droplet_metrics$score.debris, droplet_metrics$pct_spliced, method = "pearson", use = "complete.obs")
```

## Evaluation signals

- Pearson R falls in the expected high positive range (~0.7–0.99), consistent with the paper's reported mean R = 0.89 across the 8 snRNA-seq experiments
- Every droplet in the correlation input has both a debris score and a percent-spliced value (no NaNs introduced by barcode misalignment)
- The correlation is computed on raw per-droplet values, not on experiment-level summaries, preserving within-sample pairing
- Resulting correlation value is written to the debris_score_spliced_correlation output with the barcode counts used documented

## Limitations

- Debris scores cannot be assumed to exist on the same scale in independent experiments, so correlations should be computed within a single sample rather than pooled across samples
- The correlation between debris score and spliced-read percent was less successful in PBMC scRNA-seq data, likely because background and cell type profiles differ less in scRNA-seq
- The debris score relies on accurate estimation of debris-enriched genes; if few or no genes are increased in the background distribution, the resulting debris score (and thus the correlation) will likely be inaccurate

## Evidence

- [results] with a mean Pearson R of 0.89: [evidence span withheld — anchor status: not_found]
- [readme] estimate_dbr_score produces per-droplet debris scores: "sce <- estimate_dbr_score(sce)"
- [discussion] debris score threshold and scoring rationale: "Therefore, scoring and filtering individual droplets allow for finer classification as well as quantification of the amount of contamination."
- [discussion] debris scores not comparable across experiments: [evidence span withheld — anchor status: not_found]
- [discussion] PBMC scRNA-seq correlation less successful: "this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq."
