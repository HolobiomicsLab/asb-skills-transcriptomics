---
name: chimera-identification-blast-alignment
description: Use when you need to label contigs in a de novo transcriptome assembly
  as chimeric vs. correct and have (or can build) a reference transcriptome of known,
  truly expressed transcripts.
when_to_use_negative:
- No suitable reference transcriptome exists, since classification depends on unambiguous
  BLASTn matches to trusted reference transcripts
- You need reference-free quality assessment of an assembly, where TransRate (requiring
  only reads and the assembly) is the article's recommended alternative
- Input contigs representative of alternatively spliced transcripts of one gene, which
  may be misclassified as chimeras if unevenly expressed along their sequence
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BLASTn
  role: Aligns assembled contigs against the reference transcriptome; matches to multiple
    non-overlapping regions of different genes flag a contig as chimeric (90% identity,
    E-value 10 cut-offs)
- name: Trinity
  role: Produces the de novo assembly of simulated reads whose contigs are then classified
    by BLASTn
- name: Polyester
  role: Simulates paired-end RNA-Seq reads from the reference transcriptome to create
    a benchmark with known chimeric and correct contigs
- name: Bellerophon
  role: Downstream pipeline whose TPM, CD-HIT-EST, and TransRate filtering steps are
    evaluated against the BLASTn chimera labels
  repo: github.com/JesseKerkvliet/Bellerophon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:8218cac413df78328d845f264cf73513ef90c4efc499e143e7f2b39f144dc57b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - Bellerophon
  - CD-HIT-EST
  - Polyester
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# chimera-identification-blast-alignment

## Summary

Identifies chimeric contigs in a de novo transcriptome assembly by BLASTn alignment against a trusted reference transcriptome, classifying contigs with multiple non-overlapping matches to different genes as chimeras and those matching exactly one transcript as correct.

## When to use

Use when you need to label contigs in a de novo transcriptome assembly as chimeric vs. correct and have (or can build) a reference transcriptome of known, truly expressed transcripts.

## When NOT to use

- No suitable reference transcriptome exists, since classification depends on unambiguous BLASTn matches to trusted reference transcripts
- You need reference-free quality assessment of an assembly, where TransRate (requiring only reads and the assembly) is the article's recommended alternative
- Input contigs representative of alternatively spliced transcripts of one gene, which may be misclassified as chimeras if unevenly expressed along their sequence

## Inputs

- De novo assembled transcriptome contigs in FASTA (e.g. Trinity output)
- Reference transcriptome in FASTA (e.g. D. melanogaster NCBI RefSeq GCF_000001215.4 transcripts)

## Outputs

- Per-contig classification as chimeric or non-chimeric (correct)
- Counts of chimeric vs. correct contigs removed or retained at each filtering step (output table)

## How to apply

Align each contig from the assembly (e.g. a Trinity assembly in FASTA) against the reference transcriptome with BLASTn, using a percent identity cut-off of 90% and an E-value cut-off of 10. Classify a contig as chimeric when it contains two or more matches that do not overlap in their mapping region and whose hits correspond to different reference genes; classify contigs matching exactly one reference transcript as correct. Optionally, first build the reference by randomly selecting transcripts (e.g. 3,000 from the D. melanogaster NCBI RefSeq GCF_000001215.4 annotation) and simulate reads with Polyester to generate chimeras of known count for benchmarking. The resulting chimera/contig labels can then be used to quantify how many chimeras survive each downstream filtering step, as Bellerophon did when recounting which BLASTn-classified chimeras and non-chimeric contigs survived TPM, CD-HIT-EST, and TransRate filtering.

## Related tools

- **BLASTn** (Aligns assembled contigs against the reference transcriptome; matches to multiple non-overlapping regions of different genes flag a contig as chimeric (90% identity, E-value 10 cut-offs))
- **Trinity** (Produces the de novo assembly of simulated reads whose contigs are then classified by BLASTn)
- **Polyester** (Simulates paired-end RNA-Seq reads from the reference transcriptome to create a benchmark with known chimeric and correct contigs)
- **Bellerophon** (Downstream pipeline whose TPM, CD-HIT-EST, and TransRate filtering steps are evaluated against the BLASTn chimera labels) — github.com/JesseKerkvliet/Bellerophon

## Examples

```
blastn -query Trinity_assembly.fasta -subject dmel_ref_transcripts.fasta -perc_identity 90 -evalue 10 -out chimera_hits.tsv
```

## Evaluation signals

- In the simulated benchmark, reproduce the reported outcome where Bellerophon removed 136 of 295 chimeras (46.1%) and 575 of 3,283 correct sequences (17.5%)
- Check that chimeric calls are restricted to contigs whose BLASTn hits are non-overlapping and map to different reference genes; single-transcript matches must be labeled correct
- Verify BLASTn parameters applied were percent identity ≥90% and E-value ≤10
- Confirm counts of chimeras removed per filter step sum consistently (TPM removed 227/8.5%, CD-HIT-EST removed 600/24.5%, TransRate-C removed 792/42.9% of chimeras at their stages)

## Limitations

- Contigs from alternative splicing of one gene may be erroneously flagged as chimeras by TransRate if they are unevenly expressed along their sequence
- Requires a reference genome/transcriptome; when no reference is available, de novo assembly quality is difficult to assess
- The Chimera-removal trade-off is asymmetric: in the simulated experiment Bellerophon removed 17.5% of correct sequences alongside 46.1% of chimeras

## Evidence

- [methods] Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes.: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
- [discussion] 46.1% of the chimeras in the simulated D. melanogaster RNA‐Seq experiment: "46.1% of the chimeras in the simulated D. melanogaster RNA‐Seq experiment"
- [readme] Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies.: "Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies."
- [results] the TPM filtering step first removed 227 (8.5%) of the chimeras: "the TPM filtering step first removed 227 (8.5%) of the chimeras"
