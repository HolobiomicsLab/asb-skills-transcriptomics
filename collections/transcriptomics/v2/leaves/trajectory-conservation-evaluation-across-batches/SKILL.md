---
name: trajectory-conservation-evaluation-across-batches
description: Use when integrating multi-batch scRNA-seq data (e.g., ten donor batches
  of human bone marrow/PBMC immune cells) and the key question is whether a known
  developmental continuum across batches survives integration, or when benchmarking
  integration methods on trajectory conservation.
when_to_use_negative:
- The dataset contains only a single batch or no known developmental trajectory to
  conserve, so trajectory-continuum assessment has no reference to compare against
- The goal is purely qualitative cell-type identification rather than benchmarking
  integration methods on trajectory preservation and batch mixing
- The dataset size makes kBET or the eleven competing methods computationally infeasible
  (the article notes kBET could not be computed for some methods on the healthy heart
  atlas due to computational limitations)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDREAMER
  role: Primary integration method whose embedding is evaluated for conservation of
    the HSPC-to-erythrocyte trajectory; adversarial variational autoencoder with batch
    classifier trained with ADAM and batch size 128
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: Preprocessing (AnnData import, mitochondrial QC, normalize_total, log-transform,
    top 2000 HVGs with flavor='seurat') and Louvain clustering for metric computation
- name: INSCT
  role: Competing integration method run on the immune dataset for trajectory-conservation
    comparison; reported to fail to conserve the erythrocyte trajectory
  repo: https://github.com/lkmklsmn/insct
- name: Seurat
  role: MNN-based competing integration method included in the benchmark; fails to
    conserve the erythrocyte trajectory
- name: LIGER
  role: Competing integration method included in the benchmark; fails to conserve
    the erythrocyte trajectory
- name: scDML
  role: Competing integration method included in the benchmark; fails to conserve
    the erythrocyte trajectory
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:02afc06d6c8933a4ee65a18554f269dd10af81ee7778515642499f986643c92c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - BBKNN
  - Harmony
  - INSCT
  - LIGER
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# trajectory-conservation-evaluation-across-batches

## Summary

Evaluates whether a single-cell data-integration method preserves continuous developmental trajectories (e.g., HSPC-to-erythrocyte and monocyte-to-dendritic-cell differentiation in the ten-batch, 33,506-cell human immune dataset) after batch correction, using embedding visualization, subtype mixing, and quantitative bio-conservation metrics.

## When to use

Use when integrating multi-batch scRNA-seq data (e.g., ten donor batches of human bone marrow/PBMC immune cells) and the key question is whether a known developmental continuum across batches survives integration, or when benchmarking integration methods on trajectory conservation.

## When NOT to use

- The dataset contains only a single batch or no known developmental trajectory to conserve, so trajectory-continuum assessment has no reference to compare against
- The goal is purely qualitative cell-type identification rather than benchmarking integration methods on trajectory preservation and batch mixing
- The dataset size makes kBET or the eleven competing methods computationally infeasible (the article notes kBET could not be computed for some methods on the healthy heart atlas due to computational limitations)

## Inputs

- Raw count expression matrix for the multi-batch human immune dataset (33,506 cells, ten batches) as a Scanpy AnnData object
- Batch labels for the ten donor batches
- Cell-type annotations covering bone marrow (HSPCs through erythrocytes) and PBMC cell types

## Outputs

- Per-method integrated embeddings over the 33,506 immune cells
- immune_integration_metrics.csv with per-method NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, kBET (k=50), isolated label F1/ASW, iLISI/cLISI, and composite scores
- PNG figure showing the conserved (or broken) erythrocyte developmental trajectory in the integrated embedding

## How to apply

Obtain the processed human immune dataset (33,506 cells from bone marrow and PBMCs across ten batches) from the benchmarking deposit (Zenodo 10.5281/zenodo.10021936, Figshare). Import raw counts as a Scanpy AnnData object, remove low-quality cells by mitochondrial gene counts, apply scanpy.pp.normalize_total plus log-transform, and select the top 2000 highly variable genes with scanpy.pp.highly_variable_genes (flavor='seurat'). Run the integration methods under test (scDREAMER plus competitors such as scVI, Scanorama, Harmony, Seurat, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, scDML), then cluster each integrated embedding with Louvain clustering at the resolution that maximizes NMI and reuse those assignments for downstream metrics. Compute bio-conservation metrics (NMI, ARI, cell-type ASW) and batch-correction metrics (batch ASW scaled as 1 - abs(ASW), PCR batch, graph connectivity, kBET with k=50, graph iLISI/cLISI), min-max normalize each metric across methods, and average into composite bio-conservation, batch-correction, and combined scores. Finally, assess trajectory conservation qualitatively via embedding visualization and marker-gene/subtype mixing along the HSPC-to-erythrocyte and monocyte-to-DC continua, writing per-method scores to immune_integration_metrics.csv and the embedding/trajectory figure as a PNG.

## Related tools

- **scDREAMER** (Primary integration method whose embedding is evaluated for conservation of the HSPC-to-erythrocyte trajectory; adversarial variational autoencoder with batch classifier trained with ADAM and batch size 128) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing (AnnData import, mitochondrial QC, normalize_total, log-transform, top 2000 HVGs with flavor='seurat') and Louvain clustering for metric computation)
- **INSCT** (Competing integration method run on the immune dataset for trajectory-conservation comparison; reported to fail to conserve the erythrocyte trajectory) — https://github.com/lkmklsmn/insct
- **Seurat** (MNN-based competing integration method included in the benchmark; fails to conserve the erythrocyte trajectory)
- **LIGER** (Competing integration method included in the benchmark; fails to conserve the erythrocyte trajectory)
- **scDML** (Competing integration method included in the benchmark; fails to conserve the erythrocyte trajectory)

## Examples

```
sc.pp.normalize_total(adata); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, n_top_genes=2000, flavor='seurat')
```

## Evaluation signals

- Louvain clustering of each integrated embedding must be done at the resolution that maximizes NMI, and that same resolution's cluster assignments must be reused for ARI and ASW computations
- scDREAMER's integrated embedding should exhibit a perfect continuum of bone marrow cell types from HSPCs to erythrocytes, whereas Seurat, INSCT, LIGER, and scDML embeddings are expected to break the continuum
- Each metric (NMI, ARI, ASW, PCR batch, graph connectivity, kBET with k=50, iLISI/cLISI) is min-max normalized across methods before averaging into bio-conservation, batch-correction, and combined composite scores
- Subclustering and marker gene expression analysis of dendritic cell populations should confirm whether DC subtypes are separated and mixed across batches in the embedding
- Deliverables must include immune_integration_metrics.csv and a PNG trajectory/embedding visualization per method

## Limitations

- The scDREAMER model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- Trajectory conservation is partly assessed qualitatively via embedding visualization and subtype mixing, which is subjective
- Louvain resolution is chosen to maximize NMI, which may bias reuse of the same assignments for other metrics
- kBET could not be computed for some methods on large multi-batch datasets (healthy heart atlas) due to computational limitations
- scDREAMER, being deep-learning based, may require fine-tuning of some parameters for best performance

## Evidence

- [intro] conservation of development trajectory across different batches (human immune integration): "conservation of development trajectory across different batches (human immune integration)"
- [results] Through subclustering and marker gene expression analysis, we wanted to find out if an integration method was able to separate these DC subtypes: "Through subclustering and marker gene expression analysis, we wanted to find out if an integration method was able to separate these DC subtypes"
- [methods] We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value.: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [results] scDREAMER clearly separated tissue-specific cell types and exhibited a perfect continuum of bone marrow cell types after integration: "scDREAMER clearly separated tissue-specific cell types and exhibited a perfect continuum of bone marrow cell types after integration"
- [readme] scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings: "scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings"
