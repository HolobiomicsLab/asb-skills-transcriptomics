---
name: hashtag-demultiplexing-cell-assignment
description: Use when input contains a cell-hashing (hashtag oligo) ADT count matrix
  — e.g. CITE-seq-Count output for multiple hashed samples — and each cell barcode
  must be assigned to a sample, a doublet, or a negative before downstream single-cell
  analysis.
when_to_use_negative:
- The input is gene expression (GEX) data only with no hashtag/ADT oligo counts —
  there is no HTO signal to threshold, so demultiplexing cannot be performed.
- Sample deconvolution relies on genotype/SNP demultiplexing (e.g. genetic hashing)
  rather than antibody-derived hashtag oligos; this skill is specific to HTO-count-based
  Seurat demultiplexing.
- The cells have already been demultiplexed and a sample-assignment table exists;
  re-running this step would duplicate the pipeline's doublet/negative removal.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3361
tools:
- name: gExcite
  role: Start-to-end Snakemake workflow that performs hashtag normalization, demultiplexing
    of cells to sample assignments, and removal of doublets and negatives
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Seurat
  role: R framework in which HTO normalization and HTODemux-style thresholding on
    normalized hashtag counts are run to classify singlets, doublets, and negatives
- name: CITE-seq-Count
  role: Upstream tool that produces the hashtag count matrix consumed by the demultiplexing
    step
- name: Snakemake
  role: Workflow manager embedding the demultiplexing step so it runs reproducibly
    and scalably as part of gExcite
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:cb8971f52f56c7ae2eb4db57358048f495fc68fa250f5fe6ca86fecbae10e6ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - CITE-seq-Count
  - Seurat
  - Snakemake
  - gExcite
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hashtag-demultiplexing-cell-assignment

## Summary

Assigns individual cells from a multiplexed, cell-hashed experiment back to their source samples using Seurat-based hashtag oligo (HTO) normalization and HTODemux-style demultiplexing, flagging doublets and negatives. In gExcite this step turns the CITE-seq-Count hashtag count matrix into a per-cell sample-assignment table.

## When to use

Use when input contains a cell-hashing (hashtag oligo) ADT count matrix — e.g. CITE-seq-Count output for multiple hashed samples — and each cell barcode must be assigned to a sample, a doublet, or a negative before downstream single-cell analysis.

## When NOT to use

- The input is gene expression (GEX) data only with no hashtag/ADT oligo counts — there is no HTO signal to threshold, so demultiplexing cannot be performed.
- Sample deconvolution relies on genotype/SNP demultiplexing (e.g. genetic hashing) rather than antibody-derived hashtag oligos; this skill is specific to HTO-count-based Seurat demultiplexing.
- The cells have already been demultiplexed and a sample-assignment table exists; re-running this step would duplicate the pipeline's doublet/negative removal.

## Inputs

- CITE-seq-Count hashtag (HTO/ADT) count matrix, cells x hashtags
- gExcite example test data: three hashed samples of human PBMC cells
- gExcite pipeline config.yaml configured for the hashed samples

## Outputs

- hashed_sample_assignments.csv — per-cell table mapping each cell barcode to its sample identity and demultiplexing call (singlet sample, doublet, or negative)

## How to apply

Load the hashtag count matrix produced by CITE-seq-Count (for gExcite's example run, three hashed human PBMC samples) into the Seurat framework. Normalize the hashtag-derived ADT counts per cell so HTO signal is comparable across cells, then run Seurat's HTODemux-style thresholding on the normalized hashtag counts to classify every cell barcode as a singlet assigned to one hashed sample, a doublet, or a negative. Note that the number of PCA dimensions used on the ADT/HTO data cannot exceed the number of ADTs in the experiment, otherwise the script fails. Assemble the resulting calls into a per-cell assignment table mapping each cell barcode to its sample identity and demultiplexing call, and write it out as hashed_sample_assignments.csv. The step is embedded in gExcite's Snakemake workflow so normalization, demultiplexing, and doublet/negative removal run reproducibly as part of the pipeline.

## Related tools

- **gExcite** (Start-to-end Snakemake workflow that performs hashtag normalization, demultiplexing of cells to sample assignments, and removal of doublets and negatives) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Seurat** (R framework in which HTO normalization and HTODemux-style thresholding on normalized hashtag counts are run to classify singlets, doublets, and negatives)
- **CITE-seq-Count** (Upstream tool that produces the hashtag count matrix consumed by the demultiplexing step)
- **Snakemake** (Workflow manager embedding the demultiplexing step so it runs reproducibly and scalably as part of gExcite) — https://github.com/ETH-NEXUS/gExcite_pipeline

## Examples

```
Rscript -e 'hto <- Read10X(...); obj <- CreateSeuratObject(...); obj <- NormalizeData(obj, assay="HTO"); obj <- HTODemux(obj)'  # writes hashed_sample_assignments.csv via the gExcite demultiplexing rule
```

## Evaluation signals

- hashed_sample_assignments.csv exists and contains one row per cell barcode with both a sample identity and a demultiplexing call (singlet/doublet/negative).
- Every cell barcode falls into exactly one of the three call classes: a singlet assigned to one of the hashed samples, a doublet, or a negative — no unclassified barcodes remain.
- Singlet assignments are distributed across the three hashed PBMC samples (no sample receives zero or nearly all singlets for the gExcite example test data).
- The ADT/HTO PCA dimensionality used is <= the number of hashtag ADTs in the experiment, since exceeding it makes the script fail.
- The step runs within the Snakemake rulegraph of gExcite, downstream of the CITE-Seq counting steps, confirming inputs are the post-counting HTO matrix rather than raw FASTQs.

## Limitations

- The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails.
- Other Unix systems including MacOS are not supported — the workflow is compatible with and tested on Linux only.
- Starting the pipeline after the resource-intensive cellranger count and CITE-Seq steps requires dummy files in place of the FASTQ files, so input setup must match the expected folder structure.

## Evidence

- [methods] gExcite performs normalization, demultiplexing of cells to sample assignments, and removal of doublets and negatives using the Seurat framework.: [evidence span withheld — anchor status: not_found]
- [abstract] we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution"
- [readme] example data for a test run ... with three hashed samples of human PBMC cells, so that hashing deconvolution, GEX analysis and ADT analysis can be performed: "We provide [example data for a test run](https://drive.google.com/drive/folders/14clt2_E_P0-HEXlJwH1fHCk5KhpPpxMc?usp=share_link) with three hashed samples of human PBMC cells, so that hashing"
- [discussion] The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails.: "The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails."
