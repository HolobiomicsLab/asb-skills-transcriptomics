---
name: gencode-annotation-filtering
description: Use when building a pseudoalignment target (UTRome) from a GENCODE or
  other GTF/GFF annotation and transcripts must be restricted to protein-coding features
  with verified 3' ends before truncation.
when_to_use_negative:
- The goal is comprehensive noncoding RNA or novel-CS annotation — the filter discards
  lncRNA/noncoding transcripts and unvalidated ends (mRNA_end_NF) by design.
- The input annotation already encodes only validated protein-coding 3' ends (e.g.
  the deposited txcutr-db UTRome GTF), in which case re-filtering is redundant.
- Quantifying full-length transcript abundance (e.g. bulk RNA-seq alignment) rather
  than 3'-anchored isoform quantification, where 3'-end validation is not the relevant
  filter.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: txcutr
  role: consumes the filtered protein-coding/validated-3'-end transcriptome to generate
    truncated UTRome GTF/FASTA and merge tables
  repo: https://github.com/Mayrlab/txcutr-db
- name: Snakemake
  role: runs the txcutr-db pipeline that applies the filter and generates scUTRquant
    target files
  repo: https://github.com/Mayrlab/txcutr-db
- name: GENCODE
  role: source annotation (v39/vM21/vM25) whose gene_type and tag attributes drive
    the filter
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:e56e016a0f2d9b138e28ba2de02c43cdb2c4e4af616637df8f9d485132cc7b13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/txcutr-db
  tools:
  - GENCODE
  - Snakemake
  - scUTRquant
  - txcutr
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gencode-annotation-filtering

## Summary

Restrict a GENCODE GTF/GFF transcriptome annotation to protein-coding transcripts with validated 3' ends (excluding mRNA_end_NF-tagged transcripts) so that downstream 3'UTR truncation and quantification targets contain only reliably terminated transcripts.

## When to use

Use when building a pseudoalignment target (UTRome) from a GENCODE or other GTF/GFF annotation and transcripts must be restricted to protein-coding features with verified 3' ends before truncation.

## When NOT to use

- The goal is comprehensive noncoding RNA or novel-CS annotation — the filter discards lncRNA/noncoding transcripts and unvalidated ends (mRNA_end_NF) by design.
- The input annotation already encodes only validated protein-coding 3' ends (e.g. the deposited txcutr-db UTRome GTF), in which case re-filtering is redundant.
- Quantifying full-length transcript abundance (e.g. bulk RNA-seq alignment) rather than 3'-anchored isoform quantification, where 3'-end validation is not the relevant filter.

## Inputs

- GENCODE annotation GTF/GFF3 (e.g. GENCODE v39 human, vM21 mouse, vM25 for UTRome generation)
- transcript attribute table with gene_type/biotype and tag columns (incl. mRNA_end_NF)

## Outputs

- filtered GTF containing only protein-coding transcripts with validated 3' ends
- transcript ID list / merge-ready txome feeding txcutr truncateTxome and generateMergeTable

## How to apply

Load the annotation GTF (e.g. GENCODE v39 for human, vM21 for mouse, or vM25 when building the scUTRquant UTRome target) and parse the tag and gene_type/biotype attributes. Keep only records with a protein-coding transcript biotype whose 3' end is validated by PolyA_signal/manifest evidence, and drop any transcript carrying the mRNA_end_NF tag (NOVEL_'unvalidated terminator' in GENCODE). The rationale is that scUTRquant quantifies 3'UTR isoforms by pseudoaligning 10X or MWS 3'-tag reads against truncated transcripts, so unvalidated 3' ends would create spurious targets. Write the filtered transcript set back out as a GTF/GFF3 with the same coordinates, which then feeds txcutr truncateTxome (500 nt 3'-anchored truncation) and generateMergeTable (200 nt merge distance) in the txcutr-db Snakemake pipeline. Validate by checking the filtered feature counts and transcript IDs agree with the deposited txcutr-db reference (Zenodo 10.5281/zenodo.8118405).

## Related tools

- **txcutr** (consumes the filtered protein-coding/validated-3'-end transcriptome to generate truncated UTRome GTF/FASTA and merge tables) — https://github.com/Mayrlab/txcutr-db
- **Snakemake** (runs the txcutr-db pipeline that applies the filter and generates scUTRquant target files) — https://github.com/Mayrlab/txcutr-db
- **GENCODE** (source annotation (v39/vM21/vM25) whose gene_type and tag attributes drive the filter)

## Evaluation signals

- Every retained transcript record carries a protein-coding biotype; no lncRNA or noncoding features remain in the filtered GTF.
- Zero transcripts in the filtered set carry the mRNA_end_NF tag.
- Feature/transcript counts in the filtered GTF match the deposited txcutr-db reference for the corresponding GENCODE release (mouse vM25 comparison; Zenodo 10.5281/zenodo.8118405).
- Merge-group assignments after downstream txcutr generateMergeTable agree with the reference custom_target.merge.tsv.
- Downstream kallisto index (custom_target.kdx) builds without error from the filtered and truncated FASTA.

## Limitations

- Excluding mRNA_end_NF transcripts removes novel/unvalidated 3' termini, so CS not certified by GENCODE are not represented in this target.
- The filter is annotation-release-specific: the CS catalog used GENCODE v39 (human) / vM21 (mouse) while UTRome generation used vM25; mixing versions changes the retained set.
- Restricting to protein-coding transcripts means 3'UTR isoform changes in noncoding transcripts are invisible to the resulting quantification.

## Evidence

- [methods] filter recommendation: "We recommend filtering transcripts to only protein-coding transcripts with validated 3' ends"
- [methods] GENCODE versions used: "protein coding transcripts that have verified 3' ends in the GENCODE v39 and vM21 annotations"
- [methods] mRNA_end_NF exclusion: "excluding transcripts with the tag **mRNA_end_NF**"
- [readme] txcutr-db target generation: [evidence span withheld — anchor status: external_doc]
