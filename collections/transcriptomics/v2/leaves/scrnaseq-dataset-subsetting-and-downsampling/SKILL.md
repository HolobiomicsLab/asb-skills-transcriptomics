---
name: scrnaseq-dataset-subsetting-and-downsampling
description: Use when a large single-cell RNA-seq dataset (e.g. a whole-tumor immune
  atlas) must be narrowed to a biologically relevant population (immune cells) and
  a fixed number of cells (e.g. 5000) before per-cell scoring.
when_to_use_negative:
- The dataset is already a cell x gene score/feature table or small enough that subsetting
  would discard informative heterogeneity
- The analysis requires scores on the full population, since per-cell UCell scores
  are robust to dataset composition and do not require downsampling for correctness
- The input lacks cell-type annotations, making a biologically meaningful (e.g. immune-cell)
  subset impossible to define
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Holds the scRNA-seq object that is subset to immune cells and downsampled
    to 5000 cells before AddModuleScore_UCell scoring
  repo: https://github.com/satijalab/seurat
- name: SingleCellExperiment
  role: Alternative Bioconductor container for the gene-vs-cell matrix that UCell
    can score directly after subsetting
- name: UCell
  role: Downstream scorer (AddModuleScore_UCell / ScoreSignatures_UCell) applied to
    the downsampled subset
  repo: https://github.com/carmonalab/UCell
- name: future
  role: Enables serial or parallel processing of batches when the subset is still
    too large for memory
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:c9c163e5cdc90d576ef622381141bf60e29cdccee2966fb6830957d7816a3912
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat
  tools:
  - R
  - Seurat
  - SingleCellExperiment
  - UCell
  - data.table
  - future
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrnaseq-dataset-subsetting-and-downsampling

## Summary

Prepare a scRNA-seq object for signature scoring by loading the Zilionis et al. (2019) NSCLC immune-cell dataset via the scRNAseq package, subsetting it to immune cells, and downsampling to 5000 cells so that downstream UCell scoring is tractable and results are reproducible.

## When to use

Use when a large single-cell RNA-seq dataset (e.g. a whole-tumor immune atlas) must be narrowed to a biologically relevant population (immune cells) and a fixed number of cells (e.g. 5000) before per-cell scoring.

## When NOT to use

- The dataset is already a cell x gene score/feature table or small enough that subsetting would discard informative heterogeneity
- The analysis requires scores on the full population, since per-cell UCell scores are robust to dataset composition and do not require downsampling for correctness
- The input lacks cell-type annotations, making a biologically meaningful (e.g. immune-cell) subset impossible to define

## Inputs

- Zilionis et al. (2019) NSCLC scRNA-seq dataset loaded via the scRNAseq package (Seurat or SingleCellExperiment object)
- Cell-type annotations used to select the immune-cell subset
- Target cell count for downsampling (5000 cells)

## Outputs

- A Seurat/SingleCellExperiment object restricted to immune cells and downsampled to 5000 cells, ready for AddModuleScore_UCell or ScoreSignatures_UCell
- Per-cell signature score table (one column per signature, one row per cell) written to ucell_scores_zilionis.csv

## How to apply

Load the Zilionis et al. (2019) lung cancer scRNA-seq dataset through the scRNAseq package and subset the Seurat or SingleCellExperiment object to immune cells using the provided cell-type annotations. Downsample the subset to a fixed 5000 cells (random sampling of cell barcodes) so that downstream UCell scoring of the Tcell, Myeloid, NK and Plasma_cell signatures runs in minutes even on limited hardware. Because UCell ranks are computed per cell and are robust to dataset composition, the downsampled subset retains valid per-cell signature scores without composition bias. For very large inputs, let UCell automatically split the data into batches processed serially (minimizing memory) or in parallel through the future package.

## Related tools

- **Seurat** (Holds the scRNA-seq object that is subset to immune cells and downsampled to 5000 cells before AddModuleScore_UCell scoring) — https://github.com/satijalab/seurat
- **SingleCellExperiment** (Alternative Bioconductor container for the gene-vs-cell matrix that UCell can score directly after subsetting)
- **UCell** (Downstream scorer (AddModuleScore_UCell / ScoreSignatures_UCell) applied to the downsampled subset) — https://github.com/carmonalab/UCell
- **future** (Enables serial or parallel processing of batches when the subset is still too large for memory)

## Examples

```
library(Seurat); se <- subset(zilionis_seu, cells = sample(colnames(zilionis_seu), 5000)); scores <- AddModuleScore_UCell(se, features = gene.sets)
```

## Evaluation signals

- The output score file exists and contains exactly 5000 rows, one per downsampled cell
- The retained cells are all annotated as immune cells from the Zilionis NSCLC dataset
- One score column per requested signature (Tcell, Myeloid, NK, Plasma_cell) is present
- All scores are normalized Mann-Whitney U statistics (AUC-equivalent) bounded between 0 and 1
- Rerunning with a fixed random seed reproduces the same 5000-cell subset

## Limitations

- Downsampling to 5000 cells discards part of the population; UCell scores are per-cell and composition-robust, so subsetting is for compute efficiency rather than statistical necessity
- Subsetting depends on prior cell-type annotation quality; misannotated non-immune cells may be retained
- Very large datasets are automatically split into batches, so memory use still depends on batch size and serial vs. parallel processing choices

## Evidence

- [full_text] Load the Zilionis et al. (2019) lung cancer scRNA-seq dataset via the scRNAseq package, subset it to immune cells, and downsample to 5000 cells.: [evidence span withheld — anchor status: not_found]
- [abstract] UCell robust to heterogeneity and scalable: "UCell signature scores, based on the Mann-Whitney U statistic, are robust to dataset size and heterogeneity"
- [methods] Batch splitting of large datasets: "Large datasets are automatically split into batches of reduced size, which can be processed serially (minimizing memory usage) or in parallel"
- [readme] UCell input containers: "UCell can be applied to any cell vs. gene data matrix, and includes functions to directly interact with Seurat and Bioconductor's SingleCellExperiment objects."
