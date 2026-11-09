---
name: sirna-seed-match-counting
description: Use when you have DESeq2 differential expression results from an siRNA
  perturbation (e.
when_to_use_negative:
- Your input is raw RNA-seq reads or a feature count matrix rather than a DESeq2 results
  data frame containing gene_id, log2FoldChange, and padj columns — SeedMatchR() requires
  these columns to map seed matches to differential expression data
- Your sequences are not named consistently with gene or transcript IDs (without setting
  tx.id.col=TRUE for transcript-named sequences, match-to-gene summarization will
  fail)
- You need canonical-transcript or transcript-support-level filtering with rnor6,
  rnor7, or mm10 — these references cannot use ensembldb::TxIsCanonicalFilter or TxSupportLevelFilter
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: SeedMatchR
  role: Core R package providing SeedMatchR() seed-match counting, get_seed(), filter_res(),
    load_annotations(), and deseq_fc_ecdf() for ECDF/KS off-target testing
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Source of the differential expression results that SeedMatchR annotates with
    predicted seed matches
- name: AnnotationHub
  role: Provides the GTF and 2bit DNA resources (e.g., rnor6 AH93578/AH95846) from
    which load_annotations() builds 3'UTR sequences
- name: GenomicFeatures
  role: Used with getPromoterSeq to derive promoter DNAStringSet inputs for promoter-region
    seed-match searches
- name: ensembldb
  role: Backend for transcript feature extraction (e.g., 3UTR extraction); TxIsCanonicalFilter
    and TxSupportLevelFilter are unavailable for rnor6/rnor7
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:95cb685ec83b08ec7bf565146ccd11efbf3b305c64d51e35825010b52474ed30
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# sirna-seed-match-counting

## Summary

Count siRNA seed-region matches (e.g., mer7m8 from guide 'UUAUAGAGCAAGAACACUGUUUU') in 3'UTR or promoter sequences using the SeedMatchR R package, and annotate DESeq2 results with per-gene match counts to enable seed-mediated off-target detection via ECDF/KS analysis of log2 fold changes.

## When to use

Use when you have DESeq2 differential expression results from an siRNA perturbation (e.g., GalNAc-siRNA Ttr treatments from GSE184929) and need to test whether transcripts carrying a specific seed match (mer7m8/mer8) show cumulative differential expression relative to a background without seed matches.

## When NOT to use

- Your input is raw RNA-seq reads or a feature count matrix rather than a DESeq2 results data frame containing gene_id, log2FoldChange, and padj columns — SeedMatchR() requires these columns to map seed matches to differential expression data
- Your sequences are not named consistently with gene or transcript IDs (without setting tx.id.col=TRUE for transcript-named sequences, match-to-gene summarization will fail)
- You need canonical-transcript or transcript-support-level filtering with rnor6, rnor7, or mm10 — these references cannot use ensembldb::TxIsCanonicalFilter or TxSupportLevelFilter

## Inputs

- DESeq2 results data frame with columns gene_id, baseMean, log2FoldChange, padj (e.g., Schlegel_2022_Ttr_D1_30mkg from GSE184929)
- Named DNAStringSet of transcript feature sequences (3'UTRs from load_annotations with rnor6/rnor7/hg38/mm39/mm10, or promoter sequences from GenomicFeatures::getPromoterSeq)
- siRNA guide sequence (23 bp, 5'->3'), e.g., 'UUAUAGAGCAAGAACACUGUUUU' targeting Ttr
- Seed name specification, e.g., 'mer7m8' or 'mer8'

## Outputs

- DESeq2 results data frame annotated with a seed match count column (e.g., mer7m8) per gene
- Optionally, GRanges of match ranges when res.format='granges'
- ECDF plot object and KS test statistic/p-value from deseq_fc_ecdf comparing log2(FC) distributions of seed-match vs background gene sets

## How to apply

Install SeedMatchR from GitHub with devtools::install_github('tacazares/SeedMatchR'), then load species annotations with load_annotations(reference.name='rnor6', canonical=FALSE, min.feature.width=8, longest.utr=TRUE) to obtain 3'UTR-derived transcript sequences (rnor6 requires canonical=FALSE because it cannot use ensembldb::TxIsCanonicalFilter). Extract the seed with get_seed(guide.seq='UUAUAGAGCAAGAACACUGUUUU', seed.name='mer7m8') and run SeedMatchR(res, seqs=annodb$seqs, sequence=guide.seq, seed.name='mer7m8') on the filtered DESeq2 results (filter_res reduces the Ttr D1 30 mg/kg table from 32883 to 13582 rows by filtering on log2FoldChange, padj, baseMean and removing NAs). The SeedMatchR function appends match counts as a new column (e.g., 'mer7m8') keyed on the 'gene_id' column; if sequence names are transcript IDs rather than gene IDs, set tx.id.col=TRUE to summarize matches to the gene level using the GTF. Define gene sets (e.g., mer7m8 >= 1 with mer8 == 0 as targets, mer7m8 == 0 and mer8 == 0 as background) and compare log2(FC) ECDFs with deseq_fc_ecdf using a one-sided KS test (alternative='greater'); for promoter-level analysis, build a getPromoterSeq-based DNAStringSet with GenomicFeatures and pass it as the 'seqs' argument instead.

## Related tools

- **SeedMatchR** (Core R package providing SeedMatchR() seed-match counting, get_seed(), filter_res(), load_annotations(), and deseq_fc_ecdf() for ECDF/KS off-target testing) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Source of the differential expression results that SeedMatchR annotates with predicted seed matches)
- **AnnotationHub** (Provides the GTF and 2bit DNA resources (e.g., rnor6 AH93578/AH95846) from which load_annotations() builds 3'UTR sequences)
- **GenomicFeatures** (Used with getPromoterSeq to derive promoter DNAStringSet inputs for promoter-region seed-match searches)
- **ensembldb** (Backend for transcript feature extraction (e.g., 3UTR extraction); TxIsCanonicalFilter and TxSupportLevelFilter are unavailable for rnor6/rnor7)

## Examples

```
res = SeedMatchR(res = res, seqs = annodb$seqs, sequence = guide.seq, seed.name = "mer7m8")
```

## Evaluation signals

- filter_res on the Ttr D1 30 mg/kg input reduces the table from 32883 rows (8 columns) to ~13582 rows; a large deviation signals a filtering misconfiguration
- The annotated results data frame gains a numeric match-count column (e.g., 'mer7m8') with gene-level counts, and the known target Ttr (ENSRNOG00000016275) shows mer7m8 = 1
- The one-sided KS test (alternative='greater') yields a significant left-shift of the log2FC ECDF for mer7m8 seed-match genes vs the no-seed-match background, reproducing the published cumulative seed-mediated downregulation at Ttr D1 30 mg/kg
- deseq_fc_ecdf returns an ECDF plot object (ecdf.results$plot) plus D statistic and p-value; absence of either indicates the workflow did not complete
- Mutually exclusive gene sets are well-defined: mer7m8 targets require mer8 == 0 and background requires mer7m8 == 0 and mer8 == 0

## Limitations

- rnor6 and mm10 cannot use ensembldb::TxIsCanonicalFilter, so canonical=FALSE must be passed to load_annotations; rnor6/rnor7 also cannot use TxSupportLevelFilter, so transcript.support should not be passed for rat
- Using rnor7 requires R >= 4.3.0 and AnnotationHub >= 3.8.0
- Running load_annotations takes several minutes; results should be saved as an Rdata object for reuse
- Sequence names determine whether tx.id.col=TRUE is needed — gene-named sequences need no flag, transcript-named sequences require tx.id.col=TRUE with GTF info
- The forked repository adds biological target bulges and wobbles to the search space relative to the base implementation

## Evidence

- [readme] The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries.: "The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries."
- [abstract] SeedMatchR extends DESeq2 with seed-match annotation: "SeedMatchR is designed to extend current differential expression analysis tools, such as DESeq2, by annotating results with predicted seed matches."
- [intro] rnor6/mm10 canonical filter limitation: "`rnor6` and `mm10` cannot be used with the `ensembldb::TxIsCanonicalFilter` therefore the `canonical` argument for `SeedMatchR::load_annotations` must be set to `FALSE`."
- [readme] gene_id column requirement for mapping seed matches: [evidence span withheld — anchor status: external_doc]
- [readme] transcript-name tx.id.col requirement: [evidence span withheld — anchor status: not_found]
- [readme] ECDF + KS rationale for cumulative seed effects: "Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test."
- [results] getPromoterSeq promoter analysis for activation: [evidence span withheld — anchor status: not_found]
