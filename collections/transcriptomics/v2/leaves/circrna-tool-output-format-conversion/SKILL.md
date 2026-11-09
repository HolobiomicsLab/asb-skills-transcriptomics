---
name: circrna-tool-output-format-conversion
description: Use when a circRNA junction list from a BSJ detector (CIRCexplorer2 annotated_circ.txt,
  CIRI2 result.txt, UROBORUS, or any similar tool) must be reshaped into CircAST's
  circRNA_list.txt (chromosome/start/end) before full-length assembly.
when_to_use_negative:
- The upstream tool already emits CircAST-compatible format (CIRCexplorer2/CIRI2/UROBORUS
  annotated_circ.txt, result.txt, or circRNA_list.txt can be passed directly via -J).
- The goal is linear transcript assembly (e.g., Cufflinks) or de novo BSJ detection
  itself — this skill only reshapes existing junction lists.
- The circRNAs of interest are intronic or intergenic, which the current CircAST pipeline
  cannot assemble (it handles exon-annotated circular transcripts only).
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: Downstream assembler that consumes the converted circRNA_list.txt via -J to
    reconstruct full-length circular isoforms and quantify expression
  repo: https://github.com/xiaofengsong/CircAST
- name: CIRCexplorer2
  role: Upstream BSJ detector whose junction output is converted to CircAST circRNA_list.txt
    format (chromosome, start, end)
- name: CIRI2
  role: Alternative upstream BSJ detector whose result.txt can feed CircAST directly
    or after format conversion
- name: UROBORUS
  role: Alternative upstream BSJ detector compatible with CircAST's junction list
    format
- name: samtools
  role: Sorts accepted_hits.bam by read name and converts it to accepted_hits.sorted.sam
    required alongside the junction list
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:4ea491355f7aeb81d0b880264ffb910dbf11120de23167ba912e310724b858e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI2
  - CircAST
  - Cufflinks
  - TopHat2
  - UROBORUS
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# circrna-tool-output-format-conversion

## Summary

Convert back-splice junction (BSJ) lists produced by upstream circRNA detectors (CIRCexplorer2, CIRI2, UROBORUS, or other similar tools) into the CircAST circRNA_list.txt input format (chromosome, start, end columns) so CircAST can assemble full-length circular isoforms.

## When to use

Use when a circRNA junction list from a BSJ detector (CIRCexplorer2 annotated_circ.txt, CIRI2 result.txt, UROBORUS, or any similar tool) must be reshaped into CircAST's circRNA_list.txt (chromosome/start/end) before full-length assembly.

## When NOT to use

- The upstream tool already emits CircAST-compatible format (CIRCexplorer2/CIRI2/UROBORUS annotated_circ.txt, result.txt, or circRNA_list.txt can be passed directly via -J).
- The goal is linear transcript assembly (e.g., Cufflinks) or de novo BSJ detection itself — this skill only reshapes existing junction lists.
- The circRNAs of interest are intronic or intergenic, which the current CircAST pipeline cannot assemble (it handles exon-annotated circular transcripts only).

## Inputs

- circRNA junction list from CIRCexplorer2 (e.g., circ_out/denovo/annotated_circ.txt)
- circRNA junction list from CIRI2 (result.txt) or UROBORUS, or any similar tool's BSJ output
- Name-sorted SAM alignment of paired-end reads from tophat2/samtools (accepted_hits.sorted.sam)
- UCSC-compatible GTF annotation file (e.g., hg19 genes_homo.gtf)

## Outputs

- circRNA_list.txt in CircAST input format with chromosome, start, end columns
- Downstream: CircAST output table with circular transcript name, parental gene, strand, exon sizes/offsets, junction read counts, FPKM and TPM expression levels

## How to apply

Run the upstream detector on name-sorted tophat2/samtools alignments (e.g., CIRCexplorer2 on accepted_hits.sorted.bam) and capture its junction output. Convert that output into CircAST's circRNA_list.txt format, keeping the chromosome, start, and end columns consistent with CircAST's first-three-columns BED-like expectation, and be careful about 0- vs 1-based coordinate conventions of the source tool. Use the SAME UCSC-compatible GTF annotation file that was used by the upstream detector when you then run CircAST, since mismatched GTFs will break exon graph construction. Finally, feed the converted file via -J into CircAST (with -L read length and -T junction-supporting-reads threshold, default 10) and confirm CircAST parses the junction list without format errors.

## Related tools

- **CircAST** (Downstream assembler that consumes the converted circRNA_list.txt via -J to reconstruct full-length circular isoforms and quantify expression) — https://github.com/xiaofengsong/CircAST
- **CIRCexplorer2** (Upstream BSJ detector whose junction output is converted to CircAST circRNA_list.txt format (chromosome, start, end))
- **CIRI2** (Alternative upstream BSJ detector whose result.txt can feed CircAST directly or after format conversion)
- **UROBORUS** (Alternative upstream BSJ detector compatible with CircAST's junction list format)
- **samtools** (Sorts accepted_hits.bam by read name and converts it to accepted_hits.sorted.sam required alongside the junction list)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Converted circRNA_list.txt has chromosome, start, end in the first three BED-like columns expected by CircAST, with coordinates matching the upstream tool's output (no off-by-one from 0- vs 1-based shifts).
- CircAST runs to completion with 'python CircAST.py -L <length> -T 10 -G <gtf> -F accepted_hits.sorted.sam -J circRNA_list.txt' and emits the 13-column output table (chromosome ... assigned read counts) without format errors.
- The GTF passed to CircAST (-G) is byte-identical to the one used by the upstream detector, per the README requirement.
- Every back-spliced junction in the upstream detector's output has a corresponding entry in the converted file (no dropped or duplicated loci).
- In benchmark settings (task_006), the assembled transcripts' sensitivity should reach the reported 85.63%-94.32% and precision 81.96%-87.55% against simulated ground-truth isoforms, exceeding CIRCexplorer2's 74.75%-75.61% sensitivity and 58.42%-60.42% precision.

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over; remnant linear mRNA can affect assembly and generate false transcripts.
- The current CircAST pipeline assembles exon-annotated circular transcripts only and may miss intronic or intergenic circular transcripts.
- The CircAST version number is not stated in the paper (README references both v1.0.0 and a v1.0.2 download URL).

## Evidence

- [discussion] Other circRNA junction detection tools can also be used with CircAST if their output file format is converted to CircAST input file format.: "Other circRNA junction detection tools can also be used with CircAST if their output file format is converted to CircAST input file format."
- [readme] CircRNA lists containing back-spliced junctions can be obtained by CIRCexplorer2/CIRI2/UROBORUS: "CircRNA lists containing back-spliced junctions can be obtained by CIRCexplorer2/CIRI2/UROBORUS,  circ_out/denovo/annotated_circ.txt, result.txt, or circRNA_list.txt."
- [readme] The GTF annotation file should be the same one when running CircAST and its upstream softwares.: "The GTF annotation file should be the same one when running CircAST and its upstream softwares."
- [intro] CircAST can perform analysis downstream of existing back splice site detection software such as CIRCexplorer2, CIRI2, and UROBORUS: "CircAST can perform analysis downstream of existing back splice site detection software such as CIRCexplorer2, CIRI2, and UROBORUS"
