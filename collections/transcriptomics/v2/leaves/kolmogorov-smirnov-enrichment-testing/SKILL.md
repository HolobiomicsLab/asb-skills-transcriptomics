---
name: kolmogorov-smirnov-enrichment-testing
description: Use when you have DESeq2 differential expression results annotated with
  seed-match counts (e.g. mer7m8 or mer8 matches, or a curated miRNA-target list)
  and want to test whether the target gene set shows a cumulative shift in log2FC
  relative to background.
when_to_use_negative:
- Input is raw read data or a count matrix rather than a DESeq2 results data frame
  with gene_id, log2FoldChange, and padj columns — deseq_fc_ecdf is 'designed to work
  directly with a DESeq2 results data frame'
- You need per-gene differential expression calls with explicit fold-change/FDR cutoffs
  rather than a set-level cumulative distribution comparison
- Gene sets are not mutually exclusive or the seed match / background partition has
  not been defined (e.g. mer7m8 matches still include mer8 genes)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SeedMatchR
  role: Provides deseq_fc_ecdf to compare log2FC ECDFs of gene lists from DESeq2 results,
    plus filter_res, get_seed, and SeedMatchR() to build the annotated input
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Source of the differential expression results (log2FoldChange, padj) that
    SeedMatchR extends by annotating predicted seed matches before ECDF/KS testing
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:c71a774d5a68ed49d2c38e86686e97f461a3236fa517f8ca81cde3fd6b3ad9ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - R
  - SeedMatchR
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# kolmogorov-smirnov-enrichment-testing

## Summary

Compares the empirical cumulative distribution of log2(fold-change) values between a seed-match gene set and a background gene set from DESeq2 results, using a Kolmogorov-Smirnov test to detect the cumulative expression shifts characteristic of seed-mediated siRNA off-target (or miRNA-perturbation) effects.

## When to use

Use when you have DESeq2 differential expression results annotated with seed-match counts (e.g. mer7m8 or mer8 matches, or a curated miRNA-target list) and want to test whether the target gene set shows a cumulative shift in log2FC relative to background.

## When NOT to use

- Input is raw read data or a count matrix rather than a DESeq2 results data frame with gene_id, log2FoldChange, and padj columns — deseq_fc_ecdf is 'designed to work directly with a DESeq2 results data frame'
- You need per-gene differential expression calls with explicit fold-change/FDR cutoffs rather than a set-level cumulative distribution comparison
- Gene sets are not mutually exclusive or the seed match / background partition has not been defined (e.g. mer7m8 matches still include mer8 genes)

## Inputs

- Filtered DESeq2 results data frame containing gene_id, log2FoldChange, and padj columns
- Seed-match annotations per gene (e.g. mer7m8 count column from SeedMatchR()) or an external gene list (e.g. miRDB-predicted miRNA targets with score >= 90)
- Background gene list (e.g. genes with zero seed matches)

## Outputs

- ECDF plot of log2(fold-change) for target vs background gene sets (e.g. mirna_target_ecdf.png, Fig. 1F/1G-style)
- KS test statistic and p-value (e.g. exported to mirna_ecdf_ks_results.csv)
- ecdf.results object containing the plot (ecdf.results$plot)

## How to apply

Annotate filtered DESeq2 results (via SeedMatchR::filter_res and SeedMatchR() seed-match counting with seed.name='mer7m8') with seed-match counts, then partition genes into mutually exclusive gene sets, e.g. mer7m8 >= 1 vs mer7m8 == 0 (for promoter analysis, mer7m8 >= 1 with mer8 == 0). Call SeedMatchR::deseq_fc_ecdf directly on the DESeq2 results data frame with a named list of gene lists (Background plus target set) to plot the ECDF of log2(FC) and run the KS test; use stats_test='KS' and alternative='greater' when testing for a positive (up-regulation) shift, as in the Fig. 1F/1G analyses. Extract the KS statistic and p-value from the returned results object and inspect the ECDF plot for clear separation between Background and target curves. The rationale is that miRNA-like perturbations produce cumulative, often subtle, changes in target gene expression that a per-gene cutoff analysis misses but ECDF/KS testing detects.

## Related tools

- **SeedMatchR** (Provides deseq_fc_ecdf to compare log2FC ECDFs of gene lists from DESeq2 results, plus filter_res, get_seed, and SeedMatchR() to build the annotated input) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Source of the differential expression results (log2FoldChange, padj) that SeedMatchR extends by annotating predicted seed matches before ECDF/KS testing)

## Examples

```
ecdf.results = deseq_fc_ecdf(res, list("Background" = background.list, "mer7m8" = mer7m8.list))
```

## Evaluation signals

- The deseq_fc_ecdf call returns an object whose $plot renders an ECDF with Background and target (e.g. mer7m8 or miRNA-target) curves clearly separated
- KS test output (statistic and p-value) is returned alongside the plot; for alternative='greater' the result should indicate a significant positive shift of the target log2FC distribution
- Gene lists partition the filtered results without overlap: mer7m8 >= 1 genes vs mer7m8 == 0 background, or promoter mer7m8 >= 1 with mer8 == 0 vs double-zero background
- Reproduction matches the published Fig. 1F/1G patterns: positive shift of predicted miRNA targets at Ttr D1 10 mg/kg, and slight positive-log2FC enrichment for promoter mer7m8 match genes at Ttr D1 30 mg/kg

## Limitations

- Requires the input DESeq2 data frame to contain gene_id, log2FoldChange, and padj columns for seed-match mapping
- Detects cumulative set-level shifts, not which individual genes are off-target; per-gene significance is not reported
- The shift being tested can be subtle (e.g. only a 'slight increase in positive fold changes' for promoter seed matches), so interpretation depends on the KS test rather than visual separation alone

## Evidence

- [readme] Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test.: "Many factors that perturb gene expression, like miRNA, show cumulative changes in their targets gene expression."
- [methods] The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame.: "The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame."
- [readme] The input must contain the columns `gene_id`, `log2FoldChange`, and `padj`.: [evidence span withheld — anchor status: external_doc]
