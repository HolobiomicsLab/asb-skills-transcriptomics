---
name: geo-accession-data-retrieval
description: Use when a reproducibility or benchmarking task requires the CBMC CITE-seq
  RNA counts and ADT counts from GEO accession GSE100866, e.g. to test whether ALRA
  imputation improves gene-expression-space consistency with protein distances.
when_to_use_negative:
- The RNA count and ADT count matrices are already available locally in memory or
  on disk — retrieval is unnecessary and would risk re-introducing unfiltered mouse-contaminated
  cells.
- The target dataset has no GEO accession or the accession cannot be confirmed as
  GSE100866 for the CBMC benchmark — a different data-acquisition route is needed.
- The task is to run imputation itself rather than to acquire the benchmark data —
  use the ALRA skill directly on an existing normalized matrix.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: 'Downstream consumer of the retrieved CBMC RNA counts: the downloaded matrix
    is normalized with normalize_data() and imputed with alra() using randomized SVD
    at rank k=25'
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: R package providing the randomized SVD that ALRA requires; must be installed
    before running the imputation on the retrieved data
  repo: https://github.com/KlugerLab/ALRA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:ffc0cec39009ac630e5a9565d385b080e7f6002f349fddc0fcb5c1af1994c071
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - rsvd
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# geo-accession-data-retrieval

## Summary

Retrieves raw count matrices (RNA and ADT/protein modalities) for the CITE-seq cord blood mononuclear cell (CBMC) benchmark from GEO accession GSE100866, providing the paired RNA/protein inputs required to evaluate ALRA imputation against protein-space distances.

## When to use

Use when a reproducibility or benchmarking task requires the CBMC CITE-seq RNA counts and ADT counts from GEO accession GSE100866, e.g. to test whether ALRA imputation improves gene-expression-space consistency with protein distances.

## When NOT to use

- The RNA count and ADT count matrices are already available locally in memory or on disk — retrieval is unnecessary and would risk re-introducing unfiltered mouse-contaminated cells.
- The target dataset has no GEO accession or the accession cannot be confirmed as GSE100866 for the CBMC benchmark — a different data-acquisition route is needed.
- The task is to run imputation itself rather than to acquire the benchmark data — use the ALRA skill directly on an existing normalized matrix.

## Inputs

- GEO accession ID for the CBMC CITE-seq study (GSE100866)
- GEO supplementary files containing the RNA count matrix and the ADT (protein) count matrix

## Outputs

- RNA count matrix (8005 cells x 20,400 genes) with mouse-contaminated cells (>90% mouse UMI) removed
- Paired ADT protein count matrix for the same cells
- Cell-by-gene orientation suitable for ALRA (cells as rows, genes as columns)

## How to apply

Locate the GEO accession (GSE100866) cited by the ALRA paper for the CBMC dataset and download the supplementary RNA count and ADT count matrices from the GEO FTP/supplementary files. Filter cells in which more than 90% of UMIs come from mouse genes, yielding the RNA count matrix of 8005 cells x 20,400 genes plus the paired ADT count matrix. Ensure the RNA matrix is oriented with cells as rows and genes as columns, since ALRA requires 'a matrix where the cells are rows and genes are columns' before library and log normalization via normalize_data() and rank-k (k=25 for CBMC) randomized SVD imputation.

## Related tools

- **ALRA** (Downstream consumer of the retrieved CBMC RNA counts: the downloaded matrix is normalized with normalize_data() and imputed with alra() using randomized SVD at rank k=25) — https://github.com/KlugerLab/ALRA
- **rsvd** (R package providing the randomized SVD that ALRA requires; must be installed before running the imputation on the retrieved data) — https://github.com/KlugerLab/ALRA

## Evaluation signals

- The retrieved RNA count matrix has dimensions 8005 cells x 20,400 genes after filtering cells with >90% of UMIs from mouse genes.
- Each cell in the RNA matrix has a matching ADT measurement so pairwise cell distances can be computed in both gene-expression and protein space.
- The matrix orientation is cells-as-rows and genes-as-columns, matching ALRA's required input layout.
- Spearman correlation between normalized gene-expression distances and ADT distances after ALRA imputation exceeds the raw-data value, consistent with the reported improvement from ~0.36 to ~0.64 in cbmc_distance_correlations.csv.

## Limitations

- The reported ALRA improvements were obtained on the full CBMC dataset; results may differ on subsets (the README notes results 'improve when using the entire dataset').
- The mouse-cell filter threshold (>90% mouse UMI) and rank choice (k=25, 'as reported for CBMC') are dataset-specific; other GEO datasets require their own choices, e.g. via choose_k().
- Pre-compiled MKL-backed speedups (rpca-mkl) are only available for Linux and OS X, so retrieval-then-imputation pipelines on Windows fall back to the slower default rpca path.

## Evidence

- [other] Download the CBMC CITE-seq dataset (RNA counts and ADT counts) from GEO accession GSE100866.: [evidence span withheld — anchor status: not_found]
- [other] Filter cells for which more than 90% of UMIs come from mouse genes, yielding the RNA count matrix (8005 cells x 20,400 genes) and the ADT count matrix.: [evidence span withheld — anchor status: not_found]
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`.: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
