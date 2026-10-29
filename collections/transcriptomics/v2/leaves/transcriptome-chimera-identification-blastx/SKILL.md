---
name: transcriptome-chimera-identification-blastx
description: Use when you have a de novo assembled transcriptome (e.g. Trinity FASTA)
  with no reference genome and need to count or mark chimeric transcripts to assess
  or benchmark assembly filtering.
when_to_use_negative:
- A reference genome is available and reference-based chimeric transcript detection
  or alignment to annotated transcripts can be used instead
- Contigs represent alternative splicing of a single gene — such transcripts may be
  erroneously flagged, since unevenly expressed isoforms can appear as two non-overlapping
  hits
- The goal is transcriptome quality assessment in general (e.g. BUSCO completeness
  or TransRate scoring) rather than specifically detecting or counting chimeras
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: BLASTX
  role: Aligns translated assembled contigs against the NCBI NR protein database with
    E-value cut-off 1e-4 to detect two non-overlapping hits to different genes, marking
    chimeras
- name: Trinity
  role: Produces the de novo transcriptome assembly whose isoform-rich contig groups
    are screened for chimeras
- name: Bellerophon
  role: Pipeline whose chimera-removal filters (TPM, CD-HIT-EST, TransRate-C) are
    benchmarked using the BLASTX-derived chimera labels
  repo: github.com/JesseKerkvliet/Bellerophon
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:35dff03d739f8e332b5b289ae7ed298758214ab9f51a2dac63300be11611df69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTX
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# transcriptome-chimera-identification-blastx

## Summary

Reference-free identification of chimeric contigs in a de novo transcriptome assembly by BLASTX against the NCBI NR protein database, flagging contigs whose alignments contain two non-overlapping matches to different genes. This provides the ground-truth chimera labels the Bellerophon pipeline uses to benchmark chimera-removal filters.

## When to use

Use when you have a de novo assembled transcriptome (e.g. Trinity FASTA) with no reference genome and need to count or mark chimeric transcripts to assess or benchmark assembly filtering.

## When NOT to use

- A reference genome is available and reference-based chimeric transcript detection or alignment to annotated transcripts can be used instead
- Contigs represent alternative splicing of a single gene — such transcripts may be erroneously flagged, since unevenly expressed isoforms can appear as two non-overlapping hits
- The goal is transcriptome quality assessment in general (e.g. BUSCO completeness or TransRate scoring) rather than specifically detecting or counting chimeras

## Inputs

- De novo assembled transcriptome contigs in FASTA format (e.g. Trinity assembly of the H. subflexa pheromone gland RNA-Seq data)
- NCBI nonredundant protein database (NR) for BLASTX

## Outputs

- Per-contig chimera labels (chimeric vs. non-chimeric) based on two non-overlapping BLASTX hits to different genes
- Chimera counts per assembly/filtering stage (e.g. 74 chimeric isoforms before filtering, 18 after TransRate-C-before-RSEM)

## How to apply

Run BLASTX on the assembled contigs against the NCBI nonredundant protein database (NR) with an E-value cut-off of 1e-4. For each contig, inspect the mapping regions of its hits: mark a contig as chimeric when the sequence contains two matches that do not overlap in their mapping region and that hit different genes. Apply this marking procedure to the unfiltered assembly to establish a baseline chimera count (e.g. the 74 chimeric isoforms found in the isoform-rich Trinity-predicted contig groups of the H. subflexa pheromone gland assembly), and re-run it after each filtering step (TPM, CD-HIT-EST, TransRate-C) to count how many marked chimeras remain. The change in chimera count across steps is the metric used to compare filtering orders, such as the variant that places TransRate-C before RSEM, which reduced chimeras from 74 to 18 (56 removed).

## Related tools

- **BLASTX** (Aligns translated assembled contigs against the NCBI NR protein database with E-value cut-off 1e-4 to detect two non-overlapping hits to different genes, marking chimeras)
- **Trinity** (Produces the de novo transcriptome assembly whose isoform-rich contig groups are screened for chimeras)
- **Bellerophon** (Pipeline whose chimera-removal filters (TPM, CD-HIT-EST, TransRate-C) are benchmarked using the BLASTX-derived chimera labels) — github.com/JesseKerkvliet/Bellerophon

## Examples

```
blastx -query assembly.fasta -db nr -evalue 1e-4 -outfmt 6 -out chimera_hits.tsv
```

## Evaluation signals

- Every contig labelled chimeric has two BLASTX hits with non-overlapping mapping regions against different genes at E-value <= 1e-4
- Re-running the BLASTX marking after each filtering step yields monotonically decreasing or stable chimera counts, e.g. 74 -> 18 for the TransRate-C-before-RSEM variant (56 chimeras removed)
- The baseline chimera count on the unfiltered isoform-rich benchmark set matches the reported 74 chimeric isoforms
- The marked-chimera table separates chimeric from non-chimeric contigs so that filter performance can be compared against the default order (TPM first, CD-HIT-EST 95% identity second)

## Limitations

- Transcripts arising from alternative splicing of one gene might be misclassified as chimeras if they are unevenly expressed along their sequence
- Custom order of filtering steps (needed to reproduce the variant benchmark) is disabled in the current Bellerophon version
- No reference genome is available in this setting, so chimera calls cannot be validated against known gene structures

## Evidence

- [methods] Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes.: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
- [methods] blasted against the nonredundant protein database (NR), using the BLASTX algorithm: "blasted against the nonredundant protein database (NR), using the BLASTX algorithm"
- [results] the TPM filtering step first removed 227 (8.5%) of the chimeras: "the TPM filtering step first removed 227 (8.5%) of the chimeras"
- [discussion] Contigs representative of transcripts arising from alternative splicing of one gene might be considered as chimera by Transrate if they are unevenly expressed along their sequence: "Contigs representative of transcripts arising from alternative splicing of one gene might be considered as chimera by Transrate if they are unevenly expressed along their sequence"
- [readme] Custom order of filtering steps is disabled in this version: "Custom order of filtering steps is disabled in this version"
