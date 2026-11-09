---
name: cardiac-progenitor-lineage-annotation
description: Use when a bifurcating or low-population-fate trajectory (e.g., Isl1+
  cardiac progenitors splitting into cardiomyocyte and endothelial branches) must
  be resolved across scRNA-seq, scATAC-seq, or multi-omic-integrated data and standard
  TI tools miss one fate.
when_to_use_negative:
- The trajectory is a simple linear or tree-like topology with well-populated terminal
  fates — tree-based tools like SlingShot or STREAM suffice and VIA's disconnected/multi-root
  machinery is unnecessary
- The dataset is a single connected lineage with no candidate second fate — branch-probability
  outputs and fate-detection scoring would be vacuous
- No published or annotated fate labels exist to score terminal-state detection accuracy
  against
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: VIA
  role: Primary trajectory inference method; detects terminal states and computes
    pseudotime and branch probabilities on the PARC cluster-graph via lazy-teleporting
    random walks
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm that builds the single-cell graph where each node is
    a cluster of single cells, feeding VIA's cluster-graph trajectory computations
  repo: https://github.com/ShobiStassen/VIA
- name: Seurat
  role: Anchor-based integration of the scRNA-seq and scATAC-seq modalities before
    running VIA on the integrated PCA embedding
- name: UMAP
  role: Embedding used for single-cell level visualization of the VIA trajectory
- name: SlingShot
  role: Baseline TI method that fails to resolve either cardiomyocyte or endothelial
    fate on the cardiac progenitor data
- name: Palantir
  role: Baseline fate-detection method that fails to resolve either fate on the cardiac
    progenitor data
- name: STREAM
  role: Baseline TI method that detects multiple spurious branches fragmenting the
    cardiac progenitor structure
- name: Monocle3
  role: Baseline TI method that fails to resolve either cardiomyocyte or endothelial
    fate on the cardiac progenitor data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:5aa566db7b1bfc4c073101255db3702e3db93bc1c204d8ecea5a6a81cc9ac40a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PARC
  - Palantir
  - STREAM
  - Seurat
  - SlingShot
  - UMAP
  - VIA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cardiac-progenitor-lineage-annotation

## Summary

Recovering and scoring the bifurcating Isl1+ cardiac progenitor trajectory (cardiomyocyte vs endothelial fates) with VIA on individual scRNA-seq, scATAC-seq, and Seurat-integrated inputs, where SlingShot, Palantir, STREAM, and Monocle3 merge or fragment the lineages.

## When to use

Use when a bifurcating or low-population-fate trajectory (e.g., Isl1+ cardiac progenitors splitting into cardiomyocyte and endothelial branches) must be resolved across scRNA-seq, scATAC-seq, or multi-omic-integrated data and standard TI tools miss one fate.

## When NOT to use

- The trajectory is a simple linear or tree-like topology with well-populated terminal fates — tree-based tools like SlingShot or STREAM suffice and VIA's disconnected/multi-root machinery is unnecessary
- The dataset is a single connected lineage with no candidate second fate — branch-probability outputs and fate-detection scoring would be vacuous
- No published or annotated fate labels exist to score terminal-state detection accuracy against

## Inputs

- GEO-deposited Jia et al. cardiac progenitor scRNA-seq count matrix (197 cells) with time-stage/fate annotations
- GEO-deposited Jia et al. cardiac progenitor scATAC-seq count matrix (695 cells) with time-stage/fate annotations
- Seurat anchor-based integrated RNA+ATAC object with per-modality and integrated PCA embeddings (adata.obsm['X_pca'])
- Early-stage root cell index (root_user) for pseudotime initialization

## Outputs

- VIA terminal_clusters per input modality (scRNA-seq, scATAC-seq, integrated)
- Single-cell pseudotime (single_cell_pt_markov) for each run
- Branch-probability matrix single_cell_bp (n_cells x n_terminal_states)
- Fig. 5g-style grouped fate-detection accuracy comparison plot (fig5g_fate_detection_accuracy.png) across modalities and methods

## How to apply

Load the GEO-deposited Jia et al. scRNA-seq (197 cells) and scATAC-seq (695 cells) cardiac progenitor count matrices with time-stage/fate annotations, normalize each modality, and build a Seurat anchor-based integration with PCA embeddings stored in adata.obsm['X_pca'] for the VIA input. Run VIA via via_wrapper (knn=10-30, ncomps matching retained PCs, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10) with an early-stage root cell on each of the three inputs; when a modality yields separate lineage components, use via_wrapper_disconnected with root=[r1, r2] and preserve_disconnected=True. Extract terminal_clusters, single_cell_pt_markov (pseudotime), and single_cell_bp (n_cells x n_terminal_states branch probabilities) per run. Score cardiomyocyte and endothelial terminal-state detection against the published fate annotations and compare VIA against SlingShot, Palantir, STREAM, and Monocle3 baselines. Rationale: VIA's lazy-teleporting random walks on a PARC-derived cluster-graph detect less populous terminal fates that tree-based baselines merge into one branch or fragment into spurious branches.

## Related tools

- **VIA** (Primary trajectory inference method; detects terminal states and computes pseudotime and branch probabilities on the PARC cluster-graph via lazy-teleporting random walks) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm that builds the single-cell graph where each node is a cluster of single cells, feeding VIA's cluster-graph trajectory computations) — https://github.com/ShobiStassen/VIA
- **Seurat** (Anchor-based integration of the scRNA-seq and scATAC-seq modalities before running VIA on the integrated PCA embedding)
- **UMAP** (Embedding used for single-cell level visualization of the VIA trajectory)
- **SlingShot** (Baseline TI method that fails to resolve either cardiomyocyte or endothelial fate on the cardiac progenitor data)
- **Palantir** (Baseline fate-detection method that fails to resolve either fate on the cardiac progenitor data)
- **STREAM** (Baseline TI method that detects multiple spurious branches fragmenting the cardiac progenitor structure)
- **Monocle3** (Baseline TI method that fails to resolve either cardiomyocyte or endothelial fate on the cardiac progenitor data)

## Examples

```
via.via_wrapper_disconnected(adata, true_label, embedding=adata.obsm['X_pca'][:, :ncomps], knn=20, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10, root=[r1, r2], preserve_disconnected=True)
```

## Evaluation signals

- VIA recovers both cardiomyocyte and endothelial terminal fates on all three inputs (scRNA-seq, scATAC-seq, integrated), matching published fate annotations
- fig5g_fate_detection_accuracy.png exists and contains accuracy bars for all three input conditions and all compared methods
- VIA detects the less populous cell fate (endothelial) that SlingShot, Palantir, and Monocle3 miss and that STREAM fragments into spurious branches
- Branch-probability matrix single_cell_bp has shape n_cells x n_terminal_states and high probability mass on the fate-consistent terminal state per annotated cell
- Pseudotime ordering (single_cell_pt_markov) is consistent with the early-stage root and time-stage annotations

## Limitations

- Published accuracy metric values for Fig. 5g are deferred to figures; no quantitative accuracy metrics are stated in the article text
- The article does not report software version numbers for VIA or the compared baseline methods
- The article gives no accession identifiers for the biological datasets used for validation, including the cardiac multi-omic data; GEO identifiers must be recovered from the underlying Jia et al. study

## Evidence

- [methods] VIA wrapper parameters: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [methods] Disconnected VIA call: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [methods] VIA input format: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [intro] Baseline sensitivity finding: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [abstract] Rare-fate detection claim: "discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types"
- [methods] Terminal state percentile threshold: [evidence span withheld — anchor status: not_found]
- [readme] Dataset availability: [evidence span withheld — anchor status: external_doc]
