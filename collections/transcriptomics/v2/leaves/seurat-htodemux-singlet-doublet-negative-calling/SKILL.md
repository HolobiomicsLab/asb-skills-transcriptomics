---
name: seurat-htodemux-singlet-doublet-negative-calling
description: Use when a cell-hashing CITE-seq experiment yields a hashtag oligo count
  matrix (e.g. from CITE-seq-Count) and cells must be assigned to hashed samples with
  doublet/negative calls, as in gExcite's hashing deconvolution step.
when_to_use_negative:
- Input is single-cell gene expression or CITE-seq data without hashtag oligo counts
  — there are no HTOs to deconvolve
- Samples were not multiplexed via cell hashing (one FASTQ/sample, no hashtag library),
  so no demultiplexing is needed
- An existing per-cell sample-assignment table (e.g. hashed_sample_assignments.csv)
  is already available — reuse it instead of re-running demultiplexing
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Seurat
  role: R framework used to normalize the hashtag oligo counts per cell and perform
    HTODemux-style singlet/doublet/negative demultiplexing
- name: gExcite
  role: Start-to-end Snakemake pipeline whose hashing deconvolution module wraps the
    Seurat normalization and demultiplexing step
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager embedding the demultiplexing step reproducibly and scalably
    within gExcite
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: CITE-seq-Count
  role: Upstream tool that produces the hashtag count matrix consumed by the demultiplexing
    step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:490aaa4a3dea0607b74630bf90b872c4061f78b63cc0ea301b014220ffb19b6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - CITE-seq-Count
  - Seurat
  - Snakemake
  - gExcite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# seurat-htodemux-singlet-doublet-negative-calling

## Summary

Seurat-based demultiplexing of cell-hashing data as implemented in gExcite: hashtag oligo (HTO) counts are normalized per cell and HTODemux-style thresholding classifies each cell as a singlet assigned to a hashed sample, a doublet, or a negative.

## When to use

Use when a cell-hashing CITE-seq experiment yields a hashtag oligo count matrix (e.g. from CITE-seq-Count) and cells must be assigned to hashed samples with doublet/negative calls, as in gExcite's hashing deconvolution step.

## When NOT to use

- Input is single-cell gene expression or CITE-seq data without hashtag oligo counts — there are no HTOs to deconvolve
- Samples were not multiplexed via cell hashing (one FASTQ/sample, no hashtag library), so no demultiplexing is needed
- An existing per-cell sample-assignment table (e.g. hashed_sample_assignments.csv) is already available — reuse it instead of re-running demultiplexing

## Inputs

- Hashtag oligo (HTO) count matrix produced by CITE-seq-Count (cells x hashtags), e.g. for three hashed human PBMC samples from the gExcite example test data
- Hashed-sample definitions/naming used to label the demultiplexed sample identities

## Outputs

- Per-cell demultiplexing call per barcode: singlet with assigned sample identity, doublet, or negative
- hashed_sample_assignments.csv — a per-cell table mapping each cell barcode to its sample identity and demultiplexing call
- Set of doublet and negative cells removed prior to downstream gene expression and ADT analysis

## How to apply

Load the CITE-seq-Count hashtag count matrix for the hashed samples (in gExcite's example, three hashed human PBMC samples) into the Seurat framework. Normalize the hashtag oligo counts per cell so that HTO-derived counts are comparable across cells, then run Seurat's HTODemux-style thresholding on the normalized HTO counts, which classifies each cell barcode as a singlet (assigned to the specific hashed sample with maximal HTO signal), a doublet, or a negative. Assemble the per-cell assignment table mapping each cell barcode to its sample identity and demultiplexing call, and write it out as hashed_sample_assignments.csv. Downstream, retain only singlets, removing doublets and negatives, as gExcite does within its Snakemake-managed hashing deconvolution module before gene expression and CITE-seq/ADT analysis proceed.

## Related tools

- **Seurat** (R framework used to normalize the hashtag oligo counts per cell and perform HTODemux-style singlet/doublet/negative demultiplexing)
- **gExcite** (Start-to-end Snakemake pipeline whose hashing deconvolution module wraps the Seurat normalization and demultiplexing step) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager embedding the demultiplexing step reproducibly and scalably within gExcite) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **CITE-seq-Count** (Upstream tool that produces the hashtag count matrix consumed by the demultiplexing step)

## Examples

```
hto <- NormalizeData(hto_seurat, assay = "HTO", normalization.method = "CLR"); hto <- HTODemux(hto, assay = "HTO")
```

## Evaluation signals

- Every cell barcode in hashed_sample_assignments.csv matches a barcode column of the input CITE-seq-Count HTO count matrix (one row per barcode, no unassigned barcodes)
- Each cell carries exactly one call from the three allowed classes: singlet (with one of the three hashed sample identities), doublet, or negative
- Singlet assignments distribute across the expected hashed samples (e.g. roughly comparable cell counts per sample for the three hashed PBMC samples in the gExcite example)
- Doublet and negative cells are excluded from the downstream GEX and ADT analysis objects
- Normalized HTO values are comparable across cells (per-cell normalization applied) so that HTODemux-style thresholds separate confident singlets from ambiguous doublets/negatives

## Limitations

- gExcite is compatible with and tested on Linux only; other Unix systems including MacOS are currently not supported
- Running a quick test that skips the resource-intensive cellranger count and CITE-Seq steps requires dummy files in place of the FASTQ files
- The demultiplexing was showcased on three hashed samples of human PBMC cells; hashing deconvolution quality depends on sufficient per-cell HTO counts, and negatives must be explicitly removed after calling

## Evidence

- [abstract] gExcite hashing deconvolution in Seurat: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution"
- [methods] Scope of gExcite analysis modules: "provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution"
- [readme] Example hashed PBMC test data: "with three hashed samples of human PBMC cells, so that hashing deconvolution, GEX analysis and ADT analysis can be performed."
- [readme] Starting point of the demultiplexing-capable test run: "A quick test run on the example data can be performed that starts after the resource-intensive cellranger count and CITE-Seq steps."
