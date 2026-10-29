---
name: simulated-rnaseq-tool-benchmarking
description: Use when comparing IPA-detection tools on RNA-seq data with no matched
  3'-end-seq or Iso-seq ground truth, or when assessing how sequencing depth (10X–50X)
  affects identification accuracy and PAU quantification error.
when_to_use_negative:
- A matched 3'-end sequencing or Iso-seq ground-truth dataset is available for the
  same samples — direct benchmarking against 3P-seq/A-seq/polyA-seq/Iso-seq sites
  is preferable to simulation
- Only one tool is of interest and no competing tool is being compared — a full multi-tool
  benchmark adds simulation overhead without comparative value
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Polyester
  role: R package used to simulate RNA-seq data at 10X–50X coverage with five replicates
    per level from a transcript set with predefined polyA sites
- name: InPACT
  role: Tool under evaluation; identifies skipped and composite IPA sites from sorted
    BAM and RefSeq GTF and quantifies IPA usage via Salmon/InPACT_quantify
  repo: github.com/YY-TMU/InPACT
- name: APAIQ
  role: Compared method; identifies IPA sites from DNA sequence and RNA-seq read coverage
    on the simulated datasets
- name: IPAFinder
  role: Compared method; identifies IPA sites from read coverage fluctuations on the
    simulated datasets
- name: Salmon
  role: Indexes the merged transcriptome (gffread-extracted transcripts.fa) and quantifies
    transcript TPM used to derive relative IPA usage (PAU) for error computation
  repo: github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:1ad28cc2e373552d0e7d2cf8c298fe0530bc4ff26fc5c16daf0bef162976968c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - APAIQ
  - BEDTools closest
  - IPAFinder
  - InPACT
  - Polyester
  - Salmon
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# simulated-rnaseq-tool-benchmarking

## Summary

Benchmark intronic polyadenylation (IPA) identification and quantification tools (InPACT, APAIQ, IPAFinder) on RNA-seq data simulated with the R package Polyester at 10X–50X coverage, scoring sensitivity, precision, and IPA-usage error (|ΔPAU|).

## When to use

Use when comparing IPA-detection tools on RNA-seq data with no matched 3'-end-seq or Iso-seq ground truth, or when assessing how sequencing depth (10X–50X) affects identification accuracy and PAU quantification error.

## When NOT to use

- A matched 3'-end sequencing or Iso-seq ground-truth dataset is available for the same samples — direct benchmarking against 3P-seq/A-seq/polyA-seq/Iso-seq sites is preferable to simulation
- Only one tool is of interest and no competing tool is being compared — a full multi-tool benchmark adds simulation overhead without comparative value

## Inputs

- Transcript set with predefined polyA sites (ground truth) for Polyester simulation
- Simulated RNA-seq reads (FASTQ) at 10X–50X coverage, five replicates per coverage level
- Coordinate-sorted RNA-seq alignment BAM files
- Reference gene annotation GTF (e.g., RefSeq GRCh38, hg38.109.20211119.ncbiRefSeq.gtf)
- InPACT_polyAsites.hg38.saf candidate polyA site annotation

## Outputs

- Per-tool sensitivity and precision versus sequencing coverage (10X–50X) plots
- Cumulative distributions of |ΔPAU| IPA-usage error per tool
- Per-site classification of predictions as true/false positives (within/outside 50 nt of predefined polyA site)
- Summary table of sensitivity, precision, and usage-error metrics across replicates

## How to apply

Simulate RNA-seq datasets with the R package Polyester from a transcript set with predefined polyA sites, at coverage levels of 10X to 50X in 10X increments, generating five replicates per level. Run InPACT, APAIQ, and IPAFinder on every simulated dataset to predict IPA sites, then classify predictions within 50 nt of a predefined polyA site as true positives (BEDTools closest) and compute Sensitivity = TP/Predefined and Precision = TP/(TP+FP), averaged over the five replicates per coverage level. For quantification, estimate relative usage of each identified IPA site with each tool (InPACT via Salmon transcript-level TPM: salmon quant → InPACT_quantify) and take ground-truth relative usage of the predefined sites as reference, computing per-site error |ΔPAU|. Plot sensitivity and precision versus coverage per tool and the cumulative distributions of |ΔPAU| comparing tools; the benchmark succeeds when the tested tool attains the highest sensitivity/precision across the 10X–50X range and the lowest usage error.

## Related tools

- **Polyester** (R package used to simulate RNA-seq data at 10X–50X coverage with five replicates per level from a transcript set with predefined polyA sites)
- **InPACT** (Tool under evaluation; identifies skipped and composite IPA sites from sorted BAM and RefSeq GTF and quantifies IPA usage via Salmon/InPACT_quantify) — github.com/YY-TMU/InPACT
- **APAIQ** (Compared method; identifies IPA sites from DNA sequence and RNA-seq read coverage on the simulated datasets)
- **IPAFinder** (Compared method; identifies IPA sites from read coverage fluctuations on the simulated datasets)
- **Salmon** (Indexes the merged transcriptome (gffread-extracted transcripts.fa) and quantifies transcript TPM used to derive relative IPA usage (PAU) for error computation) — github.com/COMBINE-lab/salmon

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5
```

## Evaluation signals

- InPACT reports ~90% sensitivity for IPA site identification at 50X coverage; a correct reproduction should approach this value
- The evaluated tool should show the highest sensitivity and precision across all coverage levels (10X–50X) relative to APAIQ and IPAFinder
- The cumulative distribution of |ΔPAU| for InPACT should lie below (lower error than) those of IPAFinder and APAIQ
- Predictions classified as true positives must fall within 50 nt of a predefined polyA site (BEDTools closest match)
- Metrics are averaged over five replicates per coverage level; per-replicate variance should be reported or checked for stability

## Limitations

- No version number is given for InPACT or any compared tool (APAIQ, IPAFinder, Polyester), complicating exact reproduction
- No parameters or thresholds are reported for the simulations or comparative benchmarks beyond coverage levels and replicate count
- IPAFinder depends on read coverage fluctuations requiring adequate sequencing depth, so its simulated-data performance degrades at low coverage by design

## Evidence

- [methods] Simulation coverage design: "The RNA-seq data were simulated with varying sequencing coverage levels ranging from 10✕ to 50✕ with a 10✕ increment"
- [intro] Benchmark objective against simulators: "we show that InPACT outperforms IPAFinder in identifying and quantifying IPA with 3’-end sequencing data, long-read sequencing data and simulated RNA-seq data as benchmarks"
- [discussion] Expected quantification outcome: "InPACT exhibited improved accuracy in quantifying IPA events with a lower error rate compared to IPAFinder and APAIQ"
- [readme] InPACT IPA-site identification invocation: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
- [readme] Salmon quantification step: "salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant"
