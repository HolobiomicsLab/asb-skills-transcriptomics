---
name: sirna-seed-region-definition
description: Use when you have an siRNA guide sequence and need to define its seed
  region (e.g., mer7m8) and count guide matches — canonical or non-canonical with
  wobbles/bulges/3'-compensatory — across transcript/UTR sequences to interpret seed-mediated
  off-target effects in RNA-seq data.
when_to_use_negative:
- The analysis goal is differential expression testing itself rather than annotating
  results with seed matches — run DESeq2 first; SeedMatchR only annotates an existing
  results frame containing gene_id, log2FoldChange, and padj.
- A full-length or cleavage-oriented guide-target alignment is needed rather than
  seed-mediated (mer7m8 / wobble / bulge / 3'-compensatory) off-target enumeration.
- The sequence set uses gene symbols or lacks the sequence names/tx.id structure SeedMatchR
  expects (tx.id.col=TRUE is needed when sequence names are transcript IDs, not gene
  IDs).
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0659
tools:
- name: SeedMatchR
  role: Forked R package (2.0.0) providing get_seed, SeedMatchR, full_search, run_wobble_bulges_pipeline,
    and get_compensatory_sequences to define the guide seed and enumerate canonical
    and non-canonical matches
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Source of the differential expression results frame that SeedMatchR annotates
    with predicted seed match counts
- name: AnnotationHub
  role: Supplies the rnor6 GTF and 2bit DNA sequence files consumed by load_annotations
    (>= 3.8.0 required for rnor7)
- name: GenomicFeatures
  role: Basis for deriving transcript feature sequences, e.g., GenomicFeatures::extractTranscriptSeqs()
    paired with GenomicFeatures::threeUTRsByTranscript()
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:9f4daf6fb1d64ab001cccf646dc024b9d793eafa11d5000c09358de97667ecb6
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# sirna-seed-region-definition

## Summary

Defines and uses the siRNA seed region (e.g., the mer7m8 seed extracted from the 23-nt D1 guide 'UUAUAGAGCAAGAACACUGUUUU') to enumerate seed-mediated off-target matches against reference transcriptomes, extending the canonical seed search to non-canonical wobble, target-bulge, and 3'-compensatory matches via the forked SeedMatchR package.

## When to use

Use when you have an siRNA guide sequence and need to define its seed region (e.g., mer7m8) and count guide matches — canonical or non-canonical with wobbles/bulges/3'-compensatory — across transcript/UTR sequences to interpret seed-mediated off-target effects in RNA-seq data.

## When NOT to use

- The analysis goal is differential expression testing itself rather than annotating results with seed matches — run DESeq2 first; SeedMatchR only annotates an existing results frame containing gene_id, log2FoldChange, and padj.
- A full-length or cleavage-oriented guide-target alignment is needed rather than seed-mediated (mer7m8 / wobble / bulge / 3'-compensatory) off-target enumeration.
- The sequence set uses gene symbols or lacks the sequence names/tx.id structure SeedMatchR expects (tx.id.col=TRUE is needed when sequence names are transcript IDs, not gene IDs).

## Inputs

- siRNA guide sequence, RNA string oriented 5'->3' (e.g., 23-nt D1 'UUAUAGAGCAAGAACACUGUUUU')
- Feature DNAStringSet and GTF annotations from SeedMatchR::load_annotations (e.g., rnor6 3UTR sequences, min.feature.width=8, longest.utr=TRUE)
- Optional DESeq2 results data.frame with required columns gene_id, log2FoldChange, and padj (e.g., Schlegel_2022_Ttr_D1_30mkg from GEO GSE184929)

## Outputs

- Per-gene seed match count columns (e.g., mer7m8) merged onto the DESeq2 results frame
- match.df / GRanges of guide matches with transcript IDs and match coordinates (full_search, res.format='granges')
- comp_matches data.frame listing TX_ID and match coordinates for 3'-compensatory regions
- Wobble/bulge and compensatory-region count tables summarized per seed type and mismatch/indel setting; optional deseq_fc_ecdf ECDF plots with KS test

## How to apply

Install the forked package with devtools::install_github('tacazares/SeedMatchR') (SeedMatchR 2.0.0, R >= 4.3.0, AnnotationHub >= 3.8.0). Load species annotations — for rat use SeedMatchR::load_annotations(reference.name='rnor6', canonical=FALSE, min.feature.width=8, longest.utr=TRUE) — to obtain the feature DNAStringSet annodb$seqs; canonical must be FALSE for rnor6 because ensembldb::TxIsCanonicalFilter is unsupported. Define the guide 5'->3' (e.g., D1 = 'UUAUAGAGCAAGAACACUGUUUU' targeting Ttr in rat liver) and extract/declare the seed with get_seed() or seed.name='mer7m8', then count per-gene seed matches with SeedMatchR() (which annotates a DESeq2 results frame having gene_id, log2FoldChange, and padj columns) or full_search() for multiple siRNA views. To go beyond the canonical mer7m8 search, run the fork's run_wobble_bulges_pipeline on annodb$seqs to enumerate matches allowing biological target bulges and wobbles, and get_compensatory_sequences with allow_wobbles and indel options to derive a comp_matches data.frame of TX_ID and match coordinates for 3'-compensatory regions. Summarize match counts per seed type and mismatch/indel setting (col.name, max.mismatch, with.indels analogous to the inexact-match interface) into wobble/bulge and compensatory count tables, optionally paired with deseq_fc_ecdf KS tests comparing mer7m8-target vs background log2 fold changes.

## Related tools

- **SeedMatchR** (Forked R package (2.0.0) providing get_seed, SeedMatchR, full_search, run_wobble_bulges_pipeline, and get_compensatory_sequences to define the guide seed and enumerate canonical and non-canonical matches) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Source of the differential expression results frame that SeedMatchR annotates with predicted seed match counts)
- **AnnotationHub** (Supplies the rnor6 GTF and 2bit DNA sequence files consumed by load_annotations (>= 3.8.0 required for rnor7))
- **GenomicFeatures** (Basis for deriving transcript feature sequences, e.g., GenomicFeatures::extractTranscriptSeqs() paired with GenomicFeatures::threeUTRsByTranscript())

## Examples

```
guide.seq = "UUAUAGAGCAAGAACACUGUUUU"; annodb = load_annotations(reference.name="rnor6", canonical=FALSE, min.feature.width=8, longest.utr=TRUE); res = SeedMatchR(res=res, seqs=annodb$seqs, sequence=guide.seq, seed.name="mer7m8")
```

## Evaluation signals

- load_annotations log output shows 'Keep only the ENSEMBL canonical transcript: FALSE' and 'Keeping sequences that are >= 8' for the rnor6 3UTR search space
- Per-gene seed counts are non-negative integers and partition the filtered results into res$mer7m8 >= 1 (targets) vs res$mer7m8 == 0 (background) gene sets
- comp_matches is a data.frame that contains TX_ID and match coordinate columns for 3'-compensatory regions
- Filtering sanity check: filter_res on Schlegel_2022_Ttr_D1_30mkg shrinks 32883 rows to 13582 before seed counting
- SeedMatchR-annotated frame retains one row per gene_id with the seed count column named per seed.name (e.g., 'mer7m8'); example Ttr gene ENSRNOG00000016275 carries mer7m8 = 1

## Limitations

- rnor6 and mm10 cannot be used with ensembldb::TxIsCanonicalFilter, so the canonical argument to SeedMatchR::load_annotations must be FALSE for these references
- rnor6 and rnor7 cannot be used with ensembldb::TxSupportLevelFilter; do not pass transcript.support when working with rat transcriptomes
- rnor7 requires R >= 4.3.0 because AnnotationHub >= 3.8.0 is required; load_annotations takes several minutes, so save annotations as Rdata objects for reuse
- The wobble/bulge and compensatory search capabilities exist only in the forked repository (tacazares/SeedMatchR), whose stated purpose is to add biological target bulges and wobbles to the search space

## Evidence

- [readme] The goal of SeedMatchR is to help users identify potential seed-mediated effects in their RNA-seq data.: "The goal of SeedMatchR is to help users identify potential seed-mediated
effects in their RNA-seq data."
- [intro] Fork adds bulges and wobbles to the search space: "These changes in this forked repository is to add the biological target
bulges and wobbles in the search space."
- [methods] get_seed extracts seed sequence information from the siRNA input: "You can extract the seed sequence information from the siRNA input sequence using the `get_seed()` function."
- [readme] SeedMatchR requires DESeq2 result columns gene_id, log2FoldChange, padj: "The input must contain the columns `gene_id`,
`log2FoldChange`, and `padj`."
- [intro] rnor6 requires canonical=FALSE in load_annotations: "`rnor6` and `mm10` cannot be used with the `ensembldb::TxIsCanonicalFilter` therefore the `canonical` argument for `SeedMatchR::load_annotations` must be set to `FALSE`."
