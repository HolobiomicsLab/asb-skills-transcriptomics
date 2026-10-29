---
name: density-regression-gene-expression-association
description: Use when you have scRNA-seq cells genotyped for an expressed somatic
  SNV (e.g., via cb_sniffer or Vartrix on _counts_UB.tsv data) and want to test which
  genes track a mutant subclone across the whole sample, including non-genotyped cells.
when_to_use_negative:
- The sample lacks expressed somatic SNVs — in AML, CNAs are rare and CNAs rarely
  reflect the complete clonal architecture, so mutation-based subclone labels require
  tumor-specific SNVs discovered by eWGS of paired tumor/normal samples.
- Only genotyped cells are of interest and you do not intend to leverage expression
  information from the additional, non-genotyped cells in the sample.
- The goal is variant calling itself rather than expression-phenotype association
  — Vartrix does not perform variant calling; variants must be pre-called (e.g., from
  eWGS of tumor/normal pairs).
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: python3
  role: Implementation language for loading the _counts_UB.tsv matrix and t-SNE coordinates,
    computing local mutant-cell density, fitting per-gene regressions with covariates,
    and FDR adjustment
- name: pysam
  role: Parses Cell Ranger BAMs and extracts CB barcodes when identifying cells expressing
    the mutation (upstream of density regression via cb_sniffer)
- name: cb_sniffer
  role: Calls mutant and reference barcodes from 10x single-cell BAMs, producing the
    mutant-cell label that is converted to density and regressed against expression
  repo: https://github.com/sridnona/cb_sniffer
- name: Vartrix
  role: Alternative tool for extracting per-cell variant matrices from 10x data to
    obtain expressed-mutation labels for cells before density computation
  repo: https://github.com/10XGenomics/vartrix
- name: sridnona/python3:180925.v1
  role: Docker image in which the regression workflow is executed for reproducibility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:1154ede3b31427bca47a482a17f9e2bc80026a2e53213bd8d805311b6b610352
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - Vartrix
  - cb_sniffer
  - pysam
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# density-regression-gene-expression-association

## Summary

Regresses single-cell gene expression on a continuous local density of cells expressing a subclonal mutation (e.g., GATA2R361C in AML sample 721214) within the embedded/clustered expression space, controlling for the mutant gene's own expression and covariates, to identify genes whose expression is associated with the subclone (FDR < 0.05).

## When to use

Use when you have scRNA-seq cells genotyped for an expressed somatic SNV (e.g., via cb_sniffer or Vartrix on _counts_UB.tsv data) and want to test which genes track a mutant subclone across the whole sample, including non-genotyped cells.

## When NOT to use

- The sample lacks expressed somatic SNVs — in AML, CNAs are rare and CNAs rarely reflect the complete clonal architecture, so mutation-based subclone labels require tumor-specific SNVs discovered by eWGS of paired tumor/normal samples.
- Only genotyped cells are of interest and you do not intend to leverage expression information from the additional, non-genotyped cells in the sample.
- The goal is variant calling itself rather than expression-phenotype association — Vartrix does not perform variant calling; variants must be pre-called (e.g., from eWGS of tumor/normal pairs).

## Inputs

- UMI count matrix for the AML sample derived from Chromium UB_tag read counts (_counts_UB.tsv-style table)
- Precomputed t-SNE/cluster coordinates for the full sample
- List of cells identified as expressing the subclonal mutation (e.g., GATA2R361C mutant-cell barcodes from cb_sniffer/Vartrix)

## Outputs

- Ranked list of genes significantly associated with mutant-cell density (FDR-adjusted p < 0.05), e.g., gata2r361c_density_correlated_genes.tsv
- Per-cell local mutant-cell density values in the embedded/clustered space

## How to apply

Load the UMI count matrix derived from Chromium UB_tag read counts (_counts_UB.tsv) together with the precomputed t-SNE/cluster coordinates in python3, and overlay cells identified as expressing the mutation (e.g., via cb_sniffer) as a binary or graded label onto the embedded space. Compute, for every cell (genotyped and non-genotyped), a local mutant-cell density in this space so the analysis leverages expression information from the entire sample. For each gene, fit a regression of expression on mutant-cell density while controlling for the mutant gene's own expression (e.g., GATA2) and additional covariates, implemented in python3 within the sridnona/python3:180925.v1 Docker image. Convert regression p-values to FDR-adjusted p-values, retain genes with FDR-adjusted p < 0.05, rank significant genes by association strength, and write the ranked list (e.g., gata2r361c_density_correlated_genes.tsv).

## Related tools

- **python3** (Implementation language for loading the _counts_UB.tsv matrix and t-SNE coordinates, computing local mutant-cell density, fitting per-gene regressions with covariates, and FDR adjustment)
- **pysam** (Parses Cell Ranger BAMs and extracts CB barcodes when identifying cells expressing the mutation (upstream of density regression via cb_sniffer))
- **cb_sniffer** (Calls mutant and reference barcodes from 10x single-cell BAMs, producing the mutant-cell label that is converted to density and regressed against expression) — https://github.com/sridnona/cb_sniffer
- **Vartrix** (Alternative tool for extracting per-cell variant matrices from 10x data to obtain expressed-mutation labels for cells before density computation) — https://github.com/10XGenomics/vartrix
- **sridnona/python3:180925.v1** (Docker image in which the regression workflow is executed for reproducibility)

## Examples

```
python3 -c "import pandas as pd, statsmodels.api as sm; X=pd.read_csv('tsne_coords.tsv',sep='\t'); y=pd.read_csv('_counts_UB.tsv',sep='\t'); res=sm.OLS(y[g], sm.add_constant(density+gata2_expr)).fit(); padj=sm.stats.multipletests(res.pvalues,method='fdr_bh')[1]"
```

## Evaluation signals

- Top-ranked gene in the FDR-significant ranked list matches the known subclone marker (VIM was the most strikingly correlated gene for the GATA2R361C subclone in case 721214).
- GO-level enrichment of the ranked list is consistent with the expected subclone phenotype (immune response, apoptosis, and leukocyte adhesion genes enriched for GATA2R361C).
- All significant genes pass the stated threshold of FDR-adjusted p < 0.05 after multiple-testing correction of regression p-values.
- Every cell in the sample, including non-genotyped cells, receives a local mutant-cell density value (no missing densities in the embedded/clustered space).
- Output ranked list file (e.g., gata2r361c_density_correlated_genes.tsv) contains the required columns of gene, association strength, and FDR-adjusted p-value.

## Limitations

- It is impossible to determine whether a cell is truly wild-type for a given mutation because dropout (transcript and allelic) occurs on most scRNA-seq platforms.
- Dropout reduces the sensitivity of mutation detection by a factor of two, and partial transcript coverage on end-biased Chromium platforms further limits sensitivity of variant detection.
- Coverage drops non-linearly across transcript length, so some variants are much more easily detectable than others.
- The utility of the approach depends on the specific mutational composition of the sample and likely performs better for tumor types with higher mutation burdens than AML.
- Some observed expression heterogeneity without detectable genetic heterogeneity may be due in part to the limited sensitivity of mutation detection.

## Evidence

- [discussion] Analyze genotyped cells in context of whole sample: "we analyzed them in the context of the entire sample, thereby leveraging the expression information provided by the additional, non-genotyped cells"
- [methods] UB_tag count table input: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [readme] cb_sniffer usage for mutant barcode calling: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [readme] Variant-overlay-on-clustering rationale: [evidence span withheld — anchor status: external_doc]
- [abstract] Expression signatures associated with subclonal mutations: "identify expression signatures associated with subclonal mutations, and find cell surface markers that could be used to purify subclones for further study"
- [discussion] Dropout limitation: "As a result, it is impossible to determine whether a cell is truly wild-type for a given mutation."
