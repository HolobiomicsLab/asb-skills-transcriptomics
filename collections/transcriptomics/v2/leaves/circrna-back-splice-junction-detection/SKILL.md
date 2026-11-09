---
name: circrna-back-splice-junction-detection
description: Use when you have name-sorted accepted_hits.sorted.sam from TopHat2 paired-end
  RNA-seq data and need a CircAST-format circRNA_list.txt of back-spliced junctions
  (chromosome, start, end) before running full-length circRNA isoform assembly.
when_to_use_negative:
- CircAST-based assembly is intended for RNase R-treated samples — avoid this workflow
  on naive RNA-seq where remnant linear mRNA can generate false circular transcripts
- You need full-length circRNA isoform sequences or expression quantification directly
  — BSJ detection only reports back-splice sites, not internal isoform structure (use
  CircAST assembly downstream)
- Targets are intronic or intergenic circular transcripts — the CircAST pipeline assembles
  exon-annotated circular transcripts and may miss these even with correct BSJ detection
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: CIRCexplorer2
  role: Upstream back-spliced junction detection from TopHat2 alignments; output converted
    to CircAST circRNA_list.txt format
  repo: https://github.com/xiaofengsong/CircAST
- name: CIRI2
  role: Alternative BSJ detector producing result.txt as the CircAST junction input
  repo: https://github.com/xiaofengsong/CircAST
- name: UROBORUS
  role: Alternative BSJ detector producing circRNA_list.txt as the CircAST junction
    input
  repo: https://github.com/xiaofengsong/CircAST
- name: TopHat2
  role: Paired-end alignment with gene GTF (tophat -o results -p 12 -G) generating
    accepted_hits.bam that BSJ detectors parse
  repo: https://github.com/xiaofengsong/CircAST
- name: SAMtools
  role: Name-sorts accepted_hits.bam ('samtools sort -n') and converts to SAM required
    for the detection/assembly pipeline
  repo: https://github.com/xiaofengsong/CircAST
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:22f55efca8fbe4d75ab8a04edf004033d4a9940bab50f8a14b048c39bc4df924
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI
  - CIRI-AS
  - CIRI2
  - CircAST
  - Cufflinks
  - SAMtools
  - TopHat2
  - UROBORUS
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# circrna-back-splice-junction-detection

## Summary

Detect circRNA back-spliced junctions (BSJs) from TopHat2-aligned paired-end RNA-seq data using upstream detection tools (CIRCexplorer2, CIRI2, or UROBORUS), producing a circRNA junction list that feeds CircAST's full-length assembly. This detection step is mandatory because CircAST only operates downstream of BSJ calls and does not discover junctions itself.

## When to use

Use when you have name-sorted accepted_hits.sorted.sam from TopHat2 paired-end RNA-seq data and need a CircAST-format circRNA_list.txt of back-spliced junctions (chromosome, start, end) before running full-length circRNA isoform assembly.

## When NOT to use

- CircAST-based assembly is intended for RNase R-treated samples — avoid this workflow on naive RNA-seq where remnant linear mRNA can generate false circular transcripts
- You need full-length circRNA isoform sequences or expression quantification directly — BSJ detection only reports back-splice sites, not internal isoform structure (use CircAST assembly downstream)
- Targets are intronic or intergenic circular transcripts — the CircAST pipeline assembles exon-annotated circular transcripts and may miss these even with correct BSJ detection

## Inputs

- Paired-end FASTQ reads (e.g. SRA accessions SRR1636985, SRR3479244; GSA CRA002302 for mouse testis RNase R-treated data)
- accepted_hits.sorted.sam — name-sorted TopHat2 alignment ('samtools sort -n' then 'samtools view')
- UCSC-compatible gene annotation GTF (e.g. hg19 or mm10 genes.gtf)
- UCSC Bowtie genome index (e.g. UCSC hg19)
- Back-spliced junction list in CircAST input format (circRNA_list.txt with chromosome, start, end columns)

## Outputs

- circRNA back-spliced junction list (CIRCexplorer2 circ_out/denovo/annotated_circ.txt, CIRI2 result.txt, or UROBORUS circRNA_list.txt) in CircAST-compatible format
- Filtered set of BSJs passing the supporting-reads threshold (default 10) consumed by CircAST as -J input

## How to apply

Align paired-end FASTQ reads with TopHat2 against a UCSC Bowtie index using a gene annotation GTF (e.g. 'tophat -o results -p 12 -G genes_homo.gtf genome reads_1.fastq reads_2.fastq'), then name-sort the BAM with 'samtools sort -n' and convert to SAM with 'samtools view' to obtain accepted_hits.sorted.sam. Detect back-spliced junctions from these alignments with CIRCexplorer2, CIRI2, or UROBORUS, yielding circ_out/denovo/annotated_circ.txt, result.txt, or circRNA_list.txt respectively; outputs from other BSJ tools are acceptable only if converted to CircAST's input format (chromosome, start, end). Use the identical GTF annotation file for both the BSJ detector and CircAST, since CircAST requires UCSC-compatible GTF formatting and consistency between upstream and downstream runs. Apply the back-splicing junction supporting-reads threshold (-T, default 10; e.g. -T 5 was used on the mouse testis RNase R dataset) when the junction list is consumed by CircAST to filter low-support BSJs. The resulting junction list, together with the sorted SAM and GTF, becomes the -J input to 'python CircAST.py -L <read_length> -T <threshold> -G genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt'.

## Related tools

- **CIRCexplorer2** (Upstream back-spliced junction detection from TopHat2 alignments; output converted to CircAST circRNA_list.txt format) — https://github.com/xiaofengsong/CircAST
- **CIRI2** (Alternative BSJ detector producing result.txt as the CircAST junction input) — https://github.com/xiaofengsong/CircAST
- **UROBORUS** (Alternative BSJ detector producing circRNA_list.txt as the CircAST junction input) — https://github.com/xiaofengsong/CircAST
- **TopHat2** (Paired-end alignment with gene GTF (tophat -o results -p 12 -G) generating accepted_hits.bam that BSJ detectors parse) — https://github.com/xiaofengsong/CircAST
- **SAMtools** (Name-sorts accepted_hits.bam ('samtools sort -n') and converts to SAM required for the detection/assembly pipeline) — https://github.com/xiaofengsong/CircAST

## Examples

```
tophat -o results -p 12 -G genes_homo.gtf genome reads_1.fastq reads_2.fastq; samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam; samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam; python CircAST.py -L 100 -T 5 -G genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Junction list is in CircAST input format with chromosome, start, end columns, and the same GTF was used for detection and assembly
- On simulated hg19 datasets, CircAST run on the detected junctions should yield sensitivity in the reported 85.63%-94.32% range and precision in 81.96%-87.55%
- Mouse testis RNase R dataset (GSA CRA002302) with -T 5 should yield 2883 detected back-spliced circRNAs (≥10 supporting BSJ reads) and 3464 assembled full-length transcripts
- The sorted SAM passes 'samtools sort -n' + 'samtools view' chain without positional sorting errors when consumed by CircAST
- Per-sample multi-isoform BSJ fractions from downstream assembly fall within the reported 7%-22% range

## Limitations

- Remnant linear mRNA in RNase R-treated samples can affect assembly performance and generate false transcripts
- Other BSJ detection tools require output file format conversion to CircAST input format before use
- GTF annotation mismatch between the BSJ detector and CircAST breaks the pipeline; UCSC-compatible GTF formatting is required

## Evidence

- [intro] CircAST can perform analysis downstream of existing back splice site detection software such as CIRCexplorer2, CIRI2, and UROBORUS: "CircAST can perform analysis downstream of existing back splice site detection software such as CIRCexplorer2, CIRI2, and UROBORUS"
- [readme] CircRNA lists containing back-spliced junctions can be obtained by CIRCexplorer2/CIRI2/UROBORUS: "CircRNA lists containing back-spliced junctions can be obtained by CIRCexplorer2/CIRI2/UROBORUS,  circ_out/denovo/annotated_circ.txt, result.txt, or circRNA_list.txt"
- [discussion] Other circRNA junction detection tools can also be used with CircAST if their output file format is converted to CircAST input file format.: "Other circRNA junction detection tools can also be used with CircAST if their output file format is converted to CircAST input file format."
- [readme] The GTF annotation file should be the same one when running CircAST and its upstream softwares.: "The GTF annotation file should be the same one when running CircAST and its upstream softwares."
- [methods] set threshold for back-splicing junction supporting reads(optional, default: 10): "set threshold for back-splicing junction supporting reads(optional, default: 10)"
