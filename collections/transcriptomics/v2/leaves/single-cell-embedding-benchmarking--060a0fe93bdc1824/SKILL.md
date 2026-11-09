---
name: single-cell-embedding-benchmarking
description: Use when evaluating whether a single-cell embedding method (e.g., DIRECTi,
  DCA, PCA, tSNE) preserves cell-type resolution in scRNA-seq latent space, by computing
  MAP of nearest-neighbor cell retrieval between query and reference datasets.
when_to_use_negative:
- Datasets with fewer than 1000 cells, which may suffer from under-training of the
  over-parameterized DIRECTi model (3000 cells is generally sufficient)
- Mega-scale integration scenarios where highly skewed cell-type composition across
  batches risks over-correcting of batches
- Tasks where input is not cell-by-gene scRNA-seq data with reference annotations
  available for cell-querying
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST / DIRECTi
  role: Fits the deep generative model on the reference, produces posterior cell embeddings,
    and provides the BLAST cell-to-cell similarity metric used to compute MAP
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DCA
  role: Baseline dimension-reduction method embedded in the MAP comparison
- name: PCA
  role: Baseline linear dimension-reduction method in the MAP comparison
- name: tSNE
  role: Baseline non-linear dimension-reduction method in the MAP comparison
- name: anndata / Cell_BLAST.data
  role: Loads and migrates reference and query datasets in h5ad format
  repo: https://github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:ec9709c129c9e51abf02bfee4632ac77f2f79ee053dc02f3d63565b9a6705391
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - Cell BLAST / DIRECTi
  - Cell_BLAST.blast
  - Cell_BLAST.data
  - Cell_BLAST.directi
  - Cell_BLAST.latent
  - DCA
  - PCA
  - anndata / Cell_BLAST.data
  - tSNE
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-embedding-benchmarking

## Summary

Benchmarks Cell BLAST's DIRECTi deep generative embeddings against DCA, PCA, and tSNE for cell-type resolution using mean average precision (MAP) on real-world scRNA-seq reference/query datasets in h5ad format, as in Supplementary Table 1/2 and Supplementary Fig. 3 of the Cell BLAST paper.

## When to use

Use when evaluating whether a single-cell embedding method (e.g., DIRECTi, DCA, PCA, tSNE) preserves cell-type resolution in scRNA-seq latent space, by computing MAP of nearest-neighbor cell retrieval between query and reference datasets.

## When NOT to use

- Datasets with fewer than 1000 cells, which may suffer from under-training of the over-parameterized DIRECTi model (3000 cells is generally sufficient)
- Mega-scale integration scenarios where highly skewed cell-type composition across batches risks over-correcting of batches
- Tasks where input is not cell-by-gene scRNA-seq data with reference annotations available for cell-querying

## Inputs

- Reference scRNA-seq dataset in h5ad (AnnData) format with cell-type annotations
- Query scRNA-seq dataset in h5ad (AnnData) format with ground-truth cell-type labels for MAP evaluation
- Optional legacy h5 files convertible via cb.data.h5_to_h5ad

## Outputs

- MAP score per embedding method (DIRECTi/Cell BLAST, DCA, PCA, tSNE) with K = 1% of total cells
- Results table comparable to Supplementary Table 1/2
- Comparison summary visualizable as in Supplementary Fig. 3

## How to apply

Load real-world reference and query scRNA-seq benchmark datasets in h5ad (AnnData) format via Cell_BLAST.data, after converting any legacy h5 files with cb.data.h5_to_h5ad. Fit the DIRECTi deep generative model on the reference dataset with Cell_BLAST.directi.fit_DIRECTi to learn low-dimensional cell embeddings that explicitly model multilevel batch effect. Embed both reference and query cells with each method under comparison: DIRECTi posterior embeddings, DCA, PCA, and tSNE. For every query cell, retrieve nearest reference cells via Cell_BLAST.blast.BLAST using Cell BLAST's posterior-distribution-distance similarity metric rather than approximate cosine or LSH Hamming distances. Compute mean average precision (MAP) for each method with K set to 1% of the total number of cells, then compile a results table formatted like Supplementary Table 1/2 and summarize as in Supplementary Fig. 3.

## Related tools

- **Cell BLAST / DIRECTi** (Fits the deep generative model on the reference, produces posterior cell embeddings, and provides the BLAST cell-to-cell similarity metric used to compute MAP) — https://github.com/gao-lab/Cell_BLAST
- **DCA** (Baseline dimension-reduction method embedded in the MAP comparison)
- **PCA** (Baseline linear dimension-reduction method in the MAP comparison)
- **tSNE** (Baseline non-linear dimension-reduction method in the MAP comparison)
- **anndata / Cell_BLAST.data** (Loads and migrates reference and query datasets in h5ad format) — https://github.com/gao-lab/Cell_BLAST

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST && python -c "import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref); blast = cb.blast.BLAST(model, ref); hits = blast.query(q); print(hits)"
```

## Evaluation signals

- The MAP table contains scores for all four methods (Cell BLAST/DIRECTi, DCA, PCA, tSNE)
- Cell BLAST's MAP reproduces the relative ranking over DCA, PCA, and tSNE reported in Supplementary Table 1/2 and Supplementary Fig. 3
- Cell BLAST's generative model, together with DCA, PCA, and tSNE, is among the best performing methods in cell-type resolution by MAP
- K is set to 1% of the total number of cells when computing mean average precision
- Nearest-neighbor retrieval uses Cell BLAST's posterior-distribution-distance similarity metric, not cosine or Hamming distances

## Limitations

- The Cell BLAST model is over-parameterized (like all neural network-based models) and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient, but datasets with fewer than 1000 cells may suffer from under-training issues
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition

## Evidence

- [results] Dimension-reduction benchmark finding: [evidence span withheld — anchor status: not_found]
- [intro] Similarity metric rationale: "While these metrics per se are intuitive and computationally efficient, they may not reflect cell-to-cell similarity faithfully, and suffer from nonbiological confounding variation across datasets"
- [readme] Installation instructions: "Install Cell BLAST from PyPI by running:

   `pip install Cell-BLAST`"
- [methods] Data migration: "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [readme] Reproduction instructions: [evidence span withheld — anchor status: external_doc]
