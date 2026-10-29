---
name: phased-sirna-loci-prediction
description: Use when sRNA-seq reads (19–25 nt, e.g. rice SRR5049781) must be scanned
  for phased 21-nt siRNA stacks and PHAS loci, especially when genome- and transcriptome-based
  predictions should be combined to catch alternative-splicing or novel unannotated
  loci.
when_to_use_negative:
- The input is degradome data or miRNA target verification — use the separate target/initiator
  modules instead of the phase module.
- Only a single reference (genome alone) is available and no transcriptome/flnc integration
  or annotation-based locus naming is needed — the multi-reference value-add (alternative-splicing
  and novel-locus detection) is lost.
- The sRNA reads fall outside the 19–25 nt length window appropriate for 21-nt phasiRNAs
  (e.g. purely 24-nt phased siRNAs would require -pl 24 and this workflow's defaults
  would miss them).
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: PhasiHunter
  role: 'core tool: preprocess, phase (hypergeometric + phase score PHAS loci prediction),
    and integration submodules'
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: maps filtered sRNA reads to cDNA/gDNA/flnc references with max hits 10 during
    preprocess and phase
- name: Trim_galore
  role: adapter trimming in the preprocess r-mode raw pipeline
  repo: https://github.com/FelixKrueger/TrimGalore
- name: Seqkit
  role: sequence manipulation supporting the length/abundance filtering step
- name: Biopython
  role: dependency used by PhasiHunter for sequence handling in the phase module
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:cb274607422c5adff48fd6122466e15119b69135ebdd19cd6108910adc36c88a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bedtools
  - Biopython
  - Bowtie
  - Perl5
  - PhasiHunter
  - Seqkit
  - Trim_galore
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# phased-sirna-loci-prediction

## Summary

Predict phased siRNA (phasiRNA) clusters and PHAS loci from sRNA-seq data using PhasiHunter's dual-algorithm approach (hypergeometric test plus phase score) run against multiple reference sequences (genome, transcriptome, full-length transcriptome), then integrate the reference-based results with GFF3 annotation.

## When to use

Use when sRNA-seq reads (19–25 nt, e.g. rice SRR5049781) must be scanned for phased 21-nt siRNA stacks and PHAS loci, especially when genome- and transcriptome-based predictions should be combined to catch alternative-splicing or novel unannotated loci.

## When NOT to use

- The input is degradome data or miRNA target verification — use the separate target/initiator modules instead of the phase module.
- Only a single reference (genome alone) is available and no transcriptome/flnc integration or annotation-based locus naming is needed — the multi-reference value-add (alternative-splicing and novel-locus detection) is lost.
- The sRNA reads fall outside the 19–25 nt length window appropriate for 21-nt phasiRNAs (e.g. purely 24-nt phased siRNAs would require -pl 24 and this workflow's defaults would miss them).

## Inputs

- Raw sRNA-seq FASTQ/FASTQ.GZ (e.g. SRR5049781.fastq.gz)
- Reference genome FASTA (oryza_sativa_gdna.fa) and Bowtie index prefix
- Reference transcriptome FASTA (oryza_sativa_cdna.fa) and Bowtie index prefix
- Optional full-length transcriptome (flnc) FASTA and Bowtie index
- Genome GFF3 annotation (oryza_sativa_gdna.gff3)
- Trimmed, length/abundance-filtered sRNA FASTA (SRR5049781_trimmed_format_filter.fa)

## Outputs

- phase_a.txt — redundant allsiRNA cluster output (gene/PHAS_gene, strand, position, abundance, record, sequence, length, pvalue, phase_ratio, phase_number, phase_abundance, phase_score, marker)
- phase_o.txt — redundant phasiRNA cluster output with pvalue and phase_score fields
- integration_o.txt, integration_a.txt — integrated phasiRNA/PHAS cluster outputs
- integration_s.txt — integrated PHAS loci from simultaneous genome+transcriptome prediction
- integration_p.txt — integrated PHAS loci records passing the hypergeometric pvalue cutoff with only-P-method results discarded

## How to apply

First run `phasiHunter preprocess -m r` on the raw FASTQ (Trim_galore adapter trimming, normalization base 1,000,000, length filter 19–25 nt, CPM cutoff 1, Bowtie max hits 10) to build cDNA- and gDNA-based map files; optionally build a full-length transcriptome (flnc) index and map file via `-m m` for use with the phase module's -f/-fm options. Then run `phasiHunter phase` with both methods enabled (-me b): the hypergeometric test (pvalue cutoff -pv 0.001) and the phase score method (score cutoff -ps 15, ratio cutoff -pr 0.4), using phase length 21 (-pl 21), minimum phase number 4 (-pn 4), cluster island 5 (-il 5), and parallel cores (-j 20), producing phase_a.txt (redundant allsiRNA clusters) and phase_o.txt (redundant phasiRNA clusters). PhasiHunter implements forward and reverse search within the hypergeometric method to enhance PHAS loci detection sensitivity. Finally run `phasiHunter integration` on phase_o.txt/phase_a.txt against the genome GFF3 with -g y and -dp y (discard results supported only by the phase-score method) using the same pn/pl/pv/il settings to emit integration_o.txt, integration_a.txt, integration_s.txt, and integration_p.txt. Simultaneous genome+transcriptome prediction and integration recovers loci single-reference tools miss (99 additional specific PHAS genes on the test dataset) and supports flnc-based calls (122 unique PHAS genes, 96.7% novel unannotated).

## Related tools

- **PhasiHunter** (core tool: preprocess, phase (hypergeometric + phase score PHAS loci prediction), and integration submodules) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (maps filtered sRNA reads to cDNA/gDNA/flnc references with max hits 10 during preprocess and phase)
- **Trim_galore** (adapter trimming in the preprocess r-mode raw pipeline) — https://github.com/FelixKrueger/TrimGalore
- **Seqkit** (sequence manipulation supporting the length/abundance filtering step)
- **Biopython** (dependency used by PhasiHunter for sequence handling in the phase module)

## Examples

```
phasiHunter phase -cm SRR5049781_processed_cdna.map -c oryza_sativa_cdna.fa -gm SRR5049781_processed_gdna.map -g oryza_sativa_gdna.fa -fa SRR5049781_trimmed_format_filter.fa -a phase_a.txt -o phase_o.txt -me b -il 5 -pl 21 -pn 4 -pv 0.001 -ps 15 -pr 0.4 -j 20
```

## Evaluation signals

- Both phase_a.txt and phase_o.txt exist and contain the documented table header including pvalue and phase_score fields, with -pv 0.001 / -ps 15 / -pr 0.4 cutoffs applied to per-locus values
- integration_p.txt exists and contains PHAS loci records passing the hypergeometric pvalue cutoff 0.001 with only-P-method results discarded (-dp y)
- Integration with -g y confirms genome-based PHAS loci are present and flnc-based loci appear in the integrated phasiRNA cluster output
- On the test_osa rice dataset, flnc-based prediction yields 122 unique PHAS genes (96.7% novel) and combined genome+transcriptome prediction yields ~99 additional specific PHAS genes over single-reference tools (Supplementary Table S4)
- Temporary .phasiHuter_bowtieIndex is deleted after prediction when -cl y is set

## Limitations

- README documents only a changelog entry 1.0.1 (25/02/2024) with no explicit software version number; no discussion section was available so limitations/reproducibility caveats are not documented.
- Only-P-method (phase-score-only) results are discarded by default in integration (-dp y), so loci supported solely by phase score are excluded.
- Requires all dependencies (Bowtie, Biopython, Bedtools, Dnapi, Trim_galore, Seqkit, Perl5, Fasta36, TarHunter) on a Linux platform; a Docker image is provided for convenience.
- Multi-reference prediction (flnc-based) yields largely novel unannotated loci (96.7%), which may require additional downstream annotation effort.

## Evidence

- [abstract] dual algorithms: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction"
- [abstract] integration claim: "PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated"
- [readme] phase module parameters: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] integration command: "phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt -a"
- [methods] dp option: "-dp: str   -- y | n, discard only P method result, default=y"
- [abstract] alternative splicing claim: "PhasiHunter can efficiently identify the phasiRNAs generated through alternative splicing events"
