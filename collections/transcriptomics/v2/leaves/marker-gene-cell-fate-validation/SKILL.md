---
name: marker-gene-cell-fate-validation
description: Use when a trajectory-inference run has auto-detected terminal states
  that must be biologically labeled and validated, especially for rare populations
  like delta cells.
when_to_use_negative:
- Terminal states are already manually assigned or supplied as user-defined labels,
  so no marker-based disambiguation is needed
- The dataset lacks known, fate-specific marker genes, leaving no independent biological
  ground truth for validation
- The goal is global topology assessment rather than fate-level annotation of detected
  terminal clusters
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: Auto-detects terminal states and produces terminal_clusters and single_cell_bp
    branch probabilities that are subsequently validated by marker expression
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph; cluster granularity (too_big_factor=0.4)
    determines whether rare fates like delta cells are resolved
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Embedding for single-cell visualization of terminal-state marker expression
- name: scanpy
  role: AnnData assembly and marker-gene expression extraction for fate annotation
- name: scVelo
  role: Distributes the Bastidas-Ponce E15.5 pancreatic endocrine dataset used as
    input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:dbb4809eaf8c2dc5a9112409287a7df49935b4d1dd058acaa2ef094b1b0091b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - UMAP
  - VIA
  - scVelo
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Marker-gene cell-fate validation

## Summary

Validates automatically inferred terminal cell states (e.g. VIA terminal clusters) by assigning each fate to a lineage via canonical marker-gene expression, as done for the E15.5 pancreatic endocrine lineages (Alpha, Beta-1, Beta-2, Delta, Epsilon) using Sst, Hhex, Cd24a, Ins1, Ins2, and Gcg.

## When to use

Use when a trajectory-inference run has auto-detected terminal states that must be biologically labeled and validated, especially for rare populations like delta cells.

## When NOT to use

- Terminal states are already manually assigned or supplied as user-defined labels, so no marker-based disambiguation is needed
- The dataset lacks known, fate-specific marker genes, leaving no independent biological ground truth for validation
- The goal is global topology assessment rather than fate-level annotation of detected terminal clusters

## Inputs

- AnnData object with scRNA-seq counts and PCA stored in adata.obsm['X_pca'] (e.g. scVelo-distributed Bastidas-Ponce E15.5 pancreatic endocrine dataset)
- Completed VIA run with via_object.terminal_clusters and via_object.single_cell_bp branch probabilities
- Curated marker-gene list per candidate fate (e.g. Sst, Hhex, Cd24a, Ins1, Ins2, Gcg)
- UMAP embedding (umap-learn, e.g. random_state=42, n_neighbors=15) for single-cell visualization

## Outputs

- Enumerated list of auto-detected terminal clusters with fate assignments (Alpha, Beta-1, Beta-2, Delta, Epsilon)
- Marker-expression evidence table per terminal state (e.g. delta fate high in Hhex/Sst/Cd24a)
- Pseudotime ordering of sub-fates (Beta-2 immature preceding Beta-1 mature)
- Prevalence estimate of rare fates (e.g. ~3% delta population)

## How to apply

After running VIA without user-defined terminal states (relying on the default pseudotime_threshold_TS=30 percentile criterion and too_big_factor=0.4 for cluster granularity), extract via_object.terminal_clusters and via_object.single_cell_bp branch probabilities to enumerate the detected terminal states. Define a marker-gene list for candidate fates (e.g. Sst, Hhex, Cd24a for delta; Ins1, Ins2 for beta; Gcg for alpha) analogous to the marker_genes list passed to via_wrapper. Assign each terminal cluster to a lineage by which markers are exclusively or dominantly elevated in its cells. For rare fates, confirm the terminal state captures a population of the expected prevalence (e.g. ~3% delta cells with elevated Hhex/Sst/Cd24a). For related sub-fates, use pseudotime ordering to confirm maturity (e.g. Beta-2, Ins2+Ins1-low immature, should precede Beta-1, Ins1+Ins2+ mature).

## Related tools

- **VIA** (Auto-detects terminal states and produces terminal_clusters and single_cell_bp branch probabilities that are subsequently validated by marker expression) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm underlying VIA's cluster-graph; cluster granularity (too_big_factor=0.4) determines whether rare fates like delta cells are resolved) — https://github.com/ShobiStassen/VIA
- **UMAP** (Embedding for single-cell visualization of terminal-state marker expression)
- **scanpy** (AnnData assembly and marker-gene expression extraction for fate annotation)
- **scVelo** (Distributes the Bastidas-Ponce E15.5 pancreatic endocrine dataset used as input)

## Examples

```
via.via_wrapper(adata, true_label, adata.obsm['X_pca'][:,:2], knn=10, ncomps=20, jac_std_global=0.15, root=[42], marker_genes=['Sst','Hhex','Cd24a','Ins1','Ins2','Gcg']); fates = via_obj.terminal_clusters; bp = via_obj.single_cell_bp
```

## Evaluation signals

- The run recovers exactly five auto-detected terminal clusters corresponding to the five endocrine lineages (Alpha, Beta-1, Beta-2, Delta, Epsilon)
- The Delta terminal state matches the rare (~3%) population with exclusively elevated Hhex, Sst, and Cd24a
- Beta-2 (Ins2+Ins1-low immature) precedes Beta-1 (Ins1+Ins2+ mature) in pseudotime
- Each terminal cluster's marker profile is exclusive to its assigned fate (no mixed marker dominance)
- No terminal cluster is left unassigned or assigned to multiple fates with comparable branch probabilities

## Limitations

- Marker-based fate labels depend on the completeness of the curated marker list; missing markers can leave terminal clusters ambiguous
- VIA's automatic terminal-state detection depends on filter parameters (pseudotime_threshold_TS, too_big_factor, small_pop); wrong granularity can merge rare fates or split common ones
- Rare populations (~3% delta cells) are sensitive to clustering granularity; the paper notes other methods exhibit low sensitivity to cell fates

## Evidence

- [results] VIA automatically detects all hormone-producing pancreatic lineages, including the rare delta cells with exclusively elevated Hhex, Sst, and Cd24a, without manual assignment.: [evidence span withheld — anchor status: not_found]
- [methods] Pseudotime threshold for terminal states: [evidence span withheld — anchor status: not_found]
- [methods] too_big_factor granularity: [evidence span withheld — anchor status: not_found]
- [intro] Sensitivity of other methods to cell fates: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [methods] VIA wrapper invocation: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [discussion] Rare delta cell identification in pancreas: "VIA distinguished between dendritic subtypes in an scRNA-seq hematopoiesis dataset; identified the rare delta cell islet in pancreatic development"
