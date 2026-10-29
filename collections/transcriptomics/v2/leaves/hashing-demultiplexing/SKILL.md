---
name: hashing-demultiplexing
description: Use when analyzing hashed single-cell multiome data (e.g. cell hashing
  antibody oligo tags) that requires sample demultiplexing plus combined GEX and ADT/CITE-seq
  processing from raw FASTQ reads onward.
when_to_use_negative:
- The experiment has no cell-hashing antibodies — the hashing deconvolution stage
  is specific to hashed samples and cannot demultiplex unhashed captures
- Analysis starts from already-processed count matrices or a feature table rather
  than raw sequencing reads — gExcite's cellranger count stage expects raw FASTQ input
- The workflow must run on MacOS or other non-Linux Unix systems — the README states
  it is compatible with and tested on Linux only
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: gExcite
  role: Start-to-end Snakemake workflow performing hashing deconvolution plus gene
    and antibody (CITE-seq) expression analysis
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager embedding gExcite; required version >=6.12.1; module inclusion
    pulls in scAmpi; --use-conda installs per-rule environments
- name: snakedeploy
  role: Deploys the gExcite workflow from GitHub into the working directory
- name: Cell Ranger (cellranger)
  role: Runs cellranger count on raw GEX and ADT reads; must be installed manually
    and added to PATH
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation
- name: scAmpi
  role: 'Included as a Snakemake module providing basic scRNA processing: mapping,
    QC, normalisation, unsupervised clustering, cell type classification, DE analysis'
  repo: https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:2ab3242f9dbcca22f76fd3f8d83f4e5a40ec007046b198d570c295ec258b5f34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Cell Ranger (cellranger)
  - Mamba
  - Snakemake
  - gExcite
  - scAmpi
  - snakedeploy
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# hashing-demultiplexing

## Summary

Hash-sample deconvolution as implemented in the gExcite Snakemake workflow: demultiplexing hashed single-cell samples and combining gene expression (GEX) with CITE-seq/ADT analysis in one start-to-end pipeline. It addresses the gap where existing tools cover only GEX or antibody analysis in isolation and do not scale to many samples.

## When to use

Use when analyzing hashed single-cell multiome data (e.g. cell hashing antibody oligo tags) that requires sample demultiplexing plus combined GEX and ADT/CITE-seq processing from raw FASTQ reads onward.

## When NOT to use

- The experiment has no cell-hashing antibodies — the hashing deconvolution stage is specific to hashed samples and cannot demultiplex unhashed captures
- Analysis starts from already-processed count matrices or a feature table rather than raw sequencing reads — gExcite's cellranger count stage expects raw FASTQ input
- The workflow must run on MacOS or other non-Linux Unix systems — the README states it is compatible with and tested on Linux only

## Inputs

- Hashed-sample FASTQ files for GEX and ADT libraries, arranged as input_fastqs_gex/SAMPLENAME/ and input_fastqs_adt/SAMPLENAME/ following the SAMPLENAME_S[Number]_L00[LaneNumber]_[ReadType]_001.fastq.gz naming schema
- config.yaml adapted to the data (including scAmpi module config entries)
- cellranger reference genome configuration (resources section of the config)
- Example test data: three hashed samples of human PBMC cells (Google Drive bundle)

## Outputs

- Hashing deconvolution results (sample demultiplexing assignments per cell)
- GEX analysis outputs (from scAmpi_basic: mapping, QC, normalisation, unsupervised clustering, cell type classification, DE analysis)
- CITE-seq/ADT analysis outputs
- Snakemake rulegraph-tracked results directory with all documented test-run outputs

## How to apply

Deploy the gExcite pipeline with snakedeploy into a conda/Mamba environment containing Snakemake (>=6.12.1), then arrange hashed-sample FASTQs in input_fastqs_gex and input_fastqs_adt folders using the SAMPLENAME_S[Number]_L00[LaneNumber]_[ReadType]_001.fastq.gz schema with paths relative to the working directory (mandatory). Configure config.yaml per the config directory README, adapting the scAmpi module entries, and install cellranger manually with its binary on the PATH since it is not conda-installable. Run 'snakemake --use-conda --printshellcmds --cores 1'; the workflow executes cellranger count on raw reads, then hashing deconvolution, GEX analysis, and CITE-seq/ADT analysis, with scAmpi included as a Snakemake module for basic scRNA processing (QC, normalization, clustering, cell type classification). For NovaSeq combined GEX+ADT data, use the provided scripts to remove index hopping before the run. A quick test run on the bundled three hashed human PBMC samples skips the resource-intensive cellranger count and CITE-Seq steps by starting from dummy FASTQ files.

## Related tools

- **gExcite** (Start-to-end Snakemake workflow performing hashing deconvolution plus gene and antibody (CITE-seq) expression analysis) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager embedding gExcite; required version >=6.12.1; module inclusion pulls in scAmpi; --use-conda installs per-rule environments)
- **snakedeploy** (Deploys the gExcite workflow from GitHub into the working directory)
- **Cell Ranger (cellranger)** (Runs cellranger count on raw GEX and ADT reads; must be installed manually and added to PATH) — https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation
- **scAmpi** (Included as a Snakemake module providing basic scRNA processing: mapping, QC, normalisation, unsupervised clustering, cell type classification, DE analysis) — https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA

## Examples

```
snakemake --use-conda --printshellcmds --cores 1
```

## Evaluation signals

- All Snakemake rules complete without error when run with --use-conda --printshellcmds
- The documented test-run outputs for hashing deconvolution, GEX analysis, and ADT analysis are present in the results directory
- FASTQ paths resolve relative to the working directory (mandatory per the discussion notes) and sample names match the [a-zA-Z0-9_-]+ constraint expected by cellranger
- For the quick test run, dummy FASTQ files are in place of real FASTQs so the run starts after cellranger count and CITE-Seq steps
- ADT-based UMAP uses a number of PCA dimensions that does not exceed the number of ADTs in the experiment, otherwise the script fails

## Limitations

- cellranger cannot be installed via conda and must be installed manually with its binary on the PATH
- Snakemake needs internet access for the snakedeploy/module setup; local module setup is supported only from Snakemake 7.13
- The number of PCA dimensions cannot be larger than the number of ADTs in the experiment, otherwise the script fails
- Specifying FASTQ input relative to the working directory is mandatory, not merely recommended
- Sample names may only contain [a-zA-Z0-9_-]+ characters or cellranger will stop

## Evidence

- [abstract] we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution"
- [methods] gExcite is a start-to-end workflow embedded in Snakemake: "gExcite is a start-to-end workflow embedded in Snakemake"
- [methods] example data for a test run ... with three hashed samples of human PBMC cells: [evidence span withheld — anchor status: not_found]
- [readme] A quick test run on the example data can be performed that starts after the resource-intensive cellranger count and CITE-Seq steps.: "A quick test run on the example data can be performed that starts after the resource-intensive cellranger count and CITE-Seq steps."
- [readme] This workflow makes use of Snakemake's functionality to include external workflows as a module.: "This workflow makes use of Snakemake's functionality to include external workflows as a [module]"
