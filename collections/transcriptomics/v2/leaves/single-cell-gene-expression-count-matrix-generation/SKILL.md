---
name: single-cell-gene-expression-count-matrix-generation
description: Use when starting from raw 10x-style GEX and/or ADT FASTQ files (I1/R1/R2,
  SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz) and per-sample gene or antibody
  count matrices are needed before downstream analysis.
when_to_use_negative:
- Input is already Cell Ranger output (feature-barcode matrices) and the resource-intensive
  cellranger count steps can be skipped — gExcite's quick test run starts only after
  these steps
- Analysis targets only scRNA without antibody data and STARsolo is preferred as the
  open-source alternative to Cellranger (supported by scAmpi rules but not the gExcite
  GEX/ADT flow described here)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger
  role: Performs cellranger count on GEX libraries against the reference transcriptome
    and on ADT libraries against the feature-barcode reference to produce per-sample
    UMI count matrices
- name: Snakemake
  role: Workflow manager that drives the cellranger count stage (with --use-conda)
    over all samples in gExcite
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: gExcite
  role: Start-to-end Snakemake-embedded pipeline that organizes GEX and ADT FASTQ
    inputs, config, and the read-mapping stage
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:6c0f2e97cb6c46f031f175e096d34b92cda719c5c9ed9da09fc69bd8bdcab0bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Cell Ranger
  - Snakemake
  - gExcite
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-gene-expression-count-matrix-generation

## Summary

Maps raw single-cell GEX and ADT (CITE-seq) FASTQ libraries with Cell Ranger count, run independently per library via the gExcite Snakemake workflow, to infer gene-by-cell and antibody-by-cell UMI count matrices per sample.

## When to use

Use when starting from raw 10x-style GEX and/or ADT FASTQ files (I1/R1/R2, SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz) and per-sample gene or antibody count matrices are needed before downstream analysis.

## When NOT to use

- Input is already Cell Ranger output (feature-barcode matrices) and the resource-intensive cellranger count steps can be skipped — gExcite's quick test run starts only after these steps
- Analysis targets only scRNA without antibody data and STARsolo is preferred as the open-source alternative to Cellranger (supported by scAmpi rules but not the gExcite GEX/ADT flow described here)

## Inputs

- Per-sample GEX FASTQ files (I1, R1, R2) under input_fastqs_gex/SAMPLENAME following the SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming schema
- Per-sample ADT (CITE-seq) FASTQ files (I1, R1, R2) under input_fastqs_adt/SAMPLENAME
- Cell Ranger reference transcriptome for GEX alignment
- Feature-barcode reference for ADT libraries
- config.yaml with input paths specified relative to the working directory

## Outputs

- Per-sample gene x cell UMI count matrices (Cell Ranger count feature-barcode output directories for GEX)
- Per-sample antibody x cell UMI count matrices (Cell Ranger count feature-barcode output directories for ADT)

## How to apply

Organize each sample's GEX FASTQs under input_fastqs_gex/SAMPLENAME and ADT FASTQs under input_fastqs_adt/SAMPLENAME, following the SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming schema with I1, R1 and R2 files. Configure the pipeline's config.yaml so FASTQ input paths are specified relative to the working directory (mandatory per gExcite), and install cellranger manually with its binary on the PATH since it is not conda-installable. Drive Cell Ranger count on each GEX library via Snakemake (--use-conda), aligning reads against the reference transcriptome to generate a gene x cell UMI count matrix per sample; run Cell Ranger count on each ADT library against the feature-barcode reference to generate an antibody x cell UMI count matrix per sample. If GEX and ADT were sequenced together on a NovaSeq, optionally apply the provided index-hopping removal scripts before the run. The quick test run in gExcite starts only after these resource-intensive cellranger count and CITE-Seq steps, so their completion is the checkpoint for this skill.

## Related tools

- **Cell Ranger** (Performs cellranger count on GEX libraries against the reference transcriptome and on ADT libraries against the feature-barcode reference to produce per-sample UMI count matrices)
- **Snakemake** (Workflow manager that drives the cellranger count stage (with --use-conda) over all samples in gExcite) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **gExcite** (Start-to-end Snakemake-embedded pipeline that organizes GEX and ADT FASTQ inputs, config, and the read-mapping stage) — https://github.com/ETH-NEXUS/gExcite_pipeline

## Examples

```
snakemake --use-conda --cores 1 -p -s workflow/Snakefile --configfile config/config.yaml
```

## Evaluation signals

- Cell Ranger count output directories containing feature-barcode count matrices exist for all samples' GEX and ADT libraries (file_exists check)
- Input FASTQs conform to SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz with I1, R1, R2 files and one sub-directory per sample
- config.yaml input paths are specified relative to the working directory (mandatory, not merely recommended)
- Both GEX and ADT libraries were processed independently, yielding separate gene-per-cell and antibody-per-cell matrices
- SAMPLENAME contains only [a-zA-Z0-9_-]+ characters; a dot in SAMPLENAME causes cellranger to stop

## Limitations

- Cell Ranger must be installed manually and its binary added to the PATH — it cannot be installed via Snakemake's --use-conda
- Specifying input FASTQ paths relative to the working directory is mandatory, not merely recommended
- SAMPLENAME can only contain [a-zA-Z0-9_-]+; if the sample name contains a dot, cellranger stops
- If GEX and ADT were sequenced together on a NovaSeq, index-hopping cleanup scripts should be applied before the run

## Evidence

- [results] During read mapping, gExcite processes the GEX and ADT libraries independently to infer read counts per gene per cell and read counts per antibody per cell, respectively.: [evidence span withheld — anchor status: not_found]
- [readme] The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema"
- [readme] Follow the instructions on the 10xGenomics installation support page to install cellranger and to include the cellranger binary to your path.: "Follow the instructions on the 10xGenomics installation support page to install cellranger and to include the cellranger binary to your path."
- [discussion] specifying the input relative to the working directory is now mandatory, not merely recommended: "specifying the input relative to the working directory is now mandatory, not merely recommended"
- [methods] starts after the resource-intensive cellranger count and CITE-Seq steps: "starts after the resource-intensive cellranger count and CITE-Seq steps"
