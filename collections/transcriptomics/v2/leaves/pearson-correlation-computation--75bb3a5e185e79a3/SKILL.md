---
name: pearson-correlation-computation
description: Use when you have two per-droplet numeric metrics (e.g., DIEM score.debris
  and percent spliced reads) aligned on shared droplet/barcode identifiers and need
  to quantify their linear agreement, expecting a high positive R (~0.7-0.99).
when_to_use_negative:
- Applying to scRNA-seq data (e.g., PBMC single-cell data), where the debris-score
  vs. percent-spliced correlation was reported as less successful due to smaller differences
  between background and cell type profiles
- Comparing debris scores across independent experiments, since debris scores are
  normalized relative to within-sample cluster means and 'cannot be assumed to exist
  on the same scale in independent experiments'
- Using this correlation as evidence of quality when the debris-enriched gene estimation
  failed, since scores will likely be inaccurate if no genes are increased in the
  background distribution
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Computes the Pearson correlation between the aligned per-droplet debris score
    and percent spliced reads vectors
- name: diem
  role: Produces the per-droplet debris score (score.debris) via estimate_dbr_score
    that is one side of the correlation
  repo: https://github.com/marcalva/diem
- name: stats
  role: R stats functions (cor/cor.test) used to compute the Pearson coefficient
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:74083d4ec511552f9a5efb8b6f743ac65bcd876146effc8c63d4def1a4b336f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - R
  - diem
  - stats
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pearson-correlation-computation

## Summary

Computes a Pearson correlation coefficient between two per-droplet numeric vectors — specifically the DIEM debris score (from estimate_dbr_score) and percent spliced reads — to quantify how well DIEM's contamination estimate tracks extranuclear RNA contamination in snRNA-seq.

## When to use

Use when you have two per-droplet numeric metrics (e.g., DIEM score.debris and percent spliced reads) aligned on shared droplet/barcode identifiers and need to quantify their linear agreement, expecting a high positive R (~0.7-0.99).

## When NOT to use

- Applying to scRNA-seq data (e.g., PBMC single-cell data), where the debris-score vs. percent-spliced correlation was reported as less successful due to smaller differences between background and cell type profiles
- Comparing debris scores across independent experiments, since debris scores are normalized relative to within-sample cluster means and 'cannot be assumed to exist on the same scale in independent experiments'
- Using this correlation as evidence of quality when the debris-enriched gene estimation failed, since scores will likely be inaccurate if no genes are increased in the background distribution

## Inputs

- Per-droplet DIEM debris scores (score.debris) from the DIEM run, e.g., mouse brain nuclei_2k 10X data processed with estimate_dbr_score
- Per-droplet percent spliced reads values from the contamination module
- Droplet/barcode identifier mapping to align the two vectors

## Outputs

- Pearson correlation coefficient (e.g., written to debris_score_spliced_correlation), expected in the ~0.7-0.99 range for snRNA-seq

## How to apply

Load the per-droplet debris scores from the DIEM run output (computed via estimate_dbr_score on the SCE object, e.g., for the 10X fresh mouse brain nuclei_2k data), and the per-droplet percent spliced reads from the contamination module. Align both vectors on the droplet identifiers so each droplet/barcode has both values, dropping any barcode missing one measurement. In R, compute the Pearson correlation (e.g., with cor or cor.test, method = 'pearson') between the two vectors. Validate that the resulting R falls in the expected high positive range (approximately 0.7-0.99, consistent with the paper's reported mean R = 0.89 across the 8 snRNA-seq experiments, including the mouse brain sample). Record the correlation value (e.g., to debris_score_spliced_correlation) alongside the alignment method used.

## Related tools

- **R** (Computes the Pearson correlation between the aligned per-droplet debris score and percent spliced reads vectors)
- **diem** (Produces the per-droplet debris score (score.debris) via estimate_dbr_score that is one side of the correlation) — https://github.com/marcalva/diem
- **stats** (R stats functions (cor/cor.test) used to compute the Pearson coefficient)

## Examples

```
cor(droplet_data$score.debris, droplet_data$pct_spliced, method = "pearson")
```

## Evaluation signals

- The Pearson R falls in the expected high positive range (~0.7-0.99), consistent with the paper's mean R = 0.89 across 8 snRNA-seq experiments
- The two vectors align 1:1 on droplet identifiers — every paired droplet has both a debris score and a percent-spliced value, with no silent NA coercion
- The correlation is strongly positive in sign; a near-zero or negative R indicates misalignment or a failed debris score estimation
- The reported value matches what cor(...) returns when re-run independently on the same aligned inputs (reproducibility check)

## Limitations

- The correlation was less successful in PBMC scRNA-seq data, possibly 'due to smaller differences between the background and cell type profiles in scRNA-seq'
- Debris scores 'cannot be assumed to exist on the same scale in independent experiments', so a correlation computed in one sample does not transfer across samples
- The scoring approach relies on accurate estimation of debris-enriched genes; 'if there are little or no genes that are increased in the background distribution, the resulting debris score will likely' be inaccurate

## Evidence

- [other] The DIEM debris score correlates highly with the percent of spliced reads across all 8 independent snRNA-seq experiments: [evidence span withheld — anchor status: not_found]
- [methods] We estimate the debris score by ruunning the function `estimate_dbr_score`: "We estimate the debris score by ruunning the function `estimate_dbr_score`"
- [discussion] this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq.: "this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq."
- [discussion] the debris score, however, cannot be assumed to exist on the same scale in independent experiments.: [evidence span withheld — anchor status: not_found]
- [readme] sce <- estimate_dbr_score(sce): "sce <- estimate_dbr_score(sce)"
