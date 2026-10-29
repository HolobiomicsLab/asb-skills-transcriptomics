---
name: expectation-maximization-read-assignment-quantification
description: Use when you have a name-sorted paired-end SAM from TopHat2, a UCSC-format
  GTF, and a back-spliced junction list from CIRCexplorer2/CIRI2/UROBORUS, and you
  need full-length circRNA isoform sequences plus isoform-level expression (FPKM/TPM).
when_to_use_negative:
- The sample was not RNase R-treated — CircAST can be applied only to RNase R-treated
  samples, and remnant linear mRNA can degrade assembly and generate false transcripts
- You need intronic or intergenic circular transcripts — the pipeline assembles only
  exon-annotated circular transcripts
- The input junction list comes from a tool whose output format has not been converted
  to CircAST input format
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: CircAST
  role: 'Core tool: builds multiple splice graphs, runs EMPC assembly, and quantifies
    isoforms by EM/maximum-likelihood read assignment'
  repo: https://github.com/xiaofengsong/CircAST
- name: TopHat2
  role: Produces the paired-end BAM alignment (accepted_hits.bam) that is sorted by
    name and converted to the CircAST SAM input
- name: SAMtools
  role: Sorts accepted_hits.bam by read name and converts it to the accepted_hits.sorted.sam
    CircAST input
- name: CircFullSeq.py
  role: Companion script that extracts full sequences of assembled circular transcripts
    from CircAST results using UCSC chromFa files
  repo: https://github.com/xiaofengsong/CircAST
- name: CIRCexplorer2
  role: Upstream back-spliced junction detector whose output serves as the -J junction
    input
- name: CIRI2
  role: Upstream back-spliced junction detector (result.txt) compatible with CircAST
    input
- name: UROBORUS
  role: Upstream back-spliced junction detector (circRNA_list.txt) compatible with
    CircAST input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:c098bf9ff51f00af707651c0dc3e5d82585c6367567280fe2343adcbce2ce936
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI2
  - CircAST
  - CircFullSeq.py
  - SAMtools
  - TopHat2
  - UROBORUS
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# Expectation-maximization read assignment quantification

## Summary

CircAST reconstructs full-length circRNA isoforms with multiple splice graphs and the EMPC algorithm, then quantifies isoform expression by allocating reads to isoforms with an expectation-maximization / maximum likelihood estimation procedure. This yields FPKM/TPM and read counts per circular transcript.

## When to use

Use when you have a name-sorted paired-end SAM from TopHat2, a UCSC-format GTF, and a back-spliced junction list from CIRCexplorer2/CIRI2/UROBORUS, and you need full-length circRNA isoform sequences plus isoform-level expression (FPKM/TPM).

## When NOT to use

- The sample was not RNase R-treated — CircAST can be applied only to RNase R-treated samples, and remnant linear mRNA can degrade assembly and generate false transcripts
- You need intronic or intergenic circular transcripts — the pipeline assembles only exon-annotated circular transcripts
- The input junction list comes from a tool whose output format has not been converted to CircAST input format

## Inputs

- GTF gene annotation file in UCSC-compatible format (e.g. hg19 or mm10 from UCSC)
- Name-sorted paired-end SAM alignment from TopHat2 and samtools (accepted_hits.sorted.sam)
- circRNA back-spliced junction list from CIRCexplorer2/CIRI2/UROBORUS (e.g. circRNA_list.txt, annotated_circ.txt, result.txt)
- Read length (integer) for the -L parameter
- Decompressed UCSC chromFa.tar.gz chromosome FASTA folder (for CircFullSeq.py sequence extraction)

## Outputs

- Tab-separated table with chromosome, circRNA start/end, circular transcript name, parental gene, strand, exon count/sizes/offsets
- Per-isoform junction read counts and assigned read counts
- Per-isoform expression levels in FPKM and TPM
- Full-length circular transcript sequences extracted via CircFullSeq.py

## How to apply

Build multiple splice graphs at each gene locus from back-splice junction (BSJ) signatures in the aligned reads without prior assumptions, then run the EMPC algorithm on the graphs to assemble multiple circular transcripts. Assign reads to specific isoforms with the expectation-maximization algorithm and compute expression levels via maximum likelihood estimation, reporting junction read counts, FPKM, TPM, and assigned read counts per transcript. Key inputs are a UCSC-compatible GTF (the same one used by the upstream BSJ detector), a name-sorted SAM (samtools sort -n, then samtools view to SAM), and a junction list; the junction-support read threshold (-T) is optional with a default of 10. Use CircFullSeq.py afterwards to extract full sequences of the assembled circular transcripts from UCSC chromFa files. In simulations, this method reached sensitivity of 85.63%-94.32% and precision of 81.96%-87.55%, with quantification correlation coefficients of 0.85-0.99 between theoretical and estimated values.

## Related tools

- **CircAST** (Core tool: builds multiple splice graphs, runs EMPC assembly, and quantifies isoforms by EM/maximum-likelihood read assignment) — https://github.com/xiaofengsong/CircAST
- **TopHat2** (Produces the paired-end BAM alignment (accepted_hits.bam) that is sorted by name and converted to the CircAST SAM input)
- **SAMtools** (Sorts accepted_hits.bam by read name and converts it to the accepted_hits.sorted.sam CircAST input)
- **CircFullSeq.py** (Companion script that extracts full sequences of assembled circular transcripts from CircAST results using UCSC chromFa files) — https://github.com/xiaofengsong/CircAST
- **CIRCexplorer2** (Upstream back-spliced junction detector whose output serves as the -J junction input)
- **CIRI2** (Upstream back-spliced junction detector (result.txt) compatible with CircAST input)
- **UROBORUS** (Upstream back-spliced junction detector (circRNA_list.txt) compatible with CircAST input)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Simulated-data sensitivity falls within the reported 85.63%-94.32% range and precision within 81.96%-87.55% against simulation ground-truth isoforms
- Correlation coefficients between theoretical and estimated isoform expression fall in the 0.85-0.99 range in simulation
- Output file contains 13 tab-separated columns matching the documented schema, including FPKM (col 11) and TPM (col 12)
- The SAM input is sorted by read name (samtools sort -n) before CircAST, and the same GTF was used for TopHat2, the BSJ detector, and CircAST
- RT-PCR and Sanger sequencing validation reconstructs the expected isoform structures (32/37 validated isoforms in the mouse testis study)

## Limitations

- CircAST can be applied only to RNase R-treated samples; remnant linear mRNA can affect assembly performance and generate false transcripts
- The current pipeline may miss intronic or intergenic circular transcripts, assembling only exon-annotated circular transcripts
- The BSJ threshold -T defaults to 10 and must be tuned; the GTF must be UCSC-compatible and identical across mapping, BSJ detection, and CircAST runs
- Other BSJ detection tools require output file format conversion before use with CircAST

## Evidence

- [intro] uses multiple splice graphs as a circRNA model to reconstruct the full-length sequence and estimate expression levels by allocating reads to different isoforms with an: "we developed CircAST. This tool uses multiple splice graphs as a circRNA model to reconstruct the full-length sequence and estimate expression levels by allocating reads to different isoforms with an"
- [discussion] used the EMPC algorithm on these multiple splice graphs to assemble the multiple circular transcripts: "We constructed multiple splice graphs at each gene locus on the basis of the BSJ signatures of sequencing data without relying on any prior assumptions"
- [discussion] maximum likelihood estimation algorithm to calculate the expression levels: "We then used a maximum likelihood estimation algorithm to calculate the expression levels."
- [readme] CircAST is implemented in Python2 under Linux system: "CircAST is implemented in Python2 under Linux system. Since several packages are required for using CircAST, we suggest that users install corresponding version of Anaconda"
- [abstract] sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [abstract] correlation coefficients of 0.85-0.99 between theoretical and estimated values: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [readme] The GTF annotation file should be the same one when running CircAST and its upstream softwares: "The GTF annotation file should be the same one when running CircAST and its upstream softwares."
