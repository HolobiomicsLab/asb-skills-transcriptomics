---
name: genome-browser-coverage-visualization
description: Use when synthetic scRNA-seq/scATAC-seq reads must be visually validated
  against real reads at the read-sequence level, e.g. comparing coverage tracks of
  scReadSim BAM output vs minnow/bowtie2-aligned FASTQ over binned genomic windows.
when_to_use_negative:
- The comparison is at the UMI count-matrix level rather than read coverage over genomic
  coordinates — scReadSim's synthetic gene-by-cell UMI matrix is the appropriate ground
  truth for count-level benchmarks instead
- Benchmarking isoform or alternative-splicing tools — scReadSim's current version
  lacks isoform-level ground truths, so coverage tracks cannot validate isoform resolution
- The synthetic reads are not aligned to the same reference genome build (mm10) as
  the real data, making per-window coverage incomparable
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: scReadSim
  role: Generates genome-aligned synthetic paired-end reads (BAM) whose coverage track
    is visualized against real data
  repo: https://github.com/JSB-UCLA/scReadSim
- name: minnow
  role: Baseline read simulator; its FASTQ output is aligned with bowtie2 so its coverage
    track can be plotted alongside real and scReadSim data
- name: bowtie2
  role: Aligns minnow's synthetic FASTQ reads to the mm10 reference genome so per-window
    coverage is computable
- name: IGV
  role: Browser style referenced for the stacked per-window coverage track figure
    with inset close-up
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:ed9fd8dc7e82836ceae441a7a3b66f70603a7a06b83a2ec43de3a0e09c47f809
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - IGV
  - bowtie2
  - minnow
  - scReadSim
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Genome-browser coverage visualization

## Summary

Renders per-window sequencing read-coverage tracks for real and synthetic single-cell datasets in an IGV-browser-style plot with an inset close-up, enabling visual alongside quantitative (Spearman/Pearson) comparison of read-level fidelity, e.g. scReadSim (Spearman 0.930) vs minnow (0.065) on chromosome 1.

## When to use

Use when synthetic scRNA-seq/scATAC-seq reads must be visually validated against real reads at the read-sequence level, e.g. comparing coverage tracks of scReadSim BAM output vs minnow/bowtie2-aligned FASTQ over binned genomic windows.

## When NOT to use

- The comparison is at the UMI count-matrix level rather than read coverage over genomic coordinates — scReadSim's synthetic gene-by-cell UMI matrix is the appropriate ground truth for count-level benchmarks instead
- Benchmarking isoform or alternative-splicing tools — scReadSim's current version lacks isoform-level ground truths, so coverage tracks cannot validate isoform resolution
- The synthetic reads are not aligned to the same reference genome build (mm10) as the real data, making per-window coverage incomparable

## Inputs

- BAM file of real scRNA-seq reads aligned to mm10 (e.g., mouse 10x Multiome RNA-seq BAM)
- BAM file of scReadSim synthetic paired-end reads (genome-aligned)
- FASTQ pair from minnow aligned to mm10 via bowtie2
- mm10 chromosome size file (embedded within the scReadSim package)
- Window size parameter (1000 bp) and inset locus (e.g., chr1:86,425,858-86,443,378)

## Outputs

- Per-window (1000-bp) read-coverage values for real, scReadSim, and minnow datasets
- Spearman and Pearson correlation of per-window coverage between real and each synthetic dataset
- IGV-browser-style multi-track coverage figure with inset close-up view (Fig. 2a analogue)

## How to apply

First ensure all datasets are genome-aligned: scReadSim outputs a synthetic BAM already aligned to mm10, while minnow's paired-end FASTQ outputs must be aligned with bowtie2 to the same mm10 reference. Partition the chromosome (the paper uses chromosome 1) into 1000-bp windows and sum read coverage per window for the real data, scReadSim's synthetic data, and minnow's synthetic data. Compute Spearman and Pearson correlations of per-window coverage between real and each synthetic dataset across windows to accompany the plot. Render the per-window coverage tracks for all three datasets stacked in IGV-browser style, adding an inset close-up (e.g., chr1:86,425,858-86,443,378) so local concordance of read coverage is visible, reproducing Fig. 2a of the scReadSim paper.

## Related tools

- **scReadSim** (Generates genome-aligned synthetic paired-end reads (BAM) whose coverage track is visualized against real data) — https://github.com/JSB-UCLA/scReadSim
- **minnow** (Baseline read simulator; its FASTQ output is aligned with bowtie2 so its coverage track can be plotted alongside real and scReadSim data)
- **bowtie2** (Aligns minnow's synthetic FASTQ reads to the mm10 reference genome so per-window coverage is computable)
- **IGV** (Browser style referenced for the stacked per-window coverage track figure with inset close-up)

## Evaluation signals

- Spearman correlation of per-window coverage between real and scReadSim synthetic data is close to the reported 0.930
- Spearman correlation of per-window coverage between real and minnow synthetic data is close to the reported 0.065
- All three tracks use identical 1000-bp window partitions of chromosome 1 and the same mm10 reference build
- Inset view (chr1:86,425,858-86,443,378) shows local coverage peaks in real and scReadSim tracks aligning, while minnow's track deviates
- scReadSim's BAM is already genome-aligned; only minnow's FASTQ required bowtie2 alignment, confirming consistent coordinate systems before binning

## Limitations

- minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions — coverage-track mismatch is expected by design
- The default implementation of scReadSim may generate intronic synthetic reads when extracting sequences from the reference genome, which can shift coverage patterns in intronic windows
- Coverage correlation is sensitive to bin size and chromosome choice; the reported values (0.930 vs 0.065) correspond to 1000-bp windows on chromosome 1

## Evidence

- [methods] Plot the per-window read coverage tracks for real, scReadSim, and minnow data with an inset close-up view (e.g., chr1:86,425,858-86,443,378) in the IGV browser style, reproducing Fig. 2a.: [evidence span withheld — anchor status: not_found]
- [results] scReadSim outperforms minnow: its synthetic scRNA-seq reads achieve a pseudo-bulk read-coverage Spearman correlation of 0.930 with real reads, whereas minnow's synthetic reads do not mimic real coverage (Spearman correlation = 0.065).: [evidence span withheld — anchor status: not_found]
- [intro] minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions: "minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions"
- [readme] Trained on real data, scReadSim generates synthetic sequencing reads in BAM or FASTQ formats.: [evidence span withheld — anchor status: external_doc]
