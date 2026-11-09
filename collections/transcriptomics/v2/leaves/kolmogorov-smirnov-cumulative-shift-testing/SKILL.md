---
name: kolmogorov-smirnov-cumulative-shift-testing
description: Use when you have a DESeq2 results table plus gene sets (e.g., mer7m8/mer8
  seed-match targets vs. background) and need to test for cumulative down-regulation
  attributed to siRNA seed region activity.
when_to_use_negative:
- The input table lacks the required columns gene_id, log2FoldChange, and padj that
  SeedMatchR uses to map seed matches to differential expression data
- The goal is per-gene differential expression calls rather than detecting cumulative,
  seed-mediated shifts across a target gene set
- The results have not been annotated with seed-match counts (mer7m8/mer8), so no
  target vs. background gene sets can be defined
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: SeedMatchR
  role: Provides deseq_fc_ecdf() to compute the KS statistic and render the ECDF of
    log2(Fold Change) for seed-match vs. background gene sets
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Supplies the differential expression results data frame that deseq_fc_ecdf
    consumes directly
- name: GenomicFeatures
  role: Used with extractTranscriptSeqs()/threeUTRsByTranscript() to derive 3'UTR
    sequences whose seed matches define the tested gene sets
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:7d9d1226c6d756ecf24377edb59960e375108db47bfaa8a4763ba7ba4b6038e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - GenomicFeatures
  - R
  - SeedMatchR
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# kolmogorov-smirnov-cumulative-shift-testing

## Summary

Tests whether a defined gene set (e.g., siRNA mer7m8 seed-match targets) shows a cumulative shift in its log2(Fold Change) distribution versus a background set, using an ECDF plot coupled with a one-sided Kolmogorov-Smirnov test via SeedMatchR::deseq_fc_ecdf.

## When to use

Use when you have a DESeq2 results table plus gene sets (e.g., mer7m8/mer8 seed-match targets vs. background) and need to test for cumulative down-regulation attributed to siRNA seed region activity.

## When NOT to use

- The input table lacks the required columns gene_id, log2FoldChange, and padj that SeedMatchR uses to map seed matches to differential expression data
- The goal is per-gene differential expression calls rather than detecting cumulative, seed-mediated shifts across a target gene set
- The results have not been annotated with seed-match counts (mer7m8/mer8), so no target vs. background gene sets can be defined

## Inputs

- DESeq2 results data frame with gene_id, log2FoldChange, and padj columns (e.g., Schlegel_2022_Ttr_D1_30mkg from GSE184929)
- Annotated DESeq2 results with mer7m8/mer8 seed-match count columns from SeedMatchR()
- Gene sets: named list of gene_id vectors, e.g., list('Background' = background.list, 'mer7m8' = mer7m8.list)

## Outputs

- ecdf.results$plot: ECDF plot (5x5 inch) of log2(Fold Change) distributions per gene set
- One-sided Kolmogorov-Smirnov test result (D statistic and p-value) comparing seed-match targets to background

## How to apply

First filter the DESeq2 results with filter_res() (e.g., fdr_cutoff=1, fc_cutoff=0, removing NAs — D1 drops from 32883 to 13582 rows) and annotate rows with seed-match counts from SeedMatchR(res, seqs=annodb$seqs, sequence=guide.seq, seed.name='mer7m8'). Define gene sets from the annotated results: targets with mer7m8 >= 1 (and mer8 == 0), mer8 targets with mer8 >= 1, and background genes with mer7m8 == 0 and mer8 == 0. Run deseq_fc_ecdf() with stats_test='KS', alternative='greater', factor_order=c('Background','mer8','mer7m8') so the one-sided KS test detects whether the seed-match ECDF of log2(FC) is shifted toward down-regulation relative to background. Interpret a significant KS statistic as evidence of seed-mediated off-target activity; a non-significant result (as observed for the GNA-modified Ttr D4 condition) indicates the seed effect is mitigated.

## Related tools

- **SeedMatchR** (Provides deseq_fc_ecdf() to compute the KS statistic and render the ECDF of log2(Fold Change) for seed-match vs. background gene sets) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Supplies the differential expression results data frame that deseq_fc_ecdf consumes directly)
- **GenomicFeatures** (Used with extractTranscriptSeqs()/threeUTRsByTranscript() to derive 3'UTR sequences whose seed matches define the tested gene sets)

## Examples

```
ecdf.results = deseq_fc_ecdf(res, list("Background" = background.list, "mer7m8" = mer7m8.list)); ecdf.results$plot
```

## Evaluation signals

- For Ttr D1 30 mg/kg, the one-sided KS test (alternative='greater') yields a significant left-shift of the log2FC ECDF for mer7m8 seed-match genes relative to background, replicating the published result
- For the GNA-g7 modified Ttr D4 30 mg/kg condition (Fig. 1E), the same test shows no statistically significant down-shift, i.e., the seed-mediated effect is mitigated
- ecdf.results$plot returns a rendered ECDF figure; deseq_fc_ecdf runs without error on the filtered DESeq2 frame
- filter_res() reduces the D1 results from 32883 rows to 13582 rows before testing, confirming filtering was applied with fdr_cutoff=1 and fc_cutoff=0

## Limitations

- The input res data frame must contain the columns gene_id, log2FoldChange, and padj for seed-match mapping to differential expression data
- rnor6 and mm10 annotations cannot use ensembldb::TxIsCanonicalFilter, so canonical=FALSE must be passed to load_annotations; rnor7 requires R >= 4.3.0 and AnnotationHub >= 3.8.0
- Loading annotations takes several minutes; the README recommends saving annodb as an Rdata object for repeated workflows

## Evidence

- [readme] Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test.: "Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test."
- [methods] The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame.: "The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame."
- [readme] `SeedMatchR` provides functions for comparing the log2(Fold Change) of two gene sets.: "`SeedMatchR` provides functions for comparing the log2(Fold Change) of two gene sets."
- [methods] we demonstrate the ability of 'SeedMatchR' to detect cumulative changes in differential gene expression attributed to siRNA seed regions.: "we demonstrate the ability of 'SeedMatchR' to detect cumulative changes in differential gene 
expression attributed to siRNA seed regions."
