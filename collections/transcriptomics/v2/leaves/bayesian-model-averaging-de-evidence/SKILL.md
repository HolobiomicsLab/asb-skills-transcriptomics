---
name: bayesian-model-averaging-de-evidence
description: Use when scRNA-seq DE summary statistics (beta, se_beta) are already
  fitted with iDEA.fit and you want gene-level posterior DE probabilities that exploit
  consistent DE evidence across overlapping gene sets rather than gene-by-gene testing.
when_to_use_negative:
- Input is raw single-cell count data rather than per-gene DE summary statistics;
  iDEA works only on summary statistics
- No fitted iDEA model exists yet — iDEA.BMA must be run after iDEA.fit on a populated
  iDEA object
- Individual-level scRNA-seq modeling assumptions or cell-type-level uncertainty must
  be explicitly modeled, which iDEA's z-score summary-statistic framework does not
  do
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: iDEA
  role: R package providing iDEA.fit to fit the hierarchical Bayesian joint DE/GSE
    model and iDEA.BMA to aggregate DE evidence across gene sets
  repo: https://github.com/xzhoulab/iDEA
- name: zingeR
  role: Source of DE log-fold changes and p-values (zingeR-DESeq2) converted into
    iDEA beta/se_beta summary statistics
  repo: https://github.com/statOmics/zinbwaveZinger
- name: MAST
  role: Alternative DE method whose per-gene statistics feed the iDEA object prior
    to BMA aggregation
- name: SCDE
  role: Alternative DE method whose per-gene statistics feed the iDEA object prior
    to BMA aggregation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:daa238db6e7dd7c141b3dbe90e18983ce77378898c52fca76eb1e46a49264204
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CAMERA
  - DAVID
  - GSEA
  - MAST
  - PAGE
  - R
  - SCDE
  - iDEA
  - zingeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# bayesian-model-averaging-de-evidence

## Summary

Aggregate per-gene differential expression evidence across gene sets using iDEA's Bayesian model averaging (iDEA.BMA), which borrows enrichment signals from all fitted gene sets to improve DE power for individual genes in scRNA-seq summary statistics.

## When to use

Use when scRNA-seq DE summary statistics (beta, se_beta) are already fitted with iDEA.fit and you want gene-level posterior DE probabilities that exploit consistent DE evidence across overlapping gene sets rather than gene-by-gene testing.

## When NOT to use

- Input is raw single-cell count data rather than per-gene DE summary statistics; iDEA works only on summary statistics
- No fitted iDEA model exists yet — iDEA.BMA must be run after iDEA.fit on a populated iDEA object
- Individual-level scRNA-seq modeling assumptions or cell-type-level uncertainty must be explicitly modeled, which iDEA's z-score summary-statistic framework does not do

## Inputs

- iDEA object created by CreateiDEAObject from per-gene DE summary statistics (gene, beta, se_beta) derived from zingeR-DESeq2, MAST, or SCDE output
- Gene set annotation matrix (e.g., mouseGeneSets / humanGeneSets R objects from MGI GO or MSigDB sources)

## Outputs

- Per-gene Bayesian-model-averaged differential expression evidence (gene-level DE statistics/p-values aggregated across gene sets)

## How to apply

First convert DE tool output (e.g., zingeR-DESeq2, MAST, SCDE log-fold changes and p-values) into iDEA summary statistics: beta = logFC, zscore = qnorm(pvalue/2, lower.tail=FALSE), se_beta = abs(beta/zscore). Build the iDEA object with CreateiDEAObject (e.g., max_var_beta = 100, min_precent_annot = 0.0025, num_core > 1) on a gene set collection such as mouseGeneSets compiled from MGI GO annotations. Run iDEA.fit (min_degene = 5, em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5) so each gene set gets a fitted enrichment statistic, then call iDEA.BMA to aggregate, for each gene, the DE evidence contributed across all gene sets containing it. Rationale: analyzing one gene at a time loses power by ignoring consistent DE evidence across similar genes; BMA pools the fitted gene-set information so gene-level inference benefits from pathway context. Validate by checking that BMA-based DE calls increase power over the input DE method (up to 64% gain reported) at a controlled FDR (e.g., 1%).

## Related tools

- **iDEA** (R package providing iDEA.fit to fit the hierarchical Bayesian joint DE/GSE model and iDEA.BMA to aggregate DE evidence across gene sets) — https://github.com/xzhoulab/iDEA
- **zingeR** (Source of DE log-fold changes and p-values (zingeR-DESeq2) converted into iDEA beta/se_beta summary statistics) — https://github.com/statOmics/zinbwaveZinger
- **MAST** (Alternative DE method whose per-gene statistics feed the iDEA object prior to BMA aggregation)
- **SCDE** (Alternative DE method whose per-gene statistics feed the iDEA object prior to BMA aggregation)

## Examples

```
devtools::install_github('xzhoulab/iDEA'); library(iDEA); idea <- CreateiDEAObject(gene_info, gene_sets, beta, se_beta, max_var_beta=100, min_precent_annot=0.0025); idea <- iDEA.fit(idea, min_degene=5, em_iter=15, mcmc_iter=1000); idea <- iDEA.BMA(idea)
```

## Evaluation signals

- BMA DE gene list should overlap the top known DE genes (e.g., top-100 NP1 genes in Usoskin data) with a high Jaccard index against baseline DE lists
- Number of DE genes called at FDR 1% (anchoring e.g. on GO:0097458) should meet or exceed the standalone DE method's count, reflecting reported up-to-64% power gain
- Genomic-control lambda_gc for the summary statistics should be near 1, indicating calibrated z-scores before BMA
- Sanity-check conversion: se_beta = abs(beta / qnorm(pvalue/2, lower.tail=FALSE)) reproduces the input beta and p-value exactly
- Genes filtered by max_var_beta (default 100) and gene sets below min_precent_annot (default 0.0025) should be absent from the iDEA object

## Limitations

- z-score modeling effectively assumes the prior distribution of true effect sizes depends on standard errors and sample size
- DE analysis on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors, biasing BMA-derived evidence
- Future methodological work is still needed to account for uncertainty in cell type inference; iDEA does not propagate this uncertainty into gene-level DE calls

## Evidence

- [methods] iDEA.BMA aggregates DE evidence across gene sets: "we developed a Bayesian model averaging (BMA) approach to aggregate DE evidence for any given genes across all available gene sets"
- [intro] Analyzing one gene at a time causes power loss: "Analyzing one gene at a time can lead to potential power loss, as this approach fails to exploit consistent DE evidence across similar genes"
- [intro] iDEA works on summary statistics only: "iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data"
- [abstract] Power gain over DE methods: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [readme] Installation command from README: "devtools::install_github('xzhoulab/iDEA')"
