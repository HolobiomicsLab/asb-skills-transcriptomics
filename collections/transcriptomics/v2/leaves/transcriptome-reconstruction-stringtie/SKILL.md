---
name: transcriptome-reconstruction-stringtie
description: Use when you have a sorted BAM of genome-aligned RNA-Seq reads (e.g.
  HISAT2 --dta output) and need an initial GTF transcriptome as input for aptardi
  polyA site prediction or sample-specific 3'-end refinement.
when_to_use_negative:
- You need de novo polyA site discovery without an initial transcriptome — aptardi
  requires an input GTF/gff (typically StringTie output) and only evaluates 3' terminal
  exons of those transcripts
- You need coding APA within upstream exons — StringTie-based reconstruction feeding
  aptardi only refines 3' terminal exons, so upstream exon polyA events cannot be
  identified
- The input GTF already contains a curated, high-quality transcriptome with correct
  3' ends and no APA isoforms to recover
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: StringTie
  role: Reconstructs the sample-specific transcriptome from the HISAT2-sorted BAM
    using the Ensembl guide annotation (--rf -G <guide_file>) to produce stringtie.gtf
- name: HISAT2
  role: Aligns paired-end reads to hg38 with --rna-strandness RF and --dta to generate
    alignments compatible with StringTie assembly
- name: SAMtools
  role: Sorts HISAT2 alignments into sorted.bam required by StringTie and aptardi
- name: aptardi
  role: Consumes the StringTie GTF as the initial transcriptome and adds refined 3'-end
    aptardi transcripts to produce name_aptardi.gtf
  repo: https://github.com/luskry/aptardi
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:ba5bae39380bf1658ba2058b6af5bb95da1f45f117530dbf394f62cbfe3a9e1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - Cutadapt
  - FastQC
  - HISAT2
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# transcriptome-reconstruction-stringtie

## Summary

Reconstruct a sample-specific transcriptome from genome-aligned RNA-Seq reads with StringTie, guided by Ensembl annotation, to produce the stringtie.gtf that aptardi uses to identify expressed polyadenylation sites and refine transcript 3'-ends.

## When to use

Use when you have a sorted BAM of genome-aligned RNA-Seq reads (e.g. HISAT2 --dta output) and need an initial GTF transcriptome as input for aptardi polyA site prediction or sample-specific 3'-end refinement.

## When NOT to use

- You need de novo polyA site discovery without an initial transcriptome — aptardi requires an input GTF/gff (typically StringTie output) and only evaluates 3' terminal exons of those transcripts
- You need coding APA within upstream exons — StringTie-based reconstruction feeding aptardi only refines 3' terminal exons, so upstream exon polyA events cannot be identified
- The input GTF already contains a curated, high-quality transcriptome with correct 3' ends and no APA isoforms to recover

## Inputs

- sorted.bam of paired-end RNA-Seq reads aligned to hg38 with HISAT2 (--rna-strandness RF, --dta)
- Ensembl guide annotation GTF file (-G <guide_file>)

## Outputs

- stringtie.gtf — sample-specific reconstructed transcriptome in GTF format, used as aptardi's initial transcriptome input

## How to apply

Align reads with HISAT2 using --rna-strandness appropriate to the library (e.g. RF for firststrand Illumina paired-end protocols) and --dta, then sort the alignments with SAMtools into sorted.bam. Run StringTie on the sorted BAM with the strandness flag (--rf for firststrand libraries) and the Ensembl guide annotation via -G to produce stringtie.gtf; this guide-guided assembly constrains transcripts to known exons while recovering novel isoforms. The resulting stringtie.gtf is the initial transcriptome input to aptardi, which evaluates only the 3' terminal exons of these input transcripts to annotate polyA sites. Because StringTie often terminates transcripts at zero read coverage and may miss short APA isoforms (e.g. proximal 3' ends of CCND1, DICER1, and TIMP2 in CFIm25 knockdown HeLa cells), the reconstructed GTF is deliberately a starting point that aptardi extends with refined 3' termini in name_aptardi.gtf.

## Related tools

- **StringTie** (Reconstructs the sample-specific transcriptome from the HISAT2-sorted BAM using the Ensembl guide annotation (--rf -G <guide_file>) to produce stringtie.gtf)
- **HISAT2** (Aligns paired-end reads to hg38 with --rna-strandness RF and --dta to generate alignments compatible with StringTie assembly)
- **SAMtools** (Sorts HISAT2 alignments into sorted.bam required by StringTie and aptardi)
- **aptardi** (Consumes the StringTie GTF as the initial transcriptome and adds refined 3'-end aptardi transcripts to produce name_aptardi.gtf) — https://github.com/luskry/aptardi

## Examples

```
stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>
```

## Evaluation signals

- stringtie.gtf is produced without errors from sorted.bam and opens as a valid GTF for aptardi's --r argument
- Downstream, aptardi's name_aptardi.gtf contains both original StringTie transcripts and newly added aptardi transcripts (aptardi removes none from the input)
- In the CFIm25 knockdown HeLa use case, aptardi recovers proximal APA 3' ends for CCND1 (chr11:69,651,917), DICER1 (chr11:95,090,264), and TIMP2 (chr17:78,855,465) that StringTie alone missed
- Strandness flags are consistent end-to-end: HISAT2 --rna-strandness RF, StringTie --rf, and aptardi --a fr for firststrand Illumina paired-end libraries
- For cross-dataset benchmarking, the reconstructed transcriptomes feed aptardi to yield average precision near reported values (AP = 0.65 HBR, 0.63 2nd HBR, 0.63 UHR, 0.55 BNLx, 0.55 SHR)

## Limitations

- StringTie constructs a minimum path RNA-Seq cover to the position of zero read coverage, so short APA isoforms with low coverage distal to the stop can be missed or mis-terminated
- Assemblies may include pseudo transcripts from broad guide annotations (e.g. Ensembl); aptardi's authors suggest curating a more-selective input transcriptome to mitigate redundant aptardi transcript enumeration
- Standard assemblers like StringTie have substantially lower average precision and recall for polyA termini than aptardi (aptardi's average precision is roughly twice that of a standard transcriptome assembler, with recall improved over three-fold)

## Evidence

- [methods] Reconstruct transcriptome with StringTie: "stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>"
- [methods] Align reads with HISAT2 before assembly: "(hisat2 -q -p 5 --reorder -t --rna-strandness RF --dta -x <hisat2_index> -1"
- [intro] Transcript assemblers terminate at zero coverage: "Some assemblers, e.g., Cufflinks29/StringTie22, construct a minimum path RNA-Seq cover to the position where there is zero read coverage"
- [readme] aptardi takes the initial transcriptome as input: "aptardi takes as input a transcriptome (gtf/gff format), possibly constructed from RNA-Seq data, and combines RNA-Seq data from the sample with the genome (DNA sequence) of the sample"
- [results] CFIm25 knockdown short APA isoforms recovered by aptardi: [evidence span withheld — anchor status: not_found]
