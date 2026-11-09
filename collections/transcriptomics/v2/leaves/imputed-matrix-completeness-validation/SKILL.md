---
name: imputed-matrix-completeness-validation
description: Use when a sparse spot-by-poly(A) site RUD matrix has been imputed and
  you must confirm no missing entries remain and that imputed values preserve the
  known spatial structure (e.g., MOB layers).
when_to_use_negative:
- The APA ratio matrix is already dense with no missing RUD entries — imputation and
  completeness checking add nothing and risk distorting measured values.
- 'Input is image-based ST data without spatial barcodes (e.g., MERFISH/seqFISH):
  poly(A) sites cannot be identified with existing scRNA-seq tools, so no RUD matrix
  exists to validate.'
- You need statistically principled handling of truly-missing versus zero APA usage
  — the article notes no method exists to determine whether an entry is truly missing
  before imputation.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: stAPAminer
  role: Provides computeAPAIndex (raw RUD), imputeAPAIndex (KNN imputation, k=10,
    init=TRUE), optimalKvalue, createStAPAminerObject/makeStCluster for clustering
    validation, and computePearsonIndex/computeIndex/drawVolcano for correlation-based
    verification
  repo: https://github.com/BMILAB/stAPAminer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:52d92f0f3c6970d46d5f86ed35df4373df361de7138d03654412fbe62b9e0987
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# imputed-matrix-completeness-validation

## Summary

Validates that the KNN-imputed APA usage (RUD) matrix from stAPAminer is complete and trustworthy: every missing entry is filled by borrowing from k=10 expression-nearest spots, leftovers set to 0, and correctness checked by clustering and Pearson correlation against spatial layers.

## When to use

Use when a sparse spot-by-poly(A) site RUD matrix has been imputed and you must confirm no missing entries remain and that imputed values preserve the known spatial structure (e.g., MOB layers).

## When NOT to use

- The APA ratio matrix is already dense with no missing RUD entries — imputation and completeness checking add nothing and risk distorting measured values.
- Input is image-based ST data without spatial barcodes (e.g., MERFISH/seqFISH): poly(A) sites cannot be identified with existing scRNA-seq tools, so no RUD matrix exists to validate.
- You need statistically principled handling of truly-missing versus zero APA usage — the article notes no method exists to determine whether an entry is truly missing before imputation.

## Inputs

- Raw sparse RUD matrix from computeAPAIndex (spot x poly(A) site APA usage ratios)
- Gene-spot expression count matrix (count.csv)
- Spot barcode and coordinate table (position.txt)
- APA-spot count matrix stored as a movAPA PACdataset (APA.RDA)

## Outputs

- Complete imputed RUD matrix with no missing entries (written e.g. as RUD_imputed.csv)
- Cluster labels over spots (e.g., GCL, GL, ONL, OPL, MCL) for spatial validation
- Pearson-correlation-based imputation quality indices and volcano comparison (e.g., GCL vs GL)

## How to apply

First confirm the raw RUD matrix from computeAPAIndex(APA, rownames(position)) is actually sparse, then run imputeAPAIndex(RUD_RAW, count, k=10, init=TRUE), which scales the gene-spot count matrix, ranks spot-spot distances by expression similarity, and iteratively averages non-zero RUD values of the 10 nearest spots until no new entries can be filled. After the full iteration, any entries still missing are explicitly set to 0 per the package convention, so verify the output matrix has no NA/NaN cells before downstream use. Judge correctness biologically by clustering the imputed matrix (makeStCluster on a createStAPAminerObject, resolution=0.6, dims 1:10, k=30) and checking that recovered labels match known MOB morphological layers (GCL, GL, ONL, OPL, MCL), and quantitatively via computePearsonIndex/computeIndex comparisons between imputed RUD and expression-derived indices, using drawVolcano (e.g., GCL vs GL) to inspect consistency. Optionally select optimal k with optimalKvalue(RUD_RAW, count, position, ncluster=5, orders=c(seq(4,14,2))) rather than defaulting to k=10.

## Related tools

- **stAPAminer** (Provides computeAPAIndex (raw RUD), imputeAPAIndex (KNN imputation, k=10, init=TRUE), optimalKvalue, createStAPAminerObject/makeStCluster for clustering validation, and computePearsonIndex/computeIndex/drawVolcano for correlation-based verification) — https://github.com/BMILAB/stAPAminer

## Examples

```
RUD_RAW <- computeAPAIndex(APA, rownames(position)); RUD <- imputeAPAIndex(RUD_RAW, count, k=10, init=TRUE)
```

## Evaluation signals

- No NA/NaN entries remain in the imputed RUD matrix after the iterative fill-and-set-to-0 loop completes.
- Clustering the imputed matrix (makeStCluster, resolution=0.6, dims 1:10, k=30) recovers the expected five MOB morphological layers (GCL, GL, ONL, OPL, MCL).
- Pearson correlations between imputed RUD values and expression-based indices (computePearsonIndex/computeIndex) support consistency; volcano plot for a layer pair (e.g., GCL vs GL) shows sensible separation.
- Choice of k is justified: optimalKvalue over orders seq(4,14,2) with ncluster=5 agrees with (or improves on) the default k=10.
- Entries that remained missing after iteration are exactly 0 (not NA), matching the package convention.

## Limitations

- Missing RUD entries that survive all imputation iterations are arbitrarily set to 0, conflating 'unobserved' with 'zero APA usage'.
- No method is provided to determine whether an APA ratio matrix entry is truly missing before imputation.
- The transformation of the APA ratio matrix into a count matrix (e.g., for SPARK-based spatial tests) may not be theoretically reasonable or fully meet model assumptions.
- Discrepancies among replicates are not mitigated by any proposed method, so reproducibility of imputed spatial patterns across MOB replicates must be checked manually.

## Evidence

- [methods] The KNN-based imputation model in stAPAminer leverages the gene expression profile to infer the spot–spot distance and then imputes the sparse APA usage matrix: [evidence span withheld — anchor status: not_found]
- [methods] Iteratively fill missing RUD entries by averaging non-zero RUD values of k=10 nearest spots: [evidence span withheld — anchor status: not_found]
- [discussion] If there are still missing entries after the entire iteration process, we can directly set their values to 0.: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
- [discussion] First, it is crucial to determine whether an entry into the APA ratio matrix is truly missing before imputation.: "First, it is crucial to determine whether an entry into the APA ratio matrix is truly missing before imputation."
- [readme] ##Verify by correlation
pearson_RUD <- computePearsonIndex(stObj,"RUD")
index <- computeIndex(stObj)
drawVolcano(stObj,"GCL","GL"): "##Verify by correlation
pearson_RUD <- computePearsonIndex(stObj,"RUD")"
