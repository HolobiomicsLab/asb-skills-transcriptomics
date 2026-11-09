---
name: single-cell-rna-seq-dataset-subsampling
description: Use when comparing cell type matching methods (e.g., TACTiCS vs SAMap)
  whose reference performances were computed on differently sized datasets, and you
  need equal-sized per-species subsets to control for dataset size effects.
when_to_use_negative:
- When the analysis goal is absolute best performance rather than a size-controlled
  method comparison — the full unprocessed datasets should be used instead
- When subsampling would drop any included cell type below the minimum of 50 cells
  per type, violating the preserved-proportions constraint
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TACTiCS
  role: Trained and evaluated on the 50k subsets to measure how subsampling affects
    its cross-species cell type matching performance
  repo: https://github.com/kbiharie/TACTiCS
- name: SAMap
  role: Comparator method run on the identical 50k subsets with matched settings to
    enable a size-fair performance comparison
- name: PyTorch
  role: Required dependency for training TACTiCS on the subsampled data
- name: transformers
  role: Used for embedding protein sequences when protein embeddings must be regenerated
    for the subsets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:ac1355786889b4e2a23a077760e20acc4175ccef0b67d1cb3bdd634cdc8f4ae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - SAMap
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-rna-seq-dataset-subsampling

## Summary

Subsample scRNA-seq datasets to equal-sized (e.g., 50k cells per species) subsets that preserve cell type proportions with a minimum of 50 cells per included type, enabling fair cross-method comparisons controlled for dataset size differences.

## When to use

Use when comparing cell type matching methods (e.g., TACTiCS vs SAMap) whose reference performances were computed on differently sized datasets, and you need equal-sized per-species subsets to control for dataset size effects.

## When NOT to use

- When the analysis goal is absolute best performance rather than a size-controlled method comparison — the full unprocessed datasets should be used instead
- When subsampling would drop any included cell type below the minimum of 50 cells per type, violating the preserved-proportions constraint

## Inputs

- scRNA-seq raw count data for 50k-cell subsets of the human, mouse, and marmoset primary motor cortex (Zenodo 10.5281/zenodo.7582460)
- Protein sequences or precomputed protein embeddings for gene matching
- Full unprocessed dataset reference results (Zenodo 10.5281/zenodo.6786357) for quantifying performance drop on subsets

## Outputs

- Equal-sized (50k cells per species) subsampled datasets preserving cell type proportions (min. 50 cells per type)
- Per-comparison cross-species resolution scores for each method on the subsets (e.g., tactics_vs_samap_50k_results.csv)
- Quantified performance drop relative to full-dataset reference performance

## How to apply

Download the preprocessed 50k-cell subset raw count data for the human, mouse, and marmoset primary motor cortex datasets from Zenodo (10.5281/zenodo.7582460), verifying that cell type proportions are preserved with a minimum of 50 cells per included type. Optionally regenerate protein sequence embeddings with `embed_proteins` in `genes.py` and compute the gene similarity distance matrix per species pair with `calc_dist`, since the subsampled count data are intended to be used with the same TACTiCS pipeline as the full data. Train and evaluate TACTiCS on each species pair using the `TACTiCS` class in `tactics.py`, supplying the two scRNA-seq datasets and the gene distance matrix as input. Run the comparator (e.g., SAMap) on the identical 50k subsets with matched settings, then score cell type matching at the higher (cross-species) resolution and record per-comparison scores alongside full-dataset reference values to quantify the drop from subsampling.

## Related tools

- **TACTiCS** (Trained and evaluated on the 50k subsets to measure how subsampling affects its cross-species cell type matching performance) — https://github.com/kbiharie/TACTiCS
- **SAMap** (Comparator method run on the identical 50k subsets with matched settings to enable a size-fair performance comparison)
- **PyTorch** (Required dependency for training TACTiCS on the subsampled data)
- **transformers** (Used for embedding protein sequences when protein embeddings must be regenerated for the subsets)

## Examples

```
model = TACTiCS(scrna_species_a_50k, scrna_species_b_50k, calc_dist(embeddings)); model.train(); model.evaluate()
```

## Evaluation signals

- Cell type proportions in each 50k subset match the full dataset, with a minimum of 50 cells per included cell type
- Each subset contains exactly 50,000 cells per species
- TACTiCS performance on the subsets drops relative to full-dataset reference performance (expected invariant of subsampling)
- TACTiCS still outperforms SAMap at the higher (cross-species) resolution in all three pairwise species comparisons
- Per-comparison scores for both methods and both dataset sizes are recorded (e.g., in tactics_vs_samap_50k_results.csv)

## Limitations

- TACTiCS performance drops on 50k subsets relative to the full datasets, so subset results are not interchangeable with full-dataset results
- Subsampling must preserve cell type proportions; types with fewer than 50 cells cannot be reliably included
- Only 50k-cells-per-species subsets of the human, mouse, and marmoset motor cortex data are distributed on Zenodo; other subset sizes require regenerating from the full unprocessed data

## Evidence

- [results] Subset size and minimum cells per type: [evidence span withheld — anchor status: not_found]
- [methods] Zenodo deposition of 50k subsets: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo](https://zenodo.org/doi/10.5281/zenodo.7582460)"
- [methods] TACTiCS class input requirements: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [readme] calc_dist computes the gene distance matrix: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [readme] Full unprocessed dataset availability: "The full unprocessed dataset can be downloaded [here](https://doi.org/10.5281/zenodo.6786357)."
