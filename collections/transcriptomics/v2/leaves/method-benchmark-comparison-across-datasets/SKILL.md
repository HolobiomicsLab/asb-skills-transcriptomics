---
name: method-benchmark-comparison-across-datasets
description: Use when two methods were compared on differently sized datasets and
  you must re-run both on the same cell count subsets to isolate true performance
  differences from dataset-size effects.
when_to_use_negative:
- When the research question concerns absolute accuracy on the full datasets rather
  than a size-controlled head-to-head comparison — the 50k subsets deliberately trade
  performance for comparability.
- When species other than human, mouse, or marmoset motor cortex are involved — the
  50k subset resource on Zenodo covers only these three motor cortex datasets.
- When a species pair includes a cell type with fewer than 50 cells — subset construction
  requires a minimum of 50 cells per included type.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: TACTiCS
  role: 'Method under test: trained and evaluated per species pair via the TACTiCS
    class in tactics.py, taking two scRNA-seq datasets and a gene similarity distance
    matrix as input'
  repo: https://github.com/kbiharie/TACTiCS
- name: SAMap
  role: Competing state-of-the-art method run on the same three 50k subset species
    pairs with matched settings for a fair comparison
- name: PyTorch
  role: Deep-learning backend required to train the TACTiCS neural network classifier
- name: transformers
  role: Used to embed protein sequences (embed_proteins in genes.py) when the deposited
    embeddings are not reused directly
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:94b66b49c0ec1426659a002d5969547d018ee2df5e507e8787f855b959cd6b10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - SAMap
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# method-benchmark-comparison-across-datasets

## Summary

Benchmark two cross-species cell type matching methods (TACTiCS vs SAMap) on identical equal-sized 50k-cells-per-species subsets of the human, mouse, and marmoset motor cortex scRNA-seq data, to control for dataset size differences when judging relative performance.

## When to use

Use when two methods were compared on differently sized datasets and you must re-run both on the same cell count subsets to isolate true performance differences from dataset-size effects.

## When NOT to use

- When the research question concerns absolute accuracy on the full datasets rather than a size-controlled head-to-head comparison — the 50k subsets deliberately trade performance for comparability.
- When species other than human, mouse, or marmoset motor cortex are involved — the 50k subset resource on Zenodo covers only these three motor cortex datasets.
- When a species pair includes a cell type with fewer than 50 cells — subset construction requires a minimum of 50 cells per included type.

## Inputs

- Preprocessed 50k-cell subset raw count scRNA-seq data for human, mouse, and marmoset primary motor cortex (Zenodo 10.5281/zenodo.7582460)
- Protein sequence embeddings (deposited, or regenerated with embed_proteins in genes.py)
- Full-dataset reference performance scores for TACTiCS and SAMap
- Full unprocessed datasets from Zenodo 10.5281/zenodo.6786357 if reference scores must be recomputed

## Outputs

- Per-comparison higher-resolution cell type matching scores for TACTiCS and SAMap on all three species pairs
- Subset-vs-full-dataset performance deltas quantifying the drop at 50k cells per species
- tactics_vs_samap_50k_results.csv recording per-comparison scores

## How to apply

Download the preprocessed 50k-cell subset raw count data for the human, mouse, and marmoset primary motor cortex from Zenodo (10.5281/zenodo.7582460), verifying cell type proportions are preserved with a minimum of 50 cells per included type. Load the deposited protein embeddings (or regenerate them with `embed_proteins` in `genes.py`) and compute the gene similarity distance matrix for each species pair using `calc_dist`. Train and evaluate TACTiCS via the `TACTiCS` class in `tactics.py`, supplying two scRNA-seq datasets and the gene distance matrix, then run SAMap on the same three 50k subset pairs with matched settings. Score cell type matching at the higher (cross-species) resolution for each pair and record per-comparison scores alongside full-dataset reference performance to quantify the performance drop on subsets. The expected validated outcome is that TACTiCS drops relative to full data yet still outperforms SAMap at the higher resolution in all three pairwise comparisons, as reported in the paper.

## Related tools

- **TACTiCS** (Method under test: trained and evaluated per species pair via the TACTiCS class in tactics.py, taking two scRNA-seq datasets and a gene similarity distance matrix as input) — https://github.com/kbiharie/TACTiCS
- **SAMap** (Competing state-of-the-art method run on the same three 50k subset species pairs with matched settings for a fair comparison)
- **PyTorch** (Deep-learning backend required to train the TACTiCS neural network classifier)
- **transformers** (Used to embed protein sequences (embed_proteins in genes.py) when the deposited embeddings are not reused directly)

## Examples

```
from tactics import TACTiCS; model = TACTiCS(scrna_a, scrna_b, gene_dist_matrix); model.train(); model.evaluate(); model.save('pair_results')
```

## Evaluation signals

- TACTiCS performance on each 50k subset is lower than its corresponding full-dataset reference score (expected drop).
- TACTiCS still exceeds SAMap at the higher (cross-species) resolution in all three pairwise species comparisons (human-mouse, human-marmoset, mouse-marmoset).
- Each included cell type retains a minimum of 50 cells in the 50k subsets, preserving cell type proportions.
- The gene similarity distance matrix per species pair is computed from embeddings produced with embed_proteins via calc_dist.
- Per-comparison scores are written to tactics_vs_samap_50k_results.csv with full-dataset references recorded alongside.

## Limitations

- TACTiCS performance drops on 50k subsets relative to full datasets, so subset-based scores are not interchangeable with full-data results.
- The comparison is specific to primary motor cortex scRNA-seq data of human, mouse, and marmoset; generalization to other tissues or species is not established by this benchmark.
- The transformers package is only necessary for embedding protein sequences, so it can be skipped only if the deposited embeddings from Zenodo are reused.

## Evidence

- [abstract] Size-controlled comparison rationale: "our model outperforms Seurat and the state-of-the-art method SAMap"
- [methods] TACTiCS training interface: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [methods] Gene distance computation: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [readme] Data availability for 50k subsets: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo]"
- [methods] Full-dataset reference source: "The full unprocessed dataset can be downloaded [here](https://doi.org/10.5281/zenodo.6786357)"
