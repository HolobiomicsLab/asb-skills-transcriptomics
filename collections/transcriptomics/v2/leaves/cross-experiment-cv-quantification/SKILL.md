---
name: cross-experiment-cv-quantification
description: Use when you need to quantify technical variability across replicate
  sequencing runs or ERCC spike-in experiments, e.g. computing CV2 across ERCC experiments
  and correlating mean UMI counts with expected molecule counts.
when_to_use_negative:
- The goal is biological (sample-to-sample) variability among cells rather than technical
  variability of synthetic spike-in controls
- Input data lack ERCC spike-ins or known expected molecule counts, so conversion
  efficiency and correlation to expected counts cannot be computed
- UMI counts have not yet been generated from FASTQ via barcode/UMI validity filtering
  — QC the counts first
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
tools:
- name: STAR
  role: Aligns Read1 (cDNA insert) of ERCC runs to a STAR index built on the ERCC
    reference
- name: Cell Ranger Single-Cell Software Suite
  role: Reference pipeline for sample demultiplexing, barcode processing and single-cell
    3' gene counting whose UMI rules are reproduced
  repo: https://github.com/10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:b2507a3d9fb25dfb898be965b57ca359bfe988aff6f89d4e3e8568d7303a6e4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Cross-experiment CV quantification

## Summary

Quantifies the coefficient of variation squared (CV2) and Pearson correlation across replicate experiments using ERCC spike-in UMI counts, to benchmark droplet-based scRNA-seq sensitivity and conversion efficiency as in Zheng et al. 2017.

## When to use

Use when you need to quantify technical variability across replicate sequencing runs or ERCC spike-in experiments, e.g. computing CV2 across ERCC experiments and correlating mean UMI counts with expected molecule counts.

## When NOT to use

- The goal is biological (sample-to-sample) variability among cells rather than technical variability of synthetic spike-in controls
- Input data lack ERCC spike-ins or known expected molecule counts, so conversion efficiency and correlation to expected counts cannot be computed
- UMI counts have not yet been generated from FASTQ via barcode/UMI validity filtering — QC the counts first

## Inputs

- ERCC Spike-in Mix1 and Mix2 (1:50 dilution) GEM sequencing FASTQ files (SRA SRP073767 or support.10xgenomics.com/single-cell/datasets)
- ERCC reference transcript sequences (tools.thermofisher.com/content/sfs/manuals/cms_095047.txt)
- Expected ERCC molecule counts loaded into GEMs

## Outputs

- Per-experiment inferred cDNA conversion efficiency values
- Pearson correlation coefficient between mean UMI counts and expected ERCC molecule counts
- CV2 across all ERCC experiments
- Summary table (e.g. ercc_sensitivity_metrics.csv)

## How to apply

Align Read1 (the cDNA insert) of each ERCC GEM run to the ERCC reference (cms_095047.txt) with STAR, then count valid cell-barcode/UMI combinations per ERCC transcript using the paper's UMI validity rules: quality score >10, non-homopolymer UMIs, and 1-Hamming-distance correction to the more-abundant UMI for the same cell barcode and gene. Compute inferred cDNA conversion efficiency as the fraction of expected ERCC molecules represented by counted UMIs (expected: 6.7–8.1% for Mix1 and Mix2 at 1:50 dilution). Compute the Pearson correlation between mean sequenced UMI counts and expected ERCC molecule counts per transcript (expected r≈0.96). Finally compute CV2 (variance/mean squared) across all ERCC experiments to summarize technical noise (~7% reported). Write all recomputed metrics to a CSV such as ercc_sensitivity_metrics.csv.

## Related tools

- **STAR** (Aligns Read1 (cDNA insert) of ERCC runs to a STAR index built on the ERCC reference)
- **Cell Ranger Single-Cell Software Suite** (Reference pipeline for sample demultiplexing, barcode processing and single-cell 3' gene counting whose UMI rules are reproduced) — https://github.com/10XGenomics/cellranger

## Examples

```
python compute_ercc_metrics.py --fastq ercc_mix1.fastq --ercc-ref cms_095047.txt --expected counts.csv --out ercc_sensitivity_metrics.csv
```

## Evaluation signals

- Inferred cDNA conversion efficiency values fall within the reported 6.7–8.1% range for both ERCC Spike-in Mix1 and Mix2 at 1:50 dilution
- Pearson correlation between mean UMI counts and expected ERCC molecule counts is approximately 0.96
- CV2 across all ERCC experiments is approximately 7%
- All counted UMIs pass validity rules: quality score >10, non-homopolymer, 1-Hamming-distance-corrected to the more-abundant UMI
- Output CSV contains conversion efficiency, Pearson r, and CV2 columns populated for every ERCC experiment

## Limitations

- Some earlier libraries used 5 nt UMI and 5 bp Read2, introducing protocol heterogeneity that can affect count comparability across libraries
- Cell count estimation carries 15–20% error, which can propagate into calculated efficiency metrics
- Conversion-efficiency inference depends on accurate expected ERCC molecule counts loaded into GEMs; pipetting variation in spike-in dilution (e.g. 1:50) directly biases the denominator

## Evidence

- [methods] UMIs with sequencing quality score >10 were considered valid if they were not homopolymers.: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [methods] 1-Hamming-distance UMI correction rule: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [methods] ERCC reference source: "For ERCC samples, ERCC reference (https://tools.thermofisher.com/content/sfs/manuals/cms_095047.txt) was used."
- [methods] Data availability for ERCC GEM runs: "Single-cell RNA-seq data have been deposited in the Short Read Archive under accession number SRP073767."
- [readme] Repository README describing data availability: [evidence span withheld — anchor status: not_found]
