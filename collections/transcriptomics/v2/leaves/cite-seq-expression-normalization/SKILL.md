---
name: cite-seq-expression-normalization
description: Use when raw CITE-seq or scRNA-seq count matrices (e.g., GEO GSE100866
  CBMC data) must be normalized with log2(1+X) before marker selection or k-NN label
  classification.
when_to_use_negative:
- Input expression data is already normalized (e.g., log-transformed or size-factor-corrected);
  a second log2(1+X) transform would double-compress the data
- The goal is trajectory or spatial placement of unlabeled cells — the article restricts
  marker selection to categorical labels and defers manifold constraints to future
  work
- Cell type classes are not expected to be linearly separable in the reduced marker
  space (the article notes this assumption may not hold in lower dimensions)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGeneFit
  role: Consumes the log2(1+X)-normalized CBMC expression matrix and cell-type labels
    to jointly optimize selection of a marker gene panel, evaluated via k-NN label
    classification
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:87a66f4577cd008d7ae0a6b2aae3287c8cbd95b113ed46bc15dfd674cb8dade5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cite-seq-expression-normalization

## Summary

Normalizes CITE-seq scRNA-seq expression matrices (e.g., the CBMC dataset from GEO accession GSE100866) using a log2(1+X) transform so that downstream marker selection and cell-label classification are comparable across cells. This preprocessing step is the basis for the scGeneFit CBMC marker-panel benchmark reported in Dumitrascu et al. (2021).

## When to use

Use when raw CITE-seq or scRNA-seq count matrices (e.g., GEO GSE100866 CBMC data) must be normalized with log2(1+X) before marker selection or k-NN label classification.

## When NOT to use

- Input expression data is already normalized (e.g., log-transformed or size-factor-corrected); a second log2(1+X) transform would double-compress the data
- The goal is trajectory or spatial placement of unlabeled cells — the article restricts marker selection to categorical labels and defers manifold constraints to future work
- Cell type classes are not expected to be linearly separable in the reduced marker space (the article notes this assumption may not hold in lower dimensions)

## Inputs

- Raw CITE-seq count matrix for the CBMC study (GEO accession GSE100866), cells x genes
- Cell-type labels for each assayed cell (flat or hierarchical partition)

## Outputs

- log2(1+X)-transformed normalized expression matrix
- Marker-set-specific classification error rates from k-NN evaluation on a 30% held-out test split (e.g., CSV with one error entry per marker set)

## How to apply

Download the CITE-seq data (for the CBMC benchmark, GEO accession GSE100866, cord blood mononuclear cells) and normalize the expression matrix with a log2(1+X) transform applied elementwise to the raw count matrix. The normalized matrix, together with the CBMC cell-type labels, is then fed to scGeneFit, which selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods. The output marker sets (scGeneFit-selected, curated, or random) are compared by splitting cells into a 70% training / 30% test partition and training a k-nearest-neighbor classifier on the 70% training split in the reduced marker space. Correctness is judged by holding the normalization consistent across all marker sets so that error rates are directly comparable (reported: 8.05% scGeneFit, 9.90% curated, 30% random).

## Related tools

- **scGeneFit** (Consumes the log2(1+X)-normalized CBMC expression matrix and cell-type labels to jointly optimize selection of a marker gene panel, evaluated via k-NN label classification) — github.com/solevillar/scGeneFit

## Examples

```
X_norm = np.log2(1 + X_raw)  # CBMC GSE100866 counts before scGeneFit
```

## Evaluation signals

- The emitted CSV contains exactly one classification-error entry per marker set (scGeneFit, curated, random)
- The scGeneFit marker set achieves the lowest error, matching the reported ordering and values: 8.05% (scGeneFit) < 9.90% (curated) < 30% (random)
- All marker sets compared are matched in size to the scGeneFit panel and evaluated on the identical 70%/30% train/test cell split
- The normalized matrix values are non-negative and bounded, consistent with an elementwise log2(1+X) transform of raw counts

## Limitations

- A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; future work plans to relax the linear separability assumption
- Marker selection is limited to categorical labeling; placing unlabeled cells on trajectories or in spatial assays via a manifold constraint is only envisioned future work

## Evidence

- [other] log2 normalization precedes scGeneFit marker selection on CBMC data: [evidence span withheld — anchor status: not_found]
- [abstract] scGeneFit jointly optimizes label recovery: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [results] CBMC CITE-seq dataset used in the study: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [discussion] Linear separability assumption caveat: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
