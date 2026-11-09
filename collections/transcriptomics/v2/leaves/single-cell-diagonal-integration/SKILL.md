---
name: single-cell-diagonal-integration
description: Use when you have unpaired single-cell modality pairs (e.g. scRNA-scATAC,
  scRNA-protein, scRNA-smFISH, scRNA-morphology) from the same biological condition
  and need a shared cell embedding without converting features across modalities.
when_to_use_negative:
- No cross-modality feature connections are available for the modality pair — the
  paper notes scConfluence 'relies on connections between features of different modalities...
  as for example when integrating electrophysiology measurements with gene expression'
- Data are paired multimodal measurements of the same cells; the paper states integration
  of paired with unpaired multimodal data 'is not yet addressed by scConfluence'
- The primary goal involves very rare cell populations, since rare populations are
  unlikely to be co-sampled in mini-batches and this remains challenging for scConfluence
  and all state-of-the-art methods
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: 'Core method: uncoupled autoencoders per modality plus regularized Inverse
    Optimal Transport on the weakly connected feature cost matrix to produce shared
    latent embeddings'
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: MuData I/O, modality-specific normalization (tfidf for scATAC, Center Log
    Ratio for protein), and QC filtering via filter_obs/filter_var
- name: Scanpy
  role: scRNA log-normalization (normalize_total to 10,000 + log1p), neighbor graph
    and UMAP on the latent embedding
- name: SciPy
  role: scipy.spatial.distance.cdist to compute the cross-modality cost matrix from
    weakly connected features
- name: Seurat v4.3.0
  role: 'Baseline: anchor-based integration of the same unpaired benchmark splits'
- name: rliger v1.0.0
  role: 'Baseline: iNMF integration compared against scConfluence'
- name: MultiMAP
  role: Baseline diagonal integration method run on the same benchmark datasets
- name: Uniport
  role: Baseline optimal-transport-based diagonal integration method
- name: scGLUE
  role: Baseline graph-linked unpaired multi-omics integration method
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:3a9876aedf3399b67cb035eb56669e58f96531abf41013666c14a722c87ca983
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Liger
  - MultiMAP
  - Muon
  - Scanpy
  - SciPy
  - Seurat
  - Seurat v4.3.0
  - Uniport
  - rliger v1.0.0
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# single-cell-diagonal-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates unpaired single-cell multimodal data (diagonal integration) using scConfluence, which combines uncoupled autoencoders with regularized Inverse Optimal Transport (rIOT) on weakly connected features to learn a shared low-dimensional latent space across modalities such as scRNA-seq, scATAC-seq, surface protein, smFISH, and neuronal morphology images.

## When to use

Use when you have unpaired single-cell modality pairs (e.g. scRNA-scATAC, scRNA-protein, scRNA-smFISH, scRNA-morphology) from the same biological condition and need a shared cell embedding without converting features across modalities.

## When NOT to use

- No cross-modality feature connections are available for the modality pair — the paper notes scConfluence 'relies on connections between features of different modalities... as for example when integrating electrophysiology measurements with gene expression'
- Data are paired multimodal measurements of the same cells; the paper states integration of paired with unpaired multimodal data 'is not yet addressed by scConfluence'
- The primary goal involves very rare cell populations, since rare populations are unlikely to be co-sampled in mini-batches and this remains challenging for scConfluence and all state-of-the-art methods

## Inputs

- MuData object with unpaired single-cell modality pairs (e.g. scRNA-seq and scATAC-seq counts, CITE-seq protein CLR-normalized, smFISH, or morphology features)
- Cross-modality cost matrix computed on weakly connected features (scipy cdist), stored in mdata.uns with the cross_keys entry
- Cell type labels for downstream evaluation (purity, transfer accuracy, FOSCTTM)

## Outputs

- Shared latent cell embeddings (dimension 16) written to mdata.obsm['latent'], aligned to mdata.obs_names
- Joint UMAP/neighbor graph on the latent representation (Scanpy)
- Benchmark metric table (e.g. benchmark_metrics.csv) with method, scenario/dataset, purity, transfer accuracy, connectivity, and FOSCTTM columns
- Cross-modality feature imputations (e.g. held-out smFISH gene spatial patterns)

## How to apply

Load the modalities into a MuData object (mudata.read_h5mu) and preprocess per modality: scRNA log-normalization to 10,000 counts plus log1p with the first 100 PCs as encoder input, TF-IDF for scATAC via Muon, and Centered Log Ratio for protein. Compute the cross-modality cost matrix from weakly connected features with scipy.spatial.distance.cdist (Pearson similarity per the paper) and register it in mdata.uns with mdata.uns['cross_keys']. Instantiate one scconfluence AutoEncoder per modality, build the ScConfluence model, and train with model.fit(save_path='results') using latent dimension 16 and the unbalanced Sinkhorn regularization with default transported mass m=0.5 via ot.partial.partial_wasserstein, tuning sinkhorn_loss_weight/reach if modality embeddings separate. Extract embeddings with model.get_latent() into mdata.obsm['latent'], then compute neighbors and UMAP with Scanpy (sc.pp.neighbors(mdata, use_rep='latent'); sc.tl.umap(mdata)) and evaluate with purity, label-transfer accuracy, connectivity, and FOSCTTM against baselines (Seurat v4.3.0 anchors, rliger v1.0.0, MultiMAP, Uniport, scGLUE).

## Related tools

- **scConfluence** (Core method: uncoupled autoencoders per modality plus regularized Inverse Optimal Transport on the weakly connected feature cost matrix to produce shared latent embeddings) — https://github.com/cantinilab/scconfluence
- **Muon** (MuData I/O, modality-specific normalization (tfidf for scATAC, Center Log Ratio for protein), and QC filtering via filter_obs/filter_var)
- **Scanpy** (scRNA log-normalization (normalize_total to 10,000 + log1p), neighbor graph and UMAP on the latent embedding)
- **SciPy** (scipy.spatial.distance.cdist to compute the cross-modality cost matrix from weakly connected features)
- **Seurat v4.3.0** (Baseline: anchor-based integration of the same unpaired benchmark splits)
- **rliger v1.0.0** (Baseline: iNMF integration compared against scConfluence)
- **MultiMAP** (Baseline diagonal integration method run on the same benchmark datasets)
- **Uniport** (Baseline optimal-transport-based diagonal integration method)
- **scGLUE** (Baseline graph-linked unpaired multi-omics integration method)

## Examples

```
import scconfluence, mudata as md; from scipy.spatial.distance import cdist; mdata = md.read_h5mu('data.h5mu'); mdata.uns['cross_keys'] = ['rna','atac']; model = scconfluence.model.ScConfluence(unimodal_aes={'rna': scconfluence.model.AutoEncoder('rna', mdata), 'atac': scconfluence.model.AutoEncoder('atac', mdata)}); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent(k='rna')
```

## Evaluation signals

- mdata.uns contains the cross_keys entry and the cost matrix; mdata.obsm['latent'] is populated and aligned to mdata.obs_names
- Latent embeddings have the expected dimension 16 per cell, and Scanpy neighbors + UMAP on 'latent' show cell types mixed across modalities rather than separated by modality of origin
- On unbalanced cell-line benchmarks (HeLa/HCT/K562 scCATseq, 4 subsampling scenarios), scConfluence ranks first among the 6 methods on all four metrics (purity, transfer accuracy, connectivity, FOSCTTM), reproducing Fig. 2b
- benchmark_metrics.csv contains all 6 methods x 4 scenario rows with finite values for all four metrics
- Spatial imputation of held-out smFISH genes yields Spearman correlations consistent with the reported ranking, and predicted patterns for Scgn, Synpr, and Olah match Fig. 4d-f

## Limitations

- Rare cell populations are hard to handle: they are 'much less likely to be simultaneously sampled from each modality in the mini-batches', and mini-batch optimization is nonetheless required to scale to millions of cells
- scConfluence depends on cross-modality feature connections, which are unavailable for some pairs such as electrophysiology vs gene expression
- Hidden layer sizes were adapted per dataset (Supplementary Table 5), which is not provided in the article, creating a reproducibility gap; latent dimension 16 was used for all datasets
- Integration of paired with unpaired multimodal data is not yet addressed by the method

## Evidence

- [discussion] Unbalanced OT default transported mass m=0.5: [evidence span withheld — anchor status: not_found]
- [discussion] scRNA preprocessing: normalize to 10,000 then log1p; 100 PCs as encoder input: [evidence span withheld — anchor status: not_found]
