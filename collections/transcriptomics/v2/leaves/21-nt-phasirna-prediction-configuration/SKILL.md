---
name: 21-nt-phasirna-prediction-configuration
description: Use when the goal is to predict 21-nt phasiRNAs and PHAS loci from sRNA-seq
  data mapped to cDNA (transcriptome) and/or gDNA (genome) references, e.g. rice SRR5049781
  data against oryza_sativa_cdna.fa and oryza_sativa_gdna.fa.
when_to_use_negative:
- When the aim is 24-nt phasiRNA (reproductive phasiRNA) prediction instead — set
  -pl 24, this configuration targets 21-nt loci
- When only one reference is desired, the phase module is designed to integrate multiple
  reference sequences; using a single -c/-cm or -g/-gm pair discards the multi-reference
  integration capability
- When raw sRNA reads have not been preprocessed — the phase module expects .map files
  and the filtered sRNA fasta, not FASTQ input
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: PhasiHunter
  role: Main CLI tool; its 'phase' submodule performs 21-nt phasiRNA and PHAS loci
    prediction using hypergeometric test and phase score methods with multithreading
    (-j)
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Used in the preprocess module to map sRNA reads to reference sequences, producing
    the .map files consumed by the phase module
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:16a122e98347dc55a2499cb2e04dc7c1232ecca14c160866d9ed528badd31260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bowtie
  - PhaseTank
  - PhasiHunter
  - PhasiRNAnalyzer
  - unitas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# 21-nt-phasiRNA-prediction-configuration

## Summary

Configure and run PhasiHunter's phase module for 21-nt phasiRNA (phased siRNA) prediction against transcriptome and genome references using both phase score and hypergeometric methods, with the preprocessed sRNA mapping files as input.

## When to use

Use when the goal is to predict 21-nt phasiRNAs and PHAS loci from sRNA-seq data mapped to cDNA (transcriptome) and/or gDNA (genome) references, e.g. rice SRR5049781 data against oryza_sativa_cdna.fa and oryza_sativa_gdna.fa.

## When NOT to use

- When the aim is 24-nt phasiRNA (reproductive phasiRNA) prediction instead — set -pl 24, this configuration targets 21-nt loci
- When only one reference is desired, the phase module is designed to integrate multiple reference sequences; using a single -c/-cm or -g/-gm pair discards the multi-reference integration capability
- When raw sRNA reads have not been preprocessed — the phase module expects .map files and the filtered sRNA fasta, not FASTQ input

## Inputs

- Preprocessed sRNA mapping file for transcriptome reference (.map, e.g. SRR5049781_processed_cdna.map)
- Preprocessed sRNA mapping file for genome reference (.map, e.g. SRR5049781_processed_gdna.map)
- Reference transcriptome FASTA (e.g. oryza_sativa_cdna.fa)
- Reference genome FASTA (e.g. oryza_sativa_gdna.fa)
- Length- and abundance-filtered sRNA FASTA (e.g. SRR5049781_trimmed_format_filter.fa)

## Outputs

- AllsiRNA cluster output file (phase_a.txt)
- phasiRNA cluster output file (phase_o.txt)
- Optionally deleted .phasiHuter_bowtieIndex when -cl y

## How to apply

First run the PhasiHunter preprocess module (modes r/c/m) to trim adapters, normalize, apply length (19–25 nt) and abundance (CPM ≥ 1) filters, and map reads with Bowtie to produce the .map files (e.g. SRR5049781_processed_cdna.map and SRR5049781_processed_gdna.map). Then run 'phasiHunter phase' supplying the transcriptome map/reference (-cm/-c), the genome map/reference (-gm/-g), the filtered sRNA fasta (-fa), and set -pl 21 for the 21-nt phase length. Keep default prediction parameters unless justified: -me b (both hypergeometric test and phase score), -pn 4 phase number, -il 5 island, -pv 0.001 p-value cutoff, -ps 15 phase score cutoff, -pr 0.4 phase ratio cutoff, -mh 10 max hits. Use -j to set parallel threads; the article reports four-thread genome prediction completes in ~45 min versus 91.03 min single-threaded. Outputs are the allsiRNA cluster file (-a, phase_a.txt) and the phasiRNA cluster file (-o, phase_o.txt), which feed the subsequent integration module.

## Related tools

- **PhasiHunter** (Main CLI tool; its 'phase' submodule performs 21-nt phasiRNA and PHAS loci prediction using hypergeometric test and phase score methods with multithreading (-j)) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Used in the preprocess module to map sRNA reads to reference sequences, producing the .map files consumed by the phase module)

## Examples

```
phasiHunter phase -cm test_osa/SRR5049781_processed_cdna.map -c test_osa/oryza_sativa_cdna.fa -gm test_osa/SRR5049781_processed_gdna.map -g test_osa/oryza_sativa_gdna.fa -fa test_osa/SRR5049781_trimmed_format_filter.fa -a phase_a.txt -o phase_o.txt -me b -pl 21 -j 4
```

## Evaluation signals

- The phase module produces both phase_a.txt (allsiRNA clusters) and phase_o.txt (phasiRNA clusters) without errors and with non-empty phasiRNA cluster content
- Predicted phasiRNA clusters satisfy the configured cutoffs: phase score ≥ 15 (-ps), phase ratio ≥ 0.4 (-pr), hypergeometric p-value ≤ 0.001 (-pv), phase number ≥ 4 (-pn)
- All input sRNA reads are within the 19–25 nt length window and passed the CPM ≥ 1 (-e 1) abundance filter during preprocess
- Runtime sanity check: single-threaded genome-based 21-nt prediction on the rice demo data should be on the order of ~91 min, and ~45 min with -j 4
- The -cl y option removes the temporary .phasiHuter_bowtieIndex after the run completes

## Limitations

- 21-nt and 24-nt phasiRNA predictions use the same phase module but different phase lengths; using -pl 21 on reproductive-tissue data aimed at 24-nt phasiRNAs will miss them
- Max hits when mapping to reference (-mh 10) limits multi-mapping sRNAs, which may affect phasiRNAs in repetitive regions
- Only the update log 1.0.1 (25/02/2024) was available; no changelog details on limitations were extractable from the article's discussion section
- Runtime scales strongly with reference size: genome-based prediction (~91 min single-thread) is ~4.3x slower than transcriptome-based (~21.41 min)

## Evidence

- [readme] phase module parameters for 21-nt prediction: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] phase length option: "-pl: int   --  phase length, 21 | 24, default=21"
- [readme] phase score and ratio cutoffs: "-ps: float --  phase score cutoff, default=15, only function with p/b method applied"
- [abstract] two prediction algorithms: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods"
- [abstract] parallel computing rationale: "the excellent data structure and parallel computing architecture allow PhasiHunter to predict phasiRNAs and their regulatory pathways with high efficiency"
- [results] benchmark runtime finding: [evidence span withheld — anchor status: not_found]
