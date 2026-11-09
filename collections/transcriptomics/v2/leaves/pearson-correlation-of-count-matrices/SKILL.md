---
name: pearson-correlation-of-count-matrices
description: Use when two raw gene-barcode count matrices produced by different scRNA-seq
  pipelines on the same FASTQ dataset must be compared numerically to validate that
  an alternative pipeline reproduces native-pipeline output.
when_to_use_negative:
- Input matrices come from different datasets, references, or read sets — the correlation
  is only meaningful for paired pipelines run on identical FASTQ input (e.g. same
  GRCh38 chr21-mapped reads)
- The goal is to compare dimensionality-reduced or normalized expression matrices
  rather than raw UMI count GBMs
- 5' scRNA-seq data from technologies other than Chromium, which UniverSC does not
  support at the time of writing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: UniverSC
  role: Produces the alternative-pipeline GBM being benchmarked; wraps Cell Ranger
    3.0.2 to process UMI-based 3' scRNA-seq FASTQ data across platforms
  repo: https://github.com/minoda-lab/universc
- name: Cell Ranger 3.0.2
  role: Native Chromium pipeline whose GBM serves as the reference for the r = 1 comparison
    against UniverSC
- name: R stats package
  role: Computes the Pearson correlation between paired per-gene UMI counts from the
    two harmonized GBMs
- name: Docker
  role: Containerized environment for running UniverSC consistently alongside native
    pipelines
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:c4be6056f6b179497aa15cdfc81890592980dd683158415dfcc01d0110247199
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - Cell Ranger 3.0.2
  - Docker
  - R stats package
  - UniverSC
  - dropSeqPipe
  - seqtk
  - zUMIs
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pearson-correlation-of-count-matrices

## Summary

Quantifies agreement between two single-cell gene-barcode matrices (GBMs), e.g. one from UniverSC and one from a platform-specific pipeline such as Cell Ranger, dropSeqPipe, CogentAP, or zUMIs, by Pearson-correlating paired per-gene UMI counts. This is the benchmarking approach used to show UniverSC GBMs match native pipelines (r = 1 for Chromium/Cell Ranger 3.0.2, r >= 0.94 for the other three platforms).

## When to use

Use when two raw gene-barcode count matrices produced by different scRNA-seq pipelines on the same FASTQ dataset must be compared numerically to validate that an alternative pipeline reproduces native-pipeline output.

## When NOT to use

- Input matrices come from different datasets, references, or read sets — the correlation is only meaningful for paired pipelines run on identical FASTQ input (e.g. same GRCh38 chr21-mapped reads)
- The goal is to compare dimensionality-reduced or normalized expression matrices rather than raw UMI count GBMs
- 5' scRNA-seq data from technologies other than Chromium, which UniverSC does not support at the time of writing

## Inputs

- Raw gene-barcode matrix (GBM) from UniverSC (UMI count matrix, genes x barcodes)
- Raw gene-barcode matrix (GBM) from the platform-native pipeline (Cell Ranger 3.0.2, dropSeqPipe, CogentAP, or zUMIs) run on the same GRCh38 chr21 FASTQ dataset

## Outputs

- Pearson correlation coefficient r for each UniverSC-vs-native GBM pair (expected: 1.0 for Chromium, >=0.94 for Drop-seq, ICELL8, SmartSeq3)
- Harmonized GBM pairs restricted to shared barcodes with union gene sets (missing entries filled with 0 UMIs)

## How to apply

First harmonize the pair of raw GBMs: keep only barcodes found in both matrices, and add genes found in only one matrix to the other with 0 UMIs assigned so both share identical barcode and gene sets. Then compute the Pearson correlation using the R package stats, treating each gene's paired UMI counts (one value per pipeline) per cell as a single data point, excluding gene-cell pairs where both counts are zero. Rationale: matrix harmonization ensures the correlation is computed on comparable features, and the zero-pair exclusion prevents non-informative (0,0) pairs from inflating r. Expect an agreement threshold of r >= 0.94 for heterogeneous pipelines (Drop-seq, ICELL8, SmartSeq3) and r = 1 when the alternative pipeline wraps the native one (UniverSC wrapping Cell Ranger 3.0.2 for Chromium). Apply the same recipe per platform pair to reproduce the Fig. 2 left-column comparison.

## Related tools

- **UniverSC** (Produces the alternative-pipeline GBM being benchmarked; wraps Cell Ranger 3.0.2 to process UMI-based 3' scRNA-seq FASTQ data across platforms) — https://github.com/minoda-lab/universc
- **Cell Ranger 3.0.2** (Native Chromium pipeline whose GBM serves as the reference for the r = 1 comparison against UniverSC)
- **R stats package** (Computes the Pearson correlation between paired per-gene UMI counts from the two harmonized GBMs)
- **Docker** (Containerized environment for running UniverSC consistently alongside native pipelines)

## Examples

```
cor(as.vector(gbm_universc[, shared_barcodes]), as.vector(gbm_native[, shared_barcodes]), method="pearson")
```

## Evaluation signals

- Chromium UniverSC-vs-Cell Ranger 3.0.2 pair reaches Pearson correlation r = 1 (identical GBMs)
- Drop-seq (dropSeqPipe), ICELL8 (CogentAP), and SmartSeq3 (zUMIs) pairs each reach r >= 0.94, matching Fig. 2 left-column results
- Both GBMs have identical barcode sets after filtering (only barcodes present in both matrices kept) and identical gene sets (union with 0-UMI fill)
- No (0,0) gene-cell pairs are included as data points in the correlation computation
- Same reference (GRCh38 hg38 chromosome 21) and same input reads (e.g. chr21-filtered Chromium FASTQ from bamtofastq) were used for both pipelines

## Limitations

- UniverSC utilises Cell Ranger version 3.0.2 due to licensing, so results may differ from analyses run with newer Cell Ranger versions
- 5' scRNA-seq technologies other than Chromium are not supported by UniverSC, so the correlation check cannot be applied to those data
- Correlation is computed per gene on paired UMI counts; the exclusion of double-zero pairs means sparsely detected genes contribute few or no data points

## Evidence

- [methods] GBM harmonization before correlation: "only barcodes found in both GBMs were kept, and genes only found in one GBM were added to the other with 0 UMIs assigned."
- [methods] Pearson correlation computation: "the Pearson correlation between the GBMs and ARI between the two clustering outcomes were calculated"
- [discussion] Correlation thresholds reported in the article: "UniverSC utilises Cell Ranger version 3.0.2 due to licensing"
- [readme] Scope of UniverSC (readme): [evidence span withheld — anchor status: external_doc]
