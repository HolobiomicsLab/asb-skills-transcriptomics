---
name: gobp-gene-perturbation-simulation
description: 'Use when benchmarking batch-correction/integration methods for over-correction
  risk: you need a known implanted biological signal (GOBP term in one cell type of
  one batch) that a good integrator must preserve, not erase.'
when_to_use_negative:
- The goal is to measure batch-effect removal strength rather than over-correction
  risk — an unperturbed benchmark (e.g. scIB metrics on the original data) is the
  right setup, since the implanted signal here deliberately confounds batch with biology
- The dataset lacks both cell-type annotations and batch labels, so the perturbation
  cannot be confined to a specific cell type within a single batch
- The biological question concerns a real disease/condition contrast (e.g. healthy
  vs COVID-19 samples) rather than a simulated implanted signal
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: preprocessing (filtering, normalize to 10,000 counts with log1p, 2000 HVG
    selection) and UMAP embedding used for over-correction scoring
- name: scCobra
  role: primary batch-correction method evaluated on the perturbed data; expected
    to show the lowest over-correction score
  repo: https://github.com/mcgilldinglab/scCobra
- name: Seurat
  role: benchmark batch-correction method run with default parameters on the perturbed
    dataset
- name: Harmony
  role: benchmark batch-correction method run with default parameters on the perturbed
    dataset
- name: Scanorama
  role: benchmark batch-correction method run with default parameters on the perturbed
    dataset
- name: scVI
  role: benchmark deep-learning batch-correction method (scvi-tools v0.11) run on
    the perturbed dataset
- name: scDML
  role: benchmark batch-correction method run with default parameters on the perturbed
    dataset
- name: scDREAMER
  role: benchmark batch-correction method run with default parameters on the perturbed
    dataset
- name: scIB
  role: pipeline for Leiden clustering (resolutions 0.1–2.0 in 0.1 increments) supporting
    evaluation of integrated outputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:326fd0a7abe35a6b134c669f584ae388ce21effbe45402446d47438b38fab751
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gobp-gene-perturbation-simulation

## Summary

Implants a controlled biological signal into a single-cell dataset by increasing expression of genes annotated to a specific GOBP term (e.g. 'Defense response to Virus') in a defined cell type and batch, creating a perturbed dataset to test whether batch-correction methods over-correct real biology.

## When to use

Use when benchmarking batch-correction/integration methods for over-correction risk: you need a known implanted biological signal (GOBP term in one cell type of one batch) that a good integrator must preserve, not erase.

## When NOT to use

- The goal is to measure batch-effect removal strength rather than over-correction risk — an unperturbed benchmark (e.g. scIB metrics on the original data) is the right setup, since the implanted signal here deliberately confounds batch with biology
- The dataset lacks both cell-type annotations and batch labels, so the perturbation cannot be confined to a specific cell type within a single batch
- The biological question concerns a real disease/condition contrast (e.g. healthy vs COVID-19 samples) rather than a simulated implanted signal

## Inputs

- Preprocessed scRNA-seq count matrix (AnnData/h5ad) with batch and cell-type labels — e.g. immune dataset GSE149614 (33,506 cells, 16 cell types, 10 batches)
- GOBP term gene sets (e.g. genes annotated to 'Defense response to Virus')
- Specification of target cell type (CD4+ T cells) and target batch ('10X') to receive the perturbation
- Perturbation magnitude (increase expression by 1)

## Outputs

- Perturbed expression matrix in which GOBP 'Defense response to Virus' genes are upregulated by 1 in CD4+ T cells of the 10X batch
- Over-correction scores per batch-correction method (normalized, averaged over 5 seeds) for all cell types and for the perturbed CD4+ T cell subset, e.g. written to overcorrection_scores.csv (scCobra 0.043, scVI 0.086, Scanorama 0.012; CD4+ T subset: scCobra 0.038, scVI 0.158, scDML 0.278)

## How to apply

Load and preprocess the target dataset per the scCobra protocol — filter cells with fewer than 200 expressed genes, filter genes detected in fewer than 3 cells, remove cells with mitochondrial fraction above 5%, normalize to 10,000 counts per cell with log1p, and select 2000 highly variable genes using Scanpy. Then apply the perturbation: increase the expression of genes annotated to the chosen GOBP term (here 'Defense response to Virus') by 1, but only in cells of a designated cell type (CD4+ T cells) belonging to a single batch (the '10X' batch of the immune dataset GSE149614). This confines the biological difference to one batch so that naive batch correction that ignores biology will erase it, while a correct integrator preserves it. Run the batch-correction methods (scCobra, Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER with default parameters, 5 random seeds) on the perturbed data, compute over-correction scores from 2D UMAP embeddings following the SCALEX methodology, normalize, and compare across methods and cell-type subsets.

## Related tools

- **Scanpy** (preprocessing (filtering, normalize to 10,000 counts with log1p, 2000 HVG selection) and UMAP embedding used for over-correction scoring)
- **scCobra** (primary batch-correction method evaluated on the perturbed data; expected to show the lowest over-correction score) — https://github.com/mcgilldinglab/scCobra
- **Seurat** (benchmark batch-correction method run with default parameters on the perturbed dataset)
- **Harmony** (benchmark batch-correction method run with default parameters on the perturbed dataset)
- **Scanorama** (benchmark batch-correction method run with default parameters on the perturbed dataset)
- **scVI** (benchmark deep-learning batch-correction method (scvi-tools v0.11) run on the perturbed dataset)
- **scDML** (benchmark batch-correction method run with default parameters on the perturbed dataset)
- **scDREAMER** (benchmark batch-correction method run with default parameters on the perturbed dataset)
- **scIB** (pipeline for Leiden clustering (resolutions 0.1–2.0 in 0.1 increments) supporting evaluation of integrated outputs)

## Examples

```
# after Scanpy preprocessing of GSE149614 h5ad:
adata.X[adata.obs.celltype == 'CD4+ T cells'].X[:, gobp_virus_genes] += 1  # 10X batch only
```

## Evaluation signals

- After perturbation, GOBP 'Defense response to Virus' genes show elevated expression (+1) exclusively in CD4+ T cells of the 10X batch and are unchanged elsewhere
- A low-over-correction method should score ≈ scCobra values: ~0.043 (all cell types) and ~0.038 (CD4+ T subset), matching Fig. 3c reference values within tolerance
- A signal-erasing method should show elevated scores in the CD4+ T subset (e.g. scVI ≈ 0.158, scDML ≈ 0.278), confirming the perturbation was detectable
- Scores should be stable when recomputed across 5 random seeds (values reported are seed averages) and reproducible from 2D UMAP embeddings per the SCALEX methodology
- The normalized score table should contain entries for all seven methods for both the all-cell-types and CD4+ T cell subset views

## Limitations

- Over-correction scores are derived from two-dimensional UMAP embeddings (following SCALEX), introducing dependence on UMAP and its stochasticity
- Benchmark methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- The simulation relies on a simplistic +1 expression shift of one GOBP term in one cell type/batch, which may not capture realistic biological differences between batches

## Evidence

- [results] Perturbation protocol: [evidence span withheld — anchor status: not_found]
- [intro] Over-correction motivation: "this category of methods often over-corrects the batch effects since they tend to ignore the potential inherent difference associated with different batches"
- [discussion] Simulation rationale: "We utilized the immune dataset to simulate a scenario for examining the risk of over-correction"
- [discussion] Preprocessing thresholds: "we first filtered out cells that expressed fewer than 200 genes"
- [readme] README dataset description: [evidence span withheld — anchor status: not_found]
