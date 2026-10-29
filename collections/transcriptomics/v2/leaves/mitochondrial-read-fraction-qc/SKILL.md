---
name: mitochondrial-read-fraction-qc
description: Use when raw 10X droplet counts are loaded (sparse Matrix / SCE object)
  and you need per-droplet MT% or MALAT1% to assess extranuclear RNA contamination,
  tune debris-score filtering, or benchmark which droplets a filter (DIEM, EmptyDrops,
  hard cutoffs) retains.
when_to_use_negative:
- The data is not droplet-based 10X output or lacks mitochondrial/MALAT1 gene annotations
  matching '^mt-' / '^malat1$', since get_gene_pct relies on these gene-name patterns
- The goal is to replace DIEM's expression-based debris scoring entirely — MT% cutoffs
  are described as ad hoc and hard cutoffs can retain debris droplets and drop nuclear
  droplets in snRNA-seq
- Comparing debris scores or MT% thresholds across independent samples for integration
  — the paper warns the debris score 'cannot be assumed to exist on the same scale
  in independent experiments'
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: diem
  role: Provides read_10x, create_SCE and get_gene_pct used to compute per-droplet
    pct.mt and MALAT1 QC fractions
  repo: https://github.com/marcalva/diem
- name: R
  role: Implementation language; counts stored in a column-wise sparse Matrix from
    the R Matrix package
- name: Matrix
  role: R package backing the column-wise sparse count matrix that get_gene_pct operates
    on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:9946a1352c2c6f3adf9e5e8e995a9a66a84a2368ff84ca94d4301274be8e714d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - DIEM
  - Matrix
  - R
  - diem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mitochondrial-read-fraction-qc

## Summary

Compute per-droplet mitochondrial read fraction (pct.mt) and MALAT1% from a 10X feature-barcode matrix using the diem package's get_gene_pct, using these extranuclear-contamination metrics as QC covariates for droplet filtering decisions (e.g. DIEM vs EmptyDrops comparison).

## When to use

Use when raw 10X droplet counts are loaded (sparse Matrix / SCE object) and you need per-droplet MT% or MALAT1% to assess extranuclear RNA contamination, tune debris-score filtering, or benchmark which droplets a filter (DIEM, EmptyDrops, hard cutoffs) retains.

## When NOT to use

- The data is not droplet-based 10X output or lacks mitochondrial/MALAT1 gene annotations matching '^mt-' / '^malat1$', since get_gene_pct relies on these gene-name patterns
- The goal is to replace DIEM's expression-based debris scoring entirely — MT% cutoffs are described as ad hoc and hard cutoffs can retain debris droplets and drop nuclear droplets in snRNA-seq
- Comparing debris scores or MT% thresholds across independent samples for integration — the paper warns the debris score 'cannot be assumed to exist on the same scale in independent experiments'

## Inputs

- 10X feature-barcode count matrix read into a column-wise sparse Matrix via diem::read_10x
- SCE object created by diem::create_SCE with a gene_data slot (gene names matched against '^mt-' and '^malat1$')

## Outputs

- SCE object annotated with per-droplet 'pct.mt' (percent mitochondrial reads) and 'MALAT1' columns
- QC summaries/plots: plot_clust scatter of pct.mt (or MALAT1) vs score.debris; per-group mean MT%/MALAT1% tables (e.g. diem_emptydrops_pbmc_comparison.csv)

## How to apply

After reading the 10X matrix with read_10x and building the SCE object with create_SCE, identify mitochondrial genes by grepping gene_data for the pattern '^mt-' (ignore.case=TRUE, e.g. '^MT-' in human data) and the MALAT1 gene with '^malat1$', then call get_gene_pct once per gene set to add 'pct.mt' and 'MALAT1' columns to the SCE droplet data. These fractions quantify extranuclear contamination: MT% is described in the paper as 'a measure of extranuclear contamination'. Use the metrics diagnostically — plot pct.mt against score.debris via plot_clust(feat_x='pct.mt', feat_y='score.debris', log_x=TRUE) to check that debris-scored droplets carry high MT% — or as a benchmark metric when comparing kept-droplet sets across filters (e.g. mean pct.mt/MALAT1% for DIEM-only vs EmptyDrops-only droplets in the 68K PBMC data). Optionally apply an ad hoc pct.mt cutoff to filter droplets, but prefer expression-based scoring (DIEM debris score) since the paper shows hard cutoffs can both lose nuclear droplets and retain debris.

## Related tools

- **diem** (Provides read_10x, create_SCE and get_gene_pct used to compute per-droplet pct.mt and MALAT1 QC fractions) — https://github.com/marcalva/diem
- **R** (Implementation language; counts stored in a column-wise sparse Matrix from the R Matrix package)
- **Matrix** (R package backing the column-wise sparse count matrix that get_gene_pct operates on)

## Examples

```
mt_genes <- grep("^mt-", rownames(sce@gene_data), ignore.case=TRUE, value=TRUE); sce <- get_gene_pct(x = sce, genes=mt_genes, name="pct.mt")
```

## Evaluation signals

- Both 'pct.mt' and 'MALAT1' columns exist in the SCE droplet data and are fractions in [0, 1] (or percents in [0, 100]) derived from total per-droplet counts
- The mitochondrial gene set was matched case-insensitively on '^mt-' (human '^MT-') and MALAT1 on '^malat1$'; zero matched genes means the metric is invalid
- plot_clust(pct.mt vs score.debris) shows debris-scored droplets enriched in high MT%, consistent with extranuclear contamination
- In the 68K PBMC DIEM vs EmptyDrops benchmark, EmptyDrops-only droplets (1,927) show higher mean MT% and MALAT1% than droplets kept by both methods, validating MT% as a ruptured-cell indicator

## Limitations

- The correlation of debris score with extranuclear contamination was 'less successful in the PBMC single-cell RNA-seq data', likely because background and cell-type profiles differ less in scRNA-seq, so MT% agreement with DIEM scoring is weaker in single-cell data
- The paper notes 'an ad hoc cutoff of the percent of reads originating from the mitochondria' is ad hoc; hard MT% cutoffs can misfilter in snRNA-seq
- Debris scores (against which MT% is compared) are normalized within-sample and may not be comparable across experiments, especially if extranuclear RNA distributions differ between samples

## Evidence

- [intro] ad hoc mitochondria cutoff: "Additionally, an ad hoc cutoff of the percent of reads originating from the mitochondria (a measure of extranuclear contamination) can help to filter droplets"
- [methods] get_gene_pct function: "We can generate this using the `get_gene_pct` function"
- [discussion] PBMC debris-score correlation weaker: "this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq."
- [readme] README MT/MALAT1 computation: [evidence span withheld — anchor status: external_doc]
- [readme] README MT vs debris plot: [evidence span withheld — anchor status: external_doc]
