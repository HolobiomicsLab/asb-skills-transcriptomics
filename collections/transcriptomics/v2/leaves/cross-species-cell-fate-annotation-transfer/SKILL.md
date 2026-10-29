---
name: cross-species-cell-fate-annotation-transfer
description: Use when a query scRNA-seq dataset from one species must receive continuous
  cell fate (differentiation potential) annotations from a curated reference dataset
  in another species, requiring cross-batch and cross-species distribution alignment.
when_to_use_negative:
- Reference and query datasets are from the same species with only batch effects to
  correct — a standard within-species cell-type query suffices
- Datasets contain fewer than ~1000 cells, where Cell BLAST's over-parameterized neural
  network model may suffer under-training issues
- The goal is assigning discrete Cell Ontology terms via the ACA/BLAST2CO curation
  pipeline rather than continuous fate probabilities
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cell BLAST
  role: Cell-querying framework used to search reference cells and transfer annotations
    via cell-to-cell similarity
  repo: github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST
  role: Python package (import Cell_BLAST as cb) providing data handling and the BLAST
    index
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fitted to the reference data whose embeddings handle
    batch effect and enable cross-dataset alignment
  repo: github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST.directi.fit_DIRECTi
  role: API used to train DIRECTi with online tuning so embeddings adapt to the query
    dataset's batch and distribution
  repo: github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST.blast.BLAST
  role: Index built from the fitted reference model; queried to find similar cells
    and compute posterior-distribution-based similarities
  repo: github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:127d6515186c8825a74ac5eb9e890d36eb879129a786ce09e567ff7170a9c239
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST
  - Cell_BLAST.blast.BLAST
  - Cell_BLAST.directi.fit_DIRECTi
  - DIRECTi
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-species-cell-fate-annotation-transfer

## Summary

Transfers continuous cell fate annotations across species by training Cell BLAST's DIRECTi deep generative model on a reference dataset (e.g. mouse Tusi hematopoietic progenitors) with adversarial batch alignment-based online tuning, then querying an independent cross-species dataset (e.g. human Velten) via posterior-distribution cell-to-cell similarity.

## When to use

Use when a query scRNA-seq dataset from one species must receive continuous cell fate (differentiation potential) annotations from a curated reference dataset in another species, requiring cross-batch and cross-species distribution alignment.

## When NOT to use

- Reference and query datasets are from the same species with only batch effects to correct — a standard within-species cell-type query suffices
- Datasets contain fewer than ~1000 cells, where Cell BLAST's over-parameterized neural network model may suffer under-training issues
- The goal is assigning discrete Cell Ontology terms via the ACA/BLAST2CO curation pipeline rather than continuous fate probabilities

## Inputs

- Reference scRNA-seq dataset (e.g. mouse Tusi hematopoietic progenitors) as AnnData/h5ad with fate annotations
- Query scRNA-seq dataset from another species (e.g. human Velten) as AnnData/h5ad
- One-to-one ortholog mapping table to place both datasets in a shared gene space
- Known lineage marker gene lists (Supplementary Fig. 17) for validation

## Outputs

- Predicted cell fate probabilities per query cell (weighted from hit cell annotations)
- Per-fate Spearman correlation table written to fate_probability_spearman.csv

## How to apply

Load the mouse Tusi reference and human Velten query datasets as AnnData (h5ad) objects and map genes between species via one-to-one orthologs so both matrices share a common gene space. Fit a DIRECTi model on the reference with Cell_BLAST.directi.fit_DIRECTi, applying online tuning so the model's low-dimensional embeddings adapt to the query dataset's batch and distribution through adversarial batch alignment. Build a Cell_BLAST.blast.BLAST index from the fitted model and query Velten cells, computing cell-to-cell similarity as the distance between posterior distributions. Transfer fate annotations from hit cells to each query cell, yielding predicted fate probabilities from weighted hit annotations. Validate by computing Spearman rank correlations between predicted fate probabilities and expression of known lineage markers from the Supplementary Fig. 17 marker lists, then write per-fate correlations to fate_probability_spearman.csv.

## Related tools

- **Cell BLAST** (Cell-querying framework used to search reference cells and transfer annotations via cell-to-cell similarity) — github.com/gao-lab/Cell_BLAST
- **Cell_BLAST** (Python package (import Cell_BLAST as cb) providing data handling and the BLAST index) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fitted to the reference data whose embeddings handle batch effect and enable cross-dataset alignment) — github.com/gao-lab/Cell_BLAST
- **Cell_BLAST.directi.fit_DIRECTi** (API used to train DIRECTi with online tuning so embeddings adapt to the query dataset's batch and distribution) — github.com/gao-lab/Cell_BLAST
- **Cell_BLAST.blast.BLAST** (Index built from the fitted reference model; queried to find similar cells and compute posterior-distribution-based similarities) — github.com/gao-lab/Cell_BLAST

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST && python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- Spearman rank correlations between predicted fate probabilities and known lineage marker expression are computed for each fate and non-negative/meaningful for most lineages
- Online-tuned (adversarial batch alignment) model yields significantly higher marker-expression correlations than a model without online tuning, as reported in the paper's inter-species transfer to 'Velten'
- Both AnnData objects share a common ortholog-mapped gene space before model fitting
- Output file fate_probability_spearman.csv exists with one row per fate and Spearman correlation values
- Import of Cell_BLAST succeeds (import Cell_BLAST as cb; print(cb.__version__))

## Limitations

- The Cell BLAST model, like all neural network-based models, is over-parameterized and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition

## Evidence

- [intro] Online tuning rationale: "we introduce Cell BLAST, a cell-querying tool employing a customized neural network-based generative model that effectively handles batch effect"
- [methods] Install/import verification: "import Cell_BLAST as cb"
- [abstract] Model capability for continuous fates: "we demonstrate the effectiveness of Cell BLAST in annotating discrete cell types and continuous cell differentiation potential, as well as identifying novel cell types"
- [discussion] Similarity metric and uncertainty modeling: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [discussion] Under-training limitation: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
- [readme] README package usage: "Annotations in reference cells, e.g. cell type, can then be transfered to query cells based on cell-to-cell similarities."
