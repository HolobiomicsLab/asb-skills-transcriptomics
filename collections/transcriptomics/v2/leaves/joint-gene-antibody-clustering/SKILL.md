---
name: joint-gene-antibody-clustering
description: Use when a hashed PBMC-style CITE-seq dataset has passed GEX QC and cellranger/CITE-Seq
  preprocessing, and you need joint gene-plus-antibody clustering, UMAP embeddings
  per modality, and antibody-positive cell calls.
when_to_use_negative:
- The dataset contains only gene expression (no ADT/CITE-seq library), so no antibody
  modality exists to combine for joint clustering
- The goal is hashing deconvolution or upstream cellranger count / FASTQ processing
  rather than the CITE-seq analysis layer's clustering and positive-cell calling
- The ADT data has not been filtered to GEX-QC-retained cells, so joint analysis would
  mix mismatched cell sets
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: gExcite
  role: 'Workflow that implements the CITE-seq analysis layer: modality-filtered UMAP
    embedding, joint BremSC clustering, and per-antibody positive-cell thresholds'
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager embedding gExcite and orchestrating the CITE-seq analysis
    rules via --use-conda
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: scAmpi
  role: External workflow module included in gExcite that provides the upstream scRNA
    processing (mapping, QC, normalisation) whose QC-passed cells feed the ADT filtering
  repo: https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:7020360d4b796dfcb782f0605443cee7ca0d49abc64e92a6fc59994601228c5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Snakemake
  - gExcite
  - scAmpi
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# joint-gene-antibody-clustering

## Summary

CITE-seq analysis step of the gExcite Snakemake workflow that computes UMAP embeddings from gene expression, antibody (ADT) expression, or both modalities combined, and runs joint gene-plus-antibody (BremSC) clustering to assess cell similarity across data types.

## When to use

Use when a hashed PBMC-style CITE-seq dataset has passed GEX QC and cellranger/CITE-Seq preprocessing, and you need joint gene-plus-antibody clustering, UMAP embeddings per modality, and antibody-positive cell calls.

## When NOT to use

- The dataset contains only gene expression (no ADT/CITE-seq library), so no antibody modality exists to combine for joint clustering
- The goal is hashing deconvolution or upstream cellranger count / FASTQ processing rather than the CITE-seq analysis layer's clustering and positive-cell calling
- The ADT data has not been filtered to GEX-QC-retained cells, so joint analysis would mix mismatched cell sets

## Inputs

- ADT (antibody-derived tag) count matrix filtered to cells retained by GEX QC
- Gene expression (GEX) count matrix for the same cells
- gExcite pipeline config.yaml (config directory, adapted scAmpi module entries)
- ADT panel definition (number of ADTs in the experiment)

## Outputs

- UMAP embeddings computed from GEX alone, ADT alone, and gene-plus-antibody combined data
- Joint BremSC cluster labels for cells
- Per-antibody antibody-positive cell calls
- ADT ridge plots

## How to apply

In the CITE-seq layer of gExcite, first filter the ADT data to the cells retained by gene expression (GEX) QC so both modalities describe the same cells. Then compute UMAP embeddings from one of three inputs: gene expression alone, antibody expression alone, or the combination of both, ensuring the number of PCA dimensions used for the ADT-based UMAP does not exceed the number of ADTs in the experiment (otherwise the script fails). Run the joint gene-plus-antibody (BremSC) clustering on the combined data to obtain cluster labels spanning both modalities. Finally, apply per-antibody background thresholds to call antibody-positive cells per marker. Execute the surrounding workflow with Snakemake (--use-conda) since scAmpi is included as a module providing the upstream scRNA processing steps.

## Related tools

- **gExcite** (Workflow that implements the CITE-seq analysis layer: modality-filtered UMAP embedding, joint BremSC clustering, and per-antibody positive-cell thresholds) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager embedding gExcite and orchestrating the CITE-seq analysis rules via --use-conda) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **scAmpi** (External workflow module included in gExcite that provides the upstream scRNA processing (mapping, QC, normalisation) whose QC-passed cells feed the ADT filtering) — https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA

## Examples

```
snakemake --use-conda --printshellcmds --cores 1
```

## Evaluation signals

- The quick test run completes through the ADT analysis steps without errors and emits the CITE-seq analysis outputs (embeddings, cluster labels, positive-cell calls, ridge plots) for the three hashed PBMC samples
- The number of PCA dimensions used for the ADT-based UMAP does not exceed the number of ADTs in the experiment; if it does, the script fails
- UMAP embeddings are produced for all three input variants: gene expression alone, antibody expression alone, and their combination
- Cluster labels come from joint gene-plus-antibody (BremSC) clustering, not from GEX-only clustering
- Per-antibody positive-cell calls are generated using per-antibody background thresholds rather than a single global cutoff

## Limitations

- The number of PCA dimensions for ADT-based UMAP cannot be larger than the number of ADTs in the experiment, otherwise the script fails
- gExcite is compatible with and tested on Linux only; other Unix systems including MacOS are not supported
- ADT analysis is restricted to cells retained by GEX QC, so the CITE-seq layer depends on upstream GEX QC outcomes
- Requires Snakemake ≥ 6.12.1 and, when starting from raw FASTQ data, a manually installed cellranger binary in PATH

## Evidence

- [abstract] we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution"
- [discussion] PCA dimensions must not exceed number of ADTs: "The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails."
- [readme] README overview of the workflow: "gExcite is a start-to-end workflow embedded in Snakemake that provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution."
- [readme] scAmpi module dependency: "scAmpi, a workflow that provides basic scRNA processing steps, is included as a module into gExcite."
