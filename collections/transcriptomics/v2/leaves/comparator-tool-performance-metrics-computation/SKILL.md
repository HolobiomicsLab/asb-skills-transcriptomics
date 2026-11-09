---
name: comparator-tool-performance-metrics-computation
description: Use when two or more transcript-3'-end or polyA-site prediction tools
  have been run on the same aligned RNA-Seq input and you need head-to-head accuracy
  metrics (e.g., fraction of predicted termini within 100 bases of a true PolyA-Seq
  site, PPV) to select a method.
when_to_use_negative:
- No gold-standard polyA site truth set (e.g., PolyA-Seq) is available to score predictions
  against — the match-to-true-site metrics cannot be computed.
- Comparators were run on different bam/genome/transcriptome inputs, making head-to-head
  accuracy comparisons invalid.
- The goal is APA differential usage between conditions rather than annotation-accuracy
  benchmarking.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: aptardi
  role: Primary tool under evaluation; produces the aptardi-modified transcriptome
    whose 3' termini are scored against PolyA-Seq sites
  repo: github.com/luskry/aptardi
- name: TAPAS
  role: Comparator polyA-site detection tool run on the same sorted bam, genome fasta,
    and transcriptome inputs
  repo: github.com/arefeen/TAPAS
- name: APARENT
  role: Comparator deep-neural-network APA predictor run on the same inputs
  repo: github.com/johli/aparent
- name: StringTie
  role: Reconstructs the baseline transcriptome whose 3'-end annotation serves as
    the unmodified comparison arm
- name: RSEM
  role: Quantitates the aptardi transcriptome so the 2/3-samples ≥1-count filter can
    build the high-quality transcriptome before benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:9a26db65aea7c4b0d8799d366ca9c8e79454db1e9fd2c19caec254a0789ec413
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - APARENT
  - RSEM
  - StringTie
  - TAPAS
  - aptardi
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# comparator-tool-performance-metrics-computation

## Summary

Benchmarks 3'-end annotation tools (aptardi, TAPAS, APARENT) and baseline transcriptomes against gold-standard PolyA-Seq sites on HBR RNA-Seq, computing correct 3'-termini counts, positive predictive value (PPV), and unique true-site counts. It quantifies how much a modified transcriptome improves polyA site annotation over competing methods.

## When to use

Use when two or more transcript-3'-end or polyA-site prediction tools have been run on the same aligned RNA-Seq input and you need head-to-head accuracy metrics (e.g., fraction of predicted termini within 100 bases of a true PolyA-Seq site, PPV) to select a method.

## When NOT to use

- No gold-standard polyA site truth set (e.g., PolyA-Seq) is available to score predictions against — the match-to-true-site metrics cannot be computed.
- Comparators were run on different bam/genome/transcriptome inputs, making head-to-head accuracy comparisons invalid.
- The goal is APA differential usage between conditions rather than annotation-accuracy benchmarking.

## Inputs

- aptardi-modified transcriptome GTF (e.g., hbr_aptardi.gtf)
- TAPAS 3'-end predictions on the same sorted.bam, hg38.fa, and transcriptome
- APARENT 3'-end predictions on the same inputs
- original StringTie/Ensembl transcriptome GTF
- gold-standard PolyA-Seq polyA site coordinates

## Outputs

- aptardi_tapas_aparent_comparison.csv with per-tool metrics
- per-tool counts of correct 3'-termini, PPV, fraction of added transcripts matching a true PolyA-Seq site, and unique true-site counts

## How to apply

Run each comparator (aptardi Mode 1 with the pre-built model.hdf5 and scale.pk at default 100-base bins and 0.5 probability threshold; TAPAS; APARENT) on the identical sorted bam, hg38.fa, and StringTie-reconstructed transcriptome. Extract predicted 3' termini from each output GTF plus the original transcriptome's termini. Match each predicted terminus to true PolyA-Seq sites using a 100-base tolerance window, consistent with aptardi's 100-base bin resolution. Compute: (1) fraction of added aptardi transcripts matching a true site (reported: 82%, 22,846/27,853 for aptardi vs 35%, 39,842/113,923 for StringTie), (2) count of transcripts with correct 3'-termini (62,688 aptardi vs 22,804 TAPAS), (3) PPV (0.44 aptardi vs 0.31 TAPAS), and (4) unique true-site counts captured (29,327/25,180/23,685). Write all metrics to a comparison CSV such as aptardi_tapas_aparent_comparison.csv.

## Related tools

- **aptardi** (Primary tool under evaluation; produces the aptardi-modified transcriptome whose 3' termini are scored against PolyA-Seq sites) — github.com/luskry/aptardi
- **TAPAS** (Comparator polyA-site detection tool run on the same sorted bam, genome fasta, and transcriptome inputs) — github.com/arefeen/TAPAS
- **APARENT** (Comparator deep-neural-network APA predictor run on the same inputs) — github.com/johli/aparent
- **StringTie** (Reconstructs the baseline transcriptome whose 3'-end annotation serves as the unmodified comparison arm)
- **RSEM** (Quantitates the aptardi transcriptome so the 2/3-samples ≥1-count filter can build the high-quality transcriptome before benchmarking)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --g hbr_aptardi.gtf --n model.hdf5 --t scale.pk --o output_dir
```

## Evaluation signals

- aptardi correct 3'-termini count should substantially exceed TAPAS (reference result: 62,688 vs 22,804 HBR transcripts within 100 bases of a true PolyA-Seq site)
- PPV for aptardi should be ~0.44 vs ~0.31 for TAPAS on HBR
- Fraction of added aptardi transcripts matching a true site should be ~82% (22,846/27,853) vs ~35% (39,842/113,923) for the original StringTie transcriptome
- Unique true polyA-site counts should be reported for all arms (29,327/25,180/23,685) and the comparison CSV should be non-empty with all tool rows populated
- All comparators must have consumed identical sorted.bam, hg38.fa, and transcriptome inputs — verify input file paths/hashes match across runs

## Limitations

- PolyA-Seq truth sites were sequenced at much greater depth, so some reported polyA sites were likely undetectable in the shallower RNA-Seq data, biasing counts downward.
- aptardi annotates at 100-base bin resolution and annotates the 3'-most base in a positive bin; polyA sites can wiggle up to ~30 nucleotides, so the 100-base match tolerance partly reflects this resolution limit.
- aptardi can attribute a stop site to the wrong transcript isoform and enumerates an aptardi transcript for every isoform probing the same region, so benchmarking should use the filtered (≥1 count in 2/3 of samples) high-quality transcriptome to avoid penalizing false-positive redundancy.
- Manually engineered DNA sequence features may not generalize outside mammals; the HBR-trained model performed modestly worse on BNLx and SHR (unstranded RNA-Seq, different species).

## Evidence

- [results] Incorporating aptardi correctly identified 3′ termini (within 100 bases of a true PolyA-Seq site) for 62,688 HBR transcripts compared with 22,804 transcripts using TAPAS: [evidence span withheld — anchor status: not_found]
- [discussion] PPV improvement over TAPAS: "Adding aptardi transcripts increased the number of unique true polyA sites captured by the transcriptome and furthermore increased the ratio of true positive to false positive termini"
- [readme] Comparator setup must use identical inputs: "aptardi takes as input a transcriptome (gtf/gff format), possibly constructed from RNA-Seq data, and combines RNA-Seq data from the sample with the genome (DNA sequence)"
- [readme] Filtering before benchmarking avoids redundant false-positive aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [discussion] Prediction resolution limitation affecting match tolerance: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
