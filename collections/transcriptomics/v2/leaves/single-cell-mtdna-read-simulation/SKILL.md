---
name: single-cell-mtdna-read-simulation
description: Use when benchmarking mtDNA variant-calling tools against known ground-truth
  clonal variants in simulated Smart-seq2-like scRNA-seq data, or when generating
  simulated single-cell mtDNA BAMs with controlled clone counts, allele frequencies,
  and lineage trees.
when_to_use_negative:
- The goal is calling clonal variants on real single-cell data (e.g., Kim or fibroblast
  Smart-seq2 datasets) rather than benchmarking callers against a synthetic truth
  set
- The assay is scATAC-seq with NUMT-masked references, where mtDNA-specific callers
  like mgatk were primarily designed and simulation assumptions may not transfer
- Input data has already been genotyped into a SNP-by-cell AD/DP matrix and the task
  is clonal reconstruction (use vireoSNP downstream instead)
tools:
- name: NEAT-genReads
  role: Synthesizes paired-end mtDNA reads in silico on the hg19 reference at 1000x
    coverage
- name: SomatoSim
  role: Adds sequencing background noise and clonal mutations with configurable M,
    allele frequency, clone ratio, and lineage-tree settings
- name: cellsnp-lite
  role: Piles up raw reads into a SNP-by-cell VAF/AD/DP matrix used both for empirical
    VAF sampling and for caller input on simulated data
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Evaluated caller using a binomial mixture model with a ΔBIC cutoff (Kneedle
    knee point) to select informative variants
  repo: github.com/single-cell-genetics/MQuad
- name: mgatk
  role: Comparison caller whose PR/ROC curve is traced by varying the log10(VMR) cutoff
    (strand information unused)
- name: Monovar
  role: Nuclear SNV comparison caller whose curve is traced by varying its MPR statistic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:fce5ff2384ec0590c3e16ebcc0a7e95e2d85d01ae1e4517a48072b40b790051d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - MQuad
  - Monovar
  - NEAT-genReads
  - SomatoSim
  - cellsnp-lite
  - mgatk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# single-cell-mtdna-read-simulation

## Summary

In-silico simulation of single-cell mtDNA sequencing reads (NEAT-genReads + SomatoSim on hg19) to build ground-truth benchmark datasets for evaluating clonal mtDNA variant callers such as MQuad, mgatk, and Monovar by AUPRC/AUROC.

## When to use

Use when benchmarking mtDNA variant-calling tools against known ground-truth clonal variants in simulated Smart-seq2-like scRNA-seq data, or when generating simulated single-cell mtDNA BAMs with controlled clone counts, allele frequencies, and lineage trees.

## When NOT to use

- The goal is calling clonal variants on real single-cell data (e.g., Kim or fibroblast Smart-seq2 datasets) rather than benchmarking callers against a synthetic truth set
- The assay is scATAC-seq with NUMT-masked references, where mtDNA-specific callers like mgatk were primarily designed and simulation assumptions may not transfer
- Input data has already been genotyped into a SNP-by-cell AD/DP matrix and the task is clonal reconstruction (use vireoSNP downstream instead)

## Inputs

- Hematopoietic colony scRNA-seq dataset (Lareau et al., 536 cells) BAM files for empirical VAF sampling
- hg19 reference genome (UCSC)
- High-coverage mtDNA BED regions for coverage targeting
- Ground-truth list of injected clonal variants per simulation setting
- Simulated paired-end mtDNA BAM files (read length 152 bp, 1000x coverage)

## Outputs

- benchmark_auprc_auroc.csv with per-tool per-setting AUPRC/AUROC values
- Default-setting (M=10) PR and ROC curve PNG figures
- Simulated mtDNA read BAMs with known clonal variant truth sets

## How to apply

First build an empirical VAF matrix by running cellsnp-lite on the hematopoietic colony scRNA-seq dataset (536 cells) and select background SNPs with average depth >50 and heteroplasmy (VAF >1%) in more than 5% of cells (~3500 SNPs), randomly sampling VAFs into 90 simulated cells. Synthesize paired-end mtDNA reads with NEAT-genReads v3.0 on the hg19 reference (read length 152 bp, empirical fragment length distribution, 1000x coverage in high-coverage BED regions with 2% off-target coverage), then inject noise and clonal mutations with SomatoSim v1.0.0, sweeping M ∈ {5,10,25,50} variants per clone, allele frequencies Lognormal(μ, 0.005) with μ ∈ {0.01,0.05,0.1}, clone ratios {1:1:1, 2:4:4, 2:2:6, 1:1:8}, and lineage trees {linear, mixed, branched} (defaults M=10, μ=0.1, r=1:1:1, t=branched). Run each caller on the cellsnp-lite pileup (MQuad ΔBIC cutoff swept with the Kneedle knee point as default; mgatk v0.6.2 log10(VMR) cutoffs log10(VMR) ≥ −2 and strand correlation ≥0.65; Monovar varying its MPR statistic), then compute precision-recall and ROC curves against the ground-truth clonal variants for every setting.

## Related tools

- **NEAT-genReads** (Synthesizes paired-end mtDNA reads in silico on the hg19 reference at 1000x coverage)
- **SomatoSim** (Adds sequencing background noise and clonal mutations with configurable M, allele frequency, clone ratio, and lineage-tree settings)
- **cellsnp-lite** (Piles up raw reads into a SNP-by-cell VAF/AD/DP matrix used both for empirical VAF sampling and for caller input on simulated data) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Evaluated caller using a binomial mixture model with a ΔBIC cutoff (Kneedle knee point) to select informative variants) — github.com/single-cell-genetics/MQuad
- **mgatk** (Comparison caller whose PR/ROC curve is traced by varying the log10(VMR) cutoff (strand information unused))
- **Monovar** (Nuclear SNV comparison caller whose curve is traced by varying its MPR statistic)

## Evaluation signals

- At default settings (M=10 variants per clone), MQuad should reach AUPRC ≈ 0.976 and AUROC ≈ 1.00, exceeding mgatk (AUPRC 0.800, AUROC 0.999) and Monovar (AUPRC 0.147, AUROC 0.968)
- Background SNP panel should contain ~3500 SNPs meeting average depth >50 and VAF >1% in >5% of cells criteria
- Precision-recall and ROC curves are traced by sweeping each tool's cutoff (ΔBIC for MQuad, log10(VMR) for mgatk, MPR for Monovar), not a single operating point
- Per-setting AUPRC/AUROC rows for all combinations of M ∈ {5,10,25,50}, μ ∈ {0.01,0.05,0.1}, clone ratios, and lineage trees are present in benchmark_auprc_auroc.csv

## Limitations

- Simulation mimics Smart-seq2-like scRNA-seq; performance on other assays (e.g., 10x scRNA-seq, mtscATAC-seq) is not captured by this truth set
- Nuclear SNV callers like Monovar assume a diploid context violated in the mitochondrial genome, depressing their benchmarked performance
- Very few computational methods exist for analyzing mitochondrial SNVs across different sequencing assays, especially conventional scRNA-seq data, so the comparison tool set is small

## Evidence

- [discussion] Mitochondria reads were synthesized in silico with NEAT-genReads v3.038 and SomatoSim (v1.0.0)39: "Mitochondria reads were synthesized in silico with NEAT-genReads v3.038 and SomatoSim (v1.0.0)39"
- [discussion] Simulation SNP selection criteria: "we selected variants that are 1) deeply covered with average depth >50 and 2) heteroplasmic (VAF > 1%) in more than 5% of the total cells"
- [discussion] ΔBIC cutoff via Kneedle: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] mgatk suggested cutoffs: "we used the suggested cutoffs to call variants: log10(VMR) ≥ −2 and strand correlation coefficient ≥0.65"
- [discussion] cellsnp-lite pileup role: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23"
- [abstract] MQuad outperforms existing methods: "MQuad can identify mitochondrial variants with both high sensitivity and specificity, outperforming existing methods by a large extent"
- [readme] cellsnp-lite output usable for MQuad: [evidence span withheld — anchor status: not_found]
