---
name: benchmark-performance-visualization
description: Use when comparing IPA-detection tools (InPACT, APAIQ, IPAFinder) on
  simulated RNA-seq data with known ground-truth polyA sites, and you need sensitivity/precision-vs-coverage
  figures and IPA-usage error plots.
when_to_use_negative:
- Input is real RNA-seq data without matched ground-truth polyA sites — sensitivity/precision
  against 'Predefined' sites cannot be computed
- Goal is 3'UTR-APA quantification only — the compared tools and this benchmark target
  intronic polyadenylation, and existing APA tools (DaPars, APAtrap, TAPAS) are limited
  to 3'UTR APA
- Only one tool's predictions are available — sensitivity, precision, and cumulative
  |ΔPAU| comparisons require matched predictions from multiple methods
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Polyester
  role: R package used to simulate RNA-seq datasets at 10X–50X coverage with predefined
    polyA sites as ground truth
- name: InPACT
  role: 'Benchmarked method: identifies IPA sites (skipped/composite) and quantifies
    IPA usage via Salmon transcript abundances'
  repo: https://github.com/YY-TMU/InPACT
- name: APAIQ
  role: 'Comparison method: identifies IPA sites using DNA sequence and RNA-seq read
    coverage; cannot assemble IPA isoforms'
- name: IPAFinder
  role: 'Comparison method: identifies IPA sites from read coverage fluctuations;
    benchmarked head-to-head against InPACT for |ΔPAU| error'
- name: Salmon
  role: Indexes and quantifies reconstructed IPA transcripts (salmon index/quant)
    whose abundances feed PAU/usage error computation
  repo: https://github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:93b2a75427954b71bf75985c9edd2e2aa3f6e62b39c67ff159973ab3311c2704
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - APAIQ
  - IPAFinder
  - InPACT
  - Polyester
  - Salmon
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# benchmark-performance-visualization

## Summary

Benchmarking and visual comparison of intronic polyadenylation (IPA) tools by simulating RNA-seq with Polyester at 10X–50X coverage, scoring predicted sites against predefined polyA sites, and plotting sensitivity/precision curves plus cumulative |ΔPAU| quantification-error distributions.

## When to use

Use when comparing IPA-detection tools (InPACT, APAIQ, IPAFinder) on simulated RNA-seq data with known ground-truth polyA sites, and you need sensitivity/precision-vs-coverage figures and IPA-usage error plots.

## When NOT to use

- Input is real RNA-seq data without matched ground-truth polyA sites — sensitivity/precision against 'Predefined' sites cannot be computed
- Goal is 3'UTR-APA quantification only — the compared tools and this benchmark target intronic polyadenylation, and existing APA tools (DaPars, APAtrap, TAPAS) are limited to 3'UTR APA
- Only one tool's predictions are available — sensitivity, precision, and cumulative |ΔPAU| comparisons require matched predictions from multiple methods

## Inputs

- Simulated RNA-seq read sets (FASTQ/BAM) from Polyester at 10X–50X coverage, five replicates per level
- Transcript set with predefined (ground-truth) polyA sites used for simulation
- Coordinate-sorted BAM alignments and RefSeq GTF annotation (e.g., hg38.109.20211119.ncbiRefSeq.gtf) for tool runs
- IPA prediction outputs from InPACT (predict.result.txt), APAIQ, and IPAFinder
- Salmon quant.sf / InPACT_quantify ipa_usage.txt with PAU estimates for identified IPA isoforms

## Outputs

- Sensitivity-vs-sequencing-coverage curve (10X–50X) for InPACT, APAIQ, and IPAFinder
- Precision-vs-sequencing-coverage curve for the three tools
- Cumulative distribution plot of per-site |ΔPAU| quantification error across identified IPA sites
- Per-replicate TP/FP classification tables (predictions within/outside 50 nt of predefined sites)

## How to apply

Simulate RNA-seq datasets with the R package Polyester at coverage levels from 10X to 50X in 10X increments, generating five replicates per coverage level from a transcript set with predefined polyA sites. Run InPACT, APAIQ, and IPAFinder on each simulated dataset, then classify each predicted site as a true positive if it lies within 50 nt of a predefined polyA site and a false positive otherwise, computing Sensitivity = TP / Predefined and Precision = TP / (TP + FP) per coverage level, averaged across the five replicates. In parallel, quantify relative usage of each identified IPA site (e.g., with gffread + Salmon on merged IPA transcript GTFs, then InPACT_quantify to obtain PAU estimates) and compute per-site error as the absolute difference from ground-truth relative usage (|ΔPAU|). Plot sensitivity and precision versus sequencing coverage for all three tools, and plot cumulative distributions of |ΔPAU| across identified IPA sites (paper compares InPACT vs IPAFinder). Treat the benchmark as successful when InPACT shows the highest sensitivity/precision across 10X–50X and the lowest IPA-usage error, matching the reported ~90% sensitivity at 50X.

## Related tools

- **Polyester** (R package used to simulate RNA-seq datasets at 10X–50X coverage with predefined polyA sites as ground truth)
- **InPACT** (Benchmarked method: identifies IPA sites (skipped/composite) and quantifies IPA usage via Salmon transcript abundances) — https://github.com/YY-TMU/InPACT
- **APAIQ** (Comparison method: identifies IPA sites using DNA sequence and RNA-seq read coverage; cannot assemble IPA isoforms)
- **IPAFinder** (Comparison method: identifies IPA sites from read coverage fluctuations; benchmarked head-to-head against InPACT for |ΔPAU| error)
- **Salmon** (Indexes and quantifies reconstructed IPA transcripts (salmon index/quant) whose abundances feed PAU/usage error computation) — https://github.com/COMBINE-lab/salmon

## Examples

```
salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant
```

## Evaluation signals

- InPACT sensitivity at 50X coverage should reach approximately 90% of predefined polyA sites, as reported
- Predicted sites count as true positives only when within 50 nt of a predefined polyA site; Sensitivity = TP/Predefined and Precision = TP/(TP+FP) per coverage level
- Sensitivity and precision curves should be averaged over the five replicates per coverage level and improve (or remain high) from 10X to 50X
- Cumulative |ΔPAU| distribution for InPACT should lie left of (i.e., dominate) IPAFinder's, indicating lower IPA-usage error
- Identified IPA isoforms must carry in-frame stop codons only if translation filtering is applied downstream; PAU estimates come from Salmon quant.sf through InPACT_quantify

## Limitations

- No version numbers given for InPACT or any compared tool, and no parameters or thresholds reported for the simulations or comparative benchmarks
- IPAFinder relies on read coverage fluctuations, which necessitates adequate sequencing depth, and its precision of identified IPA sites raises concerns
- APAIQ is not specifically designed for IPA analysis and cannot assemble IPA isoforms or differentiate skipped from composite IPA events

## Evidence

- [methods] The RNA-seq data were simulated with varying sequencing coverage levels ranging from 10✕ to 50✕ with a 10✕ increment: "The RNA-seq data were simulated with varying sequencing coverage levels ranging from 10✕ to 50✕ with a 10✕ increment"
- [intro] we show that InPACT outperforms IPAFinder in identifying and quantifying IPA with 3’-end sequencing data, long-read sequencing data and simulated RNA-seq data as benchmarks: "we show that InPACT outperforms IPAFinder in identifying and quantifying IPA with 3’-end sequencing data, long-read sequencing data and simulated RNA-seq data as benchmarks"
- [discussion] comprehensive evaluation using simulated RNA-seq data with varying coverage depths demonstrated InPACT’s superior performance: "comprehensive evaluation using simulated RNA-seq data with varying coverage depths demonstrated InPACT’s superior performance"
- [readme] salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant: "salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant"
- [abstract] We have demonstrated that InPACT outperforms other methods in terms of IPA identification and quantification: "We have demonstrated that InPACT outperforms other methods in terms of IPA identification and quantification"
