---
name: outlier-capping-transcriptomic-values
description: Use when preparing a gene-by-cell-population expression matrix (rows
  = genes, columns = cell populations) for ICELLNET ligand–receptor scoring, and raw
  expression magnitudes would bias interaction scores toward strongly expressed molecules.
when_to_use_negative:
- The expression matrix has already been max-scaled to the 0–10 range with the cap
  applied — rescaling would distort values.
- The molecules of interest are not ligands/receptors feeding ICELLNET communication
  scores, so per-gene max normalization is not the intended preprocessing.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ICELLNET
  role: Framework whose communication-score computation requires the 0–10 max-scaled,
    capped expression values; scaling follows the ICELLNET scRNAseq tutorial data
    layout
  repo: https://github.com/soumelis-lab/ICELLNET
- name: R
  role: Language used to load the expression matrix, compute per-gene maxima (mean
    of top 5% values), scale ×10, cap outliers at 10, and write scaled_expression.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:27d93247d4aa7be3975e743230f3ebd9c7feb13a4fe742f111ca580fb3c2394b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# outlier-capping-transcriptomic-values

## Summary

Scales ligand/receptor gene expression by each gene's maximum (mean of its 5% highest values) to a 0–10 range with a cap at 10, as done in ICELLNET before computing ligand–receptor interaction and global communication scores. This prevents highly expressed molecules (e.g., chemokines) from dominating the communication score and allows detection of low-level molecules (e.g., IL-12, IL-4).

## When to use

Use when preparing a gene-by-cell-population expression matrix (rows = genes, columns = cell populations) for ICELLNET ligand–receptor scoring, and raw expression magnitudes would bias interaction scores toward strongly expressed molecules.

## When NOT to use

- The expression matrix has already been max-scaled to the 0–10 range with the cap applied — rescaling would distort values.
- The molecules of interest are not ligands/receptors feeding ICELLNET communication scores, so per-gene max normalization is not the intended preprocessing.

## Inputs

- Gene expression matrix in R (rows = ligand/receptor genes, columns = cell populations), e.g. from scRNAseq, bulk RNAseq, microarray, or spatial transcriptomics
- List of ligand/receptor genes to be scaled (e.g., from the ICELLNET curated interaction database)

## Outputs

- Scaled expression matrix with all values non-negative and capped at 10 (e.g., scaled_expression.csv)

## How to apply

Load the gene expression matrix in R following the ICELLNET scRNAseq tutorial layout (rows = genes, columns = cell populations). For each ligand/receptor gene, compute its maximum as the mean of the 5% highest expression values for that gene. Divide each gene's expression values by this computed maximum and multiply by 10 so the reference gene reaches a scaled value of 10, then cap any outlier values exceeding 10 at 10 to bound the scaled scores. The rationale is that per-gene max-scaling equalizes molecules so a small number of highly expressed molecules (e.g., chemokines) cannot dominate the communication score, exposing important low-level molecules such as IL-12 and IL-4. Write the result (e.g., scaled_expression.csv) for downstream ICELLNET communication-score computation.

## Related tools

- **ICELLNET** (Framework whose communication-score computation requires the 0–10 max-scaled, capped expression values; scaling follows the ICELLNET scRNAseq tutorial data layout) — https://github.com/soumelis-lab/ICELLNET
- **R** (Language used to load the expression matrix, compute per-gene maxima (mean of top 5% values), scale ×10, cap outliers at 10, and write scaled_expression.csv)

## Examples

```
scaled <- pmin(10 * t(t(expr) / apply(expr, 1, function(x) mean(sort(x, decreasing=TRUE)[1:ceiling(0.05*length(x))]))), 10); write.csv(scaled, "scaled_expression.csv")
```

## Evaluation signals

- scaled_expression.csv exists after the workflow runs
- All values in the scaled matrix are non-negative
- No value in the scaled matrix exceeds the cap of 10
- The reference gene (the one defining the maximum) reaches a scaled value of 10
- Downstream ICELLNET global communication scores are not dominated by a few highly expressed molecules (e.g., chemokines)

## Limitations

- The cap at 10 flattens extreme outliers, so absolute expression magnitude above the per-gene maximum is not preserved in downstream scores.
- This normalization is specific to the ICELLNET communication-score design; ICELLNET covers a curated human ligand–receptor database (~1669 interactions) and is more specific/exhaustive for cytokines than for other interaction classes.
- Version constraints apply: ICELLNET v2.0.0 requires R < 4.1; use ICELLNET v2.2.x for R > 4.1.x.

## Evidence

- [other] scaling workflow: [evidence span withheld — anchor status: not_found]
- [other] scaling rationale: [evidence span withheld — anchor status: not_found]
- [other] cap rule: [evidence span withheld — anchor status: not_found]
- [readme] applicability: "Versatile tool applicable on a wide range of transcriptomic technologies (microarray, bulk RNAseq, scRNAseq, spatial transcriptomics)"
- [methods] tutorial reference: "ICELLNET analysis on scRNAseq data:"
