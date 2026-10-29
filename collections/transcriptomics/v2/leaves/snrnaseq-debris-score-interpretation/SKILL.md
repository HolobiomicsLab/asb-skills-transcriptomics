---
name: snrnaseq-debris-score-interpretation
description: Use when DIEM debris scores have been computed for a droplet-based snRNA-seq
  sample and you must verify that the score actually quantifies extranuclear (ambient/spliced)
  RNA contamination rather than just count depth.
when_to_use_negative:
- Input is single-cell (not single-nucleus) RNA-seq data such as PBMCs — the paper
  reports this contamination tracking was 'less successful in the PBMC single-cell
  RNA-seq data' because background and cell type profiles differ less
- The sample has few or no genes increased in the background/debris distribution,
  so debris scores would be inaccurate regardless of correlation testing
- You intend to compare absolute debris score values across multiple independent samples
  — scores are within-sample normalized and not comparable, so a cross-sample correlation
  of raw scores is not meaningful
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: diem
  role: R package whose estimate_dbr_score function produces the per-droplet debris
    scores being validated; SCE object holds score.debris alongside per-droplet features
  repo: https://github.com/marcalva/diem
- name: R
  role: Environment for aligning droplet vectors on barcode IDs and computing the
    Pearson correlation between debris score and percent spliced reads
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:21e86626bcc3f45b5934c5a1b6bddfe44058e797cfa540d2eb7cee4418bd8fc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - R
  - diem
  - stats
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# snrnaseq-debris-score-interpretation

## Summary

Validates the biological meaning of DIEM's per-droplet debris score by correlating it with an independent measure of extranuclear RNA contamination — the percent of reads that are spliced — per droplet in snRNA-seq data (mean Pearson R = 0.89 across 8 snRNA-seq experiments).

## When to use

Use when DIEM debris scores have been computed for a droplet-based snRNA-seq sample and you must verify that the score actually quantifies extranuclear (ambient/spliced) RNA contamination rather than just count depth.

## When NOT to use

- Input is single-cell (not single-nucleus) RNA-seq data such as PBMCs — the paper reports this contamination tracking was 'less successful in the PBMC single-cell RNA-seq data' because background and cell type profiles differ less
- The sample has few or no genes increased in the background/debris distribution, so debris scores would be inaccurate regardless of correlation testing
- You intend to compare absolute debris score values across multiple independent samples — scores are within-sample normalized and not comparable, so a cross-sample correlation of raw scores is not meaningful

## Inputs

- Per-droplet DIEM debris scores (output of estimate_dbr_score, stored on the SCE object as score.debris)
- Per-droplet percent of reads spliced (from the contamination quantification module)
- Shared droplet/barcode identifiers linking both vectors

## Outputs

- Pearson correlation coefficient between debris score and percent spliced reads
- Validation flag/value written to debris_score_spliced_correlation

## How to apply

Load the per-droplet DIEM debris scores produced by estimate_dbr_score for the sample (e.g., the 10X fresh mouse brain nuclei_2k snRNA-seq run). Separately load the per-droplet percent of reads that are spliced, computed from the alignment (spliced reads are a proxy for mature, extranuclear mRNA contamination in nuclear preparations). Align the two vectors on droplet/barcode identifiers so every droplet has both a debris score and a percent-spliced value, then compute the Pearson correlation in R. Expect a high positive correlation (roughly 0.7–0.99); the paper reports a mean Pearson R of 0.89 across 8 independent snRNA-seq experiments including mouse brain, which is the yardstick for judging that the debris score tracks contamination. Do not compare debris score magnitudes across independent experiments — scores are normalized within-sample and are not assumed to be on the same scale between samples.

## Related tools

- **diem** (R package whose estimate_dbr_score function produces the per-droplet debris scores being validated; SCE object holds score.debris alongside per-droplet features) — https://github.com/marcalva/diem
- **R** (Environment for aligning droplet vectors on barcode IDs and computing the Pearson correlation between debris score and percent spliced reads)

## Examples

```
cor(diem_scores[names(pct_spliced)], pct_spliced, method='pearson')  # per-droplet debris score vs percent spliced reads, aligned on droplet IDs
```

## Evaluation signals

- Pearson correlation between per-droplet debris score and percent spliced reads falls in the expected high positive range (~0.7–0.99), consistent with the paper's mean R = 0.89 across 8 snRNA-seq experiments
- Every droplet in the correlation has both a non-missing debris score and a defined percent-spliced value after ID alignment — no barcode mismatches silently dropped
- Correlation is computed within a single sample; debris scores were not pooled across experiments (which would violate the within-sample normalization assumption)
- Debris score vector is reproducible from the DIEM run (set_debris_test_set with min_counts=10, run_em, assign_clusters, estimate_dbr_score in the documented order)
- Scatter diagnostics (e.g., plot_clust with feat_x='pct.mt', feat_y='score.debris' style plotting) show debris scores rising with contamination proxies rather than with total counts alone

## Limitations

- Debris scores cannot be assumed to be on the same scale in independent experiments, so cross-sample comparisons or integrations require caution, especially if the distribution of extranuclear RNA differs across samples
- The scoring approach relies on accurate estimation of debris-enriched genes; if few or no genes are increased in the background distribution, the resulting debris score will likely be inaccurate
- Correlation of debris score with spliced-read contamination was less successful in PBMC single-cell RNA-seq data, possibly due to smaller differences between background and cell type profiles in scRNA-seq

## Evidence

- [results] The DIEM debris score correlates highly with the percent of spliced reads across all 8 snRNA-seq experiments, mean Pearson R = 0.89: [evidence span withheld — anchor status: not_found]
- [discussion] Debris scores are not on a shared scale across experiments: [evidence span withheld — anchor status: not_found]
- [discussion] Scoring depends on debris-enriched gene estimation: "the scoring approach relies on accurate estimation of debris-enriched genes. If there are little or no genes that are increased in the background distribution, the resulting debris score will likely"
- [discussion] Debris score estimates contamination amount: "Therefore, scoring and filtering individual droplets allow for finer classification as well as quantification of the amount of contamination."
- [readme] estimate_dbr_score produces the debris score in the DIEM workflow: "sce <- run_em(sce)
sce <- assign_clusters(sce)
sce <- estimate_dbr_score(sce)"
- [abstract] Scoring and quantifying contamination in droplets: "a novel approach to quantify contamination and filter droplets in snRNA-seq experiments"
