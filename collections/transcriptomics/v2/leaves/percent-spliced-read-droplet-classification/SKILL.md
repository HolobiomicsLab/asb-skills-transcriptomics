---
name: percent-spliced-read-droplet-classification
description: Use when filtering snRNA-seq droplets and you need to label retained
  or removed droplets as nuclear or background (e.g., to validate DIEM, EmptyDrops,
  or quantile filtering) using per-droplet spliced-read fractions.
when_to_use_negative:
- 'Input is scRNA-seq rather than snRNA-seq: the article reports the spliced-read/debris-score
  correlation with extranuclear contamination was less successful in PBMC single-cell
  RNA-seq data, likely due to smaller differences between background and cell type
  profiles'
- The count matrix has already been filtered so background droplets are absent — there
  are no removed droplets left to classify and the nuclear/background comparison is
  meaningless
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DIEM
  role: Provides get_gene_pct to compute per-droplet percent mitochondrial and percent
    nuclear MALAT1 metrics used for classification; its debris-score filtering output
    is evaluated by this classification
  repo: https://github.com/marcalva/diem
- name: EmptyDrops
  role: Baseline filter whose kept/removed droplets are classified as nuclear/background
    via percent spliced reads for comparison (FDR q > 0.05 removal threshold)
- name: R
  role: Implementation environment; raw counts stored as a column-wise sparse matrix
    using the R package Matrix
- name: Matrix
  role: R package providing the column-wise sparse matrix representation of the raw
    10X droplet counts
- name: CellRanger
  role: Source of the aligned 10X raw_feature_bc_matrix output that supplies the droplet
    count matrix to be classified
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:da0bd9ca492077f1f9b8a0794e8e44a131253fa3969c4980f1d256dd7cf904f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - EmptyDrops
  - Matrix
  - R
  - irlba
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# percent-spliced-read-droplet-classification

## Summary

Classify droplets from a snRNA-seq experiment as nuclear versus background by computing the percent of spliced reads per droplet, providing the ground-truth-free validation metric used to benchmark DIEM, EmptyDrops, and quantile filtering on the 10X nuclei_2k mouse brain dataset.

## When to use

Use when filtering snRNA-seq droplets and you need to label retained or removed droplets as nuclear or background (e.g., to validate DIEM, EmptyDrops, or quantile filtering) using per-droplet spliced-read fractions.

## When NOT to use

- Input is scRNA-seq rather than snRNA-seq: the article reports the spliced-read/debris-score correlation with extranuclear contamination was less successful in PBMC single-cell RNA-seq data, likely due to smaller differences between background and cell type profiles
- The count matrix has already been filtered so background droplets are absent — there are no removed droplets left to classify and the nuclear/background comparison is meaningless

## Inputs

- Raw 10X CellRanger count matrix (raw_feature_bc_matrix) loaded as a column-wise sparse Matrix via diem's read_10x
- Per-droplet filter calls (kept/removed barcodes) from DIEM, EmptyDrops (FDR q > 0.05 removal), or quantile filtering
- Gene annotations or gene-name patterns identifying mitochondrial (mt-) and MALAT1 transcripts for spliced/nuclear read fractions

## Outputs

- Per-droplet classification table labeling each barcode as nuclear or background with its percent spliced reads (e.g., diem_nuclei2k_filtering_results.csv, emptydrops_droplet_calls.csv)
- Tabulated counts and proportions of nuclear versus background droplets among kept and removed droplets per filtering method

## How to apply

After loading the raw 10X CellRanger count matrix and running a droplet filter, compute for each droplet the fraction of reads originating from spliced/nuclear-localized transcripts versus extranuclear RNA, using metrics such as percent mitochondrial reads and percent nuclear MALAT1 computed via diem's get_gene_pct. Classify each passing (and removed) droplet as nuclear or background based on its percent spliced reads, then tabulate counts among the passing droplets. On the mouse brain nuclei_2k data this yields ~2,010 DIEM-passing droplets, of which 1,850 (92.0%) are nuclear, versus 89.8% for EmptyDrops and 87.6% for quantile filtering; for EmptyDrops, 65 of 101 (64.4%) removed droplets are background. The rationale is that sub-cellularly localized RNA products (e.g., intronic/MALAT1 for nuclei, mitochondrial for ambient) differ in abundance between nuclear and ambient droplets, so spliced-read fraction serves as an independent check on filter quality.

## Related tools

- **DIEM** (Provides get_gene_pct to compute per-droplet percent mitochondrial and percent nuclear MALAT1 metrics used for classification; its debris-score filtering output is evaluated by this classification) — https://github.com/marcalva/diem
- **EmptyDrops** (Baseline filter whose kept/removed droplets are classified as nuclear/background via percent spliced reads for comparison (FDR q > 0.05 removal threshold))
- **R** (Implementation environment; raw counts stored as a column-wise sparse matrix using the R package Matrix)
- **Matrix** (R package providing the column-wise sparse matrix representation of the raw 10X droplet counts)
- **CellRanger** (Source of the aligned 10X raw_feature_bc_matrix output that supplies the droplet count matrix to be classified)

## Examples

```
sce <- get_gene_pct(x = sce, genes = mt_genes, name = "pct.mt"); sce <- get_gene_pct(x = sce, genes = malat, name = "MALAT1")
```

## Evaluation signals

- On the mouse brain nuclei_2k dataset, DIEM-passing droplets should classify as 1,850 of 2,010 (92.0%) nuclear — reproduce this in diem_nuclei2k_filtering_results.csv
- EmptyDrops baseline should yield 1,868 of 2,080 kept droplets nuclear (89.8%) and 65 of 101 removed droplets background (64.4%) in emptydrops_droplet_calls.csv
- Quantile (hard count cutoff) filtering should show lower nuclear retention (~87.6%) than DIEM and EmptyDrops
- Per-droplet spliced-read fractions (pct.mt, MALAT1) must be computable for every barcode, including droplets with a single mapped read
- Removed-droplet background fractions should rank DIEM (68.4%) > EmptyDrops (64.4%) > quantile (26.5%) on this dataset

## Limitations

- The correlation between debris score and extranuclear contamination (percent reads spliced) was less successful in PBMC single-cell RNA-seq data, possibly due to smaller differences between background and cell type profiles
- If few or no genes are increased in the background distribution, debris-based scoring (and hence downstream classification support) will likely be inaccurate
- Debris scores are normalized within a sample and cannot be assumed to be on the same scale across independent experiments or when integrating multiple samples with different extranuclear RNA distributions

## Evidence

- [intro] we hypothesized that there would be genes with sub-cellular localized RNA products that show differential abundance between droplets containing nuclear vs. ambient RNA.: "we hypothesized that there would be genes with sub-cellular localized RNA products that show differential abundance between droplets containing nuclear vs. ambient RNA."
- [methods] We can generate this using the `get_gene_pct` function: "We can generate this using the `get_gene_pct` function"
- [methods] the raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]: "The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]"
- [discussion] this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq.: "this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq."
- [readme] sce <- get_gene_pct(x = sce, genes=mt_genes, name="pct.mt"): [evidence span withheld — anchor status: external_doc]
