---
name: fate-detection-accuracy-benchmarking
description: Use when comparing whether VIA recovers both branches of a bifurcating
  lineage (e.g., Isl1+ progenitor to cardiomyocyte and endothelial fates) where SlingShot,
  Palantir, STREAM or Monocle3 merge or fragment lineages across scRNA-seq, scATAC-seq
  and Seurat-integrated data.
when_to_use_negative:
- The goal is pseudotime or trajectory topology inference alone rather than automated
  terminal-fate detection benchmarked against known fate labels
- The dataset has a single (non-bifurcating) terminal fate, so there is no multi-fate
  detection accuracy to compare between methods
- No published fate annotations or time-stage labels are available to serve as ground
  truth for scoring terminal-state detection
edam_operation: http://edamontology.org/operation_3890
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: Primary TI method whose automated terminal-fate detection (terminal_clusters,
    single_cell_bp) is benchmarked; run via via_wrapper or via_wrapper_disconnected
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph of cells used for fate
    detection
- name: SlingShot
  role: Baseline fate-detection method that fails to resolve cardiomyocyte or endothelial
    fates
- name: Palantir
  role: Baseline fate-detection method compared against VIA in the accuracy benchmark
- name: STREAM
  role: Baseline TI method that detects multiple spurious branches fragmenting the
    progenitor structure
- name: Monocle3
  role: Baseline TI method that fails to resolve either cardiomyocyte or endothelial
    cell fate
- name: Seurat
  role: Anchor-based integration of scRNA-seq and scATAC-seq modalities to produce
    the integrated PCA embedding input
- name: UMAP
  role: Embedding used for single-cell-level visualization of inferred trajectories
    and fates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:28431c2ab711e6c7f9790dc75282e56e4a91346abaa593fa6f7822a1e507cf23
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
  edam_operation: http://edamontology.org/operation_3890
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# fate-detection-accuracy-benchmarking

## Summary

Benchmarks a trajectory-inference method's ability to automatically detect terminal cell fates (e.g., cardiomyocyte vs endothelial descendants of Isl1+ cardiac progenitors) against baseline TI methods on individual and integrated scRNA-seq/scATAC-seq inputs, reproducing a Fig. 5g-style grouped accuracy comparison.

## When to use

Use when comparing whether VIA recovers both branches of a bifurcating lineage (e.g., Isl1+ progenitor to cardiomyocyte and endothelial fates) where SlingShot, Palantir, STREAM or Monocle3 merge or fragment lineages across scRNA-seq, scATAC-seq and Seurat-integrated data.

## When NOT to use

- The goal is pseudotime or trajectory topology inference alone rather than automated terminal-fate detection benchmarked against known fate labels
- The dataset has a single (non-bifurcating) terminal fate, so there is no multi-fate detection accuracy to compare between methods
- No published fate annotations or time-stage labels are available to serve as ground truth for scoring terminal-state detection

## Inputs

- GEO-deposited Jia et al. scRNA-seq cardiac progenitor count matrix (197 cells) with time-stage/fate annotations
- GEO-deposited Jia et al. scATAC-seq cardiac progenitor count matrix (695 cells) with time-stage/fate annotations
- Seurat anchor-based integrated scRNA-seq/scATAC-seq object with PCA embedding in adata.obsm['X_pca']
- Published fate annotations (cardiomyocyte, endothelial terminal fates) used as ground truth

## Outputs

- Per-run VIA outputs: terminal_clusters, single_cell_pt_markov (per-cell pseudotime), single_cell_bp (n_cells x n_terminal_states branch probabilities)
- Terminal-state detection accuracy scores for VIA vs SlingShot, Palantir, STREAM, Monocle3 per modality
- Grouped accuracy comparison plot (fig5g_fate_detection_accuracy.png) covering scRNA-seq, scATAC-seq, and integrated inputs

## How to apply

Load GEO-deposited Jia et al. scRNA-seq (197 cells) and scATAC-seq (695 cells) cardiac progenitor count matrices with time-stage/fate annotations, normalize each modality, and build Seurat anchor-based integration with PCA embeddings stored in adata.obsm['X_pca'] as VIA input. Run via_wrapper on each of the three inputs (knn=10-30, ncomps matching retained PCs, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10) with an early-stage root cell, and via_wrapper_disconnected with root=[r1, r2] and preserve_disconnected=True when a modality yields separate lineage components. Extract per-run VIA outputs — terminal_clusters, single_cell_pt_markov (pseudotime), and single_cell_bp (n_cells x n_terminal_states branch probabilities). Score detection of the cardiomyocyte and endothelial terminal states against published fate annotations, and run baseline fate-detection methods (SlingShot, Palantir, STREAM, Monocle3) that merge or fragment these lineages. Plot the Fig. 5g-style grouped accuracy comparison across the three input conditions and methods as fig5g_fate_detection_accuracy.png. Rationale: VIA's lazy-teleporting random walks over the PARC cluster-graph detect less populous fates that baselines with low sensitivity to cell fates miss.

## Related tools

- **VIA** (Primary TI method whose automated terminal-fate detection (terminal_clusters, single_cell_bp) is benchmarked; run via via_wrapper or via_wrapper_disconnected) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm underlying VIA's cluster-graph of cells used for fate detection)
- **SlingShot** (Baseline fate-detection method that fails to resolve cardiomyocyte or endothelial fates)
- **Palantir** (Baseline fate-detection method compared against VIA in the accuracy benchmark)
- **STREAM** (Baseline TI method that detects multiple spurious branches fragmenting the progenitor structure)
- **Monocle3** (Baseline TI method that fails to resolve either cardiomyocyte or endothelial cell fate)
- **Seurat** (Anchor-based integration of scRNA-seq and scATAC-seq modalities to produce the integrated PCA embedding input)
- **UMAP** (Embedding used for single-cell-level visualization of inferred trajectories and fates)

## Examples

```
import via as via; v0 = via.via_wrapper(adata, true_label, embedding=adata.obsm['X_pca'][:, :2], knn=30, ncomps=20, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10, root=[r1], preserve_disconnected=True)
```

## Evaluation signals

- The reproduced figure shows VIA recovering both the cardiomyocyte and endothelial terminal fates that baselines merge or fragment, matching the reported Fig. 5g pattern of VIA detecting less populous cell fates missed by other methods
- fig5g_fate_detection_accuracy.png exists and contains accuracy bars for all three input conditions (scRNA-seq, scATAC-seq, integrated)
- single_cell_bp has shape n_cells x n_terminal_states with entries for both cardiomyocyte and endothelial terminal states per run
- Baseline methods (Monocle3, Slingshot, Palantir) fail to resolve at least one fate and STREAM produces multiple spurious branches on these inputs, consistent with the published finding
- Accuracy of VIA terminal-state assignment is scored against published fate annotations per modality, not against simulated labels

## Limitations

- No quantitative accuracy metrics for the compared methods are reported in the article discussion; results defer to supplementary figures, so absolute thresholds must come from the source task card
- No accession identifiers are given for the biological datasets cited for validation (including the cardiac multi-omic dataset); the GEO deposit must be located from the task provenance (Jia et al.)
- No software version numbers are stated for VIA or the compared methods, which may affect reproducibility of the benchmark
- VIA's fate detection depends on the percentile threshold pseudotime_threshold_TS=30 and cluster-population merging (small_pop=10), which may require retuning for datasets with different fate-population imbalance

## Evidence

- [abstract] VIA discovers elusive lineages and less populous cell fates missed by other methods across multiple data types: "discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types"
- [intro] Baseline fate-detection methods exhibit low sensitivity: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [methods] via_wrapper invocation with tuning parameters: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [methods] Disconnected-trajectory handling with multiple roots: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [methods] Terminal-state percentile threshold parameter: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [methods] AnnData PCA input requirement: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [readme] README points to datasets and tutorials for VIA: "Dataset are available in the [Datasets folder](https://github.com/ShobiStassen/VIA/tree/master/Datasets) (smaller files)"
- [full_text] Seurat integration rationale from source card workflow: [evidence span withheld — anchor status: not_found]
