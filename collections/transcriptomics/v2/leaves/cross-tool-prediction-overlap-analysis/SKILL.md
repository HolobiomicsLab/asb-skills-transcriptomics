---
name: cross-tool-prediction-overlap-analysis
description: Use when a task requires benchmarking phasiRNA prediction tools by comparing
  the number and overlap of predicted 21-PHAS genes from multiple tools on the same
  small RNA-seq dataset with a shared reference sequence.
when_to_use_negative:
- The goal is to benchmark tools on the genome reference rather than the transcriptome
  — this skill's workflow fixes the transcriptome as the reference sequence.
- The dataset is not a plant small RNA-seq library or the tools under comparison do
  not predict 21-PHAS loci (e.g., comparing 24-nt phasiRNA pipelines).
- Only one prediction tool is being run and no cross-tool overlap comparison is requested.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: PhasiHunter
  role: Preprocess (mode r), phase prediction (-me b), and integration of 21-PHAS
    gene predictions on the maize test dataset
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Mapping preprocessed sRNA reads to the transcriptome reference index during
    preprocess (max hits 10)
- name: unitas
  role: Independent 21-PHAS gene prediction tool run on the same maize test dataset
    for cross-tool comparison
- name: PhaseTank
  role: Independent 21-PHAS gene prediction tool run on the same maize test dataset
    for cross-tool comparison
- name: PhasiRNAnalyzer
  role: Independent 21-PHAS gene prediction tool run on the same maize test dataset
    for cross-tool comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:e1b2c434df5c8bc13c002f5dfe305e426d4d6930871d6572d499f5a729c2c4ce
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-tool-prediction-overlap-analysis

## Summary

Reproduce the benchmark in which PhasiHunter is run against unitas, PhaseTank, and PhasiRNAnalyzer on a maize small RNA-seq test dataset using the transcriptome as reference, counting 21-PHAS gene predictions per tool and computing cross-tool overlap fractions (e.g., 175/181 = 96.7% for PhasiHunter and 168/178 = 94.4% for PhasiRNAnalyzer).

## When to use

Use when a task requires benchmarking phasiRNA prediction tools by comparing the number and overlap of predicted 21-PHAS genes from multiple tools on the same small RNA-seq dataset with a shared reference sequence.

## When NOT to use

- The goal is to benchmark tools on the genome reference rather than the transcriptome — this skill's workflow fixes the transcriptome as the reference sequence.
- The dataset is not a plant small RNA-seq library or the tools under comparison do not predict 21-PHAS loci (e.g., comparing 24-nt phasiRNA pipelines).
- Only one prediction tool is being run and no cross-tool overlap comparison is requested.

## Inputs

- Maize small RNA-seq FASTQ file (test dataset)
- Maize reference transcriptome FASTA and Bowtie index
- PhasiHunter phase module outputs (phase_a.txt, phase_o.txt) and integration output
- 21-PHAS gene prediction lists from unitas, PhaseTank, and PhasiRNAnalyzer

## Outputs

- Per-tool 21-PHAS gene counts (PhasiHunter=181, PhasiRNAnalyzer=178)
- Cross-tool overlap fractions (175/181 = 96.7% for PhasiHunter; 168/178 = 94.4% for PhasiRNAnalyzer)
- Summary table mirroring Supplementary Fig. S4A

## How to apply

Preprocess the maize small RNA-seq test dataset with `phasiHunter preprocess` in raw mode (-m r), restricting reads to 19–25 nt, applying a CPM cutoff of 1 with normalization base 1000000, and mapping with a bowtie max-hits of 10 against the transcriptome reference index. Predict PHAS loci with `phasiHunter phase` using phase length 21, phase number 4, both hypergeometric and phase-score methods (-me b), p-value cutoff 0.001, phase score cutoff 15, and phase ratio 0.4, then consolidate 21-PHAS gene predictions via the integration module. Run unitas, PhaseTank, and PhasiRNAnalyzer independently on the same dataset with the transcriptome reference and extract their 21-PHAS gene sets. Compute per-tool gene counts and pairwise overlap fractions as the intersection size divided by each tool's prediction set size, then assemble a summary table mirroring Supplementary Fig. S4A.

## Related tools

- **PhasiHunter** (Preprocess (mode r), phase prediction (-me b), and integration of 21-PHAS gene predictions on the maize test dataset) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Mapping preprocessed sRNA reads to the transcriptome reference index during preprocess (max hits 10))
- **unitas** (Independent 21-PHAS gene prediction tool run on the same maize test dataset for cross-tool comparison)
- **PhaseTank** (Independent 21-PHAS gene prediction tool run on the same maize test dataset for cross-tool comparison)
- **PhasiRNAnalyzer** (Independent 21-PHAS gene prediction tool run on the same maize test dataset for cross-tool comparison)

## Examples

```
phasiHunter preprocess -m r -i maize_srna.fastq.gz -mi 19 -ma 25 -e 1 -n 1000000 -o maize_processed.map -in maize_cdna_index; phasiHunter phase -cm maize_processed.map -c maize_cdna.fa -fa maize_trimmed.fa -a phase_a.txt -o phase_o.txt -me b -pl 21 -pn 4 -pv 0.001 -ps 15 -pr 0.4
```

## Evaluation signals

- PhasiHunter predicts exactly 181 21-PHAS genes on the maize test dataset with the transcriptome reference
- PhasiRNAnalyzer predicts exactly 178 21-PHAS genes under the same conditions
- Overlap fraction for PhasiHunter equals 175/181 = 96.7% (predicted PHAS genes identifiable by one or more other tools)
- Overlap fraction for PhasiRNAnalyzer equals 168/178 = 94.4%
- The summary table matches the layout and values of Supplementary Fig. S4A

## Limitations

- Only the update log 1.0.1 (25/02/2024) is available; no discussion-section content was found, so no explicit limitations or reproducibility caveats are documented
- Overlap is computed as intersection over each tool's own prediction set, so overlap fractions are asymmetric and sensitive to total per-tool counts

## Evidence

- [results] overlap rates: [evidence span withheld — anchor status: not_found]
- [readme] phase parameters: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [methods] preprocess parameters: "-mi: int    --  minimal sRNA reads length cutoff, default=19"
- [abstract] two algorithms rationale: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods"
