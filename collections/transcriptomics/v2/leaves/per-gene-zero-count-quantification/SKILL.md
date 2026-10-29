---
name: per-gene-zero-count-quantification
description: 'Use when validating ALRA imputation output: after thresholding the rank-k
  SVD approximation, to confirm that zeros at biologically non-expressed positions
  are retained and that technical (dropout) zeros have been imputed, on a per-gene
  basis.'
when_to_use_negative:
- The input matrix has not yet been through ALRA's adaptive thresholding step — zero
  counts of a raw low-rank approximation are meaningless because the approximation
  fills all positions with small nonzero values
- The matrix is oriented with genes as rows and cells as columns (ALRA requires cells
  as rows, genes as columns); counting zeros on a transposed matrix would mislabel
  the per-gene statistics
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: ALRA
  role: Provides the thresholded completed matrix whose per-gene zero counts are tabulated;
    the b_nk_example dataset and normalize_data/alra functions define the input state
  repo: github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD backend used by ALRA to compute the rank-k approximation feeding
    the thresholding step whose zeros are being quantified
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:1ce118df53744722253bfd0e30a72be02b70e7cdaf1b88f6a8dc48705af0d243
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALRA
  - rsvd
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# per-gene-zero-count-quantification

## Summary

Quantifies the number of zero entries per gene (row) in a scRNA-seq expression matrix before and after ALRA's adaptive thresholding step, producing a per-gene table (per_gene_zero_counts.csv) used to validate that biological zeros are preserved while technical zeros are completed.

## When to use

Use when validating ALRA imputation output: after thresholding the rank-k SVD approximation, to confirm that zeros at biologically non-expressed positions are retained and that technical (dropout) zeros have been imputed, on a per-gene basis.

## When NOT to use

- The input matrix has not yet been through ALRA's adaptive thresholding step — zero counts of a raw low-rank approximation are meaningless because the approximation fills all positions with small nonzero values
- The matrix is oriented with genes as rows and cells as columns (ALRA requires cells as rows, genes as columns); counting zeros on a transposed matrix would mislabel the per-gene statistics

## Inputs

- Library- and log-normalized scRNA-seq expression matrix (A_norm, cells as rows, genes as columns) with observed zero positions
- ALRA thresholded/completed matrix (A_norm_completed, e.g. alra_thresholded_matrix.csv) with the same genes as rows/columns orientation

## Outputs

- per_gene_zero_counts.csv: one row per gene with zero counts before and after thresholding
- Optional per-cell-type percent-expressing table for marker genes (e.g. NCAM1, CR2) computed via aggregate on A_norm and A_norm_completed

## How to apply

After computing ALRA's rank-k approximation and applying the adaptive threshold — which sets to zero every entry in each gene row smaller in magnitude than that row's most negative value — count, for every gene, the number of zero entries in the original normalized matrix (A_norm) and in the thresholded completed matrix (A_norm_completed). Write both counts side by side to per_gene_zero_counts.csv so per-gene shifts in zero counts are auditable. The expected pattern is that genes with many biological zeros keep approximately those zeros in the completed matrix (entries at biological-zero positions are symmetrically distributed around zero, so the negative-value threshold restores them), while genes with many dropout zeros lose zeros as technical zeros are completed. For a coarse biological sanity check, also compare the percent of cells expressing marker genes across cell types before and after imputation (e.g., NCAM1 in NK cells, CR2 in B cells from the b_nk_example dataset).

## Related tools

- **ALRA** (Provides the thresholded completed matrix whose per-gene zero counts are tabulated; the b_nk_example dataset and normalize_data/alra functions define the input state) — github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD backend used by ALRA to compute the rank-k approximation feeding the thresholding step whose zeros are being quantified)

## Examples

```
library(ALRA); A_norm_completed <- alra(normalize_data(b_nk_example), k=choose_k(A_norm)$k)[[3]]; zero_counts <- data.frame(before=rowSums(A_norm==0), after=rowSums(A_norm_completed==0)); write.csv(zero_counts, 'per_gene_zero_counts.csv')
```

## Evaluation signals

- per_gene_zero_counts.csv has one row per gene and both before/after columns are populated for every gene
- Genes whose zeros are predominantly biological retain near-identical zero counts after thresholding (~97% biological-zero preservation reported on the simulated benchmark)
- Genes with dropout zeros show decreased zero counts (technical zeros completed), e.g. NCAM1 expression in NK cells rises from ~4% to near-100% and CR2 in B cells from ~1%
- Thresholded matrix retains exact zeros at positions that were zero in A_norm at biologically non-expressed loci rather than small positive values
- Any originally non-zero normalized value that fell below the threshold was restored to its original value, so zero counts do not increase for genes with dense expression

## Limitations

- Completion quality is reduced on data subsets — the README notes results improve when using the entire dataset rather than the b_nk_example subset
- The negative-value threshold assumes entries at biological-zero positions are symmetrically distributed around zero; violations of this symmetry (e.g. unusual noise structure) undermine zero restoration
- choose_k caps k at 100 (top 100 singular values via randomized SVD), and a poorly chosen k can leak noise into the thresholded matrix and distort per-gene zero counts

## Evidence

- [readme] Threshold each gene row by the magnitude of its most negative value: "Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene."
- [intro] Symmetric distribution of biological-zero entries around zero: "the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero"
- [intro] Restoring biological zeros via thresholding: "the last step of ALRA is to restore the biological zeros in the matrix by thresholding its entries"
- [readme] Marker-gene completion check in README: "NCAM1 should be expressed in all NK cells, but is only expressed in 4% of of NK cells in the original data."
- [intro] Imputation goal: preserve biological zeros while completing technical ones: "it should accurately impute the data, in particular preserving at zero true biological zeros while completing the technical ones"
