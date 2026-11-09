---
name: unpaired-multiomics-diagonal-integration
description: Use when cell populations were profiled in separate cells per modality
  (diagonal integration), cross-modality feature connections are rare/noisy, and some
  cell types may be present in only one modality.
when_to_use_negative:
- Modalities have no cross-modality feature connections at all (e.g. electrophysiology
  vs gene expression), since scConfluence relies on connections between features of
  different modalities
- Data are paired (same cells measured in each modality) rather than unpaired — scConfluence
  targets diagonal integration of unpaired data, not paired multimodal joint integration
- A feature table already in a shared feature space is sufficient and forcing conversion
  would destroy modality-specific signal — use a method that avoids converting all
  modalities to common features
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scConfluence
  role: 'Core method: uncoupled per-modality AutoEncoders plus regularized Inverse
    Optimal Transport to learn shared latent embeddings; model built via scconfluence.model.ScConfluence(mdata,
    unimodal_aes=autoencoders) and fitted with model.fit(save_path=''results'')'
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: Provides the MuData object container, tfidf and CLR normalization, and filter_obs/filter_var
    QC filtering utilities for multiomics inputs
- name: Scanpy
  role: 'Post-integration analysis: sc.pp.neighbors with use_rep=''latent'', Louvain
    clustering, sc.tl.umap and sc.pl.umap on inferred embeddings; kNN-based cell-type
    purity scoring'
- name: POT
  role: Computes the unbalanced/partial OT plan via ot.partial.partial_wasserstein
    with default transported mass m=0.5
- name: SciPy
  role: cdist used to compute cross-modality distance matrices from connected features
    for mdata.uns['cross_keys']
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:cf277ef7cda6507793332667f417b77cc6d0937d46b3506870a637d1bdfbdc7f
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
  - Muon
  - POT
  - Scanpy
  - SciPy
  - scConfluence
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# unpaired-multiomics-diagonal-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Diagonal (unpaired) integration of single-cell multiomics modalities (scRNA-seq, scATAC-seq, CyTOF, smFISH, protein) with scConfluence, which combines uncoupled autoencoders with regularized Inverse Optimal Transport (rIOT) on weakly connected features to learn a shared low-dimensional latent embedding.

## When to use

Use when cell populations were profiled in separate cells per modality (diagonal integration), cross-modality feature connections are rare/noisy, and some cell types may be present in only one modality.

## When NOT to use

- Modalities have no cross-modality feature connections at all (e.g. electrophysiology vs gene expression), since scConfluence relies on connections between features of different modalities
- Data are paired (same cells measured in each modality) rather than unpaired — scConfluence targets diagonal integration of unpaired data, not paired multimodal joint integration
- A feature table already in a shared feature space is sufficient and forcing conversion would destroy modality-specific signal — use a method that avoids converting all modalities to common features

## Inputs

- MuData object with one AnnData per modality (scRNA-seq, scATAC-seq, CyTOF/protein), optionally .h5mu read with mudata.read_h5mu
- Cross-modality distance/similarity matrix on connected features stored in mdata.uns['cross_modality1+modality2'] with mdata.uns['cross_keys']
- Normalized per-modality counts (log-normalize_total/log1p + first 100 PCs for scRNA, TF-IDF for scATAC, CLR for protein)
- Batch index annotation for use as decoder covariate

## Outputs

- Shared latent embeddings in mdata.obsm['latent'] aligned to mdata.obs_names (latent dimension 16 by default)
- Neighbor graph and UMAP computed on the 'latent' representation with Scanpy
- Louvain cluster labels and cross-modality subcluster annotations (e.g. memory/naive/plasma B cell subclusters)
- Marker-overlap statistics, e.g. Fisher's exact test table (marker_overlap_fisher_tests.csv) or sensitivity curves (sensitivity_curves.csv)

## How to apply

Load preprocessed per-modality data as a MuData object and populate mdata.uns with a cross-modality cost matrix computed from connected features (e.g. Pearson similarity via scipy cdist) plus 'cross_keys'. Initialize one scconfluence.model.AutoEncoder per modality (log-normalized scRNA counts reduced to the first 100 PCs as encoder input, TF-IDF for scATAC, Center Log Ratio for protein) and assemble a scconfluence.model.ScConfluence model. Fit with model.fit(save_path='results') using the default unbalanced OT plan computed via ot.partial.partial_wasserstein with transported mass m=0.5, which is robust when the level of unbalanceness is unknown. Extract shared embeddings with model.get_latent() into mdata.obsm['latent']; if modality proportions are very unbalanced, sweep m (e.g. 0.3, 0.4, 0.5, 0.6, 0.75) and score each setting with cell-type purity, FOSCTTM, and graph connectivity. Downstream, build a Scanpy neighbor graph on the 'latent' representation (sc.pp.neighbors with use_rep='latent'), cluster with Louvain (e.g. resolution 0.5, finer 0.2 for subclustering B cells and monocytes), and validate cross-modality subclusters via Fisher's exact tests on marker overlaps.

## Related tools

- **scConfluence** (Core method: uncoupled per-modality AutoEncoders plus regularized Inverse Optimal Transport to learn shared latent embeddings; model built via scconfluence.model.ScConfluence(mdata, unimodal_aes=autoencoders) and fitted with model.fit(save_path='results')) — https://github.com/cantinilab/scconfluence
- **Muon** (Provides the MuData object container, tfidf and CLR normalization, and filter_obs/filter_var QC filtering utilities for multiomics inputs)
- **Scanpy** (Post-integration analysis: sc.pp.neighbors with use_rep='latent', Louvain clustering, sc.tl.umap and sc.pl.umap on inferred embeddings; kNN-based cell-type purity scoring)
- **POT** (Computes the unbalanced/partial OT plan via ot.partial.partial_wasserstein with default transported mass m=0.5)
- **SciPy** (cdist used to compute cross-modality distance matrices from connected features for mdata.uns['cross_keys'])

## Examples

```
import scconfluence, mudata as md; mdata = md.read_h5mu('pbmc.h5mu'); model = scconfluence.model.ScConfluence(mdata, unimodal_aes=autoencoders); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent()
```

## Evaluation signals

- The mdata.obsm['latent'] embeddings are returned for all cells and are index-aligned to mdata.obs_names
- Metrics on the shared latent space — cell-type purity (kNN label agreement), FOSCTTM, and graph connectivity — lie in a near-optimal flat region around the default m=0.5 configuration (matching Supplementary Fig. 13 pattern)
- UMAP on the latent representation preserves known cell-type structure and modality mixing, e.g. B cell and monocyte subclusters remain coherent
- Differentially expressed genes of cross-modality subclusters significantly overlap between scRNA and scATAC (reported overlaps: 30 genes for cluster 0, 12 for cluster 1, 232 for cluster 2), confirmed by Fisher's exact tests
- Louvain clustering on the latent space yields biologically coherent clusters at the chosen resolution (0.5 global, 0.2 for subclustering)

## Limitations

- Rare cell populations are hard to handle: they are under-represented in parameter estimation and rarely co-sampled from each modality in mini-batches, though mini-batching is needed to scale to millions of cells
- Relies on cross-modality feature connections, which are not always available (e.g. electrophysiology vs gene expression)
- Integration of paired with unpaired multimodal data is not addressed by scConfluence
- A GPU is not required but strongly recommended; the number of neurons in hidden layers must be adapted to dataset dimensionality (Supplementary Table 5, not provided)

## Evidence

- [discussion] We set by default m=0.5 which produces robust performances when prior information about the level of unbalanceness of the data is not available: [evidence span withheld — anchor status: not_found]
