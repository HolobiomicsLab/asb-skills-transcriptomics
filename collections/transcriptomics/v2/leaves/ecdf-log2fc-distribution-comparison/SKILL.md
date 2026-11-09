---
name: ecdf-log2fc-distribution-comparison
description: Use when you have DESeq2 differential expression results from an siRNA
  perturbation and want to test whether transcripts carrying seed matches (e.g. mer7m8/mer8)
  show a cumulative log2FC shift versus background.
when_to_use_negative:
- Input is raw count data or an unannotated DESeq2 results frame lacking gene_id/log2FoldChange/padj
  columns — seed matches must first be appended via SeedMatchR()
- You need per-guide exhaustive match views across multiple seed types rather than
  an ECDF comparison — use full_search() or SeedMatchReport() instead
- The perturbation effect is expected to be a single strong driver rather than cumulative
  seed-mediated repression, making a distributional ECDF/KS comparison uninformative
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SeedMatchR
  role: Provides SeedMatchR(), filter_res(), and deseq_fc_ecdf() to annotate DESeq2
    results with seed matches and run the ECDF/KS comparison
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Supplies the differential expression results data frame (log2FoldChange, padj)
    that deseq_fc_ecdf consumes directly
- name: AnnotationHub
  role: 'Provides the GTF and DNA sequence files used by load_annotations to derive
    3''UTR transcript sequences (rnor6: AH93578/AH95846)'
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:5bc3157ce8c5f2c6574c0f84edeaaaab3e1906442f36ceccaf3c78f2dd3a889d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - R
  - SeedMatchR
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ecdf-log2fc-distribution-comparison

## Summary

Compares empirical cumulative distribution functions of log2(fold change) between siRNA seed-match target gene sets and a no-match background from DESeq2 results, using a one-sided Kolmogorov-Smirnov test to detect cumulative seed-mediated off-target downregulation.

## When to use

Use when you have DESeq2 differential expression results from an siRNA perturbation and want to test whether transcripts carrying seed matches (e.g. mer7m8/mer8) show a cumulative log2FC shift versus background.

## When NOT to use

- Input is raw count data or an unannotated DESeq2 results frame lacking gene_id/log2FoldChange/padj columns — seed matches must first be appended via SeedMatchR()
- You need per-guide exhaustive match views across multiple seed types rather than an ECDF comparison — use full_search() or SeedMatchReport() instead
- The perturbation effect is expected to be a single strong driver rather than cumulative seed-mediated repression, making a distributional ECDF/KS comparison uninformative

## Inputs

- DESeq2 results data frame with columns gene_id, log2FoldChange, padj (e.g. Schlegel_2022_Ttr_D1_30mkg from GSE184929)
- 23-bp siRNA guide sequence oriented 5'->3' (e.g. UUAUAGAGCAAGAACACUGUUUU)
- 3'UTR-derived transcript sequences (annodb$seqs, DNAStringSet) from load_annotations for rnor6
- Seed name (e.g. 'mer7m8') extracted with get_seed()

## Outputs

- Annotated DESeq2 results data frame with seed-match count columns (e.g. mer7m8)
- ECDF plot of log2(Fold Change) per gene set (ecdf.results$plot, 5x5 inches)
- Kolmogorov-Smirnov test statistic and p-value comparing target vs background ECDFs

## How to apply

Annotate the filtered DESeq2 results (via filter_res with fdr_cutoff=1, fc_cutoff=0, e.g. 32883 rows reduced to 13582 for Schlegel_2022_Ttr_D1_30mkg) with seed-match counts by running SeedMatchR(res, seqs=annodb$seqs, sequence=guide.seq, seed.name='mer7m8') on 3'UTR transcript sequences loaded with load_annotations(reference.name='rnor6', canonical=FALSE, min.feature.width=8, longest.utr=TRUE). Build mutually exclusive gene lists: mer7m8 targets (mer7m8 >= 1 & mer8 == 0), mer8 targets (mer8 >= 1), and a background set (both == 0). Call deseq_fc_ecdf(res, gene.lists, stats_test='KS', null_name='Background', factor_order=c('Background','mer8','mer7m8'), alternative='greater', height=5, width=5) to render the ECDF of log2(FC) and return the KS statistic and p-value. The one-sided 'greater' alternative tests for a left-shift (downregulation) of seed-match genes, mirroring the cumulative target repression seen with miRNA-like seed activity; the same design applies to promoter-sequence features to probe high-dose siRNA gene activation (Fig. 1G).

## Related tools

- **SeedMatchR** (Provides SeedMatchR(), filter_res(), and deseq_fc_ecdf() to annotate DESeq2 results with seed matches and run the ECDF/KS comparison) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Supplies the differential expression results data frame (log2FoldChange, padj) that deseq_fc_ecdf consumes directly)
- **AnnotationHub** (Provides the GTF and DNA sequence files used by load_annotations to derive 3'UTR transcript sequences (rnor6: AH93578/AH95846))

## Examples

```
ecdf.results = deseq_fc_ecdf(res, list("Background" = background.list, "mer7m8" = mer7m8.list))
```

## Evaluation signals

- For Schlegel_2022_Ttr_D1_30mkg, the one-sided KS test (alternative='greater') yields a significant left-shift of the log2FC ECDF for mer7m8 targets vs background, reproducing Fig. 1
- filter_res reduces the D1 30 mg/kg table from 32883 to 13582 rows (README shows the intermediate 8124-row dim output)
- The annotated results gain a seed-match count column (e.g. Ttr shows mer7m8 == 1) and the input columns gene_id, log2FoldChange, padj are preserved
- Gene lists are mutually exclusive: mer7m8 set requires mer8 == 0, background requires both counts == 0
- For GNA-modified Ttr D4 (g7), the mer7m8 set shows NO significant down-shift relative to background (Fig. 1E); for promoter-sequence analysis (Fig. 1G), a slight enrichment of positive log2FCs among seed-match genes is expected

## Limitations

- rnor6 and mm10 cannot use ensembldb::TxIsCanonicalFilter, so load_annotations must be called with canonical = FALSE; rat references also cannot use TxSupportLevelFilter
- rnor7 requires R >= 4.3.0 and AnnotationHub >= 3.8.0
- The SeedMatchR() res argument requires the columns gene_id, log2FoldChange, and padj; if sequence names are transcript IDs, tx.id.col must be set TRUE to summarize to gene level
- The forked repository adds biological target bulges and wobbles to the search space, so results may differ from the CRAN/stable build

## Evidence

- [abstract] SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches.: "SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches."
- [methods] The function deseq_fc_ecdf is designed to work directly with a DESeq2 results data frame.: "The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame."
- [readme] Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test.: "Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test"
- [methods] The function filter_res() can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries.: "The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries."
- [readme] The input must contain the columns gene_id, log2FoldChange, and padj.: "The input must contain the columns `gene_id`,
`log2FoldChange`, and `padj`."
