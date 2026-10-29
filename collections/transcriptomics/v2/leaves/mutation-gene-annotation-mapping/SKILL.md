---
name: mutation-gene-annotation-mapping
description: Use when SNV callsets carry chromosome coordinates but the scRNA-seq
  count matrix (e.g., the featurecounts.qc slot of GBM10_preproc) is keyed by gene
  identifier, and the two matrices must be row-aligned before bursting estimation.
when_to_use_negative:
- SNVs are already annotated with host-gene HGNC symbols and the expression matrix
  rows already match the SNV row order
- Only read-count matrices are being processed without any need to link mutations
  to gene expression (the mapping is irrelevant to pure bulk DNA tree inference)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: biomaRt
  role: Queries Ensembl (build 37) to map mutation chromosome positions to host-gene
    HGNC symbols; installed via BiocManager inside the Canopy2 dependency set
- name: Canopy2
  role: Provides get_gene_expression()/get_BM_fun for the mapping and get_burstiness_bpsc()/get_burstiness_scale()
    that consume the aligned expression data
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:2982b26a969bd2b1309e95d55ee9b8e391b4a5605442ce98a912c27098e2ea59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - Canopy2
  - R
  - biomaRt
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mutation-gene-annotation-mapping

## Summary

Maps the chromosome positions of SNVs to HGNC gene symbols using biomaRt (Ensembl build 37) so that single-cell gene expression rows align one-to-one with mutation-host-gene rows in Canopy2, a prerequisite for estimating mutation-specific bursting kinetics (alpha, beta).

## When to use

Use when SNV callsets carry chromosome coordinates but the scRNA-seq count matrix (e.g., the featurecounts.qc slot of GBM10_preproc) is keyed by gene identifier, and the two matrices must be row-aligned before bursting estimation.

## When NOT to use

- SNVs are already annotated with host-gene HGNC symbols and the expression matrix rows already match the SNV row order
- Only read-count matrices are being processed without any need to link mutations to gene expression (the mapping is irrelevant to pure bulk DNA tree inference)

## Inputs

- M x N matrix of alternative read counts (Rs) from single-cell RNA sequencing, rows keyed by mutation with chromosome positions
- M x N matrix of total read counts (Xs) from single-cell RNA sequencing
- Single-cell gene expression count matrix (e.g., featurecounts.qc slot of GBM10_preproc) keyed by gene identifier
- Ensembl BioMart connection (build 37) via biomaRt

## Outputs

- Mutation-to-HGNC-gene-symbol mapping aligning SNV rows to gene expression rows
- Row-aligned expression matrix suitable for get_burstiness_bpsc() / get_burstiness_scale()
- 1 x M vectors of mutation-specific gene activation (alpha) and deactivation (beta) rates with an id.g vector of mutations with estimable kinetics

## How to apply

Use Canopy2's get_BM_fun/get_gene_expression functions, which internally call biomaRt::useEnsembl() under Ensembl build 37, to convert each mutation's chromosome position to its host gene's HGNC symbol. Reorder or subset the single-cell gene expression count matrix so that gene order matches the SNV rows of the Rs and Xs read-count matrices. Note that Canopy2 expects bursting rates derived from germline mutations within the same gene body as the somatic mutations, so gene-body membership—not just nearest-gene mapping—drives the annotation. Once aligned, the expression matrix feeds get_burstiness_bpsc() (MCMC, recommended) or get_burstiness_scale() (moment estimators) to obtain 1 x M vectors of mutation-specific activation (alpha) and deactivation (beta) rates. If the biomaRt call fails with a db_collect/lazy-table error, reinstall BiocFileCache per the README compatibility fix before retrying.

## Related tools

- **biomaRt** (Queries Ensembl (build 37) to map mutation chromosome positions to host-gene HGNC symbols; installed via BiocManager inside the Canopy2 dependency set)
- **Canopy2** (Provides get_gene_expression()/get_BM_fun for the mapping and get_burstiness_bpsc()/get_burstiness_scale() that consume the aligned expression data) — https://github.com/annweideman/canopy2

## Examples

```
BiocManager::install("biomaRt"); devtools::install_github("annweideman/canopy2")
```

## Evaluation signals

- Every SNV row resolves to a host-gene HGNC symbol under Ensembl build 37, with no unmatched coordinates
- Row order of the gene expression matrix exactly matches the SNV rows of Rs and Xs after mapping
- Returned alpha and beta values are all non-NA and non-negative; the paper notes BPSC had improved estimability of non-NA or negative values
- The id.g output vector marks exactly the mutations for which gene burstiness was estimable, and Rs/Xs are subset accordingly
- No db_collect()/lazy-table error from biomaRt::useEnsembl(); if it occurs, reinstalling BiocFileCache resolves it

## Limitations

- biomaRt depends on Ensembl web services; a BiocFileCache/dbplyr compatibility bug can break get_gene_expression() with a 'Failed to collect lazy table' error, requiring BiocFileCache reinstallation
- Mapping uses Ensembl build 37; coordinates from other genome builds must be lifted over first
- Bursting kinetics are not estimable for all mutation-host genes, so only the id.g subset of mutations can carry alpha/beta hyperparameters downstream

## Evidence

- [readme] get_gene_expression internally uses biomaRt: [evidence span withheld — anchor status: external_doc]
- [discussion] Estimate gene activation and deactivation rates from germline mutations within same gene body as somatic mutations: [evidence span withheld — anchor status: not_found]
- [methods] Subset read counts to mutations with estimable burstiness: "Subset alternative and total read counts to include only mutations for which gene burstiness can be estimated."
- [readme] alpha/beta are mutation-specific vectors estimated by BPSC or SCALE: [evidence span withheld — anchor status: external_doc]
