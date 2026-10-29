---
name: rna-seq-transcript-assembly
description: Use when raw RNA-Seq FASTQ samples (e.g., 180 TARGET osteosarcoma tumors
  plus 9 normal bone controls) must be converted into a consensus transcript candidate
  set prior to novel lncRNA detection.
when_to_use_negative:
- Input is already an assembled transcriptome (GTF/GFF) or a FASTA of candidate transcript
  sequences — go directly to classification with LncDC
- Only a single aligner/assembler combination is available, since the reliability
  argument rests on consensus between STAR+Cufflinks and HISAT2+StringTie routes
edam_operation: http://edamontology.org/operation_3258
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: STAR
  role: Route 1 read aligner producing alignments for Cufflinks transcript reconstruction
- name: Cufflinks
  role: Route 1 transcript assembler reconstructing transcripts from STAR alignments
- name: HISAT2
  role: Route 2 read aligner producing alignments for StringTie transcript reconstruction
- name: StringTie
  role: Route 2 transcript assembler reconstructing transcripts from HISAT2 alignments
- name: IGV
  role: Visual spot-check validation of consensus candidate assemblies
- name: LncDC
  role: Downstream classifier consuming the consensus candidate FASTA to label 'lncrna'
    vs 'mrna'
  repo: https://github.com/lim74/LncDC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:76fdcd6ce961351705b5984a72ed10a4db2c17f94e0f556cb30bb11d348dd725
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
  edam_operation: http://edamontology.org/operation_3258
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# rna-seq-transcript-assembly

## Summary

Reconstruct transcript assemblies from RNA-Seq reads via two independent aligner/assembler routes (STAR+Cufflinks and HISAT2+StringTie) and retain only candidates whose chromosome positions agree between routes, yielding a high-confidence candidate set for downstream lncRNA classification.

## When to use

Use when raw RNA-Seq FASTQ samples (e.g., 180 TARGET osteosarcoma tumors plus 9 normal bone controls) must be converted into a consensus transcript candidate set prior to novel lncRNA detection.

## When NOT to use

- Input is already an assembled transcriptome (GTF/GFF) or a FASTA of candidate transcript sequences — go directly to classification with LncDC
- Only a single aligner/assembler combination is available, since the reliability argument rests on consensus between STAR+Cufflinks and HISAT2+StringTie routes

## Inputs

- Raw RNA-Seq FASTQ reads from tumor and normal samples (e.g., 180 TARGET-OS samples and 9 normal bone controls)
- Reference genome for STAR and HISAT2 alignment
- Existing gene annotation (e.g., GENCODE/RefSeq) for known-transcript comparison

## Outputs

- Consensus candidate transcript set with chromosome positions identical across both assembly routes
- FASTA file of consensus candidate transcript sequences (plain text or gzip-compressed) ready for LncDC prediction
- Optionally IGV spot-check records supporting assembly reliability

## How to apply

Align and reconstruct transcripts from all samples via two independent routes: STAR followed by Cufflinks (Route 1) and HISAT2 followed by StringTie (Route 2). Intersect the two assemblies, keeping only candidate transcript assemblies whose chromosome positions are identical between Route 1 and Route 2; this consensus filtering plus visual spot-checks in IGV is what the authors credit for the high reliability of the identified novel lncRNAs. Export the consensus candidate transcript sequences to a FASTA file (plain text or gzip-compressed) suitable for LncDC input. The full pipeline was demonstrated on 180 TARGET OS RNA-Seq samples with 9 normal bone controls, producing 101 OS-specific novel transcript candidates before LncDC classification.

## Related tools

- **STAR** (Route 1 read aligner producing alignments for Cufflinks transcript reconstruction)
- **Cufflinks** (Route 1 transcript assembler reconstructing transcripts from STAR alignments)
- **HISAT2** (Route 2 read aligner producing alignments for StringTie transcript reconstruction)
- **StringTie** (Route 2 transcript assembler reconstructing transcripts from HISAT2 alignments)
- **IGV** (Visual spot-check validation of consensus candidate assemblies)
- **LncDC** (Downstream classifier consuming the consensus candidate FASTA to label 'lncrna' vs 'mrna') — https://github.com/lim74/LncDC

## Examples

```
python bin/lncDC.py -i candidates.fa -o lncdc_os.csv -t 8
```

## Evaluation signals

- Every consensus candidate has chromosome positions identical between the STAR/Cufflinks and HISAT2/StringTie assemblies
- The exported candidate FASTA is readable by LncDC (plain text or gzip-compressed *.gz accepted)
- Downstream LncDC prediction CSV contains 'lncrna'-labeled novel transcripts; the TARGET-OS run recovered 97 novel lncRNAs (from 94 genes) out of 101 candidates
- IGV visual spot-checks of candidate loci support assembly reliability
- Novel candidates are absent from existing annotations

## Limitations

- Consensus filtering discards any candidate assembled by only one route, potentially missing genuine transcripts
- Visual validation in IGV is manual and qualitative rather than quantitative
- The pipeline output is candidate transcripts only; biological functions and regulatory mechanisms of the 97 identified OS-specific lncRNAs remain uninvestigated and lack wet-lab validation

## Evidence

- [discussion] by using consensus assemblies from both routes in the pipeline (Supplementary Fig. S5) and validating them by visual check through IGV, the novel lncRNAs we identified are highly reliable: "by using consensus assemblies from both routes in the pipeline (Supplementary Fig. S5) and validating them by visual check through IGV, the novel lncRNAs we identified are highly reliable"
- [discussion] we developed a bioinformatics pipeline to identify OS-specific novel lncRNAs from 180 OS samples: "we developed a bioinformatics pipeline to identify OS-specific novel lncRNAs from 180 OS samples"
- [other] The pipeline produced 101 OS-specific novel transcript candidates, of which LncDC classified 97 as lncRNAs (from 94 lncRNA genes) and 4 as mRNAs.: [evidence span withheld — anchor status: not_found]
- [readme] The fasta file could be regular text file or gzip compressed file (*.gz).: "The fasta file could be regular text file or gzip compressed file (*.gz)."
