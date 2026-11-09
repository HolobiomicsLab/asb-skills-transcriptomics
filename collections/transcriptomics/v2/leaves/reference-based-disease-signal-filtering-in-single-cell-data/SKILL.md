---
name: reference-based-disease-signal-filtering-in-single-cell-data
description: Use when case-control scRNA-seq data (e.g., COVID-19 patient vs healthy
  control PBMCs) is available, healthy cells can serve as a reference in a harmonized
  joint latent space, and the goal is to remove the healthy signal to reveal disease-related
  dynamics and disease-informative subpopulations.
when_to_use_negative:
- When no healthy/reference samples exist to define the signal to be filtered — the
  kernel requires reference mapping to healthy cells
- When the prior knowledge needed to define the cell-cell similarity kernel is inaccurate
  or coarse-grained, since SiFT 'strongly relies on the prior knowledge used to define
  the cell-cell similarity kernel' and a coarse mapping 'may lead to a loss of signal'
- When the downstream task requires properly log-transformed expression, since filtered
  data 'is no longer properly log-transformed'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: 'Core kernel-based projection package: builds the knn kernel against healthy
    reference cells, projects disease-cell expression onto the healthy signal, and
    deducts it to produce the filtered matrix'
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.kernels.KnnKernel
  role: KNN cell-cell similarity kernel in the harmonized PCA space mapping each disease
    cell to its nearest healthy reference cells
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.sifter
  role: SiFT API entry point used to run the filtering procedure on the disease cells
    with the reference-based kernel
  repo: https://github.com/nitzanlab/sift-sc
- name: bbknn
  role: Batch-correction integration approach (bbknn-style neighbor correction) used
    to harmonize disease and healthy cells in a joint PCA latent space before kernel
    construction
- name: pip
  role: Installation channel for the sift-sc package (pip install sift-sc)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:07725b66dc4ace2c43482ee7f532fcb4c5ea0c08e7099cb71123a347cdb50010
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# reference-based-disease-signal-filtering-in-single-cell-data

## Summary

A SiFT-based case-control skill that uses healthy reference samples to define a cell-cell similarity kernel mapping disease cells to their nearest healthy counterparts, projects disease-cell gene expression onto the healthy signal, deducts that projection, and exposes disease-informative cells (e.g., interferon-responsive monocytes, DCs, and HSPC in COVID-19 PBMC data).

## When to use

Use when case-control scRNA-seq data (e.g., COVID-19 patient vs healthy control PBMCs) is available, healthy cells can serve as a reference in a harmonized joint latent space, and the goal is to remove the healthy signal to reveal disease-related dynamics and disease-informative subpopulations.

## When NOT to use

- When no healthy/reference samples exist to define the signal to be filtered — the kernel requires reference mapping to healthy cells
- When the prior knowledge needed to define the cell-cell similarity kernel is inaccurate or coarse-grained, since SiFT 'strongly relies on the prior knowledge used to define the cell-cell similarity kernel' and a coarse mapping 'may lead to a loss of signal'
- When the downstream task requires properly log-transformed expression, since filtered data 'is no longer properly log-transformed'

## Inputs

- scRNA-seq count matrix with disease (case) and healthy control cells (e.g., Stephenson et al. PBMC COVID-19 dataset)
- Cell type annotations for disease and healthy cells (e.g., CD14+ monocytes, DCs, HSPC, Plasmablasts)
- Interferon-response (IFN) gene set for enrichment scoring
- Harmonized joint PCA latent representation of disease and healthy cells (batch-corrected, e.g., bbknn-style)

## Outputs

- Filtered (healthy-signal-removed) gene expression matrix for disease cells
- IFN-response enrichment scores per cell type on filtered data (expected in monocytes, DCs, HSPC)
- Binary disease-informative vs non-informative cell labels via cluster purity threshold tau = 0.55
- Per-cell-type aggregation of informative cells (expected in CD14+ mono., CD16+ mono., Plasmablast, Platelet)

## How to apply

First, load the case-control scRNA-seq dataset (e.g., Stephenson et al. PBMC COVID-19 data) and harmonize disease and healthy cells in a joint PCA latent space using a batch-correction integration approach (e.g., bbknn-style neighbor correction) so healthy cells can serve as reference. Second, build a cell-cell similarity knn kernel with sift.kernels.KnnKernel in the harmonized PCA space, mapping each disease cell to its nearest healthy reference cells to encode the healthy signal to remove. Third, run SiFT (sift.SiFT or sift.sifter) with this kernel to project gene expression onto the healthy signal and deduct the projection, yielding a filtered expression matrix for the disease cells. Then perform downstream analysis on the filtered data: compute an interferon-response (IFN) gene-set score, evaluate cluster purity across the filtered neighborhood structure, and classify cells as disease-informative vs non-informative using the paper's purity threshold tau = 0.55. Finally, aggregate informative-cell labels per annotated cell type to recover disease-informative subpopulations such as CD14+ monocytes, CD16+ monocytes, Plasmablasts, and Platelets.

## Related tools

- **SiFT (sift-sc)** (Core kernel-based projection package: builds the knn kernel against healthy reference cells, projects disease-cell expression onto the healthy signal, and deducts it to produce the filtered matrix) — https://github.com/nitzanlab/sift-sc
- **sift.kernels.KnnKernel** (KNN cell-cell similarity kernel in the harmonized PCA space mapping each disease cell to its nearest healthy reference cells) — https://github.com/nitzanlab/sift-sc
- **sift.sifter** (SiFT API entry point used to run the filtering procedure on the disease cells with the reference-based kernel) — https://github.com/nitzanlab/sift-sc
- **bbknn** (Batch-correction integration approach (bbknn-style neighbor correction) used to harmonize disease and healthy cells in a joint PCA latent space before kernel construction)
- **pip** (Installation channel for the sift-sc package (pip install sift-sc))

## Examples

```
pip install sift-sc; import sift; kernel = sift.kernels.KnnKernel(n_neighbors=...); sft = sift.SiFT(kernel=kernel); sft.fit(adata); sft.transform()
```

## Evaluation signals

- After filtering, IFN-response gene-set enrichment is enhanced in monocytes, dendritic cells, and HSPC, whereas the same analysis on the original data fails to expose these cell types
- Cluster purity classification with threshold tau = 0.55 recovers disease-informative subpopulations in CD14+ monocytes, CD16+ monocytes, Plasmablasts, and Platelets, matching the paper's Fig. 6-7 summary
- The filtered neighborhood structure (not the original one) exposes the relevant innate immune cell types via the IFN score
- The kernel correctly maps each disease cell to its nearest healthy reference cells in the harmonized PCA space (integration/label-mixing sanity checks)

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- No ground truth reference is available to evaluate computational filtering results; integration metrics (which may be application-specific) are suggested as mitigation
- SiFT modifies gene expression so the filtered data is no longer properly log-transformed, a property expected in certain downstream analysis tasks
- Statistical implications of applying commonly used analysis tools to corrected gene expression data remain unexplored

## Evidence

- [discussion] COVID-19 PBMC dataset: "In the context of COVID-19, SiFT exposed disease-related signals and single-cell dynamics by filtering a corresponding healthy trajectory"
- [discussion] Workflow step: define kernel capturing signal to filter: "The SiFT procedure starts by defining a cell–cell similarity kernel, capturing similarities with respect to the signal to be filtered."
- [intro] Workflow step: projection and deduction: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [intro] Reference-based filtering objective: "Healthy samples are used as a reference to filter the healthy signal and identify disease-related dynamics"
- [readme] Installation via pip: "Install SiFT via PyPI by running::

    pip install sift-sc"
- [discussion] Limitation: reliance on prior knowledge: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [discussion] Limitation: coarse mapping loses signal: "As we showcase in various examples, an inaccurate or coarse-grained mapping may lead to a loss of signal."
