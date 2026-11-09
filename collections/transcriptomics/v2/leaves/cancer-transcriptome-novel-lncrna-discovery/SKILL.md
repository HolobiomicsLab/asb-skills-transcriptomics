---
name: cancer-transcriptome-novel-lncrna-discovery
description: Use when you have bulk RNA-Seq from a tumor cohort (e.g., 180 TARGET-OS
  samples) plus normal controls and need to discover novel, tumor-specific lncRNA
  transcripts absent from existing annotations.
when_to_use_negative:
- The transcript candidates are already annotated — the goal is novel lncRNAs absent
  from existing annotations, not re-classification of known ones
- Input is raw reads without a viable reference/assembly strategy — this pipeline
  requires transcript reconstruction from alignments, not de novo genome-free discovery
- You need functional characterization or quantification of lncRNAs — the study explicitly
  notes the 97 novel lncRNAs lack wet-lab validation and their biological functions/regulatory
  mechanisms remain uninvestigated
edam_operation: http://edamontology.org/operation_0435
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: LncDC
  role: XGBoost-based classifier labeling consensus candidate transcripts as 'lncrna'
    or 'mrna' from FASTA input
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: Machine learning algorithm underlying the LncDC default human prediction model
  repo: https://github.com/lim74/LncDC
- name: Python
  role: Implementation language for running LncDC prediction and filtering the output
    CSV
  repo: https://github.com/lim74/LncDC
- name: ViennaRNA
  role: Optional RNAfold secondary structure computation enabling the '-r' SSF prediction
    mode
  repo: https://www.tbi.univie.ac.at/RNA/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:aa5bb5066092a69268faaa61d295705c5ca159ffbc2f28fb1a9744859a3c94d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - ViennaRNA
  - XGBoost
  edam_operation: http://edamontology.org/operation_0435
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cancer-transcriptome-novel-lncrna-discovery

## Summary

A bioinformatics pipeline that identifies novel cancer-specific (e.g., osteosarcoma) lncRNAs from tumor RNA-Seq by reconstructing transcripts via two independent assembly routes, taking the consensus, and classifying candidates as lncRNA vs mRNA with the LncDC XGBoost classifier.

## When to use

Use when you have bulk RNA-Seq from a tumor cohort (e.g., 180 TARGET-OS samples) plus normal controls and need to discover novel, tumor-specific lncRNA transcripts absent from existing annotations.

## When NOT to use

- The transcript candidates are already annotated — the goal is novel lncRNAs absent from existing annotations, not re-classification of known ones
- Input is raw reads without a viable reference/assembly strategy — this pipeline requires transcript reconstruction from alignments, not de novo genome-free discovery
- You need functional characterization or quantification of lncRNAs — the study explicitly notes the 97 novel lncRNAs lack wet-lab validation and their biological functions/regulatory mechanisms remain uninvestigated

## Inputs

- Bulk RNA-Seq FASTQ files from a tumor cohort (e.g., 180 TARGET OS samples) and normal tissue controls (e.g., 9 normal bone samples)
- Reference genome alignment toolchain (STAR or HISAT2) and transcript assemblers (Cufflinks or StringTie)
- Consensus candidate transcript sequences in FASTA format (plain text or gzip-compressed *.gz)
- Existing gene annotations (e.g., GENCODE/RefSeq) for filtering known transcripts

## Outputs

- Consensus candidate transcript set (identical chromosome positions across Route 1 and Route 2)
- LncDC prediction CSV labeling each candidate as 'lncrna' or 'mrna'
- Filtered novel tumor-specific lncRNA set (e.g., os_specific_lncrnas.csv recovering the 97 novel lncRNAs from 94 lncRNA genes)

## How to apply

Align and reconstruct transcripts from the tumor samples (e.g., 180 TARGET-OS RNA-Seq) and normal controls via two independent routes — STAR+Cufflinks (Route 1) and HISAT2+StringTie (Route 2) — then retain only candidate assemblies whose chromosome positions are identical in both routes; spot-check the consensus in IGV. Export the consensus transcript sequences to a FASTA file (plain text or gzip-compressed) and run LncDC with the default human model ('python bin/lncDC.py -i candidates.fa -o lncdc_os.csv -t 8'), optionally adding '-r' for secondary structure features computed by RNAfold/ViennaRNA at the cost of long runtime. Finally filter the prediction CSV to transcripts labeled 'lncrna' that are absent from existing annotations to obtain the novel tumor-specific lncRNA set. The dual-route consensus plus IGV visual validation is what makes the resulting novel lncRNAs highly reliable.

## Related tools

- **LncDC** (XGBoost-based classifier labeling consensus candidate transcripts as 'lncrna' or 'mrna' from FASTA input) — https://github.com/lim74/LncDC
- **XGBoost** (Machine learning algorithm underlying the LncDC default human prediction model) — https://github.com/lim74/LncDC
- **Python** (Implementation language for running LncDC prediction and filtering the output CSV) — https://github.com/lim74/LncDC
- **ViennaRNA** (Optional RNAfold secondary structure computation enabling the '-r' SSF prediction mode) — https://www.tbi.univie.ac.at/RNA/

## Examples

```
python bin/lncDC.py -i candidates.fa -o lncdc_os.csv -t 8
```

## Evaluation signals

- os_specific_lncrnas.csv exists, is valid CSV, and contains 'lncrna'-labeled novel transcripts
- The result recovers the 97 novel lncRNAs (from 94 lncRNA genes) plus 4 mRNAs reported from the 180 TARGET-OS samples (101 candidates total)
- Every retained candidate has identical chromosome positions in both STAR/Cufflinks (Route 1) and HISAT2/StringTie (Route 2) assemblies
- Consensus assemblies visually verified in IGV as done in the study
- LncDC runs the default human model ('XGB_model_SIF_PF.pkl') with test_requirements.py passing before prediction

## Limitations

- Adding '-r' for secondary structure features is time-consuming because RNAfold calculation is slow
- The 97 novel lncRNAs lack wet-lab validation and functional characterization; their biological functions and regulatory mechanisms remain uninvestigated
- Further accuracy improvement was difficult because SIFs and PFs already achieved high accuracy, and the enhancement from the 6 selected SSFs is marginal
- For non-model organisms without enough annotated lncRNAs, a self-trained model from an evolutionarily close model organism is needed via lncDC-train.py

## Evidence

- [discussion] consensus candidate set from two assembly routes: "by using consensus assemblies from both routes in the pipeline (Supplementary Fig. S5) and validating them by visual check through IGV, the novel lncRNAs we identified are highly reliable"
- [methods] LncDC prediction command: "python bin/lncDC.py -i test/human_test_toy.fasta -o lncdc_human.csv -t 8"
- [abstract] LncDC classifier description: "we present our tool LncDC (Long non-coding RNA detection) that is able to accurately predict lncRNAs with an XGBoost model"
- [readme] input/output format spec: "The inputfile with RNA sequences in fasta format. The fasta file could be regular text file or gzip compressed file (*.gz)."
- [readme] secondary structure runtime caveat: "This may take a longer time because secondary structure calculation by RNAfold (The main program of ViennaRNA) is time-consuming."
- [discussion] future validation gap: "Future works should focus on validating the 97 novel lncRNAs by wet-lab approaches and investigating their biological functions and regulatory mechanisms"
