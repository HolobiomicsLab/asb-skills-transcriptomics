---
name: cell-type-specific-ipa-usage-comparison
description: Use when single-cell RNA-seq alignments and InPACT-derived IPA usage
  estimates are available and the goal is to flag IPA events whose usage is restricted
  to one cell type versus others.
when_to_use_negative:
- The input is bulk RNA-seq from a single condition with no cell-type labels — cell-type-specific
  comparison is undefined
- IPA sites have not yet been identified and quantified (no InPACT predict.result.txt
  or ipa_usage table) — run IPA identification and InPACT_quantify first
- The goal is 3'UTR APA quantification rather than novel intronic IPA events — tools
  like DaPars or QAPA target that scope instead
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3320
tools:
- name: InPACT
  role: Identify novel IPA sites/events (skipped and composite), assemble IPA isoforms
    (InPACT_transcript), and compute per-cell IPA usage (InPACT_quantify)
  repo: https://github.com/YY-TMU/InPACT
- name: Salmon
  role: Index (k-mer 31) and quantify merged GTF transcript abundance; quant.sf TPM
    feeds InPACT_quantify and the TPM < 1 filter
  repo: https://github.com/COMBINE-lab/salmon
- name: gffread
  role: Extract transcript sequences from the merged IPA isoform GTF for Salmon indexing
- name: HISAT2
  role: Align each cell's SMART-seq2 reads to the GRCh38 reference before PCR duplicate
    removal
- name: samtools
  role: Remove PCR duplicates with samtools rmdup on per-cell alignments
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:450e62a28c196d01deced7a7be3ccafb5c92c22939a6d23180d8a6e4573b3fb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - HISAT2
  - InPACT
  - Salmon
  - gffread
  - samtools
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-type-specific-ipa-usage-comparison

## Summary

Identify cell-type-specific intronic polyadenylation (IPA) events by comparing per-cell IPA usage from InPACT_quantify across cell types in scRNA-seq data, after filtering low-abundance genes with TPM < 1, and test GO enrichment of harboring genes with a hypergeometric test.

## When to use

Use when single-cell RNA-seq alignments and InPACT-derived IPA usage estimates are available and the goal is to flag IPA events whose usage is restricted to one cell type versus others.

## When NOT to use

- The input is bulk RNA-seq from a single condition with no cell-type labels — cell-type-specific comparison is undefined
- IPA sites have not yet been identified and quantified (no InPACT predict.result.txt or ipa_usage table) — run IPA identification and InPACT_quantify first
- The goal is 3'UTR APA quantification rather than novel intronic IPA events — tools like DaPars or QAPA target that scope instead

## Inputs

- Sorted coordinate BAM files of scRNA-seq reads (e.g., SMART-seq2 human fetal bone marrow, E-MTAB-9801) aligned to GRCh38
- RefSeq GRCh38 GTF annotation (hg38.109.20211119.ncbiRefSeq.gtf.gz)
- Precomputed polyA-site annotation InPACT_polyAsites.hg38.saf
- Reference genome FASTA
- Per-cell Salmon quant.sf transcript abundances
- Cell-type labels for each cell

## Outputs

- Table of novel IPA events (2635 events in 2157 genes; skipped vs composite) with IPA usage (PAU) per cell
- Table of cell-type-specific IPA events (e.g., 533 events) with cell-type assignments
- GO term enrichment results (hypergeometric test) for genes harboring cell-type-specific IPA events

## How to apply

Pool aligned reads across cells (e.g., SMART-seq2 fetal bone marrow cells aligned to GRCh38 with HISAT2 and PCR-deduplicated via samtools rmdup), run InPACT with the RefSeq GRCh38 GTF and InPACT_polyAsites.hg38.saf using default thresholds (minimum 5 spliced reads for skipped and 10 spanning reads for composite terminal exons), then assemble novel IPA isoforms with InPACT_transcript, extract sequences with gffread, and quantify with Salmon (k-mer 31 index). Compute per-cell IPA usage with InPACT_quantify, filter out low-abundance genes with TPM < 1, and compare IPA usage between cells of the same cell type and those of other cell types to designate cell-type-specific events (the reference analysis reported 533 of 2635 novel events across 2157 genes). Finally, test GO term enrichment for genes harboring cell-type-specific IPA events using the hypergeometric test.

## Related tools

- **InPACT** (Identify novel IPA sites/events (skipped and composite), assemble IPA isoforms (InPACT_transcript), and compute per-cell IPA usage (InPACT_quantify)) — https://github.com/YY-TMU/InPACT
- **Salmon** (Index (k-mer 31) and quantify merged GTF transcript abundance; quant.sf TPM feeds InPACT_quantify and the TPM < 1 filter) — https://github.com/COMBINE-lab/salmon
- **gffread** (Extract transcript sequences from the merged IPA isoform GTF for Salmon indexing)
- **HISAT2** (Align each cell's SMART-seq2 reads to the GRCh38 reference before PCR duplicate removal)
- **samtools** (Remove PCR duplicates with samtools rmdup on per-cell alignments)

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5; InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt
```

## Evaluation signals

- Output TSV files exist and contain the identified IPA events, cell-type-specific assignments, and GO enrichment terms matching the reported counts (2635 events / 2157 genes; 533 cell-type-specific events)
- Every retained event meets the read-module minima: ≥5 spliced reads for skipped and ≥10 boundary-spanning reads for composite terminal exons
- Genes with TPM < 1 are absent from the usage-comparison table
- IPA usage column contains valid PAU estimates between 0 and 1 per cell/event
- Inferred isoform GTF is parseable by gffread and yields a Salmon index that quantifies all cells

## Limitations

- The InPACT read module needs adequate read depth; the source dataset is small (486 cells), so per-cell usage estimates may be sparse
- No version number for InPACT is specified in the article; results may vary across releases
- Only IPA isoforms with an in-frame stop codon are considered for downstream translation analysis, so cell-type-specific events may not all be translatable

## Evidence

- [methods] Filter low-abundance genes with TPM < 1: "low-abundance genes with TPM < 1 were filtered out"
- [readme] Read-module thresholds for skipped and composite events: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
- [readme] InPACT_quantify invocation for IPA usage: "InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt"
- [abstract] Single-cell IPA expression in human fetal bone marrow: "application on single-cell RNA-seq data of human fetal bone marrow reveals the expression of several IPA isoforms in a context-specific manner"
