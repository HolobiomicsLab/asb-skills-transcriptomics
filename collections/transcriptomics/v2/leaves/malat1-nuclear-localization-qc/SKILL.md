---
name: malat1-nuclear-localization-qc
description: Use when comparing droplet-filtering methods (e.g. DIEM debris-cluster
  mode vs EmptyDrops) on a 10x feature-barcode matrix and you need a quantitative
  extranuclear-contamination readout (MALAT1% alongside MT%) for kept vs removed droplets.
when_to_use_negative:
- The matrix is already filtered to cell barcodes only — contamination signal from
  background droplets has been discarded, so MALAT1%/MT% comparisons across kept sets
  are meaningless
- The gene annotation lacks MALAT1 (or mitochondrial genes) — grep for '^malat1$'
  will return no genes and the contamination metric cannot be computed
- You are scoring intra-nuclear quality of a bulk or plate-based RNA-seq sample rather
  than comparing droplet-level filtering methods
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DIEM (diem R package)
  role: Provides read_10x, create_SCE and get_gene_pct used to compute the MALAT1%
    and MT% per-droplet contamination metrics on the sparse count Matrix
  repo: https://github.com/marcalva/diem
- name: R Matrix
  role: Stores the raw counts as a column-wise sparse matrix before metric computation
- name: EmptyDrops
  role: Alternative droplet-filtering method whose uniquely kept droplets are profiled
    with MALAT1%/MT% to detect retained ruptured cells
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:8112bf1b55f7bc6c31e804083bd43c220bb0af506bac694601f92896e4254d6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - DIEM
  - DIEM (diem R package)
  - EmptyDrops
  - R
  - R Matrix
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# malat1-nuclear-localization-qc

## Summary

Computes the fraction of per-droplet counts mapping to MALAT1 (and mitochondria) as a metric of extranuclear/debris RNA contamination in droplet-based scRNA- or snRNA-seq, used to judge whether a filtering method removed ruptured-cell droplets.

## When to use

Use when comparing droplet-filtering methods (e.g. DIEM debris-cluster mode vs EmptyDrops) on a 10x feature-barcode matrix and you need a quantitative extranuclear-contamination readout (MALAT1% alongside MT%) for kept vs removed droplets.

## When NOT to use

- The matrix is already filtered to cell barcodes only — contamination signal from background droplets has been discarded, so MALAT1%/MT% comparisons across kept sets are meaningless
- The gene annotation lacks MALAT1 (or mitochondrial genes) — grep for '^malat1$' will return no genes and the contamination metric cannot be computed
- You are scoring intra-nuclear quality of a bulk or plate-based RNA-seq sample rather than comparing droplet-level filtering methods

## Inputs

- 10x Genomics feature-barcode matrix (raw, unfiltered; e.g. fresh_68k_pbmc_donor_a) readable by diem::read_10x
- SCE object created with diem::create_SCE from the column-wise sparse Matrix of raw counts
- Gene name vector from sce@gene_data for pattern matching ('^malat1$', '^mt-')

## Outputs

- Per-droplet MALAT1% and pct.mt columns added to the SCE object via get_gene_pct
- Grouped summaries of mean MT%/MALAT1% for shared, DIEM-only, and EmptyDrops-only kept droplets (e.g. diem_emptydrops_pbmc_comparison.csv)

## How to apply

After loading the 10x feature-barcode matrix into a column-wise sparse Matrix via diem's read_10x and building an SCE object with create_SCE, grep the gene IDs for '^malat1$' (ignore.case=TRUE) and pass the match to get_gene_pct to store a per-droplet MALAT1 percent column; do the same with '^mt-' for pct.mt. Because MALAT1 is a nuclear-retained lncRNA, droplets dominated by MALAT1 (often co-elevated with MT%) indicate ruptured cells whose nuclear RNA leaked into the ambient pool. Restrict both filtering methods to the identical droplet test set (e.g. set_debris_test_set with min_counts = 100) so contamination metrics are computed on comparable droplets, then summarize mean MALAT1%/MT% per kept-droplet group (shared, DIEM-only, EmptyDrops-only). A filtering method is judged better when its uniquely kept droplets show MALAT1% and MT% similar to shared passing droplets rather than elevated.

## Related tools

- **DIEM (diem R package)** (Provides read_10x, create_SCE and get_gene_pct used to compute the MALAT1% and MT% per-droplet contamination metrics on the sparse count Matrix) — https://github.com/marcalva/diem
- **R Matrix** (Stores the raw counts as a column-wise sparse matrix before metric computation)
- **EmptyDrops** (Alternative droplet-filtering method whose uniquely kept droplets are profiled with MALAT1%/MT% to detect retained ruptured cells)

## Examples

```
malat <- grep(pattern="^malat1$", x=rownames(sce@gene_data), ignore.case=TRUE, value=TRUE); sce <- get_gene_pct(x = sce, genes=malat, name="MALAT1")
```

## Evaluation signals

- On the 68K PBMC data restricted to droplets with >=100 UMIs, the CSV of kept-droplet counts should report 75,847 kept for DIEM and 77,585 for EmptyDrops
- The 1,927 EmptyDrops-only droplets must show higher mean MT% and MALAT1% than droplets kept by both methods, consistent with ruptured cells
- The 189 DIEM-only droplets should have MT% and MALAT1% similar to shared passing droplets, not elevated
- grep('^malat1$', rownames(sce@gene_data), ignore.case=TRUE) returns exactly one gene so the pct column is well-defined
- Per-droplet MALAT1% and pct.mt values are bounded in [0, 1] (or [0, 100] if scaled) and sum-consistent with per-droplet total counts

## Limitations

- Debris-score/contamination correlation was 'less successful in the PBMC single-cell RNA-seq data', possibly because background and cell-type profiles differ less in scRNA-seq than snRNA-seq
- Debris scores are not on the same scale across independent experiments, so MALAT1%/MT%-based group comparisons should be made within a sample
- In this 68K PBMC analysis the droplet universe was restricted to >=100 UMI counts so DIEM and EmptyDrops operated on an identical set; unrestricted runs are not directly comparable

## Evidence

- [methods] get_gene_pct: "We can generate this using the `get_gene_pct` function"
- [readme] readme malat pattern: "malat <- grep(pattern="^malat1$", x=rownames(sce@gene_data), ignore.case=TRUE, value=TRUE)
sce <- get_gene_pct(x = sce, genes=malat, name="MALAT1")"
- [results] task finding on ruptured cells: [evidence span withheld — anchor status: not_found]
- [discussion] scRNA-seq limitation: "this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq."
