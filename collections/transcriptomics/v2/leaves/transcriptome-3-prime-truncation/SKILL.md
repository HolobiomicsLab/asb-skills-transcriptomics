---
name: transcriptome-3-prime-truncation
description: Use when building a custom pseudoalignment target for 3'-end tag scRNA-seq
  data (e.g., 10X Chromium) where reads do not span cleavage sites and full-length
  transcripts would collapse 3'UTR isoforms.
when_to_use_negative:
- Reads come from a full-length or long-read protocol that spans cleavage sites —
  de novo 3' end mapping, not 3'-anchored truncation, is appropriate.
- You only need gene-level counts; a standard transcriptome/kallisto index without
  UTRome truncation suffices.
- A pre-built txcutr-db UTRome index already exists for your exact annotation release
  — reuse it rather than regenerating.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: txcutr
  role: Performs the 3'-anchored truncation (truncateTxome, 500 nt), merge-table generation
    (generateMergeTable, 200 nt distance), and GTF/FASTA export of the truncated UTRome
  repo: https://github.com/Mayrlab/txcutr-db
- name: kallisto
  role: Builds the pseudoalignment index (kdx) over the truncated transcripts and
    is the quantification engine whose resolution limit motivates the 200-nt merge
    distance
- name: Snakemake
  role: Orchestrates the UTRome generation pipeline in the txcutr-db repository
  repo: https://github.com/Mayrlab/txcutr-db
- name: scUTRquant
  role: Downstream consumer of the truncated UTRome for simultaneous gene and 3'UTR
    isoform quantification
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:f3979695dc571b1c0e35b88521e8bc900308b5b354ab2fc469fea1ee8b5cc489
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/txcutr-db
  tools:
  - Snakemake
  - kallisto
  - scUTRquant
  - txcutr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# transcriptome-3-prime-truncation

## Summary

Generate a 3'-truncated UTRome (e.g., via txcutr) from a GENCODE GTF/GFF annotation so that kallisto bus pseudoalignment of 3'-tagged scRNA-seq reads can distinguish 3'UTR isoforms; the 500-nt truncation is empirically justified for 10X Chromium 3' libraries.

## When to use

Use when building a custom pseudoalignment target for 3'-end tag scRNA-seq data (e.g., 10X Chromium) where reads do not span cleavage sites and full-length transcripts would collapse 3'UTR isoforms.

## When NOT to use

- Reads come from a full-length or long-read protocol that spans cleavage sites — de novo 3' end mapping, not 3'-anchored truncation, is appropriate.
- You only need gene-level counts; a standard transcriptome/kallisto index without UTRome truncation suffices.
- A pre-built txcutr-db UTRome index already exists for your exact annotation release — reuse it rather than regenerating.

## Inputs

- GENCODE GTF/GFF transcriptome annotation (e.g., vM25 mouse or v39 human)
- Transcript FASTA corresponding to the annotation
- txcutr-db Snakemake reference pipeline for validation

## Outputs

- custom_target.gtf (3'-truncated transcript features)
- custom_target.merge.tsv (merge groups of collapsed truncated transcripts)
- custom_target.kdx (kallisto index over the truncated UTRome)

## How to apply

Start from a GENCODE annotation (vM25 for the mouse UTRome in this study) and filter to protein-coding transcripts with validated 3' ends, excluding transcripts tagged mRNA_end_NF. Apply txcutr truncateTxome with a 500-nt 3'-anchored truncation length — chosen empirically because more than 95% of 3' end reads of tested reference genes map within 500 nt upstream of cleavage sites, matching 10X Chromium 3' Single Cell library geometry. Deduplicate identical 3'-truncated sequences so they collapse to a single feature, then run txcutr generateMergeTable with a 200-nt merge distance, matching the empirical resolution limit of kallisto quantification. Export with txcutr exportGTF/exportFASTA (custom_target.gtf, custom_target.merge.tsv) and build the kallisto index (custom_target.kdx). Finally, validate feature counts and merge-group assignments against the deposited txcutr-db reference index (Zenodo 10.5281/zenodo.8118405).

## Related tools

- **txcutr** (Performs the 3'-anchored truncation (truncateTxome, 500 nt), merge-table generation (generateMergeTable, 200 nt distance), and GTF/FASTA export of the truncated UTRome) — https://github.com/Mayrlab/txcutr-db
- **kallisto** (Builds the pseudoalignment index (kdx) over the truncated transcripts and is the quantification engine whose resolution limit motivates the 200-nt merge distance)
- **Snakemake** (Orchestrates the UTRome generation pipeline in the txcutr-db repository) — https://github.com/Mayrlab/txcutr-db
- **scUTRquant** (Downstream consumer of the truncated UTRome for simultaneous gene and 3'UTR isoform quantification) — https://github.com/Mayrlab/scUTRquant

## Examples

```
Rscript -e 'txome_trunc <- txcutr::truncateTxome(txome, max_length=500); txcutr::generateMergeTable(txome_trunc, distance=200, out_prefix="custom_target"); txcutr::exportGTF(txome_trunc, "custom_target.gtf"); txcutr::exportFASTA(txome_trunc, "custom_target.fa"); kallisto index -i custom_target.kdx custom_target.fa'
```

## Evaluation signals

- custom_target.gtf, custom_target.kdx, and custom_target.merge.tsv all exist and parse without error.
- Transcript/feature counts in the generated GTF agree with the deposited txcutr-db reference for the same GENCODE release.
- Merge-group assignments in custom_target.merge.tsv match the txcutr-db reference groups.
- All retained transcripts are protein-coding with validated 3' ends and none carry the mRNA_end_NF tag.
- Truncated sequences are 3'-anchored with a maximum 500-nt length and identical 3'-truncated sequences collapse to single features.

## Limitations

- The 500-nt truncation cutoff is calibrated to 10X Chromium 3' Single Cell libraries; other 3'-tag protocols with different fragment/peak widths may need re-derivation (cf. the Tabula Muris peak-width analysis across 28 libraries).
- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, so the truncated UTRome supports isoform-aware quantification but not exact de novo cleavage-site mapping.
- The 200-nt merge distance reflects the empirical resolution limit of kallisto; isoform pairs closer than this are collapsed rather than resolved.

## Evidence

- [methods] protein-coding transcripts with validated 3' ends filter: "We recommend filtering transcripts to only protein-coding transcripts with validated 3' ends"
- [methods] mRNA_end_NF exclusion: "excluding transcripts with the tag **mRNA_end_NF**"
- [readme] txcutr-db pipeline purpose: [evidence span withheld — anchor status: external_doc]
- [intro] 10x reads do not span cleavage sites: "most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS)"
- [readme] kallisto truncation benchmark motivation: [evidence span withheld — anchor status: external_doc]
