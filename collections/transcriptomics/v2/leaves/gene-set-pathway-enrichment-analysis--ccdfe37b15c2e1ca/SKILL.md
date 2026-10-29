---
name: gene-set-pathway-enrichment-analysis
description: Use when a single-cell GEX matrix has already been QC-filtered, clustered,
  and cell-type classified within the gExcite/scAmpi workflow, and differentially
  expressed genes per cluster must be assigned biological pathway context.
when_to_use_negative:
- The input is ADT/CITE-seq or hashing data only, with no gene expression counts —
  gExcite routes those samples through the ADT analysis and hashing deconvolution
  branches instead
- Clusters and differentially expressed genes have not yet been computed — pathway
  enrichment in this pipeline depends on the completed scAmpi_basic QC, clustering,
  and DE analysis steps
- The workflow cannot run on the target system — gExcite is compatible with and tested
  on Linux only; other Unix systems including MacOS are not supported
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: gExcite
  role: Start-to-end Snakemake workflow whose GEX analysis layer hosts the pathway
    enrichment step
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: scAmpi
  role: Module embedded into gExcite that performs QC, clustering, cell type assignment,
    and downstream analysis on the GEX matrix
  repo: https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- name: Snakemake
  role: Workflow manager that orchestrates the GEX rules and module inclusion; requires
    >=6.12.1
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: snakedeploy
  role: Deploys the gExcite workflow into the working directory before the GEX/pathway
    rules can run
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:6837c71542e416538ab93c1957dff18796f0a17020a3f572c49753051fc8b118
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Mamba
  - Snakemake
  - gExcite
  - scAmpi
  - snakedeploy
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gene-set-pathway-enrichment-analysis

## Summary

Functional interpretation step of gExcite's GEX analysis layer, in which the embedded scAmpi Snakemake module runs pathway enrichment on the gene expression matrix after QC, normalisation, clustering, cell type classification, and DE analysis.

## When to use

Use when a single-cell GEX matrix has already been QC-filtered, clustered, and cell-type classified within the gExcite/scAmpi workflow, and differentially expressed genes per cluster must be assigned biological pathway context.

## When NOT to use

- The input is ADT/CITE-seq or hashing data only, with no gene expression counts — gExcite routes those samples through the ADT analysis and hashing deconvolution branches instead
- Clusters and differentially expressed genes have not yet been computed — pathway enrichment in this pipeline depends on the completed scAmpi_basic QC, clustering, and DE analysis steps
- The workflow cannot run on the target system — gExcite is compatible with and tested on Linux only; other Unix systems including MacOS are not supported

## Inputs

- GEX count matrix from cellranger count (or STARsolo) for each sample, derived from FASTQs in the structure /path/to/input_fastqs_gex/SAMPLENAME/SAMPLENAME_S[Number]_L00[Lane]_[Read Type]_001.fastq.gz
- Tab-delimited sample_map listing all samples with a 'sample' header column
- config.yaml with inputOutput and resources sections adapted to the disease and reference genome

## Outputs

- Pathway enrichment results per cluster based on the differentially expressed genes from the scAmpi GEX analysis
- Downstream scAmpi GEX outputs (QC-filtered, normalised, clustered, cell-type-assigned matrices) for all samples

## How to apply

Deploy the gExcite pipeline from github.com/ETH-NEXUS/gExcite_pipeline via snakedeploy (conda environment with Mamba, Snakemake >=6.12.1 and snakedeploy), then run the embedded scAmpi module with --use-conda so per-rule conda environments resolve dependencies automatically. The GEX branch chains scAmpi_basic steps — mapping (cellranger or STARsolo), QC that removes empty droplets and ambient RNA, normalisation, unsupervised clustering, cell type classification, and DE analysis — and pathway enrichment consumes the resulting differentially expressed genes. Before running, adapt config.yaml: the inputOutput section needs the full path to input FASTQs, the resources section supplies inputs for cell type classification and the cellranger genomic reference, and disease-dependent config entries must be taken from the scAmpi_single_cell_RNA repository. Validate with a Snakemake dry-run (snakemake --use-conda --printshellcmds --dry-run) to check the rulegraph, then execute the full run so pathway enrichment outputs are produced for all samples.

## Related tools

- **gExcite** (Start-to-end Snakemake workflow whose GEX analysis layer hosts the pathway enrichment step) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **scAmpi** (Module embedded into gExcite that performs QC, clustering, cell type assignment, and downstream analysis on the GEX matrix) — https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- **Snakemake** (Workflow manager that orchestrates the GEX rules and module inclusion; requires >=6.12.1) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **snakedeploy** (Deploys the gExcite workflow into the working directory before the GEX/pathway rules can run) — https://github.com/ETH-NEXUS/gExcite_pipeline

## Examples

```
snakemake --use-conda --printshellcmds --cores 1
```

## Evaluation signals

- A Snakemake dry-run (snakemake --use-conda --printshellcmds --dry-run) resolves the full rulegraph including the scAmpi GEX rules without configuration errors
- The full run completes without failed rules on the three hashed human PBMC test samples
- scAmpi GEX processing outputs are produced for all three samples, confirming the module embedding works end to end
- Pathway enrichment is generated only after upstream scAmpi_basic outputs (QC, normalisation, clusters, cell type labels, DE genes) exist for each sample
- Per-rule conda environments install successfully under --use-conda, matching the envs/{rule_name}.yaml specifications

## Limitations

- All scAmpi documentation, especially config file entries that must be adapted depending on the disease, is only in the scAmpi_single_cell_RNA repository, not the gExcite README
- The scAmpi clinical part (disease-relevant drug target search on DE genes) is only applied if at least one cluster is indicated as a diseased ('malignant') cell type
- Some scAmpi steps perform online queries, so the computing system must permit internet/proxy access; snakedeploy deployment likewise needs internet access (local module setup only from Snakemake 7.13)
- Running from raw data with cellranger requires manual installation of cellranger (or STARsolo) since neither is available via conda

## Evidence

- [readme] scAmpi, a workflow that provides basic scRNA processing steps, is included as a module into gExcite.: "scAmpi, a workflow that provides basic scRNA processing steps, is included as a module into gExcite."
- [readme] scAmpi_basic includes general scRNA processing steps, such as mapping, QC, normalisation, unsupervised clustering, cell type classification, and DE analysis.: [evidence span withheld — anchor status: external_doc]
- [abstract] we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution"
- [readme] Note that all documentation regarding scAmpi (especially regarding config file entries that must be adapted depending on the disease) can only be found in the: "Note that all documentation regarding scAmpi (especially regarding config file entries that must be adapted depending on the disease) can only be found in the"
