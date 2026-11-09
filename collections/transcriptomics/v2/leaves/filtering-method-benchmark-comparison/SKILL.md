---
name: filtering-method-benchmark-comparison
description: Use when two or more droplet/background-filtering methods (e.g., DIEM
  and EmptyDrops) must be compared on the same scRNA-seq or snRNA-seq count matrix
  to determine which retains dying/debris-contaminated droplets.
when_to_use_negative:
- When comparing methods across different droplet sets — restrict both methods to
  the identical droplet set (same min_counts) first, or the kept-set differences are
  confounded
- When the goal is filtering a single dataset for downstream analysis rather than
  benchmarking method agreement (use the DIEM filtering skill alone)
- When the debris score itself needs to be interpreted across independent experiments
  — debris scores cannot be assumed to be on the same scale across samples
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DIEM
  role: Debris-cluster filtering method whose kept-droplet set is one arm of the comparison;
    run in debris-cluster removal mode (call_targets with clusters = 'debris') for
    single-cell RNA-seq data
  repo: https://github.com/marcalva/diem
- name: EmptyDrops
  role: Comparison filtering method that estimates a Dirichlet-Multinomial distribution
    of ambient RNA; its kept droplets form the other arm of the comparison
- name: diem
  role: R package providing read_10x, create_SCE, get_gene_pct, set_debris_test_set,
    and the EM pipeline used to prepare and filter droplets
  repo: https://github.com/marcalva/diem
- name: R
  role: Language in which the benchmark workflow is executed
- name: Matrix
  role: R package storing raw counts as a column-wise sparse matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:54855c841e35837cb45124a1e8fd216d94e550c5ef832e78448fc2e962b65d65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - DIEM
  - EmptyDrops
  - Matrix
  - R
  - diem
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# filtering-method-benchmark-comparison

## Summary

Benchmarks competing droplet-filtering methods (DIEM's debris-cluster removal mode vs. EmptyDrops) on the identical restricted droplet set, comparing uniquely kept droplets by total counts and extranuclear contamination metrics (MT%, MALAT1%).

## When to use

Use when two or more droplet/background-filtering methods (e.g., DIEM and EmptyDrops) must be compared on the same scRNA-seq or snRNA-seq count matrix to determine which retains dying/debris-contaminated droplets.

## When NOT to use

- When comparing methods across different droplet sets — restrict both methods to the identical droplet set (same min_counts) first, or the kept-set differences are confounded
- When the goal is filtering a single dataset for downstream analysis rather than benchmarking method agreement (use the DIEM filtering skill alone)
- When the debris score itself needs to be interpreted across independent experiments — debris scores cannot be assumed to be on the same scale across samples

## Inputs

- Raw 10X feature-barcode matrix (from 10xgenomics dataset, e.g., fresh_68k_pbmc_donor_a) readable by diem's read_10x into a column-wise sparse Matrix
- Gene-level regex patterns for contamination markers: '^mt-' (mitochondrial) and '^malat1$'
- Minimum UMI count threshold defining the shared droplet test set (e.g., min_counts = 100)

## Outputs

- Per-method kept-droplet counts (e.g., 75,847 for DIEM, 77,585 for EmptyDrops on the 68K PBMC data)
- Group-wise mean MT% and MALAT1% summaries for shared, DIEM-only, and EmptyDrops-only droplets
- CSV file diem_emptydrops_pbmc_comparison.csv with kept-droplet counts and metric summaries

## How to apply

Read the raw 10X feature-barcode matrix into a column-wise sparse Matrix with diem's read_10x, build an SCE object via create_SCE, and compute per-droplet contamination metrics with get_gene_pct using genes matching '^mt-' (pct.mt) and '^malat1$' (MALAT1). Restrict both methods to an identical droplet set with set_debris_test_set (e.g., min_counts = 100) so the comparison is apples-to-apples. Run the DIEM pipeline (filter_genes, get_pcs, init, run_em, assign_clusters, estimate_dbr_score) and call targets in debris-cluster removal mode (call_targets with clusters = 'debris', appropriate for single-cell RNA-seq data), then run EmptyDrops on the same restricted matrix. Compute the intersection and set differences of kept droplets, summarize mean pct.mt and MALAT1 per group (shared, DIEM-only, EmptyDrops-only), and write counts and summaries to a CSV. The rationale: EmptyDrops was developed and tested on single-cell data and retains high-MT%/MALAT1% ruptured cells, while DIEM removes them; group-wise metric means reveal this.

## Related tools

- **DIEM** (Debris-cluster filtering method whose kept-droplet set is one arm of the comparison; run in debris-cluster removal mode (call_targets with clusters = 'debris') for single-cell RNA-seq data) — https://github.com/marcalva/diem
- **EmptyDrops** (Comparison filtering method that estimates a Dirichlet-Multinomial distribution of ambient RNA; its kept droplets form the other arm of the comparison)
- **diem** (R package providing read_10x, create_SCE, get_gene_pct, set_debris_test_set, and the EM pipeline used to prepare and filter droplets) — https://github.com/marcalva/diem
- **R** (Language in which the benchmark workflow is executed)
- **Matrix** (R package storing raw counts as a column-wise sparse matrix)

## Examples

```
sce <- call_targets(sce, clusters = "debris", thresh = NULL)
```

## Evaluation signals

- CSV reports 75,847 kept droplets for DIEM and 77,585 for EmptyDrops on the 68K PBMC dataset
- The 1,927 EmptyDrops-only droplets show higher mean MT% and MALAT1% than droplets kept by both methods
- The 189 DIEM-only droplets have MT% and MALAT1% similar to the shared passing droplets
- Both methods operated on the identical droplet set (droplets with at least 100 UMI counts)
- Group membership is internally consistent: shared + DIEM-only counts sum to DIEM kept total, and shared + EmptyDrops-only counts sum to EmptyDrops kept total

## Limitations

- DIEM's debris score correlation with extranuclear contamination (percent reads spliced) was less successful in PBMC single-cell RNA-seq data, possibly due to smaller differences between background and cell type profiles in scRNA-seq
- Debris scores are normalized relative to within-sample cluster means and cannot be assumed to be on the same scale across independent experiments
- EmptyDrops was originally developed and tested on single-cell data, so its model assumptions differ from DIEM's; method agreement may vary by assay type

## Evidence

- [intro] The recent method EmptyDrops12 addresses this filtering issue for scRNA-seq by estimating a Dirichlet-Multinomial distribution of the ambient RNA.: "The recent method EmptyDrops12 addresses this filtering issue for scRNA-seq by estimating a Dirichlet-Multinomial distribution of the ambient RNA."
- [readme] README call_targets for single-cell data: "# Call targets by removing droplets in debris cluster(s) for single-cell data
sce <- call_targets(sce, clusters = "debris", thresh = NULL)"
- [readme] README MT%/MALAT1 metric computation: [evidence span withheld — anchor status: external_doc]
- [readme] README restrict debris test set: "mb_small <- set_debris_test_set(mb_small, min_counts = 10)"
- [discussion] EmptyDrops model assumptions differ: "EmptyDrops, however, was originally developed and tested on single cell data and thus, the assumptions behind the model are different than that of DIEM."
- [discussion] Debris score comparability across experiments: [evidence span withheld — anchor status: not_found]
