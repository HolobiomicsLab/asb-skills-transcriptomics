---
name: sirna-off-target-mechanism-analysis
description: Use when RNA-seq differential expression results (e.g. DESeq2 output
  with gene_id, log2FoldChange, padj) from an siRNA treatment should be checked for
  cumulative down-regulation attributable to siRNA seed-region pairing in 3'UTRs.
when_to_use_negative:
- The input is not RNA-seq differential expression results (e.g. proteomics or raw
  counts without gene_id/log2FoldChange/padj columns) — SeedMatchR's res mode requires
  those specific columns
- The question is about a single gene's on-target knockdown rather than cumulative
  seed-mediated shifts across a gene set
- The organism lacks a predefined load_annotations species (built-ins cover human,
  rat, and mouse only)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
tools:
- name: SeedMatchR
  role: Core R package that annotates DESeq2 results with predicted seed matches and
    provides deseq_fc_ecdf for ECDF/KS testing of seed-mediated off-target effects
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Source of the differential expression results data.frame that SeedMatchR extends
    by annotating with predicted seed matches
- name: AnnotationHub
  role: Provides the 2bit DNA and ensembldb GTF resources (e.g. AH93578/AH95846 for
    rnor6) used by load_annotations; version >= 3.8.0 required for rnor7
- name: GenomicFeatures
  role: Used for extracting transcript sequences, e.g. GenomicFeatures::extractTranscriptSeqs()
    paired with GenomicFeatures::threeUTRsByTranscript() for building the 3'UTR search
    set
- name: ensembldb
  role: Provides transcript filters during annotation loading; note TxIsCanonicalFilter
    and TxSupportLevelFilter are unsupported for rnor6/rnor7
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:8e36e8986d6eb7c0317249b52ac82b1cd55673b77b0e6749bbd145a25cf6557f
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
  - ensembldb
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# sirna-off-target-mechanism-analysis

## Summary

Detect and visualize seed-mediated siRNA off-target effects by counting siRNA seed matches (e.g. mer7m8, mer8) in 3'UTR sequences, annotating DESeq2 results with per-gene match counts, and testing cumulative log2 fold-change shifts of seed-target gene sets versus background with an ECDF/Kolmogorov-Smirnov analysis.

## When to use

Use when RNA-seq differential expression results (e.g. DESeq2 output with gene_id, log2FoldChange, padj) from an siRNA treatment should be checked for cumulative down-regulation attributable to siRNA seed-region pairing in 3'UTRs.

## When NOT to use

- The input is not RNA-seq differential expression results (e.g. proteomics or raw counts without gene_id/log2FoldChange/padj columns) — SeedMatchR's res mode requires those specific columns
- The question is about a single gene's on-target knockdown rather than cumulative seed-mediated shifts across a gene set
- The organism lacks a predefined load_annotations species (built-ins cover human, rat, and mouse only)

## Inputs

- DESeq2 differential expression results data.frame with gene_id, log2FoldChange, and padj columns (e.g. Schlegel_2022_Ttr_D1_30mkg from GEO GSE184929, 32883 rows before filtering, 13582 after filter_res)
- siRNA guide sequence, 23 bp, 5'->3' oriented (e.g. UUAUAGAGCAAGAACACUGUUUU)
- AnnotationHub-derived annotation database (annodb) with GTF GRanges gene/transcript annotations and a DNAStringSet of 3'UTR feature sequences (e.g. rat rnor6, ENSEMBL 104, AH93578/AH95846)

## Outputs

- Annotated DESeq2 results data.frame with appended per-gene seed-match count columns (e.g. mer7m8, mer8)
- ECDF plot of log2(Fold Change) for seed-target versus background gene sets
- Kolmogorov-Smirnov test statistics comparing seed-match gene sets to background
- Optionally, GRanges of seed-match ranges (res.format='granges') or a SeedMatchReport object

## How to apply

Load species annotations with SeedMatchR::load_annotations (e.g. reference.name='rnor6', canonical=FALSE, min.feature.width=8, longest.utr=TRUE) to obtain a GTF GRanges object and a DNAStringSet of 3'UTR sequences. Filter the DESeq2 results with filter_res() (by log2FoldChange, padj, baseMean, removing NA entries), extract the seed from the 23-bp guide (e.g. 'UUAUAGAGCAAGAACACUGUUUU') using get_seed() with seed.name='mer7m8', then call SeedMatchR() with vcountPattern-style exact matching (max.mismatch=0, no indels) to append per-gene seed counts to the results frame. Define gene sets (mer7m8 targets: mer7m8>=1 & mer8==0; mer8 targets: mer8>=1; background: both zero) and run deseq_fc_ecdf() with stats_test='KS', alternative='greater' to render the ECDF of log2(Fold Change) and compute Kolmogorov-Smirnov statistics. A significant down-shift of seed-match gene sets relative to background indicates seed-mediated off-target activity; its absence (as for the GNA-modified Ttr D4 at position g7) indicates mitigation of the effect.

## Related tools

- **SeedMatchR** (Core R package that annotates DESeq2 results with predicted seed matches and provides deseq_fc_ecdf for ECDF/KS testing of seed-mediated off-target effects) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Source of the differential expression results data.frame that SeedMatchR extends by annotating with predicted seed matches)
- **AnnotationHub** (Provides the 2bit DNA and ensembldb GTF resources (e.g. AH93578/AH95846 for rnor6) used by load_annotations; version >= 3.8.0 required for rnor7)
- **GenomicFeatures** (Used for extracting transcript sequences, e.g. GenomicFeatures::extractTranscriptSeqs() paired with GenomicFeatures::threeUTRsByTranscript() for building the 3'UTR search set)
- **ensembldb** (Provides transcript filters during annotation loading; note TxIsCanonicalFilter and TxSupportLevelFilter are unsupported for rnor6/rnor7)

## Examples

```
res = SeedMatchR(res = res, seqs = annodb$seqs, sequence = guide.seq, seed.name = "mer7m8"); ecdf.results = deseq_fc_ecdf(res, list("Background" = background.list, "mer7m8" = mer7m8.list))
```

## Evaluation signals

- The annotated output table's mer7m8 column values match counts produced by the package's own SeedMatchR() call on identical inputs (per task_005 validation)
- The filtered results frame has the expected dimensions (e.g. 13582 rows x 8 columns after filter_res on the 32883-row Schlegel_2022_Ttr_D1_30mkg input)
- The output frame contains the required columns gene_id, log2FoldChange, padj plus the appended seed-count column, and Ttr shows mer7m8 = 1 in the README example
- For Ttr D1 (unmodified seed), the mer7m8 gene set shows a statistically significant down-shift of log2FC ECDF versus background (KS, alternative='greater'); for the GNA g7-modified Ttr D4, no significant down-shift
- Seed counts derive from exact matching semantics: max.mismatch = 0 and no indels via vcountPattern over named 3'UTR sequences, aggregated to gene_id

## Limitations

- rnor6 and mm10 cannot be used with ensembldb::TxIsCanonicalFilter, so canonical must be FALSE; rnor6/rnor7 also cannot use TxSupportLevelFilter, so transcript.support should not be passed for rat
- rnor7 requires R 4.3.0 or greater because AnnotationHub >= 3.8.0 is required
- Running load_annotations takes several minutes; the README recommends saving annotation objects as Rdata for repeated workflows
- If sequence names are transcript IDs rather than gene IDs, the tx.id.col argument must be set to TRUE to summarize transcript matches to the gene level via the GTF
- The forked repository adds biological target bulges and wobbles to the search space; the fork's owner notes the package looks for specific columns (gene_id, log2FoldChange, padj) in res mode

## Evidence

- [abstract] SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches.: "SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches."
- [methods] The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries.: "The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries."
- [readme] The input must contain the columns `gene_id`, `log2FoldChange`, and `padj`.: [evidence span withheld — anchor status: external_doc]
- [readme] If they sequence names are transcripts, then the argument `tx.id.col` should be set to `TRUE`.: [evidence span withheld — anchor status: not_found]
- [intro] `rnor6` and `mm10` cannot be used with the `ensembldb::TxIsCanonicalFilter`: "`rnor6` and `mm10` cannot be used with the `ensembldb::TxIsCanonicalFilter` therefore the `canonical` argument for `SeedMatchR::load_annotations` must be set to `FALSE`."
- [readme] Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test.: "Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test."
