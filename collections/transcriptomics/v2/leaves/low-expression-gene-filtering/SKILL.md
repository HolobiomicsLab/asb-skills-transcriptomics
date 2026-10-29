---
name: low-expression-gene-filtering
description: Use when running the scdrake single-sample pipeline on a droplet scRNA-seq
  matrix (e.g. 10x Cell Ranger output like PBMC 1k) and the gene dimension needs trimming
  before normalization and HVG selection.
when_to_use_negative:
- Input is already a filtered gene expression table with low-abundance genes removed
  upstream
- Working in a downstream scdrake stage (e.g. normalization or clustering) where gene
  filtering was already completed in 01_input_qc
- Analyzing spot-based spatial transcriptomics data where gene-detection characteristics
  differ from droplet scRNA-seq without re-tuning min_ratio_cells/min_umi
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scdrake
  role: Pipeline implementing the 01_input_qc stage that contains the get_gene_filter
    gene-filtering target
  repo: https://github.com/bioinfocz/scdrake
- name: drake
  role: Workflow framework that tracks get_gene_filter as a target so only changed
    parts of the pipeline rerun
  repo: https://github.com/bioinfocz/scdrake
- name: R
  role: Language in which get_gene_filter and the pipeline are implemented
  repo: https://github.com/bioinfocz/scdrake
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad089
    title: 'Scdrake: a reproducible and scalable pipeline for scRNA-seq data analysis'
  source_content_hash: sha256:a10d77ca72a96504883a1dacc0e3b89759b660f1167a2552aba3e480fdc19508
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfocz/scdrake
  tools:
  - Docker
  - R
  - drake
  - future
  - scdrake
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# low-expression-gene-filtering

## Summary

Filters lowly expressed genes during the scdrake 01_input_qc stage using the get_gene_filter function, which applies minimum expression ratios and UMI-count criteria so that downstream normalization, HVG detection, and clustering operate only on adequately detected genes.

## When to use

Use when running the scdrake single-sample pipeline on a droplet scRNA-seq matrix (e.g. 10x Cell Ranger output like PBMC 1k) and the gene dimension needs trimming before normalization and HVG selection.

## When NOT to use

- Input is already a filtered gene expression table with low-abundance genes removed upstream
- Working in a downstream scdrake stage (e.g. normalization or clustering) where gene filtering was already completed in 01_input_qc
- Analyzing spot-based spatial transcriptomics data where gene-detection characteristics differ from droplet scRNA-seq without re-tuning min_ratio_cells/min_umi

## Inputs

- SingleCellExperiment object (sce) after empty-droplet removal and cell QC filtering
- 10x Genomics Cell Ranger feature-barcode matrix (e.g. example_data/pbmc1k)
- config/single_sample/01_input_qc.yaml gene-filter parameters (min_ratio_cells, min_umi)

## Outputs

- Gene filter object from get_gene_filter applied to the SCE
- Filtered SingleCellExperiment object with low-expression genes removed
- Stage report at output/single_sample/01_input_qc/01_input_qc.html showing how each filtering type affects retained cells/genes

## How to apply

In the scdrake 01_input_qc stage, gene filtering is applied after empty-droplet removal (empty_droplets_fn) and MAD-threshold cell QC filtering (cell_qc_fn with configurable MAD_THRESHOLD, e.g. 2). The concrete step is the get_gene_filter(sce, min_ratio_cells, min_umi) target, which builds a gene filter from the filtered SingleCellExperiment using the min_ratio_cells (minimum fraction of cells in which a gene is detected) and min_umi (minimum UMI count) parameters set in config/single_sample/01_input_qc.yaml. The rationale, per the scdrake paper, is that cell and gene filtering precede normalization, highly variable gene detection, and dimensionality reduction in the pipeline, so poorly detected genes must be removed early. The filtering result feeds the downstream drake targets that assemble the filtered SingleCellExperiment object and render the stage report.

## Related tools

- **scdrake** (Pipeline implementing the 01_input_qc stage that contains the get_gene_filter gene-filtering target) — https://github.com/bioinfocz/scdrake
- **drake** (Workflow framework that tracks get_gene_filter as a target so only changed parts of the pipeline rerun) — https://github.com/bioinfocz/scdrake
- **R** (Language in which get_gene_filter and the pipeline are implemented) — https://github.com/bioinfocz/scdrake

## Examples

```
scdrake --pipeline-type single_sample run  # executes drake targets incl. get_gene_filter(sce, min_ratio_cells, min_umi) in 01_input_qc
```

## Evaluation signals

- get_gene_filter target completes within the drake plan without errors and produces a gene filter from the cell-filtered SCE
- The rendered report output/single_sample/01_input_qc/01_input_qc.html exists and contains QC summaries showing how each filtering type affects the number of retained cells
- Genes failing min_ratio_cells / min_umi criteria are absent from the downstream normalized SCE used for HVG detection
- Changing gene-filter parameters in 01_input_qc.yaml causes only downstream targets to rerun while up-to-date targets are skipped

## Limitations

- Filtering thresholds (min_ratio_cells, min_umi) are dataset-sensitive and must be tuned; defaults may not suit sparse or small datasets
- scdrake is marked as experimental/lifecycle 'stable' only recently; parameter changes trigger partial reruns, so poorly chosen thresholds cost recompute time

## Evidence

- [methods] get_gene_filter target signature: "`get_gene_filter(sce, min_ratio_cells, min_umi)`"
- [abstract] Pipeline stage ordering including gene filtering: "The pipeline includes quality control, cell and gene filtering, normalization, detection of highly variable genes"
- [readme] README feature list covering gene filtering: "Quality control and filtering of cells/spots and genes, removal of
  empty droplets."
- [readme] Drake partial rerun behavior: "Want to change some parameter? No problem! Only parts of the
    pipeline which changed will rerun, while up-to-date ones will be
    skipped."
