---
name: single-cell-qc-cell-filtering
description: Use when running gExcite (or its embedded scAmpi module) on hashed 10x
  PBMC samples where cell-level QC from the gene-expression layer must gate downstream
  CITE-seq/ADT analysis and antibody-positive cell calls.
when_to_use_negative:
- The data are bulk RNA-seq or already-aggregated feature tables rather than per-cell
  10x GEX/ADT output from hashed samples
- The task is hashing deconvolution or index-hopping removal rather than cell-level
  QC filtering
- The workflow must run on MacOS or other Unix systems, which gExcite does not support
  (Linux only)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: gExcite
  role: Start-to-end Snakemake workflow whose GEX QC output gates the CITE-seq/ADT
    cell filtering
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: scAmpi
  role: Embedded module providing the basic scRNA processing steps (mapping, QC, normalisation,
    clustering) used for cell filtering
  repo: https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- name: cellranger
  role: Upstream 10x read mapping and counting whose outputs feed the QC step; binary
    must be installed manually and added to PATH
- name: Snakemake
  role: Workflow manager used to execute the QC and filtering steps with --use-conda
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:55b2b0d51a7cf066d822a197853779055bf52f9880bf836d6da6ac16f7a6cb17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Snakemake
  - cellranger
  - gExcite
  - scAmpi
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-qc-cell-filtering

## Summary

Quality control and cell filtering within the gExcite Snakemake workflow, where gene-expression (GEX) QC determines which cells are retained and the ADT/CITE-seq layer is restricted to those same cells before UMAP embedding and joint BremSC clustering.

## When to use

Use when running gExcite (or its embedded scAmpi module) on hashed 10x PBMC samples where cell-level QC from the gene-expression layer must gate downstream CITE-seq/ADT analysis and antibody-positive cell calls.

## When NOT to use

- The data are bulk RNA-seq or already-aggregated feature tables rather than per-cell 10x GEX/ADT output from hashed samples
- The task is hashing deconvolution or index-hopping removal rather than cell-level QC filtering
- The workflow must run on MacOS or other Unix systems, which gExcite does not support (Linux only)

## Inputs

- GEX FASTQ files per sample in cellranger folder structure (SAMPLENAME_S[N]_L00[Lane]_[Read Type]_001.fastq.gz)
- ADT FASTQ files per sample in the same naming schema (input_fastqs_adt)
- pipeline config.yaml adapted to the data (including scAmpi module entries)
- cellranger count and CITE-Seq step outputs (or dummy files for the quick test run)

## Outputs

- GEX-QC-retained cell set that also gates the ADT layer
- UMAP embeddings computed from gene expression, antibody expression, or both
- BremSC joint clustering cluster labels
- antibody-positive cell calls per antibody via background thresholds
- ridge plots per antibody

## How to apply

Deploy gExcite from ETH-NEXUS/gExcite_pipeline via snakedeploy inside a conda environment with Mamba and Snakemake (>=6.12.1), then adapt the config.yaml in the config directory, including scAmpi module entries that control the basic scRNA processing steps (mapping, QC, normalisation, unsupervised clustering). QC is carried out by the embedded scAmpi_basic module; only cells retained by GEX QC are carried forward, and the CITE-seq layer filters ADT data to exactly those retained cells so both modalities describe the same cell set. When computing UMAP embeddings from gene expression, antibody expression, or their combination, keep the number of PCA dimensions for ADT-based UMAP at or below the number of ADTs in the experiment, otherwise the script fails. After QC, run joint gene-plus-antibody (BremSC) clustering and apply per-antibody background thresholds to call antibody-positive cells.

## Related tools

- **gExcite** (Start-to-end Snakemake workflow whose GEX QC output gates the CITE-seq/ADT cell filtering) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **scAmpi** (Embedded module providing the basic scRNA processing steps (mapping, QC, normalisation, clustering) used for cell filtering) — https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- **cellranger** (Upstream 10x read mapping and counting whose outputs feed the QC step; binary must be installed manually and added to PATH)
- **Snakemake** (Workflow manager used to execute the QC and filtering steps with --use-conda) — https://github.com/ETH-NEXUS/gExcite_pipeline

## Examples

```
snakemake --use-conda --printshellcmds --cores 1
```

## Evaluation signals

- The quick test run completes through the ADT analysis steps without errors on the three hashed human PBMC example samples
- The ADT analysis operates on exactly the cells retained by GEX QC (no cell present in ADT outputs that was dropped by GEX QC)
- Number of PCA dimensions used for ADT-based UMAP does not exceed the number of ADTs in the experiment, so the script does not fail
- CITE-seq outputs are emitted: embeddings, cluster labels, antibody-positive cell calls, and ridge plots
- Per-antibody background thresholds produce a binary positive/negative call per cell per antibody

## Limitations

- The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails
- The workflow is compatible with and tested on Linux only; other Unix systems including MacOS are currently not supported
- Specifying FASTQ inputs relative to the working directory is mandatory, not merely recommended
- Cellranger cannot be installed via conda and must be installed manually with its binary on the PATH

## Evidence

- [readme] scAmpi_basic includes general scRNA processing steps, such as mapping, QC, normalisation, unsupervised clustering, cell type classification, and DE analysis.: [evidence span withheld — anchor status: external_doc]
- [discussion] The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails.: "The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails."
- [methods] provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution: "provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution"
- [readme] scAmpi, a workflow that provides basic scRNA processing steps, is included as a module into gExcite.: "scAmpi, a workflow that provides basic scRNA processing steps, is included as a module into gExcite."
