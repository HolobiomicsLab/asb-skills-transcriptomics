---
name: blast-based-contig-gene-mapping
description: Use when you have a de novo assembled transcriptome (e.g., Trinity contigs
  from FASTQ reads) and a reference transcript set, and need to identify which contigs
  are chimeric, correct, or unidentified to benchmark chimera-removal pipelines.
when_to_use_negative:
- No reference transcript set or genome exists for the organism — without reference
  transcripts, contigs cannot be classified as chimeric vs. correct, which the paper
  notes makes de novo assembly quality difficult to assess
- The goal is quality assessment of the assembly itself (expression/contiguity metrics)
  rather than chimera identification — TransRate and BUSCO are the appropriate tools
  in the Bellerophon workflow for that
- Input is raw RNA-Seq reads rather than assembled contigs — reads cannot be classified
  as chimeric contigs without prior de novo assembly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BLASTn
  role: Aligns Bellerophon-filtered contigs against the reference transcript set to
    detect single vs. multiple non-overlapping gene hits
- name: Bellerophon
  role: Produces the filtered contig set whose chimeric/correct/unidentified composition
    is measured by this mapping
  repo: github.com/JesseKerkvliet/Bellerophon
- name: Trinity
  role: De novo assembler generating the contigs that are subsequently BLAST-mapped
    to reference genes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:43315fdaa062189efeb8f830a0c27b8596b68297cec5a2606becc4849627bf39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# BLAST-based contig-to-gene mapping

## Summary

Maps de novo assembled transcriptome contigs onto a reference transcript set with BLAST to classify each contig as chimeric (multiple non-overlapping hits to different genes), nonchimeric (exactly one gene), or unidentified (no hits), enabling quantification of chimera removal by filtering pipelines such as Bellerophon.

## When to use

Use when you have a de novo assembled transcriptome (e.g., Trinity contigs from FASTQ reads) and a reference transcript set, and need to identify which contigs are chimeric, correct, or unidentified to benchmark chimera-removal pipelines.

## When NOT to use

- No reference transcript set or genome exists for the organism — without reference transcripts, contigs cannot be classified as chimeric vs. correct, which the paper notes makes de novo assembly quality difficult to assess
- The goal is quality assessment of the assembly itself (expression/contiguity metrics) rather than chimera identification — TransRate and BUSCO are the appropriate tools in the Bellerophon workflow for that
- Input is raw RNA-Seq reads rather than assembled contigs — reads cannot be classified as chimeric contigs without prior de novo assembly

## Inputs

- FASTA file of assembled/filtered transcriptome contigs (e.g., Bellerophon-filtered Trinity assembly)
- Reference transcript set (e.g., D. melanogaster expressed transcripts, or NCBI RefSeq GCF_000001215.4-derived transcripts)

## Outputs

- Per-contig classification table: chimeric, nonchimeric, or unidentified
- Counts and removal percentages of chimeric, correct, and unidentified contigs removed by each filter step
- Number of unique genes represented before and after filtering (e.g., 10,527 -> 9,148)

## How to apply

BLAST the Bellerophon-filtered contigs against the reference set of transcripts using strict hit criteria: E-value <= 1e-3, percentage identity >= 90, and alignment length >= 300 nucleotides. Classify each contig as chimeric when a sequence contains two matches that do not overlap in their mapping region and have hits with different genes; as nonchimeric when it matches exactly one gene; and as unidentified when it matches no genes. Tabulate how many contigs of each class were removed by each filtering step (e.g., TPM filtering, CD-HIT-EST, TransRate-C) to compute removal percentages per class. This mapping is also how the pipeline was validated on the D. melanogaster assembly (33,484 contigs), where Bellerophon removed 1,619/2,673 chimeras (60.6%), 14,325/26,528 correct sequences (54%), and 3,005/5,183 unidentified sequences (58%).

## Related tools

- **BLASTn** (Aligns Bellerophon-filtered contigs against the reference transcript set to detect single vs. multiple non-overlapping gene hits)
- **Bellerophon** (Produces the filtered contig set whose chimeric/correct/unidentified composition is measured by this mapping) — github.com/JesseKerkvliet/Bellerophon
- **Trinity** (De novo assembler generating the contigs that are subsequently BLAST-mapped to reference genes)

## Examples

```
blastn -query bellerophon_filtered_contigs.fasta -subject dmel_reference_transcripts.fasta -evalue 1e-3 -perc_identity 90 -outfmt 6
```

## Evaluation signals

- Chimeric-class contigs show at least two non-overlapping BLAST hit regions mapping to different genes
- Nonchimeric contigs match exactly one reference gene under the thresholds E-value <= 1e-3, identity >= 90%, alignment length >= 300 nt
- Unidentified contigs produce no reference hits at the chosen thresholds
- Reproduced removal percentages fall near the reported benchmark: chimeras ~60.6%, correct ~54%, unidentified ~58%
- Unique-gene count reduction matches the expected range (e.g., 10,527 to 9,148 for the D. melanogaster SRA benchmark)

## Limitations

- Contigs from alternative splicing of one gene may be erroneously considered chimeric by TransRate if unevenly expressed along their sequence, so BLAST-based chimera calls can misclassify valid isoforms
- Without a reference genome or transcript set, the quality of a de novo assembled transcriptome is difficult to assess
- No tool version numbers for BUSCO, TransRate, CD-HIT-EST, Trinity, or Polyester are stated, affecting reproducibility of the mapping context

## Evidence

- [methods] Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes.: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
- [results] we used 3,000 transcripts expressed by D. melanogaster as a reference to simulate RNA-Seq reads.: "we used 3,000 transcripts expressed by D. melanogaster as a reference to simulate RNA‐Seq reads."
- [readme] Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies.: "Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies."
- [discussion] When there is no reference genome available, the quality of a de novo assembled transcriptome is difficult to assess: "When there is no reference genome available, the quality of a de novo assembled transcriptome is difficult to assess"
