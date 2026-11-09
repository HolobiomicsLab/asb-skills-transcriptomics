---
name: pseudobulk-read-coverage-window-binning
description: Use when comparing synthetic scRNA-seq/scATAC-seq reads against real
  reads at the read-sequence level, or when benchmarking a read simulator's ability
  to mimic pseudo-bulk coverage patterns across genomic windows.
when_to_use_negative:
- Evaluating count-level fidelity of a simulator that outputs only a gene-by-cell
  count matrix and no reads — use count-matrix correlations instead
- Benchmarking isoform-level or alternative-splicing ground truths — scReadSim provides
  gene-level UMI ground truths only
- Comparing simulators when reads are aligned to different reference genomes — correlations
  are only comparable on the same reference (mm10)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: scReadSim
  role: Generates genome-aligned synthetic scRNA-seq reads (BAM) trained on real data;
    its output coverage is the high-fidelity case (~0.930 Spearman)
  repo: https://github.com/JSB-UCLA/scReadSim
- name: minnow
  role: Baseline read simulator; its synthetic reads (two FASTQ files) are aligned
    with bowtie2 and show near-zero coverage similarity (~0.065 Spearman)
- name: bowtie2
  role: Aligns minnow's synthetic FASTQ reads to the mm10 reference so per-window
    coverage is comparable to scReadSim's BAM output
- name: IGV
  role: Browser-style visualization of per-window coverage tracks for real and synthetic
    data with inset close-up views
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:adff9cbabb874b52a658d29eb52970278cd8c61683b5661603c7ff70c2fb1829
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - IGV
  - bowtie2
  - minnow
  - scReadSim
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pseudobulk-read-coverage-window-binning

## Summary

A read-level fidelity assessment for synthetic scRNA-seq data: real and simulated BAM/aligned reads are binned per chromosome (e.g., 1000-bp windows on chr1), per-window coverage is summed, and Spearman/Pearson correlations quantify how well a simulator (scReadSim vs minnow) reproduces real read-coverage structure.

## When to use

Use when comparing synthetic scRNA-seq/scATAC-seq reads against real reads at the read-sequence level, or when benchmarking a read simulator's ability to mimic pseudo-bulk coverage patterns across genomic windows.

## When NOT to use

- Evaluating count-level fidelity of a simulator that outputs only a gene-by-cell count matrix and no reads — use count-matrix correlations instead
- Benchmarking isoform-level or alternative-splicing ground truths — scReadSim provides gene-level UMI ground truths only
- Comparing simulators when reads are aligned to different reference genomes — correlations are only comparable on the same reference (mm10)

## Inputs

- Genome-aligned scRNA-seq reads (BAM) from real data, e.g., mouse 10x Multiome RNA-seq on mm10
- Synthetic reads from scReadSim (BAM, already genome-aligned)
- Synthetic paired-end FASTQ files from minnow, aligned with bowtie2 to the same reference genome (mm10)

## Outputs

- Per-window (1000-bp) read-coverage tracks for real and synthetic datasets
- Spearman and Pearson correlations of per-window coverage between real and each synthetic dataset
- IGV-style coverage track plots with inset close-up views (Fig. 2a style)

## How to apply

Generate or obtain genome-aligned reads for real data (e.g., scReadSim output BAM aligned to mm10) and for each simulator under test (e.g., minnow FASTQ output aligned with bowtie2 to the same mm10 reference). Bin a target chromosome (chr1 in the reference experiment) into fixed 1000-bp windows and sum read counts per window to form pseudo-bulk coverage tracks for the real data and each synthetic dataset. Compute Spearman and Pearson correlations of per-window coverage between real and synthetic tracks across all windows on the chromosome. Optionally visualize the per-window coverage tracks in an IGV-style plot with an inset close-up (e.g., chr1:86,425,858-86,443,378) as in Fig. 2a. A simulator that learns from real reads (scReadSim) should yield high Spearman correlation (~0.930), while a count-based simulator not trained on real reads (minnow, which only simulates annotated genes) yields near-zero correlation (~0.065).

## Related tools

- **scReadSim** (Generates genome-aligned synthetic scRNA-seq reads (BAM) trained on real data; its output coverage is the high-fidelity case (~0.930 Spearman)) — https://github.com/JSB-UCLA/scReadSim
- **minnow** (Baseline read simulator; its synthetic reads (two FASTQ files) are aligned with bowtie2 and show near-zero coverage similarity (~0.065 Spearman))
- **bowtie2** (Aligns minnow's synthetic FASTQ reads to the mm10 reference so per-window coverage is comparable to scReadSim's BAM output)
- **IGV** (Browser-style visualization of per-window coverage tracks for real and synthetic data with inset close-up views)

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(real_cov_chr1, screadsim_cov_chr1)  # real=0.930 vs minnow=0.065
```

## Evaluation signals

- Spearman correlation of per-window coverage between real and scReadSim synthetic reads is close to 0.930
- Spearman correlation between real and minnow synthetic reads is close to 0.065
- All datasets are evaluated on identical 1000-bp windows of chromosome 1 against the same mm10 reference
- Coverage tracks visually overlap between real and scReadSim (including intergenic regions) in the IGV-style plot, unlike minnow which lacks intergenic reads
- Pearson correlation computed alongside Spearman on the same window-level coverage vectors

## Limitations

- minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions, biasing window-level comparisons in gene-poor regions
- Default scReadSim implementation may generate intronic synthetic reads, which can affect downstream tool benchmarks such as cellranger default mode
- Multi-sample/condition comparisons require one BAM file per sample or condition

## Evidence

- [results] scReadSim outperforms minnow: its synthetic scRNA-seq reads achieve a pseudo-bulk read-coverage Spearman correlation of 0.930 with real reads, whereas minnow's synthetic reads do not mimic real coverage (Spearman correlation = 0.065).: [evidence span withheld — anchor status: not_found]
- [intro] minnow lacks intergenic reads: "minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions"
- [methods] scReadSim has three major steps: "As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [readme] scReadSim generates synthetic reads in BAM or FASTQ formats trained on real data: [evidence span withheld — anchor status: external_doc]
