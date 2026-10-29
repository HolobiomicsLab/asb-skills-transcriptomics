---
name: single-cell-mosaic-multiomics-integration
description: Use when a dataset contains multiple single-cell batches that jointly
  profile different subsets of modalities (scRNA-seq, scATAC-seq, CITE-seq protein,
  chromVAR motifs), especially with unequal cell-type composition across batches.
when_to_use_negative:
- All batches jointly profile the same set of modalities (horizontal/diagonal/vertical
  integration only) — methods like LIGER, Seurat v3/v4, MultiVI, CoupleNMF, MMD-MA,
  scDART, or MultiMAP may be more appropriate
- Input consists of feature/bio-marker tables rather than raw per-cell count matrices
  that need to be stored in the counts dictionary format
- One jointly profiled batch already bridges all modalities and dictionary-learning
  bridge integration suffices
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMoMaT
  role: 'Primary method: matrix tri-factorization for mosaic integration and bio-marker
    detection; installed via pip install scmomat'
  repo: https://github.com/PeterZZQ/scMoMaT
- name: LIGER
  role: Baseline/benchmarking context for horizontal and diagonal integration (integrative
    NMF)
  repo: https://github.com/welch-lab/liger
- name: MultiMAP
  role: Baseline method for multi-omics integration comparison (e.g., MultiMap benchmark
    on PBMC)
  repo: https://github.com/Teichlab/MultiMAP
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:de58added1418d98175f0b4779ae9f99fdaa92db5da8dac4692f8cbd1346e34d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - LIGER
  - MultiMAP
  - NumPy
  - PyTorch
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-mosaic-multiomics-integration

## Summary

Integrate arbitrary numbers of single-cell multi-omics batches under the mosaic scenario (each batch profiles a different subset of modalities) using matrix tri-factorization with scMoMaT, producing a unified cell-factor embedding and cluster-specific multi-modal bio-markers.

## When to use

Use when a dataset contains multiple single-cell batches that jointly profile different subsets of modalities (scRNA-seq, scATAC-seq, CITE-seq protein, chromVAR motifs), especially with unequal cell-type composition across batches.

## When NOT to use

- All batches jointly profile the same set of modalities (horizontal/diagonal/vertical integration only) — methods like LIGER, Seurat v3/v4, MultiVI, CoupleNMF, MMD-MA, scDART, or MultiMAP may be more appropriate
- Input consists of feature/bio-marker tables rather than raw per-cell count matrices that need to be stored in the counts dictionary format
- One jointly profiled batch already bridges all modalities and dictionary-learning bridge integration suffices

## Inputs

- Mosaic multi-omics count matrices (scRNA-seq, scATAC-seq, protein/ADT, motif deviation) organized per batch, shape (ncells, nfeatures) per matrix
- counts dictionary with nbatches, feats_name per modality, and per-modality lists with None placeholders for missing batch-modality slots
- Connection matrix GxR.npz for generating pseudo-counts of missing modalities
- Reference cell-type labels (e.g., Mimitou et al. PBMC annotations) for benchmarking

## Outputs

- Cell-factor embedding zs (nfields of K=30 latent dimensions per cell), saveable as .npy
- Post-processed KNN graph: s_pair_dist, knn_indices, knn_dists
- Leiden cluster labels (resolution=0.9)
- 2D UMAP embedding (ncells, 2) and joint latent plot
- Cluster-specific marker scores (score_feats) from extract_feature_factors(), e.g., top-20 marker genes per cluster
- Benchmark metrics (GC, ARI, NMI) against reference cell-type labels

## How to apply

Assemble all count matrices into a counts dictionary with keys nbatches, feats_name, and one list per modality ordered by batch ID, inserting None placeholders for batch-modality slots that are missing; match feature sets per modality (gene intersection or peak remapping) and preprocess each matrix with utils.preprocess(). Generate pseudo-counts for missing modalities using the provided GxR.npz connection matrix, then run the first-stage model scmomat.scmomat_model(counts, K=30, device) for T=4000 iterations (lambda default 0.001) and extract cell factors with model.extract_cell_factors(). Post-process cell factors into a KNN graph with scmomat.post_process(zs, n_neighbors=30, njobs=8, r=None), Leiden-cluster at resolution=0.9, and visualize with scmomat.UMAP (n_components=2, min_dist=0.20, metric='precomputed'). Optionally run the second stage with scmomat.scmomat_retrain(model, counts, labels=labels_leiden) for T=2000 iterations and extract cluster-specific marker scores via model2.extract_feature_factors() to annotate clusters (e.g., Rorb -> L4/5, Sulf1/Foxp2 -> L6) in the MOp or PBMC datasets.

## Related tools

- **scMoMaT** (Primary method: matrix tri-factorization for mosaic integration and bio-marker detection; installed via pip install scmomat) — https://github.com/PeterZZQ/scMoMaT
- **LIGER** (Baseline/benchmarking context for horizontal and diagonal integration (integrative NMF)) — https://github.com/welch-lab/liger
- **MultiMAP** (Baseline method for multi-omics integration comparison (e.g., MultiMap benchmark on PBMC)) — https://github.com/Teichlab/MultiMAP

## Examples

```
import scmomat, torch; model = scmomat.scmomat_model(counts=counts, K=30, device=torch.device('cuda:0')); model.train_func(T=4000); zs = model.extract_cell_factors(); s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, 30, njobs=8, r=None)
```

## Evaluation signals

- First- and second-stage training losses decrease over T=4000 and T=2000 iterations respectively
- Every cell receives exactly one Leiden label; s_pair_dist, knn_indices, knn_dists are returned for all cells and the UMAP embedding has shape (ncells, 2)
- On the PBMC 4-batch dataset, GC/ARI/NMI against Mimitou et al. labels outperform MultiMap, UINMF, and StabMap baselines
- Top markers reproduce expected cortical cell-type patterns in MOp: Rorb -> L4/5, Calb1 -> L2/3, Tshz2 -> near-projecting, Sulf1/Foxp2 -> L6
- NMI/ARI are stable across lambda {1e-4,1e-3,1e-2}, k {15,30,50}, r {0.7,0.9,1}, and plateau once latent dimension d exceeds a dataset-specific threshold

## Limitations

- Cross-modality feature relationships are not learned within the integration framework (deferred to future work); the second stage can only optionally include additional feature modalities such as motifs
- Latent dimension K (d) must be large enough for the dataset — NMI/ARI plateau only after d reaches a threshold, so too-small K degrades integration
- Testing datasets other than PBMC and MOp_5batches (BMMC healthy, Spleen) are available only upon request
- Post-processing can suffer memory overflow on large data; v0.2.2 updated this step to address the issue

## Evidence

- [abstract] Mosaic integration via tri-factorization: "scMoMaT, a method that is able to integrate single cell multi-omics data under the mosaic integration scenario using matrix tri-factorization"
- [abstract] Bio-marker detection for cluster annotation: "scMoMaT is also able to uncover the cluster specific bio-markers across modalities. These multi-modal bio-markers are used to interpret and annotate the clusters to cell types."
- [readme] Counts dictionary structure: "The matrices must be ordered according to the batch ID in each list, so that scMoMaT can detact parallel sequenced batches."
- [readme] First-stage training parameters: "# number of latent dimensions (20~30)
K = 30
# number of iterations
T = 4000"
- [readme] Post-processing and clustering call: "s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors, njobs = 8, r = r)"
- [methods] Second-stage marker extraction: "model2 = scmomat.scmomat_retrain(model = model, counts =  counts, labels = labels_leiden, device = device)"
