---
name: dual-route-consensus-transcript-filtering
description: Use when assembling a de novo transcript set from RNA-Seq (e.g. 180 TARGET
  OS tumor samples plus 9 normal bone controls) and you need a high-confidence novel
  candidate set prior to lncRNA/mRNA classification with LncDC.
when_to_use_negative:
- You only have a single assembler available and cannot build the two-route consensus
  — intersecting one route with itself provides no independent validation
- Input is already a curated, annotated transcript set and the goal is differential
  expression rather than novel transcript discovery
- You need coding-potential classification of reads directly rather than reconstructed
  transcript assemblies (route the reads through LncDC only after assembly)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STAR
  role: Route 1 splice-aware aligner producing BAM input for Cufflinks transcript
    assembly
- name: Cufflinks
  role: Route 1 transcript reconstruction tool from STAR alignments
- name: HISAT2
  role: Route 2 splice-aware aligner producing BAM input for StringTie assembly
- name: StringTie
  role: Route 2 transcript reconstruction tool from HISAT2 alignments
- name: IGV
  role: Visual spot-check of consensus transcript loci to confirm assembly reliability
- name: LncDC
  role: Downstream XGBoost-based lncRNA/mRNA classifier consuming the consensus candidate
    FASTA (default human model, optional -r for secondary structure features)
  repo: https://github.com/lim74/LncDC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:675f2d04322882373fd24c061ddf9546c037eb9013ae9a58991262b424fc4905
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - Cufflinks
  - HISAT2
  - IGV
  - LncDC
  - Python
  - STAR
  - StringTie
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# dual-route-consensus-transcript-filtering

## Summary

A transcript-discovery safeguard from the LncDC osteosarcoma pipeline: novel lncRNA candidates are assembled twice (STAR+Cufflinks and HISAT2+StringTie over the same 180 TARGET-OS RNA-Seq samples) and only assemblies with identical chromosome positions across both routes are kept, then spot-checked in IGV before LncDC classification.

## When to use

Use when assembling a de novo transcript set from RNA-Seq (e.g. 180 TARGET OS tumor samples plus 9 normal bone controls) and you need a high-confidence novel candidate set prior to lncRNA/mRNA classification with LncDC.

## When NOT to use

- You only have a single assembler available and cannot build the two-route consensus — intersecting one route with itself provides no independent validation
- Input is already a curated, annotated transcript set and the goal is differential expression rather than novel transcript discovery
- You need coding-potential classification of reads directly rather than reconstructed transcript assemblies (route the reads through LncDC only after assembly)

## Inputs

- RNA-Seq FASTQ reads from 180 TARGET osteosarcoma samples and 9 normal bone controls
- Reference genome for STAR and HISAT2 alignment
- Existing transcript/gene annotations (e.g. GENCODE, NCBI RefSeq) used to exclude already-annotated transcripts
- Optionally gzip-compressed FASTA of candidate transcripts for downstream LncDC prediction

## Outputs

- Consensus candidate transcript set: assemblies with identical chromosome positions across STAR+Cufflinks and HISAT2+StringTie routes
- FASTA file of consensus candidate transcript sequences (plain text or *.gz) for LncDC
- Downstream lncdc output CSV with 'lncrna'/'mrna' labels (e.g. lncdc_os.csv), yielding the os_specific_lncrnas.csv novel set

## How to apply

Run two independent reconstruction routes on the same RNA-Seq samples: Route 1 aligns with STAR and assembles with Cufflinks, Route 2 aligns with HISAT2 and assembles with StringTie. Intersect the two candidate transcript sets by exact chromosome position, keeping only assemblies whose coordinates are identical in both routes — this consensus step is the paper's stated reliability mechanism for novel lncRNAs. Visually validate a subset of consensus loci in IGV, then export the consensus candidate transcript sequences to FASTA (plain text or gzip-compressed *.gz) suitable as LncDC input ('python bin/lncDC.py -i candidates.fa -o lncdc_os.csv -t 8'). In the TARGET-OS study this produced 101 OS-specific novel transcript candidates, of which LncDC's default human XGBoost model labeled 97 as lncRNAs (from 94 genes) and 4 as mRNAs.

## Related tools

- **STAR** (Route 1 splice-aware aligner producing BAM input for Cufflinks transcript assembly)
- **Cufflinks** (Route 1 transcript reconstruction tool from STAR alignments)
- **HISAT2** (Route 2 splice-aware aligner producing BAM input for StringTie assembly)
- **StringTie** (Route 2 transcript reconstruction tool from HISAT2 alignments)
- **IGV** (Visual spot-check of consensus transcript loci to confirm assembly reliability)
- **LncDC** (Downstream XGBoost-based lncRNA/mRNA classifier consuming the consensus candidate FASTA (default human model, optional -r for secondary structure features)) — https://github.com/lim74/LncDC

## Examples

```
python bin/lncDC.py -i candidates.fa -o lncdc_os.csv -t 8
```

## Evaluation signals

- Every retained consensus transcript has identical chromosome coordinates (chromosome positions) in both the Route 1 and Route 2 assemblies
- The exported candidate file is a valid FASTA (regular text or gzip-compressed *.gz) accepted by LncDC's '-i' input
- LncDC prediction CSV on the consensus candidates contains only 'lncrna'/'mrna' labels; the study recovered 101 candidates with 97 'lncrna' and 4 'mrna' labels
- Final os_specific_lncrnas.csv is valid CSV, contains only 'lncrna'-labeled transcripts absent from existing annotations, and reproduces the 97 novel lncRNAs reported by the study
- IGV visual checks of consensus loci show concordant coverage and splice junctions supporting the assembled transcripts

## Limitations

- The 97 novel OS-specific lncRNAs lack wet-lab validation and their biological functions and regulatory mechanisms remain uninvestigated
- Consensus reliability is supported by IGV visual spot-checks rather than exhaustive experimental validation of every locus
- If the optional LncDC '-r' secondary-structure mode is used downstream, RNAfold/ViennaRNA computation is time-consuming

## Evidence

- [discussion] Consensus assembly from two pipeline routes followed by visual validation in IGV: "by using consensus assemblies from both routes in the pipeline (Supplementary Fig. S5) and validating them by visual check through IGV, the novel lncRNAs we identified are highly reliable"
- [intro] Pipeline combining LncDC with assembly tools on 180 OS samples: "we developed a bioinformatics pipeline that combines LncDC and other popular bioinformatics tools to detect novel OS-specific lncRNAs from 180 OS RNA-Seq data"
- [readme] Downstream LncDC invocation pattern: "python bin/lncDC.py -i test/human_test_toy.fasta -o lncdc_human.csv -t 8"
- [discussion] Unvalidated novel lncRNAs: "Future works should focus on validating the 97 novel lncRNAs by wet-lab approaches and investigating their biological functions and regulatory mechanisms"
