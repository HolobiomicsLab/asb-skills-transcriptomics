---
name: utr-sequence-seed-match-counting
description: Use when you have a DESeq2 results data.frame and an siRNA guide sequence,
  and need per-gene counts of a defined seed (e.g., mer7m8) across 3'UTRs to test
  for seed-mediated off-target activity.
when_to_use_negative:
- Input results frame lacks the gene_id, log2FoldChange, or padj columns required
  by SeedMatchR's res argument
- You need mismatch-tolerant or indel-permitting seed search (e.g., target bulges
  and wobbles) rather than exact vcountPattern matching
- Sequences are full transcripts or CDS rather than the 3'UTR features the annotation
  loader extracts
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3308
tools:
- name: SeedMatchR
  role: Primary function that counts seed matches per feature and appends counts to
    the DE results frame
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Source of the differential expression results data.frame being annotated
- name: AnnotationHub
  role: Supplies the GTF and DNA sequence files (e.g., Rnor_v6.0, AH93578/AH95846)
    behind load_annotations
- name: GenomicFeatures
  role: Underlies extraction of 3'UTR feature sequences (threeUTRsByTranscript / extractTranscriptSeqs)
    used as the search space
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:10a02d27dcdcab2113c8245ae4bb45ee57af26c1576a3208fd5eb8802a0789c9
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
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# utr-sequence-seed-match-counting

## Summary

Annotate a differential expression results frame with per-gene counts of siRNA seed matches (e.g., mer7m8) found in 3'UTR feature sequences, using exact-pattern counting over a reference transcriptome so seed-mediated off-target effects can be detected in RNA-seq data.

## When to use

Use when you have a DESeq2 results data.frame and an siRNA guide sequence, and need per-gene counts of a defined seed (e.g., mer7m8) across 3'UTRs to test for seed-mediated off-target activity.

## When NOT to use

- Input results frame lacks the gene_id, log2FoldChange, or padj columns required by SeedMatchR's res argument
- You need mismatch-tolerant or indel-permitting seed search (e.g., target bulges and wobbles) rather than exact vcountPattern matching
- Sequences are full transcripts or CDS rather than the 3'UTR features the annotation loader extracts

## Inputs

- Differential expression results data.frame (DESeq2) with columns gene_id, log2FoldChange, padj
- DNAStringSet of feature (3'UTR) sequences with gene_id or transcript names, from load_annotations (e.g., rnor6 GTF GRanges plus sequences)
- siRNA guide sequence, 23 bp, 5'->3' oriented (e.g., UUAUAGAGCAAGAACACUGUUUU)
- Seed name (e.g., mer7m8) or seed derived via get_seed()

## Outputs

- DE results data.frame with an appended per-gene seed-match count column (e.g., mer7m8)
- Optionally, match ranges as GRanges when res.format = "granges"

## How to apply

Load species annotations via SeedMatchR::load_annotations (e.g., reference.name = "rnor6", canonical = FALSE since rnor6 cannot use ensembldb::TxIsCanonicalFilter, min.feature.width = 8, longest.utr = TRUE) to obtain a DNAStringSet of 3'UTR sequences keyed to gene_id. Derive the seed from the 23-bp guide (e.g., "UUAUAGAGCAAGAACACUGUUUU") with get_seed(guide.seq, seed.name = "mer7m8"). Filter the DESeq2 results with filter_res() to remove NA entries, ensuring the frame has the required gene_id, log2FoldChange, and padj columns. Then call SeedMatchR(), which counts seed matches per feature using vcountPattern semantics (exact matching, max.mismatch = 0, no indels), aggregates transcript-level matches to the gene level when sequence names are transcripts (tx.id.col = TRUE), and appends a seed-count column (e.g., mer7m8) to the results frame. Confirm correctness by checking that counts match the package's own output on identical inputs (e.g., Ttr should carry mer7m8 = 1 in the Schlegel_2022_Ttr_D1_30mkg example).

## Related tools

- **SeedMatchR** (Primary function that counts seed matches per feature and appends counts to the DE results frame) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Source of the differential expression results data.frame being annotated)
- **AnnotationHub** (Supplies the GTF and DNA sequence files (e.g., Rnor_v6.0, AH93578/AH95846) behind load_annotations)
- **GenomicFeatures** (Underlies extraction of 3'UTR feature sequences (threeUTRsByTranscript / extractTranscriptSeqs) used as the search space)

## Examples

```
res = SeedMatchR(res = res, seqs = annodb$seqs, sequence = guide.seq, seed.name = "mer7m8")
```

## Evaluation signals

- Output frame retains the filtered row count (13,582 rows from 32,883 after filter_res with fdr_cutoff=1, fc_cutoff=0) and gains a mer7m8 column
- Ttr (ENSRNOG00000016275) shows mer7m8 = 1 in the Schlegel_2022_Ttr_D1_30mkg example; genes with no seed match show 0
- Counts are non-negative integers aggregated to gene_id, with transcript-level matches summed when tx.id.col = TRUE
- Recomputing counts independently with vcountPattern (exact, max.mismatch = 0, no indels) over annodb$seqs reproduces the package's column values
- Head of output matches the README example rows (Ttr with -8.16 log2FoldChange, Kpna6 with 0 counts)

## Limitations

- rnor6 and mm10 cannot be used with ensembldb::TxIsCanonicalFilter, so canonical must be FALSE; rnor6/rnor7 also cannot use TxSupportLevelFilter
- rnor7 requires R >= 4.3.0 and AnnotationHub >= 3.8.0
- Sequence-name semantics matter: gene-ID names need no extra flags, transcript names require tx.id.col = TRUE to summarize to gene level
- The base matching is exact (max.mismatch = 0, no indels), so bulged or wobbled seed alignments are not captured

## Evidence

- [readme] README column requirement: [evidence span withheld — anchor status: external_doc]
- [readme] README tx.id.col semantics: [evidence span withheld — anchor status: not_found]
- [methods] Methods: seed extraction: "You can extract the seed sequence information from the siRNA input sequence using the `get_seed()` function."
- [methods] Methods: filter_res: "The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries."
- [intro] Intro: rnor6 canonical filter restriction: "`rnor6` and `mm10` cannot be used with the `ensembldb::TxIsCanonicalFilter`"
