---
name: single-cell-umap-dimensionality-reduction
description: Use when running gExcite's CITE-seq/ADT analysis layer on paired GEX+ADT
  (e.g. 10x hashed PBMC) data and you need cell-level similarity embeddings from one
  or both modalities, or joint BremSC clustering inputs.
when_to_use_negative:
- The analysis target is gene-expression-only scRNA processing — that is handled by
  the upstream scAmpi module (mapping, QC, normalisation, unsupervised clustering,
  cell type classification, DE analysis), not the CITE-seq UMAP layer.
- Cells have not passed GEX QC — ADT data must first be filtered to cells retained
  by GEX QC before computing cross-modality embeddings.
- You plan to set the PCA dimension count for ADT-based UMAP higher than the number
  of ADTs in the experiment — the script fails in that case, so the embedding must
  be configured within that bound.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: gExcite
  role: start-to-end Snakemake workflow whose CITE-seq analysis layer performs the
    UMAP embedding, BremSC clustering, and antibody-positive calling
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: workflow manager that executes the CITE-seq/UMAP rules, deploying per-rule
    conda environments via --use-conda
  repo: https://snakemake.readthedocs.io
- name: scAmpi
  role: included module providing the basic scRNA (GEX) processing steps — mapping,
    QC, normalisation — that precede the CITE-seq UMAP layer
  repo: https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- name: Cell Ranger (cellranger)
  role: resource-intensive upstream count step whose output the quick test run starts
    after; must be installed and on PATH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:58c32e48ce59c1d1a4230dd899a548b830a749028146f00cf1c24f90e082c162
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Cell Ranger (cellranger)
  - Snakemake
  - gExcite
  - scAmpi
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-umap-dimensionality-reduction

## Summary

Compute UMAP embeddings of cells within gExcite's CITE-seq analysis layer from gene expression, antibody-derived tag (ADT) expression, or the combination of both, to assess cell similarity across the two data modalities of the hashed PBMC workflow.

## When to use

Use when running gExcite's CITE-seq/ADT analysis layer on paired GEX+ADT (e.g. 10x hashed PBMC) data and you need cell-level similarity embeddings from one or both modalities, or joint BremSC clustering inputs.

## When NOT to use

- The analysis target is gene-expression-only scRNA processing — that is handled by the upstream scAmpi module (mapping, QC, normalisation, unsupervised clustering, cell type classification, DE analysis), not the CITE-seq UMAP layer.
- Cells have not passed GEX QC — ADT data must first be filtered to cells retained by GEX QC before computing cross-modality embeddings.
- You plan to set the PCA dimension count for ADT-based UMAP higher than the number of ADTs in the experiment — the script fails in that case, so the embedding must be configured within that bound.

## Inputs

- GEX and ADT FASTQ files per sample in the folder structure /path/to/input_fastqs_gex/SAMPLENAME/SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz (or dummy FASTQs for the quick test run starting after cellranger count and CITE-Seq steps)
- cellranger count and CITE-Seq preprocessing outputs for the hashed samples
- config.yaml pipeline configuration (including scAmpi module entries adapted from config/)

## Outputs

- UMAP embeddings computed from gene expression, antibody expression, or their combination
- BremSC joint gene-plus-antibody cluster labels
- antibody-positive cell calls from per-antibody background thresholds
- ridge plots of ADT expression

## How to apply

After GEX quality control, filter the ADT data to the cells retained by GEX QC so that both modalities describe the same cell set. Choose the UMAP input: gene expression alone, antibody expression alone, or the combination of both — the combination enables assessing cell similarity across data types. For ADT-based embeddings, the number of PCA dimensions must not exceed the number of ADTs in the experiment, otherwise the script fails. The resulting embeddings feed joint gene-plus-antibody (BremSC) clustering, followed by per-antibody background thresholds that call antibody-positive cells. The workflow runs under Snakemake with --use-conda on Linux, using scAmpi as an included module for the basic scRNA processing steps upstream of the CITE-seq layer.

## Related tools

- **gExcite** (start-to-end Snakemake workflow whose CITE-seq analysis layer performs the UMAP embedding, BremSC clustering, and antibody-positive calling) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (workflow manager that executes the CITE-seq/UMAP rules, deploying per-rule conda environments via --use-conda) — https://snakemake.readthedocs.io
- **scAmpi** (included module providing the basic scRNA (GEX) processing steps — mapping, QC, normalisation — that precede the CITE-seq UMAP layer) — https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- **Cell Ranger (cellranger)** (resource-intensive upstream count step whose output the quick test run starts after; must be installed and on PATH)

## Examples

```
snakemake --use-conda --printshellcmds --cores 1
```

## Evaluation signals

- The quick test run on the three hashed human PBMC example samples completes through the ADT analysis steps without errors.
- CITE-seq outputs are emitted: embeddings, cluster labels, positive-cell calls, and ridge plots for the three hashed PBMC samples.
- For ADT-based UMAP, the configured PCA dimension count is at most the number of ADTs in the experiment (script fails otherwise).
- The ADT matrix was filtered to exactly the cell set retained by GEX QC before embedding.
- Deployed workflow is executed with 'snakemake --use-conda --printshellcmds --cores 1' on Linux, per-rule conda environments resolve successfully.

## Limitations

- The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the ADT UMAP script fails.
- The workflow is compatible with and tested on Linux only; other Unix systems including MacOS are currently not supported.
- Running from raw data requires manually installed cellranger on the PATH (not conda-installable); the quick test run needs dummy FASTQ files to start after the cellranger count and CITE-Seq steps.
- Specifying the FASTQ input relative to the working directory is mandatory, not merely recommended.

## Evidence

- [abstract] gExcite provides dual-modality CITE-seq analysis: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution"
- [discussion] PCA dimension limit for ADT UMAP: "The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails."
- [methods] scAmpi module upstream of CITE-seq layer: "scAmpi, a workflow that provides basic scRNA processing steps, is included as a module into gExcite"
- [readme] Quick test run scope and example data: "A quick test run on the example data can be performed that starts after the resource-intensive cellranger count and CITE-Seq steps."
- [readme] Workflow scope embedded in Snakemake: "gExcite is a start-to-end workflow embedded in Snakemake that provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution."
