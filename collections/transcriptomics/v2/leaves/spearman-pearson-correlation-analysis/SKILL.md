---
name: spearman-pearson-correlation-analysis
description: Use when comparing read-level coverage tracks of synthetic vs real scRNA-seq/scATAC-seq
  data binned into genomic windows (e.g., 1000-bp bins on chr1) to quantify simulator
  fidelity.
when_to_use_negative:
- Benchmarking isoform abundance or alternative splicing tools — scReadSim provides
  gene-level UMI-count ground truths only, so coverage correlations do not validate
  isoform-level fidelity
- Data that is already a feature/count matrix without genomic coordinates — windowed
  read-coverage correlation requires aligned reads with chromosomal positions
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scReadSim
  role: Generates synthetic scRNA-seq reads (BAM) whose per-window coverage is correlated
    against real data
  repo: github.com/JSB-UCLA/scReadSim
- name: minnow
  role: Baseline read simulator; its synthetic FASTQ reads are aligned with bowtie2
    and serve as the negative comparison in the coverage correlation
- name: bowtie2
  role: Aligns minnow's synthetic FASTQ reads to the mm10 reference genome so per-window
    coverage can be computed
- name: scDesign2
  role: Count simulator trained by scReadSim to produce the synthetic UMI count matrix
    that seeds synthetic read generation
  repo: github.com/JSB-UCLA/scDesign2
- name: IGV
  role: Browser-style visualization of per-window coverage tracks with inset close-up
    view (Fig. 2a)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:cdcc70030d1150186c7bae44b5c04c519926e6faa5a0056bb65c8195f7346135
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IGV
  - bowtie2
  - minnow
  - scDesign2
  - scReadSim
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spearman-pearson-correlation-analysis

## Summary

Computes Spearman and Pearson correlations of per-window sequencing read coverage between real and synthetic datasets to quantify how faithfully a read simulator (e.g., scReadSim vs minnow) mimics real read-sequence-level coverage.

## When to use

Use when comparing read-level coverage tracks of synthetic vs real scRNA-seq/scATAC-seq data binned into genomic windows (e.g., 1000-bp bins on chr1) to quantify simulator fidelity.

## When NOT to use

- Benchmarking isoform abundance or alternative splicing tools — scReadSim provides gene-level UMI-count ground truths only, so coverage correlations do not validate isoform-level fidelity
- Data that is already a feature/count matrix without genomic coordinates — windowed read-coverage correlation requires aligned reads with chromosomal positions

## Inputs

- Real scRNA-seq BAM file (e.g., mouse 10x Multiome RNA-seq modality, mm10 reference genome)
- Synthetic read BAM file from scReadSim (already genome-aligned)
- Synthetic read FASTQ files from minnow
- Reference genome (mm10) for aligning minnow FASTQ reads with bowtie2
- Chromosome size / binning definition (1000-bp windows, e.g., chr1:86,425,858-86,443,378 inset region)

## Outputs

- Per-window (1000-bp) read coverage vectors for real, scReadSim synthetic, and minnow synthetic data
- Spearman correlation of per-window coverage (expected ~0.930 for scReadSim, ~0.065 for minnow)
- Pearson correlation of per-window coverage
- IGV-browser-style coverage track plots with inset close-up (reproducing Fig. 2a)

## How to apply

Bin a chromosome (in the scReadSim benchmark, chromosome 1) into 1000-bp windows and sum read coverage per window for the real data, and for each synthetic dataset (scReadSim's genome-aligned BAM output; minnow's FASTQ output first aligned to the same mm10 reference with bowtie2). Compute per-window Spearman and Pearson correlations between real and synthetic coverage vectors across all windows. In the scReadSim paper this yields Spearman 0.930 for scReadSim versus 0.065 for minnow, demonstrating scReadSim's synthetic scRNA-seq reads mimic real pseudo-bulk coverage. Report Spearman as the primary fidelity metric since it is rank-based and robust to coverage-magnitude outliers across windows.

## Related tools

- **scReadSim** (Generates synthetic scRNA-seq reads (BAM) whose per-window coverage is correlated against real data) — github.com/JSB-UCLA/scReadSim
- **minnow** (Baseline read simulator; its synthetic FASTQ reads are aligned with bowtie2 and serve as the negative comparison in the coverage correlation)
- **bowtie2** (Aligns minnow's synthetic FASTQ reads to the mm10 reference genome so per-window coverage can be computed)
- **scDesign2** (Count simulator trained by scReadSim to produce the synthetic UMI count matrix that seeds synthetic read generation) — github.com/JSB-UCLA/scDesign2
- **IGV** (Browser-style visualization of per-window coverage tracks with inset close-up view (Fig. 2a))

## Examples

```
bin chr1 into 1000-bp windows, sum per-window coverage for real/screadsim/minnow BAMs, then: spearmanr(cov_real, cov_screadsim)  # expect ~0.930; spearmanr(cov_real, cov_minnow)  # expect ~0.065
```

## Evaluation signals

- Spearman correlation of per-window coverage between real and scReadSim synthetic data should be close to 0.930
- Spearman correlation between real and minnow synthetic data should be close to 0.065, confirming minnow fails to mimic real coverage
- Coverage tracks are computed on identical 1000-bp windows of chromosome 1 for all three datasets to ensure paired comparison
- Real-data coverage must be recomputed from the same BAM/binning scheme used for synthetic data, avoiding binning mismatches
- Visual inspection of the IGV-style plot (with chr1:86,425,858-86,443,378 inset) should show scReadSim tracks overlaying real coverage while minnow tracks diverge

## Limitations

- The default implementation of scReadSim may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks like cellranger default mode
- scReadSim generates scRNA-seq reads with gene-level ground truths (UMI counts) only, not isoform-level ground truths including isoform abundance and alternative splicing events
- minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions — a coverage correlation over intergenic windows will be structurally low for minnow

## Evidence

- [other] Spearman correlation of per-window read coverage between real and scReadSim synthetic data should be close to the reported 0.930, and between real and minnow close to 0.065.: [evidence span withheld — anchor status: not_found]
- [other] Bin chromosome 1 into 1000-bp windows and sum read coverage per window for the real data, scReadSim's synthetic data, and minnow's synthetic data.: [evidence span withheld — anchor status: not_found]
- [methods] scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation.: "scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [intro] minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions: "minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions"
- [methods] This BAM file uses mm10 as reference genome, the required chromosome size file is also embedded within the package.: "This BAM file uses mm10 as reference genome, the required chromosome size file is also embedded within the package."
