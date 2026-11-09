---
name: polya-site-benchmarking-against-polyaseq-gold-standard
description: Use when comparing predicted transcript 3'-end/polyA site calls from
  multiple tools (e.g., aptardi vs TAPAS vs APARENT) against a PolyA-Seq gold standard
  and quantitative accuracy metrics are needed.
when_to_use_negative:
- No gold-standard polyA site coordinates (e.g., PolyA-Seq) are available for the
  sample or species being analyzed, so accuracy cannot be scored
- The question is differential APA or 3' UTR usage between conditions rather than
  benchmarking 3'-end prediction accuracy
- Input transcripts fall outside the 3' terminal exon scope aptardi analyzes (upstream
  exons are not analyzed, so coding APA cannot be evaluated)
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: aptardi
  role: Primary 3'-end predictor under benchmark; run in Mode 1 with the pre-built
    HBR model (model.hdf5, scale.pk, 100-base bins, 0.5 probability threshold)
  repo: github.com/luskry/aptardi
- name: TAPAS
  role: Comparator polyA-site prediction tool run on the same sorted bam, genome fasta,
    and transcriptome inputs
  repo: https://github.com/arefeen/TAPAS
- name: APARENT
  role: Comparator deep-learning 3' UTR polyadenylation predictor run on the same
    inputs
  repo: https://github.com/johli/aparent
- name: StringTie
  role: Reconstructs the original HBR transcriptome (stringtie sorted.bam --rf -o
    stringtie.gtf -G guide_file) used as the baseline comparator and aptardi input
- name: RSEM
  role: Quantitates the aptardi transcriptome so transcripts without at least 1 count
    in 2/3 of samples can be filtered before benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:f258a0ced0d04c5195e500a6ed1a11e19b3286cbbf222f2e6849df4293bdaf98
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - APARENT
  - RSEM
  - StringTie
  - TAPAS
  - aptardi
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# polya-site-benchmarking-against-polyaseq-gold-standard

## Summary

Benchmark predicted transcript 3'-ends (from aptardi, TAPAS, APARENT, or an original StringTie/Ensembl transcriptome) against true PolyA-Seq sites on the HBR data set, counting correct termini (within 100 bases), PPV, and unique true-site counts.

## When to use

Use when comparing predicted transcript 3'-end/polyA site calls from multiple tools (e.g., aptardi vs TAPAS vs APARENT) against a PolyA-Seq gold standard and quantitative accuracy metrics are needed.

## When NOT to use

- No gold-standard polyA site coordinates (e.g., PolyA-Seq) are available for the sample or species being analyzed, so accuracy cannot be scored
- The question is differential APA or 3' UTR usage between conditions rather than benchmarking 3'-end prediction accuracy
- Input transcripts fall outside the 3' terminal exon scope aptardi analyzes (upstream exons are not analyzed, so coding APA cannot be evaluated)

## Inputs

- Predicted transcript 3'-end GTF files from aptardi (hbr_aptardi.gtf), TAPAS, and APARENT runs on the same sorted.bam, hg38.fa, and stringtie.gtf inputs
- Original StringTie/Ensembl transcriptome (stringtie.gtf)
- True PolyA-Seq polyA site coordinates (gold standard labels)
- RSEM quantitation counts on the aptardi transcriptome

## Outputs

- Comparison metrics table (aptardi_tapas_aparent_comparison.csv) with correct 3'-termini counts, PPV, unique true-site counts, and fraction of added transcripts matching true PolyA-Seq sites
- High-quality filtered transcriptome (transcripts with at least 1 count in 2/3 of samples)

## How to apply

Predicted 3'-termini from each method (aptardi Mode 1 with the pre-built HBR model, TAPAS, APARENT, and the original StringTie transcriptome) are compared against true PolyA-Seq sites, scoring a prediction as correct if it falls within 100 bases of a true site (aptardi's 100-base bin resolution). Quantitate the aptardi transcriptome with RSEM, remove transcripts without at least 1 count in 2/3 of samples to build the high-quality transcriptome, and compute metrics: fraction of added aptardi transcripts matching a true site (22,846/27,853 = 82% vs 39,842/113,923 = 35% for original), correct 3'-termini counts (62,688 aptardi vs 22,804 TAPAS), positive predictive value (0.44 vs 0.31), and unique true-site counts (29,327/25,180/23,685). Write all metrics to a comparison table such as aptardi_tapas_aparent_comparison.csv.

## Related tools

- **aptardi** (Primary 3'-end predictor under benchmark; run in Mode 1 with the pre-built HBR model (model.hdf5, scale.pk, 100-base bins, 0.5 probability threshold)) — github.com/luskry/aptardi
- **TAPAS** (Comparator polyA-site prediction tool run on the same sorted bam, genome fasta, and transcriptome inputs) — https://github.com/arefeen/TAPAS
- **APARENT** (Comparator deep-learning 3' UTR polyadenylation predictor run on the same inputs) — https://github.com/johli/aparent
- **StringTie** (Reconstructs the original HBR transcriptome (stringtie sorted.bam --rf -o stringtie.gtf -G guide_file) used as the baseline comparator and aptardi input)
- **RSEM** (Quantitates the aptardi transcriptome so transcripts without at least 1 count in 2/3 of samples can be filtered before benchmarking)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --g hbr_aptardi.gtf --n model.hdf5 --t scale.pk --o output_dir
```

## Evaluation signals

- Correct 3'-termini count (within 100 bases of a true PolyA-Seq site) should exceed the TAPAS comparator, e.g., 62,688 aptardi vs 22,804 TAPAS on HBR
- Positive predictive value should improve over the original transcriptome, e.g., 0.44 for aptardi vs 0.31 for TAPAS
- Fraction of added aptardi transcripts matching a true PolyA-Seq site should be ~82% (22,846/27,853) vs ~35% for the original transcriptome (39,842/113,923)
- Unique true polyA sites captured should increase relative to the original transcriptome (e.g., 29,327/25,180/23,685 across methods)
- All metrics must be present in the output comparison table (aptardi_tapas_aparent_comparison.csv)

## Limitations

- aptardi makes predictions on 100 base bins, a resolution limitation, though polyA sites wiggle up to 30 nucleotides and few bins contained multiple sites
- aptardi adds polyA sites for a transcript even if another isoform of the same gene already has a stop site at the location, possibly attributing a stop site to the wrong transcript; filtering (RSEM quantitation plus removing transcripts without at least 1 count in 2/3 of samples) is highly recommended to remove redundant false positives
- Some PolyA-Seq sites sequenced at much greater depth were likely not detectable at lower RNA-Seq depths; aptardi performed modestly worse on the BNLx and SHR data sets

## Evidence

- [results] Incorporating aptardi into the pipeline correctly identified 3′ termini (within 100 bases of a true PolyA-Seq site) for 62,688 HBR transcripts compared with 22,804 transcripts using TAPAS, and raised positive predictive value from 0.31 (TAPAS) to 0.44.: [evidence span withheld — anchor status: not_found]
- [discussion] aptardi-modified transcriptome increased unique true polyA sites and true-to-false-positive termini ratio versus original transcriptome: "Adding aptardi transcripts increased the number of unique true polyA sites captured by the transcriptome and furthermore increased the ratio of true positive to false positive termini"
- [readme] Filter pipeline from README: quantitate with RSEM then remove low-count transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [discussion] 100-base bin resolution limitation: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [discussion] PolyA-Seq depth caveat: "many polyA sites reported in the PolyA-Seq data—which was sequenced at much greater depths—were likely not detectable here"
