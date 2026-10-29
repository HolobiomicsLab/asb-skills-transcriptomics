---
name: knn-kernel-construction-in-harmonized-pca-space
description: Use when filtering a biological signal from scRNA-seq data via reference
  mapping — i.e., disease cells must be linked to their nearest healthy reference
  cells before projection-and-subtraction with SiFT.
when_to_use_negative:
- Disease and healthy cells have not been integrated/batch-corrected into a joint
  latent space — a coarse or biased mapping may lead to loss of the disease signal
- No reference population exists that represents the signal to remove (SiFT strongly
  relies on prior knowledge used to define the cell-cell similarity kernel)
- The signal of interest is defined by gene attributes rather than cell-cell reference
  mapping (use distance/RBF/Laplacian kernels or gene kernels instead)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: Python package whose kernels module provides KnnKernel for building the cell-cell
    similarity kernel used in filtering
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.kernels.KnnKernel
  role: The k-nearest-neighbor kernel class that maps each disease cell to its nearest
    healthy reference cells in the harmonized PCA space
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.sifter
  role: Entry point consuming the knn kernel to project expression onto the signal
    and deduct it
  repo: https://github.com/nitzanlab/sift-sc
- name: bbknn
  role: Batch-correction integration approach used to harmonize disease and healthy
    cells in a joint PCA latent space prior to kernel construction
- name: pip
  role: Installation channel for the sift-sc package (pip install sift-sc)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:13497bae752bc5adc82e5a734f85c3de2dea1cf75a38746f178496714e32ec5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - bbknn
  - pip
  - sift.kernels.KnnKernel
  - sift.sifter
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# knn-kernel-construction-in-harmonized-pca-space

## Summary

Construct a k-nearest-neighbor cell-cell similarity kernel in a batch-corrected joint PCA latent space so that query cells (e.g., COVID-19 patient PBMCs) map to reference cells (healthy control PBMCs), encoding the signal to be filtered out by SiFT.

## When to use

Use when filtering a biological signal from scRNA-seq data via reference mapping — i.e., disease cells must be linked to their nearest healthy reference cells before projection-and-subtraction with SiFT.

## When NOT to use

- Disease and healthy cells have not been integrated/batch-corrected into a joint latent space — a coarse or biased mapping may lead to loss of the disease signal
- No reference population exists that represents the signal to remove (SiFT strongly relies on prior knowledge used to define the cell-cell similarity kernel)
- The signal of interest is defined by gene attributes rather than cell-cell reference mapping (use distance/RBF/Laplacian kernels or gene kernels instead)

## Inputs

- Jointly embedded scRNA-seq AnnData object with disease and healthy reference cells harmonized in a common PCA latent space (e.g., bbknn-corrected neighbors)
- Cell-type/sample annotations distinguishing query (disease) cells from reference (healthy control) cells

## Outputs

- Cell-cell similarity knn kernel (sift.kernels.KnnKernel) mapping each disease cell to its nearest healthy reference cells
- Kernel-derived projection of the expression matrix onto the healthy signal, subsequently deducted to yield a filtered expression matrix

## How to apply

First integrate disease and healthy control cells (e.g., Stephenson et al. PBMC COVID-19 scRNA-seq) into a joint PCA latent space using a batch-correction approach such as bbknn-style neighbor correction, so healthy cells can serve as a valid reference. Then instantiate sift.kernels.KnnKernel in this harmonized space to map each disease cell to its nearest healthy reference cells, producing a probabilistic cell-cell similarity kernel that captures similarity with respect to the signal to be filtered. This kernel is passed to sift.SiFT/sift.sifter, which uses it to project gene expression onto the healthy signal and deduct the projection. Rationale: SiFT strongly relies on the prior knowledge encoded in the kernel, so an inaccurate or coarse-grained mapping may lead to a loss of signal — the harmonized embedding is therefore a critical precondition for correct filtering.

## Related tools

- **SiFT (sift-sc)** (Python package whose kernels module provides KnnKernel for building the cell-cell similarity kernel used in filtering) — https://github.com/nitzanlab/sift-sc
- **sift.kernels.KnnKernel** (The k-nearest-neighbor kernel class that maps each disease cell to its nearest healthy reference cells in the harmonized PCA space) — https://github.com/nitzanlab/sift-sc
- **sift.sifter** (Entry point consuming the knn kernel to project expression onto the signal and deduct it) — https://github.com/nitzanlab/sift-sc
- **bbknn** (Batch-correction integration approach used to harmonize disease and healthy cells in a joint PCA latent space prior to kernel construction)
- **pip** (Installation channel for the sift-sc package (pip install sift-sc))

## Examples

```
pip install sift-sc; import sift; kernel = sift.kernels.KnnKernel(n_neighbors=30, space=harmonized_pca_coords, ref_indices=healthy_cells)
```

## Evaluation signals

- Sanity-check the kernel neighborhood: each disease cell's nearest neighbors should be healthy reference cells, not other disease cells (validating the harmonized embedding)
- After filtering, IFN-response gene-set enrichment should be enhanced in monocytes, dendritic cells, and HSPC populations, whereas the same analysis on original data fails to expose these cell types
- Cluster purity across the filtered neighborhood structure should support classifying disease-informative cells at threshold tau = 0.55
- Informative-cell labels aggregated per cell type should recover disease-informative subpopulations in CD14+ monocytes, CD16+ monocytes, Plasmablasts, and Platelets (Fig. 6-7 pattern)
- Filtered data will no longer be properly log-transformed — verify downstream tasks account for this

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- No ground truth reference exists to directly evaluate computational filtering results; application-specific integration metrics must be used instead
- Filtering modifies gene expression so data is no longer properly log-transformed, which certain downstream analysis tasks expect

## Evidence

- [methods] knn kernel in harmonized PCA space: "sift.kernels.RBFKernel
    sift.kernels.LaplacianKernel
    sift.kernels.KnnKernel
    sift.kernels.MappingKernel"
- [intro] kernel captures similarity to signal to filter: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [discussion] COVID-19 healthy reference filtering: "SiFT exposed disease-related signals and single-cell dynamics by filtering a corresponding healthy trajectory obtained by mapping to reference healthy samples."
- [readme] install via pip: "Install SiFT via PyPI by running::

    pip install sift-sc"
- [discussion] prior knowledge dependency and mapping risk: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [intro] projection and deduction step: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
