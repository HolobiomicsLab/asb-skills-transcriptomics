---
name: cell-type-replicability-subsampling-simulation
description: Use when you must stress-test whether a cross-dataset cell-type identification
  method still recovers a target type that is rare (1–20% of cells) and only subtly
  different from the out-group, rather than well-separated abundant types.
when_to_use_negative:
- The question is identifying replicable cell types across datasets at natural abundance
  — use the unsupervised MetaNeighbor workflow (variableGenes + reciprocal top hits,
  AUROC ≥ 0.95) instead of subsampling simulation.
- Gene sets contain fewer than two genes or their gene names do not match the expression-matrix
  row names, per the README requirements.
- No cell-type labels exist for the datasets, since the supervised MetaNeighbor analysis
  requires known corresponding types across datasets.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaNeighbor
  role: 'supervised pipeline: rank-correlation cell network + neighbor-voting leave-one-dataset-out
    cross-validation yielding AUROC per condition'
  repo: github.com/gillislab/MetaNeighbor
- name: R
  role: analysis environment for subsampling simulation and aggregation of AUROC summaries
- name: SummarizedExperiment
  role: container for the scRNA-seq expression datasets
- name: SingleCellExperiment
  role: container for the scRNA-seq expression datasets
- name: Matrix
  role: sparse matrix handling for large expression matrices
- name: gplots
  role: heatmap visualization of neighbor-voting score matrices (celltype_NV)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:6c81d5fed721d305a21f7dd24f28a91caf9ad23dc54cf06b37dd948263d83762
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matrix
  - MetaNeighbor
  - R
  - SingleCellExperiment
  - SummarizedExperiment
  - gplots
  - matrixStats
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cell-type-replicability-subsampling-simulation

## Summary

Empirical robustness test for cross-dataset cell-type identification (Fig. 3b) that subsamples scRNA-seq cells to make a target cell type rare (1–20% incidence) and subtly distinct (up to ~88% subtlety), then measures MetaNeighbor supervised AUROC per condition against random 20-gene controls.

## When to use

Use when you must stress-test whether a cross-dataset cell-type identification method still recovers a target type that is rare (1–20% of cells) and only subtly different from the out-group, rather than well-separated abundant types.

## When NOT to use

- The question is identifying replicable cell types across datasets at natural abundance — use the unsupervised MetaNeighbor workflow (variableGenes + reciprocal top hits, AUROC ≥ 0.95) instead of subsampling simulation.
- Gene sets contain fewer than two genes or their gene names do not match the expression-matrix row names, per the README requirements.
- No cell-type labels exist for the datasets, since the supervised MetaNeighbor analysis requires known corresponding types across datasets.

## Inputs

- Tasic, Zeisel, and Habib scRNA-seq gene-by-sample expression matrices
- GO:0022857 transporter-activity gene set (gene name vectors matching expression matrix row names)
- Random 20-gene control sets matched in size to GO:0022857
- Cell-type labels and experiment/dataset-of-origin labels per sample

## Outputs

- Per-condition mean AUROC table (rarity_subtlety_aurocs.csv)
- GO:0022857 vs. random-control AUROC comparison across incidence (1–20%) and subtlety (up to ~88%) conditions

## How to apply

Load Tasic, Zeisel, and Habib scRNA-seq expression datasets in R with SummarizedExperiment/SingleCellExperiment, and extract the GO:0022857 transporter-activity gene set. For each condition, subsample cells to instantiate a target cell type at incidence levels spanning 1–20% and at subtlety levels up to ~88%, defining the held-out type against the remaining cell types. For each of 100 repeats per condition, run MetaNeighbor's supervised pipeline: build a cell-cell rank correlation network for the gene set, standardize network values to 0–1, and perform leave-one-dataset-out cross-validation with the neighbor-voting predictor to obtain AUROC scores. Repeat the same 100-repeat procedure with random 20-gene control sets matched in size to GO:0022857, then aggregate mean AUROCs per condition and gene-set class. Write the condition-by-condition AUROC summary to rarity_subtlety_aurocs.csv.

## Related tools

- **MetaNeighbor** (supervised pipeline: rank-correlation cell network + neighbor-voting leave-one-dataset-out cross-validation yielding AUROC per condition) — github.com/gillislab/MetaNeighbor
- **R** (analysis environment for subsampling simulation and aggregation of AUROC summaries)
- **SummarizedExperiment** (container for the scRNA-seq expression datasets)
- **SingleCellExperiment** (container for the scRNA-seq expression datasets)
- **Matrix** (sparse matrix handling for large expression matrices)
- **gplots** (heatmap visualization of neighbor-voting score matrices (celltype_NV))

## Examples

```
var_genes = variableGenes(dat = dat, studies = studies); celltype_NV = MetaNeighborUS(var_genes = var_genes, dat = dat, studies = studies, cell_types = cell_types)
```

## Evaluation signals

- Mean AUROC for the target type reaches ~0.71 at ~88% subtlety and only falls to chance (0.5) beyond 99% subtlety.
- AUROC is nearly flat across 1–20% target-type incidence — performance is 'nearly unaffected by rarity'.
- GO:0022857 gene set consistently outperforms size-matched random 20-gene control sets across conditions.
- Each condition aggregates 100 repeats; per-condition mean AUROCs are stable across repeats before writing rarity_subtlety_aurocs.csv.
- Network values are standardized to the 0–1 range before neighbor voting in every run.

## Limitations

- Distinguishing only subtly different types is difficult for any analysis beyond the chance boundary (>99% subtlety); discovery of such types requires appropriate controls.
- The ranking step does not scale well in R — runtime grows steeply with total sample count (e.g. ~3653 s for 10000 samples × 10 gene sets on a laptop).
- AUROC >0.95 reflects extreme confidence; moderate scores are still meaningful but must be interpreted hierarchically from broad to subtle categories.
- Current estimates of cell identity are optimistic since clusters are defined from gene expression; orthogonal data is needed to establish biological basis.

## Evidence

- [intro] Empirical modeling of rare/subtle types: "With detailed control experiments and empirical modeling, we validate the use of HVGs for cross-dataset cell identification"
- [methods] MetaNeighbor supervised pipeline mechanics: "first, we build a network of rank correlations between all cells for a gene set"
- [readme] Interpretation of chance-level performance: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
- [discussion] Difficulty limit for subtle types: "Moving past this point and distinguishing between only subtly different types will be difficult for any analysis, and their discovery will require consideration of appropriate controls"
- [intro] Cross-validation design: [evidence span withheld — anchor status: not_found]
