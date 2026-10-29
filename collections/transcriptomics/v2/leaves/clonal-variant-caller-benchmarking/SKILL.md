---
name: clonal-variant-caller-benchmarking
description: Decision support for choosing an mtDNA variant caller for scRNA-seq clonality
  analysis, or for reproducing a simulation-based benchmark comparing MQuad, mgatk,
  and Monovar on ground-truth clonal variants.
when_to_use_negative:
- The goal is calling variants on one real dataset rather than comparing callers on
  data with known ground-truth clonal variants
- Input is nuclear-genome scDNA-seq data, where diploid-context callers such as Monovar
  or Conbase are appropriate (mitochondrial genome violates the diploid assumption)
- Data are scATAC-seq, for which mtDNA-specific methods like EMBLEM and mgatk were
  originally designed
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MQuad
  role: Clonal variant caller under test; scores variant informativeness with a binomial
    mixture model and a ΔBIC cutoff determined via the Kneedle algorithm
  repo: github.com/single-cell-genetics/MQuad
- name: mgatk
  role: Comparison caller (v0.6.2), swept over log10(VMR) with suggested cutoffs log10(VMR)
    ≥ −2 and strand correlation coefficient ≥0.65
- name: Monovar
  role: Nuclear SNV caller used as comparison baseline, run with recommended parameters
    and swept over its MPR statistic
- name: NEAT-genReads
  role: Simulates paired-end mtDNA reads (152 bp, hg19, 1000x coverage in high-coverage
    BED regions) for the benchmark
- name: SomatoSim
  role: Adds background noise and clonal mutations (v1.0.0) to simulated reads, varying
    variant number, allele frequency, clone ratio, and lineage tree
- name: cellsnp-lite
  role: Piles up simulated BAMs into SNP-by-cell matrices that feed MQuad; also builds
    the cell-by-SNP VAF matrix for background SNP selection
  repo: github.com/single-cell-genetics/cellsnp-lite
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:2236c781aa201174bf9f86177f8d2e8304d3117e30298c6cb3a5fdce23ce6383
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MQuad
  - Monovar
  - NEAT-genReads
  - SomatoSim
  - cellsnp-lite
  - mgatk
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# clonal-variant-caller-benchmarking

## Summary

Benchmark single-cell mtDNA clonal variant callers (MQuad vs mgatk vs Monovar) on simulated Smart-seq2-like mtDNA read data by sweeping each tool's calling threshold to trace PR/ROC curves and report AUPRC/AUROC per simulation setting.

## When to use

Decision support for choosing an mtDNA variant caller for scRNA-seq clonality analysis, or for reproducing a simulation-based benchmark comparing MQuad, mgatk, and Monovar on ground-truth clonal variants.

## When NOT to use

- The goal is calling variants on one real dataset rather than comparing callers on data with known ground-truth clonal variants
- Input is nuclear-genome scDNA-seq data, where diploid-context callers such as Monovar or Conbase are appropriate (mitochondrial genome violates the diploid assumption)
- Data are scATAC-seq, for which mtDNA-specific methods like EMBLEM and mgatk were originally designed

## Inputs

- BAM files of a real scRNA-seq dataset for background SNP harvesting (e.g., hematopoietic colony dataset, 536 cells)
- hg19 (UCSC) mtDNA reference for in silico read synthesis with NEAT-genReads
- Simulated BAM/allele-count data from SomatoSim with known ground-truth clonal variants
- cellsnp-lite pileup (SNP-by-cell AD/DP matrices) per simulated dataset
- Per-caller thresholds: MQuad ΔBIC cutoffs, mgatk log10(VMR) values, Monovar MPR values

## Outputs

- benchmark_auprc_auroc.csv with per-tool, per-setting AUPRC and AUROC values
- Precision-recall and ROC curve PNG figures for the default setting (M=10 variants per clone)
- Per-tool variant call sets at each cutoff for comparison against ground truth

## How to apply

Select background SNPs from real scRNA-seq pileups (here: cellsnp-lite on the 536-cell hematopoietic colony dataset, keeping SNPs with average depth >50 and heteroplasmy VAF >1% in >5% of cells, ~3500 SNPs), then synthesize paired-end mtDNA reads with NEAT-genReads v3.0 (152 bp reads, hg19, 1000x in high-coverage BED regions, 2% off-target) and add noise/clones with SomatoSim v1.0.0, varying M ∈ {5,10,25,50} variants per clone, log-normal allele frequencies Lognormal(μ, 0.005) with μ ∈ {0.01,0.05,0.1}, clone ratios {1:1:1, 2:4:4, 2:2:6, 1:1:8}, and lineage trees {linear, mixed, branched}. Run MQuad on the cellsnp-lite pileup of each simulation and sweep its ΔBIC cutoff (Kneedle knee point as default) to trace PR/ROC curves; sweep mgatk v0.6.2's log10(VMR) (suggested cutoffs log10(VMR) ≥ −2 and strand correlation ≥0.65, strand information unused) and Monovar's MPR statistic at recommended parameters. Compare every call against ground-truth clonal variants at each cutoff, compute precision-recall and ROC curves with AUPRC and AUROC per setting, and judge success by reproducing the reported ordering: at default settings MQuad AUPRC ≈ 0.976 / AUROC ≈ 1.00 should exceed mgatk (0.800/0.999) and Monovar (0.147/0.968).

## Related tools

- **MQuad** (Clonal variant caller under test; scores variant informativeness with a binomial mixture model and a ΔBIC cutoff determined via the Kneedle algorithm) — github.com/single-cell-genetics/MQuad
- **mgatk** (Comparison caller (v0.6.2), swept over log10(VMR) with suggested cutoffs log10(VMR) ≥ −2 and strand correlation coefficient ≥0.65)
- **Monovar** (Nuclear SNV caller used as comparison baseline, run with recommended parameters and swept over its MPR statistic)
- **NEAT-genReads** (Simulates paired-end mtDNA reads (152 bp, hg19, 1000x coverage in high-coverage BED regions) for the benchmark)
- **SomatoSim** (Adds background noise and clonal mutations (v1.0.0) to simulated reads, varying variant number, allele frequency, clone ratio, and lineage tree)
- **cellsnp-lite** (Piles up simulated BAMs into SNP-by-cell matrices that feed MQuad; also builds the cell-by-SNP VAF matrix for background SNP selection) — github.com/single-cell-genetics/cellsnp-lite

## Examples

```
cellsnp-lite -s simulated.bam -b barcodes.tsv -O sim_out -R mt_variants.vcf -p 20 --minCOUNT 20 && mquad --input sim_out --output mquad_out --adbicutoff 0
```

## Evaluation signals

- At default settings (M=10, μ=0.1, ratio 1:1:1, branched tree), MQuad reaches AUPRC ≈ 0.976 and AUROC ≈ 1.00, above mgatk (AUPRC 0.800, AUROC 0.999) and Monovar (AUPRC 0.147, AUROC 0.968)
- PR/ROC curves are traced by sweeping each caller's own threshold (ΔBIC, log10(VMR), MPR), not by a single fixed cutoff
- Every simulated dataset has recorded ground-truth clonal variants to enable exact precision/recall and sensitivity/specificity computation at each cutoff
- Background SNPs for simulation satisfy depth >50 and VAF >1% in >5% of cells (~3500 SNPs sampled into 90 simulated cells)
- benchmark_auprc_auroc.csv contains one AUPRC/AUROC row per tool per simulation setting

## Limitations

- Monovar and Conbase assume a diploid context, which is violated in the mitochondrial genome, weakening the comparison on the nuclear-caller side
- mgatk and EMBLEM were designed primarily for scATAC-seq; performance on conventional scRNA-seq is the benchmark's question, not an established strength
- Simulated reads use a specific configuration (152 bp reads, 1000x high-coverage, 2% off-target, hg19) and Smart-seq2-like assumptions; results may not transfer to other protocols or coverages

## Evidence

- [intro] Nuclear SNV callers such as Monovar and Conbase assume a diploid context which is violated in the mitochondrial genome.: "Nuclear SNV callers such as Monovar20 and Conbase21, assume a diploid context which is violated in the mitochondrial genome."
- [discussion] In silico synthesis of mitochondrial reads for the benchmark.: "Mitochondria reads were synthesized in silico with NEAT-genReads v3.038 and SomatoSim (v1.0.0)39"
- [discussion] Background SNP selection thresholds for the simulation.: "we selected variants that are 1) deeply covered with average depth >50 and 2) heteroplasmic (VAF > 1%) in more than 5% of the total cells"
- [discussion] MQuad's default threshold selection for tracing its PR/ROC curve.: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] mgatk comparison cutoffs used in the benchmark.: "we used the suggested cutoffs to call variants: log10(VMR) ≥ −2 and strand correlation coefficient ≥0.65"
- [readme] MQuad's core model as described in its README.: "It uses a binomial mixture model to assess the heteroplasmy of mtDNA variants among background noise."
