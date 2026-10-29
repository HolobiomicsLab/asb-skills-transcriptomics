---
name: seed-match-counting-in-3utr-transcript-sequences
description: Use when a DESeq2 RNA-seq results table from an siRNA treatment needs
  to be annotated with predicted seed matches (mer7m8, mer8) in 3'UTRs, or when you
  must test whether seed-match gene sets show a cumulative down-shift in log2FoldChange
  versus background.
when_to_use_negative:
- Your input is raw sequencing data or an unfiltered count matrix rather than a DESeq2
  results data frame with gene_id, log2FoldChange, and padj columns — run differential
  expression first.
- You need seed matches in coding sequences or full transcripts rather than 3'UTR
  features, or you are working with a species/annotation build outside the predefined
  human (hg38), mouse (mm10/mm39), and rat (rnor6/rnor7) sets supported by load_annotations.
- You only need general differential expression testing without seed-region off-target
  interpretation — DESeq2 alone suffices.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: SeedMatchR
  role: 'Core package: SeedMatchR() counts seed matches in annodb$seqs and appends
    counts to DESeq2 results; filter_res, load_annotations, get_example_data, and
    deseq_fc_ecdf support the workflow'
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Produces the differential expression results (log2FoldChange, padj) that SeedMatchR
    annotates and tests
- name: AnnotationHub
  role: Supplies the GTF and 2bit DNA sequence resources (e.g., Rnor_v6.0, AH93578/AH95846)
    from which load_annotations builds the 3'UTR sequence set
- name: GenomicFeatures
  role: Provides transcript sequence extraction patterns (extractTranscriptSeqs, threeUTRsByTranscript)
    that SeedMatchR's annotation format is designed to work with
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:bdcdb0aab14c717d60e29e0c08ad7c2e50330885af305f208275d7084c473e32
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
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# seed-match-counting-in-3utr-transcript-sequences

## Summary

Uses the SeedMatchR R package to count mer7m8/mer8 siRNA seed matches in 3'UTR transcript sequences and annotate DESeq2 differential expression results, enabling ECDF/KS testing for cumulative seed-mediated off-target down-regulation (e.g., GalNAc-siRNA Ttr D1 vs the GNA-modified Ttr D4 in rat liver).

## When to use

Use when a DESeq2 RNA-seq results table from an siRNA treatment needs to be annotated with predicted seed matches (mer7m8, mer8) in 3'UTRs, or when you must test whether seed-match gene sets show a cumulative down-shift in log2FoldChange versus background.

## When NOT to use

- Your input is raw sequencing data or an unfiltered count matrix rather than a DESeq2 results data frame with gene_id, log2FoldChange, and padj columns — run differential expression first.
- You need seed matches in coding sequences or full transcripts rather than 3'UTR features, or you are working with a species/annotation build outside the predefined human (hg38), mouse (mm10/mm39), and rat (rnor6/rnor7) sets supported by load_annotations.
- You only need general differential expression testing without seed-region off-target interpretation — DESeq2 alone suffices.

## Inputs

- DESeq2 results data frame containing gene_id, log2FoldChange, and padj columns (e.g., Schlegel_2022_Ttr_D1_30mkg, Schlegel_2022_Ttr_D4_30mkg, Schlegel_2022_Ttr_D1_10mkg from GEO accession GSE184929)
- 3'UTR transcript sequences and GTF-derived gene/transcript annotations from SeedMatchR::load_annotations (annodb$seqs; e.g., rnor6 via AnnotationHub 2bit AH93578 / ensembldb AH95846)
- siRNA guide sequence, 23 bp, oriented 5'->3' (e.g., 'UUAUAGAGCAAGAACACUGUUUU')

## Outputs

- DESeq2 results data frame annotated with seed match count columns (e.g., mer7m8) per gene
- Gene set lists: mer7m8 targets, mer8 targets, and background genes
- ECDF plot of log2(Fold Change) by gene set and Kolmogorov-Smirnov test results (ecdf.results$plot and test statistics)

## How to apply

Load species 3'UTR annotations with SeedMatchR::load_annotations (e.g., reference.name='rnor6', canonical=FALSE, min.feature.width=8, longest.utr=TRUE — rnor6 requires canonical=FALSE because it cannot use ensembldb::TxIsCanonicalFilter). Filter the DESeq2 results with filter_res (by log2FoldChange, padj, baseMean, removing NA entries; the README example goes from 32883 rows to 13582 then 8124) and pass the 23-bp guide sequence oriented 5'->3' (e.g., 'UUAUAGAGCAAGAACACUGUUUU') to SeedMatchR(res, seqs=annodb$seqs, sequence=guide.seq, seed.name='mer7m8'), which appends seed match counts as columns and requires the input to contain gene_id, log2FoldChange, and padj. Build gene sets — mer7m8 targets (mer7m8 >= 1 & mer8 == 0), mer8 targets (mer8 >= 1), background (mer7m8 == 0 & mer8 == 0) — and run deseq_fc_ecdf with stats_test='KS', alternative='greater' to render the ECDF of log2(Fold Change) and compute the Kolmogorov-Smirnov statistic. Rationale: miRNA-like perturbations produce cumulative shifts in their targets' expression, so a significant down-shift of seed-match genes versus background indicates a seed-mediated off-target effect; in the Ttr D4 GNA-g7 condition the down-shift is mitigated (Fig. 1E), whereas Ttr D1 shows it.

## Related tools

- **SeedMatchR** (Core package: SeedMatchR() counts seed matches in annodb$seqs and appends counts to DESeq2 results; filter_res, load_annotations, get_example_data, and deseq_fc_ecdf support the workflow) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Produces the differential expression results (log2FoldChange, padj) that SeedMatchR annotates and tests)
- **AnnotationHub** (Supplies the GTF and 2bit DNA sequence resources (e.g., Rnor_v6.0, AH93578/AH95846) from which load_annotations builds the 3'UTR sequence set)
- **GenomicFeatures** (Provides transcript sequence extraction patterns (extractTranscriptSeqs, threeUTRsByTranscript) that SeedMatchR's annotation format is designed to work with)

## Examples

```
res = SeedMatchR(res = res, seqs = annodb$seqs, sequence = guide.seq, seed.name = "mer7m8")
```

## Evaluation signals

- Annotated results must retain the required columns gene_id, log2FoldChange, and padj plus the appended mer7m8 count column; the top hit in the README example is Ttr (ENSRNOG00000016275, mer7m8 = 1).
- Row counts should change as expected after filter_res (README: 32883 -> 13582 -> 8124 rows) and no NA entries should remain.
- Gene set definitions must partition the filtered results: mer7m8 targets = mer7m8 >= 1 & mer8 == 0, mer8 targets = mer8 >= 1, background = mer7m8 == 0 & mer8 == 0.
- deseq_fc_ecdf should return a rendered ECDF plot plus KS statistics; for the D4 (GNA-g7) condition the mer7m8 set must show no statistically significant down-shift relative to background, while Ttr D1 shows a significant seed-mediated down-shift.
- If sequence names are transcript IDs rather than gene IDs, tx.id.col = TRUE must have been used so matches are summarized to the gene level via the GTF.

## Limitations

- rnor6 and mm10 cannot be used with ensembldb::TxIsCanonicalFilter, so canonical must be FALSE; rnor6/rnor7 also cannot be used with ensembldb::TxSupportLevelFilter, so transcript.support should not be passed for rat transcriptomes.
- rnor7 requires R >= 4.3.0 and AnnotationHub >= 3.8.0.
- load_annotations takes several minutes and should be cached as an Rdata object for repeated workflows; sequences shorter than min.feature.width (8) are dropped.
- The SeedMatchR function expects specific column names (gene_id, log2FoldChange, padj) when mapping seed matches to differential expression data, and tx.id.col must be set if seqs names are transcript IDs rather than gene IDs.

## Evidence

- [abstract] SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches.: "SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches."
- [methods] SeedMatchR() seed match counting: "You can perform a seed match for a single seed using the `SeedMatchR()` function."
- [methods] deseq_fc_ecdf works with DESeq2 results: "The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame."
- [intro] rnor6 canonical filter constraint: "`rnor6` and `mm10` cannot be used with the `ensembldb::TxIsCanonicalFilter` therefore the `canonical` argument for `SeedMatchR::load_annotations` must be set to `FALSE`."
- [readme] README: required input columns: [evidence span withheld — anchor status: external_doc]
- [readme] README: tx.id.col decision point: [evidence span withheld — anchor status: not_found]
