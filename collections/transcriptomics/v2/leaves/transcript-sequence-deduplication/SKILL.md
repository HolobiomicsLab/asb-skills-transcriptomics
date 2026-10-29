---
name: transcript-sequence-deduplication
description: Use when preparing a custom kallisto target transcriptome (GTF + FASTA
  + kdx + merge table) for 3' UTR isoform quantification from 10X Chromium 3'-tag
  scRNA-seq data, where reads are short and must be anchored near cleavage sites.
when_to_use_negative:
- Reads span the cleavage site or come from full-length/long-read protocols — the
  500-nt 3'-anchored truncation is designed for short 3'-tag reads (e.g., 10X Chromium)
  that do not span cleavage sites
- You only need gene-level counts — the deduplicated UTRome and merge table are specific
  to resolving 3'UTR isoforms
- An equivalent txcutr-db reference index already exists for your annotation — prefer
  downloading the deposited index (e.g., Zenodo 10.5281/zenodo.8118405 for GENCODE
  vM25) over regenerating
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: txcutr
  role: Performs the 500-nt 3'-anchored truncation (truncateTxome), sequence deduplication,
    merge-table generation (generateMergeTable, 200-nt distance), and GTF/FASTA export
- name: Snakemake
  role: Wraps the txcutr-db pipeline that generates the target files for scUTRquant
  repo: https://github.com/Mayrlab/txcutr-db
- name: kallisto
  role: Consumes the custom_target.kdx index built from the deduplicated truncated
    UTRome for pseudoalignment (kallisto bus)
- name: scUTRquant
  role: Downstream pipeline that uses the deduplicated UTRome targets for 3'UTR isoform
    quantification
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:72360a1012e1c1a9152f73d0d89b2f8ed09cbc5ef4a7e0398045178bd374e737
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
n_claims: 9
---

# transcript-sequence-deduplication

## Summary

Build a 3'-truncated, deduplicated UTRome target transcriptome for pseudoalignment of 3' end scRNA-seq data using txcutr: truncate GENCODE transcripts to a 500-nt 3'-anchored window, collapse identical truncated sequences to single features, and emit a merge table that maps collapsed sequences back to full-length transcripts.

## When to use

Use when preparing a custom kallisto target transcriptome (GTF + FASTA + kdx + merge table) for 3' UTR isoform quantification from 10X Chromium 3'-tag scRNA-seq data, where reads are short and must be anchored near cleavage sites.

## When NOT to use

- Reads span the cleavage site or come from full-length/long-read protocols — the 500-nt 3'-anchored truncation is designed for short 3'-tag reads (e.g., 10X Chromium) that do not span cleavage sites
- You only need gene-level counts — the deduplicated UTRome and merge table are specific to resolving 3'UTR isoforms
- An equivalent txcutr-db reference index already exists for your annotation — prefer downloading the deposited index (e.g., Zenodo 10.5281/zenodo.8118405 for GENCODE vM25) over regenerating

## Inputs

- GENCODE GTF/GFF transcriptome annotation (e.g., v39 human, vM21/vM25 mouse)
- Protein-coding transcripts with validated 3' ends (excluding mRNA_end_NF-tagged transcripts)
- Truncation length parameter (500 nt) and merge distance parameter (200 nt)

## Outputs

- custom_target.gtf — deduplicated 3'-truncated transcript annotation (via txcutr exportGTF)
- Truncated transcript FASTA (via txcutr exportFASTA)
- custom_target.kdx — kallisto index over the truncated UTRome
- custom_target.merge.tsv — merge table mapping truncated features to original transcript IDs

## How to apply

Start from a GENCODE annotation (e.g., vM25/vM21 human v39) and keep only protein-coding transcripts with validated 3' ends, excluding transcripts tagged mRNA_end_NF. Apply txcutr truncateTxome with a 500-nt 3'-anchored truncation length — chosen empirically because >95% of 3' end reads of tested reference genes fall within 500 nt upstream of cleavage sites, matching the peak widths of 10X Chromium 3' Single Cell libraries. Deduplicate so identical 3'-truncated sequences collapse to a single feature, then run txcutr generateMergeTable with a 200-nt merge distance, matching the empirical resolution limit of kallisto pseudoalignment (benchmarked in the kallisto-overlap project). Export with txcutr exportGTF/exportFASTA to custom_target.gtf and build the kallisto index (custom_target.kdx), writing the merge file as custom_target.merge.tsv; validate feature counts and merge-group assignments against the deposited txcutr-db reference (Zenodo 10.5281/zenodo.8118405).

## Related tools

- **txcutr** (Performs the 500-nt 3'-anchored truncation (truncateTxome), sequence deduplication, merge-table generation (generateMergeTable, 200-nt distance), and GTF/FASTA export)
- **Snakemake** (Wraps the txcutr-db pipeline that generates the target files for scUTRquant) — https://github.com/Mayrlab/txcutr-db
- **kallisto** (Consumes the custom_target.kdx index built from the deduplicated truncated UTRome for pseudoalignment (kallisto bus))
- **scUTRquant** (Downstream pipeline that uses the deduplicated UTRome targets for 3'UTR isoform quantification) — https://github.com/Mayrlab/scUTRquant

## Examples

```
cd txcutr-db && snakemake --use-conda --cores all  # produces custom_target.gtf, custom_target.kdx, custom_target.merge.tsv via txcutr truncateTxome(500) / generateMergeTable(200)
```

## Evaluation signals

- custom_target.gtf, custom_target.kdx, and custom_target.merge.tsv all exist and parse without error
- Transcript/feature counts in the generated GTF agree with the deposited txcutr-db reference index for the same GENCODE release (e.g., vM25)
- Merge-group assignments in custom_target.merge.tsv match the deposited reference merge table
- All retained transcripts are protein-coding and none carry the mRNA_end_NF tag in the pre-truncation filtering step
- Identical 3'-truncated sequences appear at most once as a feature in the FASTA/GTF (deduplication invariant)

## Limitations

- The 500-nt truncation and 200-nt merge distance are calibrated to 10X Chromium 3' library peak widths and kallisto's empirical resolution limit; other protocols may require re-deriving these cutoffs
- Filtering to validated 3' ends excludes transcripts with mRNA_end_NF tags, so non-protein-coding or unverified-3'-end transcripts are not quantifiable in this UTRome
- Merging at 200 nt trades isoform resolution for quantification accuracy: truncated isoforms closer than 200 nt cannot be reliably distinguished by kallisto

## Evidence

- [methods] protein-coding filter rationale: "We recommend filtering transcripts to only protein-coding transcripts with validated 3' ends"
- [methods] mRNA_end_NF exclusion: "excluding transcripts with the tag **mRNA_end_NF**"
- [readme] txcutr-db purpose: [evidence span withheld — anchor status: external_doc]
- [discussion] 3'-tag library constraint: "By building on the kallisto-bustools workflow, we provide a fast and direct means of quantification that is compatible with most 3′-end tag scRNA-seq libraries"
