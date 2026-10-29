---
name: single-cell-count-matrix-generation
description: Use when raw FASTQ files (or Cell Ranger BAMs convertible via bamtofastq)
  from any UMI-based scRNA-seq platform must be processed into gene-by-barcode matrices
  under one uniform pipeline, e.g. for cross-platform integration benchmarks.
when_to_use_negative:
- Input data comes from 5' scRNA-seq technologies other than Chromium, which UniverSC
  does not support
- Data are already processed gene-by-barcode matrices and the goal is integration
  or clustering rather than raw count generation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: UniverSC
  role: Universal processing pipeline (Cell Ranger wrapper) that converts FASTQ to
    gene-by-barcode matrices with automatic chemistry determination
  repo: https://github.com/minoda-lab/universc
- name: Cell Ranger 3.0.2
  role: Underlying alignment/quantification engine invoked by UniverSC
- name: bamtofastq
  role: Converts Cell Ranger BAM output back to FASTQ so datasets can be reprocessed
    with UniverSC
  repo: https://github.com/10XGenomics/bamtofastq
- name: Docker
  role: Containerised delivery of UniverSC for consistent, reproducible cross-platform
    processing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:e27e6d3b4546467212c3d654d390f1e656b6ce51d3cbb5aebfafcfa66008cee9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - Cell Ranger 3.0.2
  - Docker
  - UniverSC
  - bamtofastq
  - cluster
  - zUMIs
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Single-cell count matrix generation

## Summary

Generate gene-by-barcode count (GBM) matrices from single-cell RNA-seq FASTQ data using UniverSC, a Cell Ranger-wrapping pipeline that supports any UMI-based platform (Chromium, Drop-seq, ICELL8, SmartSeq2/3) with automatic chemistry determination and a single processing configuration across platforms.

## When to use

Use when raw FASTQ files (or Cell Ranger BAMs convertible via bamtofastq) from any UMI-based scRNA-seq platform must be processed into gene-by-barcode matrices under one uniform pipeline, e.g. for cross-platform integration benchmarks.

## When NOT to use

- Input data comes from 5' scRNA-seq technologies other than Chromium, which UniverSC does not support
- Data are already processed gene-by-barcode matrices and the goal is integration or clustering rather than raw count generation

## Inputs

- FASTQ files from UMI-based scRNA-seq platforms (Chromium, Drop-seq, ICELL8, SmartSeq2/3)
- Cell Ranger BAM output convertible to FASTQ via bamtofastq
- Reference genome (e.g. full mouse GRCm39 or human GRCh38 chr21)

## Outputs

- Gene-by-barcode count matrix (GBM) per platform/pipeline
- Cell Ranger alignment/quantification outputs for each processed dataset

## How to apply

Run UniverSC (a shell wrapper around Cell Ranger 3.0.2, available as a Docker image) on the FASTQ input, letting the tool automatically determine the 3' scRNA-seq chemistry appropriate for the platform. Choose a reference genome matching the organism — e.g. the full mouse genome GRCm39 for mouse primary cell data, or human GRCh38 chromosome 21 with reads filtered to chr21 to reduce output size — and avoid downsampling unless required. For Cell Ranger 1.1.0 output BAMs, first convert to FASTQ with the 10XGenomics bamtofastq tool. The output gene-by-barcode matrices can then be harmonised across pipelines by keeping only intersecting barcodes and adding missing genes with 0 UMIs before downstream Seurat integration and kBET/Silhouette evaluation.

## Related tools

- **UniverSC** (Universal processing pipeline (Cell Ranger wrapper) that converts FASTQ to gene-by-barcode matrices with automatic chemistry determination) — https://github.com/minoda-lab/universc
- **Cell Ranger 3.0.2** (Underlying alignment/quantification engine invoked by UniverSC)
- **bamtofastq** (Converts Cell Ranger BAM output back to FASTQ so datasets can be reprocessed with UniverSC) — https://github.com/10XGenomics/bamtofastq
- **Docker** (Containerised delivery of UniverSC for consistent, reproducible cross-platform processing)

## Examples

```
universc count --id=smartseq2_sample --fastqs=fastqs/ --transcriptome=refdata-gex-GRChm39 --chemistry=SC3Pv2
```

## Evaluation signals

- GBMs from different pipelines share identical gene and barcode sets after harmonisation (intersecting barcodes kept, missing genes filled with 0 UMIs)
- No downsampling performed against the chosen reference (e.g. full GRCm39) so quantification is not read-limited
- Downstream integration reproduces expected metric direction: UniverSC-processed SmartSeq2 gives kBET ~0.06 vs zUMIs ~0.11 and Silhouette ~0.43 vs ~0.36 (Fig. 3B, C)
- UniverSC Docker/GitHub CI test suites for each platform (10x Genomics, DropSeq, ICELL8, SCI-Seq, inDrops v3, Smart-Seq3) pass for the target chemistry
- Pearson correlation between harmonised GBMs and ARI between clustering outcomes computed as cross-pipeline agreement checks

## Limitations

- UniverSC uses Cell Ranger version 3.0.2 due to licensing, not the latest Cell Ranger release
- 5' scRNA-seq technologies other than Chromium are not supported at the time of writing
- launch_universc.sh requires write access to a Cell Ranger installation, so Cell Ranger must be installed in a user's home directory (no admin rights needed but not a system install)
- Reads may need filtering/downsampling (e.g. chr21-only, ICELL8 downsampled to 250 K reads with seqtk) to reduce output size in test settings

## Evidence

- [readme] UniverSC supports any UMI-based platform: [evidence span withheld — anchor status: external_doc]
- [methods] Automatic chemistry determination: [evidence span withheld — anchor status: not_found]
- [methods] Full mouse genome reference, no downsampling: "The full mouse genome (GRCm39) was used as the reference genome and no downsampling was performed for these datasets."
- [methods] GBM harmonisation across pipelines: "only barcodes found in both GBMs were kept, and genes only found in one GBM were added to the other with 0 UMIs assigned."
- [methods] Cell Ranger wrapper and BAM conversion workflow: "the 10x Genomics bamtofastq tool (https://github.com/10XGenomics/bamtofastq) was used to convert Cell Ranger 1.1.0 output"
