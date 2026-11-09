---
name: disease-informative-subpopulation-identification-across-cell-types
description: Use when case-control single-cell RNA-seq data have a healthy control
  population usable as a reference trajectory, and the goal is to expose disease-related
  cells and dynamics masked by healthy shared signal.
when_to_use_negative:
- No healthy reference samples or prior knowledge are available to define the cell-cell
  similarity kernel — SiFT strongly relies on this prior knowledge and an inaccurate
  or coarse-grained mapping may lead to a loss of signal
- Downstream analysis strictly requires properly log-transformed expression — SiFT
  modifies gene expression so the filtered data is no longer properly log-transformed
- The goal is generic batch correction rather than filtering a specific biological
  (disease) signal; use dedicated integration methods (bbknn, Harmony, scVI) directly
  instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Computes the probabilistic cell-cell similarity kernel, projects expression
    onto the healthy signal, and deducts the projection to produce the filtered matrix
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.kernels.KnnKernel
  role: Defines the kNN similarity kernel mapping each disease cell to its nearest
    healthy reference cells in the harmonized PCA space
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.sifter
  role: High-level SiFT entry point used to run the filtering routine with the KNN
    kernel
  repo: https://github.com/nitzanlab/sift-sc
- name: bbknn
  role: Batch-correction integration to harmonize disease and healthy cells in a joint
    PCA latent space so healthy cells can act as reference
- name: pip
  role: Installs the SiFT package from PyPI
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:247d5b28241b624e9863e2cd345be7a235e52225826defe1112f3a4ecfa97b80
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# disease-informative-subpopulation-identification-across-cell-types

## Summary

Identifies disease-informative subpopulations across annotated cell types by SiFT-filtering the healthy reference signal from case PBMC scRNA-seq data, then classifying cells as disease-informative vs non-informative via neighborhood cluster purity (tau = 0.55). It recovers disease-related dynamics (e.g., interferon-response in monocytes, DCs, HSPC) that the original data fails to expose.

## When to use

Use when case-control single-cell RNA-seq data have a healthy control population usable as a reference trajectory, and the goal is to expose disease-related cells and dynamics masked by healthy shared signal.

## When NOT to use

- No healthy reference samples or prior knowledge are available to define the cell-cell similarity kernel — SiFT strongly relies on this prior knowledge and an inaccurate or coarse-grained mapping may lead to a loss of signal
- Downstream analysis strictly requires properly log-transformed expression — SiFT modifies gene expression so the filtered data is no longer properly log-transformed
- The goal is generic batch correction rather than filtering a specific biological (disease) signal; use dedicated integration methods (bbknn, Harmony, scVI) directly instead

## Inputs

- Case-control single-cell RNA-seq count data (e.g., Stephenson et al. COVID-19 PBMC disease and healthy control samples)
- Cell type annotations (CD14+ monocytes, CD16+ monocytes, Plasmablasts, Platelets, DCs, HSPC, etc.)
- Harmonized joint PCA latent representation of disease and healthy cells
- Interferon (IFN) response gene set for enrichment scoring
- Purity threshold tau = 0.55 for informative-cell classification

## Outputs

- Filtered gene expression matrix for disease cells with the healthy reference signal deducted
- Per-cell disease-informative vs non-informative classification labels (cluster purity > tau = 0.55)
- Per-cell-type aggregate of informative-cell labels identifying disease-informative subpopulations (CD14+ mono., CD16+ mono., Plasmablast, Platelet)
- IFN-response enrichment scores per cell type on the filtered data (monocytes, DCs, HSPC highlighted)

## How to apply

Load the case-control scRNA-seq dataset (e.g., Stephenson et al. PBMC COVID-19 with disease and healthy control samples) and integrate disease and healthy cells into a joint PCA latent space using a batch-correction method such as bbknn-style neighbor correction so healthy cells can serve as a reference. Build a cell-cell similarity kernel with sift.kernels.KnnKernel in the harmonized latent space, mapping each disease cell to its nearest healthy reference cells to encode the healthy signal to be removed. Apply sift.SiFT (sift.sifter) to project gene expression onto this kernel and deduct the projection, yielding a filtered expression matrix for the disease cells. Run IFN-response gene-set enrichment on the filtered data — expect enrichment in monocytes, dendritic cells, and HSPC, which the same analysis on the original data fails to expose. Compute cluster purity over the filtered neighborhood structure and label cells disease-informative vs non-informative using the paper's purity threshold tau = 0.55. Finally aggregate informative-cell labels per annotated cell type to recover disease-informative subpopulations in CD14+ monocytes, CD16+ monocytes, Plasmablasts, and Platelets.

## Related tools

- **SiFT (sift-sc)** (Computes the probabilistic cell-cell similarity kernel, projects expression onto the healthy signal, and deducts the projection to produce the filtered matrix) — https://github.com/nitzanlab/sift-sc
- **sift.kernels.KnnKernel** (Defines the kNN similarity kernel mapping each disease cell to its nearest healthy reference cells in the harmonized PCA space) — https://github.com/nitzanlab/sift-sc
- **sift.sifter** (High-level SiFT entry point used to run the filtering routine with the KNN kernel) — https://github.com/nitzanlab/sift-sc
- **bbknn** (Batch-correction integration to harmonize disease and healthy cells in a joint PCA latent space so healthy cells can act as reference)
- **pip** (Installs the SiFT package from PyPI)

## Examples

```
pip install sift-sc; import sift; kernel = sift.kernels.KnnKernel(); sifter = sift.sifter(); sifter.fit(db=args, kernel=kernel)
```

## Evaluation signals

- IFN-response gene-set enrichment on the filtered data is concentrated in monocytes, dendritic cells, and HSPC, whereas the same analysis on the original (unfiltered) data fails to expose these cell types
- Cluster-purity classification with tau = 0.55 recovers disease-informative subpopulations in CD14+ monocytes, CD16+ monocytes, Plasmablasts, and Platelets, reproducing the paper's Fig. 6-7 summary
- The joint latent representation shows disease and healthy cells harmonized (bbknn-style corrected neighborhoods) before kernel construction
- Filtered expression values are original minus kernel-projected expression, i.e., the healthy signal has been deducted at the gene-expression level
- Qualitative agreement with the paper's figures generated from the healthy-trajectory-filtered data (enriched IFN response + informative subpopulation recovery)

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- No ground truth reference is available to evaluate computational filtering results; integration metrics used for evaluation may be application-specific
- Filtered data is no longer properly log-transformed, a property expected by certain downstream analysis tasks; statistical implications of applying common tools to corrected expression are unexplored

## Evidence

- [intro] Healthy samples are used as a reference to filter the healthy signal and identify disease-related dynamics: "Healthy samples are used as a reference to filter the healthy signal and identify disease-related dynamics"
- [discussion] COVID-19 discussion quote: "SiFT exposed disease-related signals and single-cell dynamics by filtering a corresponding healthy trajectory obtained by mapping to reference healthy samples."
- [discussion] kernel prior-knowledge dependency: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [discussion] log-transform caveat: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
- [intro] workflow objective: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
