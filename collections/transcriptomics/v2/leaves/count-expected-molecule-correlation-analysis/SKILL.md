---
name: count-expected-molecule-correlation-analysis
description: Use when benchmarking detection sensitivity and quantitative linearity
  of a UMI-based scRNA-seq protocol against ERCC spike-ins with known molecule concentrations
  loaded into GEMs.
when_to_use_negative:
- Input data contain no ERCC spike-ins or no known expected molecule counts, so no
  expected-vs-observed correlation can be computed
- Goal is gene-level expression quantification of cellular transcriptomes rather than
  synthetic-RNA sensitivity benchmarking
- Reads come from a non-UMI or full-length protocol where molecule counts cannot be
  de-duplicated via cell barcode + UMI combinations
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: STAR
  role: Aligns Read1 (the cDNA insert) of ERCC runs to a STAR index built on the ERCC
    reference transcripts
- name: Cell Ranger Single-Cell Software Suite
  role: Reference pipeline for sample demultiplexing, barcode processing and single-cell
    3' gene/UMI counting logic mirrored in the UMI validity and counting rules
  repo: 10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:72545996fbac8d0fe2f01d6118971a1a40c6f1cad2d08c3be818b9e9cf826202
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# count-expected-molecule-correlation-analysis

## Summary

Quantifies how faithfully a droplet-based scRNA-seq system converts known input synthetic RNA molecules (ERCC spike-ins) into detected UMI counts, reporting inferred cDNA conversion efficiency, Pearson correlation between sequenced UMI counts and expected molecule counts, and CV2 across ERCC experiments.

## When to use

Use when benchmarking detection sensitivity and quantitative linearity of a UMI-based scRNA-seq protocol against ERCC spike-ins with known molecule concentrations loaded into GEMs.

## When NOT to use

- Input data contain no ERCC spike-ins or no known expected molecule counts, so no expected-vs-observed correlation can be computed
- Goal is gene-level expression quantification of cellular transcriptomes rather than synthetic-RNA sensitivity benchmarking
- Reads come from a non-UMI or full-length protocol where molecule counts cannot be de-duplicated via cell barcode + UMI combinations

## Inputs

- ERCC Spike-in Mix1 and Mix2 (1:50 dilution) GEM sequencing FASTQ files from SRA accession SRP073767 or the 10x Genomics datasets site
- ERCC reference transcript sequences (FASTA) from tools.thermofisher.com/content/sfs/manuals/cms_095047.txt

## Outputs

- Per-ERCC-experiment inferred cDNA conversion efficiency values (fraction of expected molecules represented by counted UMIs)
- Pearson correlation coefficient (r) between mean UMI counts and expected ERCC molecule counts
- Coefficient of variation squared (CV2) across ERCC experiments
- ercc_sensitivity_metrics.csv containing conversion efficiency, Pearson r, and CV2

## How to apply

Download ERCC Spike-in Mix1 and Mix2 (1:50 dilution) GEM FASTQ files (SRA SRP073767 or support.10xgenomics.com/single-cell/datasets) and the ERCC reference sequences from tools.thermofisher.com/cms_095047.txt, then build a STAR index on the ERCC reference. Align Read1 (the cDNA insert) of each ERCC run to the ERCC reference with STAR. Apply the paper's UMI validity rules — UMIs with sequencing quality score >10 that are not homopolymers, with 1-Hamming-distance correction to the UMI with more reads for the same cell barcode and gene — to count valid barcode/UMI combinations per ERCC transcript. Compute per-experiment conversion efficiency as the fraction of expected ERCC molecules represented by counted UMIs (expected range 6.7–8.1%), then compute the Pearson correlation between mean UMI counts and expected molecule counts (expected ~0.96) and the CV2 across ERCC experiments (expected ~7%); write the metrics to ercc_sensitivity_metrics.csv.

## Related tools

- **STAR** (Aligns Read1 (the cDNA insert) of ERCC runs to a STAR index built on the ERCC reference transcripts)
- **Cell Ranger Single-Cell Software Suite** (Reference pipeline for sample demultiplexing, barcode processing and single-cell 3' gene/UMI counting logic mirrored in the UMI validity and counting rules) — 10XGenomics/cellranger

## Examples

```
STAR --genomeDir ercc_index --readFilesIn Read1.fastq.gz --readFilesCommand zcat --outFileNamePrefix ercc_mix1_ && python ercc_metrics.py --umi-q 10 --out ercc_sensitivity_metrics.csv
```

## Evaluation signals

- Inferred conversion efficiency values fall within the reported 6.7–8.1% range for both ERCC Spike-in Mix1 and Mix2 at 1:50 dilution
- Pearson correlation between mean UMI counts and expected ERCC molecule counts is approximately 0.96
- CV2 across all ERCC experiments is approximately 7%
- UMI counts pass the paper's validity rules: quality score >10, non-homopolymer, 1-Hamming-distance corrected to the more-abundant UMI per cell barcode and gene
- ercc_sensitivity_metrics.csv is populated with all three recomputed metrics (conversion efficiency, Pearson r, CV2)

## Limitations

- Conversion efficiency is inferred from ERCC spike-ins at 1:50 dilution and may not extrapolate to cellular transcripts with different lengths or structures
- Some earlier libraries used 5 nt UMI and 5 bp Read2 instead of the standard 10 nt UMI, introducing protocol heterogeneity that can affect UMI counting
- The paper notes a 15–20% error in cell counts, which could account for variability in calculated efficiencies

## Evidence

- [methods] ERCC alignment and UMI counting rules: "Read1, which contains the cDNA insert, was aligned to an appropriate reference genome using STAR35."
- [methods] UMI validity thresholds: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [methods] UMI correction rule: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [abstract] Sensitivity validated with synthetic RNAs: "We validated the sensitivity of the system and its ability to detect rare populations using cell lines and synthetic RNAs."
- [readme] Data availability via 10x datasets site: [evidence span withheld — anchor status: not_found]
