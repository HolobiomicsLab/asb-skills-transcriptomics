---
name: command-line-bioinformatics-workflow-execution
description: Use when the analysis goal is to run a documented multi-step phasiRNA/PHAS
  loci prediction workflow from the command line on a Linux platform, using sRNA FASTQ/FASTA
  inputs, cDNA- and genome-based Bowtie map files, and reference transcriptome/genome
  FASTAs.
when_to_use_negative:
- The input is already a post-prediction product (e.g. an existing phase_o.txt/phase_a.txt)
  and the goal is annotation-level integration only — run `phasiHunter integration`
  rather than re-executing preprocess/phase.
- The platform is not Linux or Bowtie/Biopython/Bedtools/Trim_galore/Seqkit/Perl5/Dnapi
  dependencies cannot be installed — PhasiHunter is documented as 'a CLI program runing
  on linux platform'.
- Only a single reference sequence (genome OR transcriptome) is available and the
  multi-reference cDNA+genome+full-length-transcriptome mapping design of PhasiHunter
  cannot be realized.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: PhasiHunter
  role: primary CLI program whose preprocess/phase/integration submodules execute
    the phasiRNA and PHAS loci prediction workflow
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: short-read mapper used by the preprocess module to build map files against
    cDNA and genome references (max hits -mh 10)
- name: Trim_galore
  role: adaptor trimming step inside r-mode preprocessing (trim adaptor --> normalization
    --> length and abundance filter --> mapping)
  repo: https://github.com/FelixKrueger/TrimGalore
- name: Biopython
  role: Python dependency used by PhasiHunter for sequence handling
- name: Bedtools
  role: dependency supporting interval operations in the pipeline
- name: Seqkit
  role: dependency supporting FASTA/FASTQ manipulation in preprocessing
- name: Dnapi
  role: dependency for phasiRNA initiator-related prediction steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:7ce4ef5f6be2ad2ef58163b6ad5b35a0459e5404f55bf6d608b789ca6b7a38d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bedtools
  - Biopython
  - Bowtie
  - Dnapi
  - PhasiHunter
  - Seqkit
  - Trim_galore
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# command-line-bioinformatics-workflow-execution

## Summary

Execute the PhasiHunter command-line pipeline — a multithreaded CLI program for mining phasiRNA regulation pathways — through its documented submodules (preprocess, phase, integration, target/initiator) with the exact parameter vocabulary (e.g. -me b, -pv 0.001, -ps 15, -pr 0.4) that the tool expects on Linux.

## When to use

Use when the analysis goal is to run a documented multi-step phasiRNA/PHAS loci prediction workflow from the command line on a Linux platform, using sRNA FASTQ/FASTA inputs, cDNA- and genome-based Bowtie map files, and reference transcriptome/genome FASTAs.

## When NOT to use

- The input is already a post-prediction product (e.g. an existing phase_o.txt/phase_a.txt) and the goal is annotation-level integration only — run `phasiHunter integration` rather than re-executing preprocess/phase.
- The platform is not Linux or Bowtie/Biopython/Bedtools/Trim_galore/Seqkit/Perl5/Dnapi dependencies cannot be installed — PhasiHunter is documented as 'a CLI program runing on linux platform'.
- Only a single reference sequence (genome OR transcriptome) is available and the multi-reference cDNA+genome+full-length-transcriptome mapping design of PhasiHunter cannot be realized.

## Inputs

- sRNA sequencing FASTQ or FASTQ.gz file (e.g. SRR5049781.fastq.gz) for r-mode preprocessing
- length- and abundance-filtered sRNA FASTA (e.g. SRR5049781_trimmed_format_filter.fa) for m-mode preprocessing and as the phase module -fa input
- preprocessed cDNA-based map file (-cm, e.g. SRR5049781_processed_cdna.map)
- preprocessed genome-based map file (-gm, e.g. SRR5049781_processed_gdna.map)
- reference transcriptome FASTA (-c, e.g. oryza_sativa_cdna.fa)
- reference genome FASTA (-g, e.g. oryza_sativa_gdna.fa)
- Bowtie index prefixes (e.g. index/oryza_sativa_cdna_index, index/oryza_sativa_gdna_index)
- reference annotation GFF3 for the integration module (-an, e.g. oryza_sativa_gdna.gff3)

## Outputs

- redundant allsiRNA cluster output file (phase_a.txt) with columns gene/PHAS_gene, strand, position, abundance, record, sequence, length, pvalue, phase_ratio, phase_number, phase_abundance, phase_score, marker
- redundant phasiRNA cluster output file (phase_o.txt) with the same documented columns
- integrated result files from the integration module (e.g. integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt)

## How to apply

Run the workflow in the article's documented order: (1) `phasiHunter preprocess` to trim adaptor, normalize (CPM cutoff -e 1, normalization base -n 1000000), length-and-abundance filter (sRNA lengths 19–25 nt via -mi 19 -ma 25), and map reads with Bowtie (max hits -mh 10) against the oryza_sativa_cdna and oryza_sativa_gdna indexes, producing the cDNA map (-cm) and genome map (-gm) files. (2) `phasiHunter phase` with both map files, both reference FASTAs, and the filtered sRNA FASTA (-fa), using the combined prediction method -me b (phase score plus hypergeometric test), island 5 (-il 5), phase length 21 (-pl 21), phase number 4 (-pn 4), pvalue cutoff -pv 0.001, phase score cutoff -ps 15, phase ratio cutoff -pr 0.4, and parallel jobs -j 20, writing phase_a.txt (redundant allsiRNA clusters) and phase_o.txt (redundant phasiRNA clusters); pass -cl y to delete the temporary .phasiHuter_bowtieIndex. (3) `phasiHunter integration` with -io phase_o.txt, -ia phase_a.txt and a reference GFF3 (-an) to merge results. The rationale for -me b is that PhasiHunter employs two major phasiRNA prediction algorithms (phase score and hypergeometric distribution-based methods) to ensure prediction integrity and accuracy, and the hypergeometric method runs both forward and reverse searches to enhance PHAS loci detection sensitivity.

## Related tools

- **PhasiHunter** (primary CLI program whose preprocess/phase/integration submodules execute the phasiRNA and PHAS loci prediction workflow) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (short-read mapper used by the preprocess module to build map files against cDNA and genome references (max hits -mh 10))
- **Trim_galore** (adaptor trimming step inside r-mode preprocessing (trim adaptor --> normalization --> length and abundance filter --> mapping)) — https://github.com/FelixKrueger/TrimGalore
- **Biopython** (Python dependency used by PhasiHunter for sequence handling)
- **Bedtools** (dependency supporting interval operations in the pipeline)
- **Seqkit** (dependency supporting FASTA/FASTQ manipulation in preprocessing)
- **Dnapi** (dependency for phasiRNA initiator-related prediction steps)

## Examples

```
phasiHunter phase -cm SRR5049781_processed_cdna.map -c oryza_sativa_cdna.fa -gm SRR5049781_processed_gdna.map -g oryza_sativa_gdna.fa -fa SRR5049781_trimmed_format_filter.fa -a phase_a.txt -o phase_o.txt -me b -il 5 -pl 21 -pn 4 -mh 10 -j 20 -pv 0.001 -ps 15 -pr 0.4 -cl y
```

## Evaluation signals

- Both phase_a.txt and phase_o.txt exist and their table headers contain the documented columns including pvalue, phase_ratio, phase_score, phase_number and phase_abundance fields.
- Reported per-locus values respect the applied cutoffs: pvalue <= 0.001 (-pv 0.001), phase_score >= 15 (-ps 15), and phase_ratio >= 0.4 (-pr 0.4) for loci retained under -me b.
- Preprocess outputs SRR5049781_processed_cdna.map and SRR5049781_processed_gdna.map are non-empty and consistent with the 19–25 nt length filter (-mi 19 -ma 25) and CPM cutoff -e 1.
- With -cl y, the temporary .phasiHuter_bowtieIndex directory is deleted after prediction completes.
- Running with -v prints version information and `phasihunter -h` shows the expected help content, confirming a correct installation before execution.

## Limitations

- PhasiHunter is a CLI program running on the Linux platform only; dependencies (Bowtie, Biopython, Bedtools, Dnapi, Trim_galore, Seqkit, Perl5, Fasta36, TarHunter) must all be installed, or the provided Docker image or conda/mamba env.yaml must be used.
- The article's changelog only lists an update-log entry 1.0.1 (25/02/2024) and no discussion-section changelog was found, so version-specific limitations and reproducibility details are sparse.
- The redundant output files phase_a.txt and phase_o.txt are pre-integration outputs; annotation-aware results require a subsequent `phasiHunter integration` step with a reference GFF3.

## Evidence

- [abstract] PhasiHunter employs two major phasiRNA prediction algorithms: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction"
- [readme] phase module parameters: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] preprocess filter chain: "raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping"
- [readme] phase cutoff options: "-pv: float --  pvalue cutoff, default=0.001, only function with h/b method applied"
- [methods] CLI platform statement: "phasihunter is a CLI program runing on linux platform."
