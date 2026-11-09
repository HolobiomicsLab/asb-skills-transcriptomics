---
name: small-rna-read-preprocessing-and-mapping
description: Use when raw small RNA-seq FASTQ/FASTQ files (e.g. SRR5049781) must be
  adapter-trimmed, normalized, length-filtered, and mapped to a reference before phasiRNA
  phase prediction with PhasiHunter.
when_to_use_negative:
- Input is already a processed .map file or phase-module output — rerunning preprocessing
  would double-filter reads
- Reads fall outside the 19-25 nt window after trimming, or the goal is long-RNA/mRNA
  alignment rather than sRNA mapping with up to 10 hits
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: PhasiHunter
  role: preprocess module orchestrating trim, normalize, filter, and mapping steps
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Trim_galore
  role: adapter trimming of raw sRNA reads in mode r
  repo: https://github.com/FelixKrueger/TrimGalore
- name: Bowtie
  role: maps filtered sRNA reads to the reference index with max hits (-mh 10)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:616dfc914c628c290e693b8b1f59aaaeeef2e4f932d346ddbcc25ac289fff468
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Biopython
  - Bowtie
  - Perl5
  - PhasiHunter
  - Trim_galore
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# small-rna-read-preprocessing-and-mapping

## Summary

Preprocess raw small RNA-seq reads with PhasiHunter's preprocess module (adapter trimming via Trim_galore, normalization, length/abundance filtering, Bowtie mapping) to produce the .map files required for phasiRNA/PHAS loci prediction against cdna, gdna, or flnc references.

## When to use

Use when raw small RNA-seq FASTQ/FASTQ files (e.g. SRR5049781) must be adapter-trimmed, normalized, length-filtered, and mapped to a reference before phasiRNA phase prediction with PhasiHunter.

## When NOT to use

- Input is already a processed .map file or phase-module output — rerunning preprocessing would double-filter reads
- Reads fall outside the 19-25 nt window after trimming, or the goal is long-RNA/mRNA alignment rather than sRNA mapping with up to 10 hits

## Inputs

- raw sRNA-seq FASTQ or FASTQ.gz file (e.g. SRR5049781.fastq.gz) for mode r
- cleaned FASTA/FASTA.gz file for mode c
- length- and abundance-filtered FASTA file for mode m
- Bowtie index prefix for the reference (e.g. oryza_sativa_cdna_index) or reference FASTA via -r

## Outputs

- processed sRNA mapping file (.map) such as SRR5049781_processed_cdna.map or SRR5049781_processed_gdna.map
- trimmed/filtered sRNA FASTA (e.g. SRR5049781_trimmed_format_filter.fa) usable as phase module -fa input

## How to apply

Run `phasiHunter preprocess -m r` on the raw FASTQ(.gz) to execute the pipeline trim adaptor -> normalization -> length and abundance filter -> mapping: Trim_galore removes adapters, reads are normalized to a base of 1,000,000 (-n 1000000), filtered to 19-25 nt (-mi 19 -ma 25) at a CPM cutoff of 1 (-e 1), and mapped with Bowtie allowing up to 10 hits (-mh 10). Use -j (adaptor trim parallel cores, 8 recommended) and -bj (bowtie parallel cores) for speed. Supply a Bowtie index prefix via -in (e.g. oryza_sativa_cdna_index); if only clean/mapped FASTA exists, run -m c or -m m instead. The output is a processed .map file (e.g. SRR5049781_processed_cdna.map) that feeds the phase module's -cm/-gm/-fm options; for a full-length transcriptome (flnc) reference, generate a separate flnc-based map via -m m and supply it with -fm and -f in the phase module.

## Related tools

- **PhasiHunter** (preprocess module orchestrating trim, normalize, filter, and mapping steps) — https://github.com/HuangLab-CBI/PhasiHunter
- **Trim_galore** (adapter trimming of raw sRNA reads in mode r) — https://github.com/FelixKrueger/TrimGalore
- **Bowtie** (maps filtered sRNA reads to the reference index with max hits (-mh 10))

## Examples

```
phasiHunter preprocess -m r -i /home/user/test_osa/SRR5049781.fastq.gz -mi 19 -ma 25 -e 1 -n 1000000 -o /home/user/test_osa/SRR5049781_processed_cdna.map -in /home/user/test_osa/index/oryza_sativa_cdna_index
```

## Evaluation signals

- Output .map file exists at the path given to -o and is non-empty (e.g. SRR5049781_processed_cdna.map)
- Mapped reads respect the length window 19-25 nt and CPM cutoff of 1 after normalization to 1,000,000
- No read maps to more than 10 reference loci (Bowtie -mh 10)
- The map file is accepted downstream: the phase module runs without error when the file is passed via -cm/-gm/-fm

## Limitations

- PhasiHunter is a CLI program running on the Linux platform only; multiple PhasiHunter versions/changelogs are undocumented (only update log 1.0.1 - 25/02/2024 is noted)
- Reference choice (cdna, gdna, or flnc) changes the result set — a separate -m m run and -fm/-f supply are required for full-length-transcriptome-based prediction
- Correct execution depends on external dependencies (Bowtie, Biopython, Bedtools, Dnapi, Trim_galore, Seqkit, Perl5, Fasta36, TarHunter) being installed

## Evidence

- [methods] raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping: "raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping"
- [readme] -mi: int -- minimal sRNA reads length cutoff, default=19; -ma: int -- maxmial sRNA reads length cutoff, default=25; -e: float -- sRNA reads cpm cutoff, default=1: "-mi: int    --  minimal sRNA reads length cutoff, default=19
    -ma: int    --  maxmial sRNA reads length cutoff, default=25
    -e: float  --  sRNA reads cpm cutoff, default=1 "
- [readme] -mh: int -- max hits when mapping to ref sequence, default=10: "-mh: int    --  max hits when mapping to ref sequence, default=10"
- [results] prediction based on the full-length transcriptome yields 122 unique PHAS genes, of which almost all (96.7%) are novel unannotated genes: [evidence span withheld — anchor status: not_found]
