---
name: cite-seq-hashtag-count-normalization
description: 'Use when cell-hashing CITE-seq data must be demultiplexed: a hashtag
  count matrix per cell barcode exists and each cell needs a sample identity call
  plus doublet/negative classification before downstream GEX/ADT analysis.'
when_to_use_negative:
- Input is GEX-only scRNA-seq without hashtag oligo counts — there is nothing to deconvolve;
  use the scAmpi_basic gene-expression module instead.
- Each cell barcode already has a known sample identity (no multiplexing), so demultiplexing
  would be a no-op.
- Data are still raw FASTQs that have not been through cellranger count and CITE-seq
  counting — run those upstream steps (or the gExcite quick test run with dummy FASTQ
  files) first.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: gExcite
  role: Start-to-end Snakemake workflow that embeds the hashtag normalization, demultiplexing,
    and doublet/negative removal steps
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Seurat
  role: R framework used to normalize per-cell hashtag counts and run HTODemux-style
    demultiplexing into singlet/doublet/negative calls
- name: CITE-seq-Count
  role: Upstream tool that produces the hashtag count matrix consumed by the normalization/demultiplexing
    step
- name: Snakemake
  role: Workflow manager that orchestrates the demultiplexing rules reproducibly and
    scalably
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:ea651cdd231e5ec69f0adbf3952eaaeec04d9979b9bb75d5c4c42454ff79625e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - CITE-seq-Count
  - Seurat
  - Snakemake
  - gExcite
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cite-seq-hashtag-count-normalization

## Summary

Demultiplexes cell-hashing (hashtag oligo) data inside the gExcite Snakemake pipeline: hashtag counts produced by CITE-seq-Count are normalized per cell in the Seurat framework and used to classify every cell barcode as a singlet, doublet, or negative, yielding a per-cell sample assignment table.

## When to use

Use when cell-hashing CITE-seq data must be demultiplexed: a hashtag count matrix per cell barcode exists and each cell needs a sample identity call plus doublet/negative classification before downstream GEX/ADT analysis.

## When NOT to use

- Input is GEX-only scRNA-seq without hashtag oligo counts — there is nothing to deconvolve; use the scAmpi_basic gene-expression module instead.
- Each cell barcode already has a known sample identity (no multiplexing), so demultiplexing would be a no-op.
- Data are still raw FASTQs that have not been through cellranger count and CITE-seq counting — run those upstream steps (or the gExcite quick test run with dummy FASTQ files) first.

## Inputs

- CITE-seq-Count hashtag (HTO) count matrix: hashtag oligo counts per cell barcode
- gExcite config.yaml configured for the hashed samples
- gExcite example test data: three hashed samples of human PBMC cells
- Cell barcodes from the upstream cellranger count / CITE-seq processing steps

## Outputs

- hashed_sample_assignments.csv: per-cell table mapping each cell barcode to its sample identity and demultiplexing call (singlet/doublet/negative)
- Normalized hashtag-derived ADT assay within the Seurat object

## How to apply

Load the CITE-seq-Count hashtag count matrix for the hashed samples (e.g. the three hashed human PBMC samples shipped as gExcite example test data) into a Seurat object as the hashtag-derived ADT assay. Normalize the hashtag oligo counts per cell so that HTO/ADT counts are comparable across cells within the Seurat framework. Run Seurat-based hashtag demultiplexing with HTODemux-style thresholding on the normalized hashtag counts to classify each cell barcode as a singlet assigned to a specific hashed sample, a doublet, or a negative. Assemble a per-cell assignment table mapping every barcode to its sample identity and demultiplexing call, and write it out as hashed_sample_assignments.csv. In gExcite this step is embedded in the Snakemake workflow and typically follows the resource-intensive cellranger count and CITE-seq counting steps.

## Related tools

- **gExcite** (Start-to-end Snakemake workflow that embeds the hashtag normalization, demultiplexing, and doublet/negative removal steps) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Seurat** (R framework used to normalize per-cell hashtag counts and run HTODemux-style demultiplexing into singlet/doublet/negative calls)
- **CITE-seq-Count** (Upstream tool that produces the hashtag count matrix consumed by the normalization/demultiplexing step)
- **Snakemake** (Workflow manager that orchestrates the demultiplexing rules reproducibly and scalably)

## Examples

```
snakedeploy deploy-workflow https://github.com/ETH-NEXUS/gExcite_pipeline --tag main . && snakemake --use-conda --configfile config/config.yaml -j 1 -p
```

## Evaluation signals

- hashed_sample_assignments.csv is produced and contains an assignment call for every input cell barcode.
- Every barcode carries exactly one call among {singlet, doublet, negative}; singlets are assigned only to the hashed sample identities.
- Hashtag-derived ADT counts are normalized per cell before thresholding, so HTO-derived values are comparable across cells.
- Running the gExcite quick test on the three-hashed-PBMC example data reproduces demultiplexing output without rerunning cellranger.
- Sum of singlets + doublets + negatives equals the number of cells in the CITE-seq-Count HTO matrix.

## Limitations

- gExcite is compatible with and tested on Linux only; other Unix systems including MacOS are currently not supported.
- Starting after the resource-intensive cellranger count and CITE-Seq steps (e.g. the quick test run) requires dummy files in place of the FASTQ files.
- In the broader ADT analysis of the same pipeline, the number of PCA dimensions cannot be larger than the number of ADTs in the experiment, otherwise the script fails.

## Evidence

- [other] After hashtag counting with CITE-seq-Count, gExcite performs normalization, demultiplexing of cells to sample assignments, and removal of doublets and negatives using the Seurat framework.: [evidence span withheld — anchor status: not_found]
- [other] Seurat-based hashtag demultiplexing with HTODemux-style thresholding and hashed_sample_assignments.csv output.: [evidence span withheld — anchor status: not_found]
- [abstract] gExcite embedded in Snakemake provides hashing deconvolution.: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution. Embedded in the Snakemake workflow manager, gExcite"
- [methods] Pipeline starts after the resource-intensive cellranger count and CITE-Seq steps.: "starts after the resource-intensive cellranger count and CITE-Seq steps"
- [readme] README: quick test run on three hashed PBMC samples enables hashing deconvolution.: "A quick test run on the example data can be performed that starts after the resource-intensive cellranger count and CITE-Seq steps."
