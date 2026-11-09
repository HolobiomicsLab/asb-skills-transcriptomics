---
name: subclone-expression-signature-verification
description: Use when genotyped cells expressing a specific somatic SNV are available
  for a scRNA-seq sample and you need to test whether the mutant subclone has an associated
  expression signature, including by extending mutation labels to non-genotyped cells
  via density in embedded/clustered space.
when_to_use_negative:
- 'The variant has not been pre-called: VarTrix explicitly ''does not perform variant
  calling'', so mutations must come from prior calls (e.g. eWGS of paired tumor/normal
  samples)'
- 'You need to determine cells that are truly wild-type for the mutation: dropout
  makes this impossible, so the method should only assert mutant status, not wild-type
  status'
- 'Subclone inference based on copy number alterations is desired and the tumor type
  is AML: CNAs are rare in AML and rarely capture complete subclonal complexity'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: python3
  role: Scripting environment for loading count matrices, computing mutant-cell density,
    fitting gene-wise regressions, and FDR correction
  repo: https://github.com/genome/scrna_mutations
- name: pysam
  role: Parses the 10x Cell Ranger BAM alongside cb_sniffer to call mutant/ref barcodes
    per cell before the expression regression
  repo: https://github.com/sridnona/cb_sniffer
- name: Docker (sridnona/python3:180925.v1)
  role: Container image providing the reproducible python3 environment used in the
    paper's methods
  repo: https://github.com/genome/scrna_mutations
- name: VarTrix
  role: Alternative tool to assign single cells to the pre-called variant loci used
    as the mutant-cell label input
  repo: https://github.com/10XGenomics/vartrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:1eed47231061235f72a237371539758c00496e64589c3928c5566e8f84c479d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genome/scrna_mutations
  tools:
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - VarTrix
  - pysam
  - python3
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# subclone-expression-signature-verification

## Summary

Regress single-cell gene expression on local density of cells expressing a somatic subclonal mutation (e.g. GATA2R361C in AML case 721214), controlling for covariates, to verify whether a subclone carries a distinct expression signature. This is how the paper connects genotype to phenotype in heterogeneous AML samples using 10x Chromium scRNA-seq.

## When to use

Use when genotyped cells expressing a specific somatic SNV are available for a scRNA-seq sample and you need to test whether the mutant subclone has an associated expression signature, including by extending mutation labels to non-genotyped cells via density in embedded/clustered space.

## When NOT to use

- The variant has not been pre-called: VarTrix explicitly 'does not perform variant calling', so mutations must come from prior calls (e.g. eWGS of paired tumor/normal samples)
- You need to determine cells that are truly wild-type for the mutation: dropout makes this impossible, so the method should only assert mutant status, not wild-type status
- Subclone inference based on copy number alterations is desired and the tumor type is AML: CNAs are rare in AML and rarely capture complete subclonal complexity

## Inputs

- UMI count matrix for the sample (_counts_UB.tsv-style Chromium UB_tag read count table)
- Precomputed t-SNE/cluster coordinates for the full sample including non-genotyped cells
- List of cells identified as expressing the GATA2R361C mutation (cb_sniffer/VarTrix output from the Cell Ranger BAM)
- Covariate table (e.g. GATA2 expression per cell)

## Outputs

- Ranked list of genes significantly associated with mutant-cell density (FDR-adjusted p < 0.05), written to gata2r361c_density_correlated_genes.tsv
- Per-cell local mutant-cell density values overlaid on the t-SNE/cluster space

## How to apply

Load the UMI count matrix for the sample (the _counts_UB.tsv-style table of Chromium molecular barcode UB_tag read counts) together with precomputed t-SNE/cluster coordinates in python3 (e.g. from the sridnona/python3:180925.v1 Docker image with pysam available). Overlay the cells identified as expressing the mutation (called by cb_sniffer or VarTrix from the Cell Ranger BAM) as a binary/graded label onto the t-SNE/cluster space of the entire sample, thereby leveraging expression information from non-genotyped cells. Compute a local mutant-cell density for every cell, then for each gene regress expression on mutant-cell density while controlling for GATA2 expression and additional covariates. Convert regression p-values to FDR-adjusted p-values, retain genes with FDR-adjusted p < 0.05, rank them by association strength, and write the ranked list to gata2r361c_density_correlated_genes.tsv. Correct application is validated when the top-ranked gene is VIM and immune response, apoptosis, and leukocyte adhesion genes are enriched in the ranked list.

## Related tools

- **python3** (Scripting environment for loading count matrices, computing mutant-cell density, fitting gene-wise regressions, and FDR correction) — https://github.com/genome/scrna_mutations
- **pysam** (Parses the 10x Cell Ranger BAM alongside cb_sniffer to call mutant/ref barcodes per cell before the expression regression) — https://github.com/sridnona/cb_sniffer
- **Docker (sridnona/python3:180925.v1)** (Container image providing the reproducible python3 environment used in the paper's methods) — https://github.com/genome/scrna_mutations
- **VarTrix** (Alternative tool to assign single cells to the pre-called variant loci used as the mutant-cell label input) — https://github.com/10XGenomics/vartrix

## Examples

```
python3 gata2r361c_density_regression.py --counts 721214_counts_UB.tsv --tsne 721214_tsne_clusters.tsv --mutants gata2r361c_cells.tsv --covariates gata2_expression.tsv --fdr 0.05 --out gata2r361c_density_correlated_genes.tsv
```

## Evaluation signals

- The top-ranked gene in the FDR-significant list (FDR-adjusted p < 0.05) is VIM
- Gene set enrichment of the ranked list shows immune response, apoptosis, and leukocyte adhesion genes among significant hits
- Every gene in the output table has an FDR-adjusted p-value, and only genes with FDR-adjusted p < 0.05 are retained
- Mutant-cell density is defined for every cell in the sample, including non-genotyped cells overlaid onto the t-SNE/cluster space
- The output file gata2r361c_density_correlated_genes.tsv is produced with genes ranked by association strength

## Limitations

- Dropout (transcript and allelic) occurs with most scRNA-seq platforms, reducing mutation detection sensitivity, so it is impossible to determine whether a cell is truly wild-type for a given mutation
- Partial transcript coverage on end-biased platforms such as Chromium limits sensitivity of variant detection, and coverage drops non-linearly across transcript length so some variants are more easily detectable than others
- Utility depends on the mutational composition of the sample and will likely perform better for tumor types with higher mutation burdens than AML
- Some observed expression heterogeneity without detectable genetic heterogeneity may be due to the limited sensitivity of mutation detection

## Evidence

- [abstract] Abstract finding of subclone signatures: "identify expression signatures associated with subclonal mutations, and find cell surface markers that could be used to purify subclones for further study"
- [methods] Count table format used as regression input: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [discussion] Leveraging non-genotyped cells: "we analyzed them in the context of the entire sample, thereby leveraging the expression information provided by the additional, non-genotyped cells"
- [readme] cb_sniffer README describes the genotyping input: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [readme] VarTrix README on overlaying variants with expression clustering: [evidence span withheld — anchor status: external_doc]
