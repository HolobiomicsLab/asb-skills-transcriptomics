---
name: phasirna-phas-loci-prediction
description: Use when sRNA-seq libraries (e.g. 19–25 nt reads) need phased siRNA/PHAS
  locus prediction against one or more reference sequences (transcriptome and/or genome),
  or when comparing reference-based phasiRNA predictions.
when_to_use_negative:
- Input is already a per-library Bowtie map file — use the phase module directly rather
  than re-preprocessing.
- The goal is microRNA discovery or miRNA target prediction with no interest in phasing/PHAS
  loci — this skill is specific to phased siRNA cluster mining.
- Reads fall outside the 19–25 nt sRNA window; the preprocess length cutoffs (-mi
  19, -ma 25) would discard the data.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: PhasiHunter
  role: CLI pipeline providing preprocess, phase, integration, visualization, target/initiator,
    and deg modules for PHAS loci and phasiRNA prediction
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Maps length- and CPM-filtered sRNA reads to transcriptome and genome reference
    indexes during preprocessing (max hits 10)
- name: Bedtools
  role: Interval overlap against gff3 annotation to classify reference-unique PHAS
    loci by exon/intron span
- name: Trim_galore
  role: Adapter trimming of raw sRNA-seq reads in raw (r) mode preprocessing
  repo: https://github.com/FelixKrueger/TrimGalore
- name: Seqkit
  role: Sequence file manipulation in preprocessing and formatting steps
- name: Dnapi
  role: Dependency used for dicer-position analysis within PhasiHunter
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:81391cb94e400b8e9a86034c35ac33345c17b4a7db084dc3247843e1baf2154a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bedtools
  - Bowtie
  - Dnapi
  - Perl5
  - PhaseTank
  - PhasiHunter
  - PhasiRNAnalyzer
  - Seqkit
  - Trim_galore
  - unitas
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# phasiRNA-phas-loci-prediction

## Summary

Predict phased siRNA (phasiRNA) clusters and PHAS loci from sRNA-seq data using PhasiHunter's dual prediction algorithms (hypergeometric test and phase score) across transcriptome and genome reference sequences, with downstream integration to consolidate results.

## When to use

Use when sRNA-seq libraries (e.g. 19–25 nt reads) need phased siRNA/PHAS locus prediction against one or more reference sequences (transcriptome and/or genome), or when comparing reference-based phasiRNA predictions.

## When NOT to use

- Input is already a per-library Bowtie map file — use the phase module directly rather than re-preprocessing.
- The goal is microRNA discovery or miRNA target prediction with no interest in phasing/PHAS loci — this skill is specific to phased siRNA cluster mining.
- Reads fall outside the 19–25 nt sRNA window; the preprocess length cutoffs (-mi 19, -ma 25) would discard the data.

## Inputs

- sRNA-seq FASTQ or FASTQ.gz files (raw, for -m r mode)
- Trimmed/length-and-abundance-filtered sRNA fasta (for -m m mode, e.g. SRR5049781_trimmed_format_filter.fa)
- Reference transcriptome fasta (e.g. oryza_sativa_cdna.fa) and prebuilt Bowtie index
- Reference genome fasta (e.g. oryza_sativa_gdna.fa) and prebuilt Bowtie index
- Genome gff3 annotation file (e.g. oryza_sativa_gdna.gff3) for the integration module

## Outputs

- Per-library Bowtie map files (*_processed_cdna.map and *_processed_gdna.map)
- phase_a.txt — allsiRNA cluster output file
- phase_o.txt — phasiRNA cluster output file
- integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt — consolidated PHAS loci and phasiRNA results
- PHAS.fa and phasiRNA.fa sequences; phasiRNA_cluster plots (visualization module)

## How to apply

First preprocess each library with the PhasiHunter preprocess module mapping reads via Bowtie to both a reference transcriptome and reference genome index (length filter 19–25 nt, CPM cutoff 1, normalization base 1000000, Bowtie max hits 10), producing *_processed_cdna.map and *_processed_gdna.map files. Then run the phase module on the map files with the sRNA fasta using both prediction methods (-me b), phase length 21, phase number cutoff 4, cluster island 5, p-value cutoff 0.001, phase score cutoff 15, and phase ratio cutoff 0.4, generating phase_a.txt (allsiRNA clusters) and phase_o.txt (phasiRNA clusters). Finally integrate the transcriptome-based and genome-based phase results with the integration module using the genome gff3 annotation (-pn 4, -pl 21, -pv 0.001, -il 5, -dp y to discard only-P-method results), producing integration_o/a/s/p.txt files. The dual reference strategy lets PhasiHunter identify phasiRNAs generated through alternative splicing events that a single reference would miss, and unique loci per reference can be classified by exon/intron structure with Bedtools interval overlap against the gff3.

## Related tools

- **PhasiHunter** (CLI pipeline providing preprocess, phase, integration, visualization, target/initiator, and deg modules for PHAS loci and phasiRNA prediction) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Maps length- and CPM-filtered sRNA reads to transcriptome and genome reference indexes during preprocessing (max hits 10))
- **Bedtools** (Interval overlap against gff3 annotation to classify reference-unique PHAS loci by exon/intron span)
- **Trim_galore** (Adapter trimming of raw sRNA-seq reads in raw (r) mode preprocessing) — https://github.com/FelixKrueger/TrimGalore
- **Seqkit** (Sequence file manipulation in preprocessing and formatting steps)
- **Dnapi** (Dependency used for dicer-position analysis within PhasiHunter)

## Examples

```
phasiHunter phase -cm SRR5049781_processed_cdna.map -c oryza_sativa_cdna.fa -gm SRR5049781_processed_gdna.map -g oryza_sativa_gdna.fa -fa SRR5049781_trimmed_format_filter.fa -a phase_a.txt -o phase_o.txt -me b -pl 21 -pn 4 -pv 0.001 -ps 15 -pr 0.4 -il 5
```

## Evaluation signals

- phase module emits both phase_o.txt (phasiRNA clusters) and phase_a.txt (allsiRNA clusters) for each reference; both files must exist and be non-empty.
- Integration produces all four files (integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt); -dp y should remove predictions supported only by the phase-score method.
- Reference-specific unique-loci counts must be reproducible by set difference: on the 32-library maize test dataset, transcriptome-based prediction yields 30 unique PHAS genes and genome-based prediction 95 unique PHAS genes.
- Exon/intron classification of genome-unique loci via Bedtools/gff3 must match reported statistics (69.5% spanning intron and exon, 1% intronic; transcriptome-unique loci 60% multi-exon / 40% mono-exon).
- Cross-tool benchmark check: PhasiHunter's 181 predicted 21-PHAS genes on the transcriptome reference, with 96.7% (175) identifiable by unitas/PhaseTank/PhasiRNAnalyzer.
- `phasihunter -h` returns help text confirming correct installation.

## Limitations

- Only PHAS loci supported by both methods (or discarded only-P results with -dp y) are retained; hypergeometric-only vs phase-score-only predictions are filtered during integration.
- Only 21-nt and 24-nt phase lengths are supported (-pl 21 | 24); other phasing sizes cannot be analyzed.
- Runs on Linux only and requires external dependencies (Bowtie, Biopython, Bedtools, Dnapi, Trim_galore, Seqkit, Perl5, Fasta36, TarHunter); the Docker image or conda env.yaml is recommended to satisfy these.
- Only-update-log 1.0.1 available; no detailed version/changelog or limitations discussion extractable from the article.

## Evidence

- [abstract] Dual algorithms with multi-reference support: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods"
- [abstract] Integration of multi-reference results: "PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated"
- [readme] phase module invocation with thresholds: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] preprocess module invocation: "phasiHunter preprocess -m r -i /home/user/test_osa/SRR5049781.fastq.gz -mi 19 -ma 25 -e 1 -n 1000000 -o /home/user/test_osa/SRR5049781_processed_cdna.map"
- [methods] integration discards only-P-method results: "-dp: str   -- y | n, discard only P method result, default=y"
- [methods] Linux CLI and dependencies: "phasihunter is a CLI program runing on linux platform"
