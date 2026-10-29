---
name: cite-seq-adt-count-processing
description: Use when raw ADT/CITE-seq FASTQ files (I1/R1/R2) from hashed samples
  need to be converted into antibody-by-cell UMI count matrices, optionally after
  removing index-hopped reads from combined NovaSeq GEX+ADT data.
when_to_use_negative:
- Data are gene-expression-only with no ADT/CITE-seq library — only the GEX branch
  of the pipeline is needed
- Count matrices already exist (pipeline start configured after the resource-intensive
  cellranger count and CITE-Seq steps — only dummy FASTQ files are then required)
- GEX and ADT were sequenced separately, not combined on one NovaSeq run — the index-hopping
  removal step does not apply
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: Cell Ranger
  role: Aligns ADT (CITE-seq) library reads against the feature-barcode reference
    to generate antibody-by-cell UMI count matrices per sample; must be installed
    manually and added to PATH
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation
- name: gExcite
  role: Snakemake workflow embedding the ADT processing stage, including the index-hopping
    removal scripts (workflow/scripts/index_hopping_removal) for combined NovaSeq
    GEX+ADT data
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Drives the cellranger count and CITE-Seq rules reproducibly; run with --use-conda
    for automated per-rule environments
- name: DropletUtils
  role: Package used by the gExcite index-hopping removal scripts to identify reads
    misassigned between hashed samples and yield a cleaned count matrix
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:b5d3623b59a8c81074d1809ea141caf17d6d69d5759fd5df36cfdbb4fa3618e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell Ranger
  - DropletUtils
  - Snakemake
  - gExcite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cite-seq-adt-count-processing

## Summary

Processing of CITE-seq (antibody-derived tag, ADT) libraries in the gExcite Snakemake workflow: per-sample ADT FASTQs are mapped with Cell Ranger count against a feature-barcode reference to yield antibody-by-cell UMI count matrices, with optional index-hopping cleanup (DropletUtils) when GEX and ADT were sequenced together on a NovaSeq run.

## When to use

Use when raw ADT/CITE-seq FASTQ files (I1/R1/R2) from hashed samples need to be converted into antibody-by-cell UMI count matrices, optionally after removing index-hopped reads from combined NovaSeq GEX+ADT data.

## When NOT to use

- Data are gene-expression-only with no ADT/CITE-seq library — only the GEX branch of the pipeline is needed
- Count matrices already exist (pipeline start configured after the resource-intensive cellranger count and CITE-Seq steps — only dummy FASTQ files are then required)
- GEX and ADT were sequenced separately, not combined on one NovaSeq run — the index-hopping removal step does not apply

## Inputs

- ADT (CITE-seq) FASTQ files per sample: SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz with I1, R1, R2 reads
- Feature-barcode reference for the antibody panel
- pipeline config.yaml with FASTQ paths relative to the working directory
- Unfiltered GEX and ADT count data from the combined NovaSeq sequencing run (for optional index-hopping removal)

## Outputs

- Antibody-by-cell UMI count matrices per sample (feature-barcode count matrices from Cell Ranger count output directories)
- index_hopping_cleaned_count_matrix.tsv (cleaned count matrix produced by the index-hopping removal step, when applied)

## How to apply

Arrange each sample's ADT FASTQs under input_fastqs_adt/SAMPLENAME following the SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming schema with I1, R1 and R2 files. Configure config.yaml so FASTQ input paths are specified relative to the working directory (mandatory). Then let the Snakemake pipeline (--use-conda) drive Cell Ranger count on each ADT library against the feature-barcode reference, producing antibody-by-cell UMI count matrices per sample; this runs in parallel with the GEX libraries being mapped against the transcriptome reference. Optionally, before the main run, execute the gExcite index-hopping removal scripts (workflow/scripts/index_hopping_removal) on the unfiltered GEX and ADT count data of the combined NovaSeq run to identify and reassign reads misassigned between hashed samples; the cleanup is applied only in the combined GEX+ADT NovaSeq sequencing scenario. Run cellranger from a manually installed binary in PATH since it cannot be installed via conda.

## Related tools

- **Cell Ranger** (Aligns ADT (CITE-seq) library reads against the feature-barcode reference to generate antibody-by-cell UMI count matrices per sample; must be installed manually and added to PATH) — https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation
- **gExcite** (Snakemake workflow embedding the ADT processing stage, including the index-hopping removal scripts (workflow/scripts/index_hopping_removal) for combined NovaSeq GEX+ADT data) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Drives the cellranger count and CITE-Seq rules reproducibly; run with --use-conda for automated per-rule environments)
- **DropletUtils** (Package used by the gExcite index-hopping removal scripts to identify reads misassigned between hashed samples and yield a cleaned count matrix)

## Examples

```
snakedeploy deploy-workflow https://github.com/ETH-NEXUS/gExcite_pipeline --tag main .
```

## Evaluation signals

- Cell Ranger count output directories containing feature-barcode count matrices exist for all samples' ADT libraries (file_exists check)
- If index-hopping removal was applied: index_hopping_cleaned_count_matrix.tsv exists and is readable
- A dry run of the Snakemake workflow completes without errors, confirming the pre-processed data is accepted by the downstream pipeline
- SAMPLENAME folders and file names conform to the naming schema, and SAMPLENAME uses only [a-zA-Z0-9_-]+ characters (cellranger stops otherwise)

## Limitations

- Cell Ranger (and STARsolo, if chosen) cannot be installed via conda and must be installed manually and added to PATH
- SAMPLENAME can only contain [a-zA-Z0-9_-]+ characters; a dot in the name causes cellranger to stop
- Specifying FASTQ input paths relative to the working directory is mandatory, not merely recommended
- Index-hopping removal is an optional cleanup provided only for combined GEX and ADT NovaSeq sequencing data; no quantitative threshold is reported in the paper for this step

## Evidence

- [readme] cellranger count and CITE-Seq steps: "A quick test run on the example data can be performed that starts after the resource-intensive cellranger count and CITE-Seq steps."
- [methods] combined NovaSeq index hopping: "In case of combined GEX and ADT NovaSeq sequencing data, scripts are provided to clean up the data before a run"
- [readme] FASTQ folder structure: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema:"
- [readme] cellranger manual install: "install cellranger and to include the cellranger binary to your path"
