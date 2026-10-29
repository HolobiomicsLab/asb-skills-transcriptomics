---
name: shapley-based-annotation-integration
description: Use when multiple conflicting clustering annotations (e.g., Leiden at
  resolutions 1/2/3 on RNA and ATAC AnnData objects) must be mix-and-matched into
  a single stable, biologically coherent cell-population labeling that consensus ensemble
  methods cannot achieve.
when_to_use_negative:
- When a single clustering resolution already yields the desired discrete cell states
  and no conflicting annotations exist to reconcile
- When the goal is data-level integration into a joint latent space (e.g., Seurat-WNN,
  TotalVI) rather than decision-level annotation reconciliation
- When annotation sets are already standardized reference labels and no mix-and-match
  across modalities/algorithms/resolutions is needed
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTriangulate
  role: 'Core framework: Shapley-value decision-level integration of conflicting cluster
    annotations via lazy_run, stability scoring, and pruning'
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Preprocessing framework (scanpy_recipe) for normalization, HVG selection,
    neighbor graphs, PCA, UMAP, and Leiden clustering that generate input annotations
- name: ClusterEnsembles
  role: Benchmarking comparison methods (HGPA, MCLA, HBGF ensemble clustering) against
    scTriangulate on Homogeneity/Completeness/V-Measure
  repo: https://github.com/tsano430/ClusterEnsembles
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:220c55b5365af27ed358460d292a762282ef27a06f1067ce421fc6bfd9184595
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# shapley-based-annotation-integration

## Summary

scTriangulate integrates conflicting single-cell clustering solutions (across modalities, algorithms, and resolutions) at the decision level using the Shapley value from coalitional game theory combined with cluster stability metrics, producing reconciled cell-population assignments without assuming any distribution of raw underlying values.

## When to use

Use when multiple conflicting clustering annotations (e.g., Leiden at resolutions 1/2/3 on RNA and ATAC AnnData objects) must be mix-and-matched into a single stable, biologically coherent cell-population labeling that consensus ensemble methods cannot achieve.

## When NOT to use

- When a single clustering resolution already yields the desired discrete cell states and no conflicting annotations exist to reconcile
- When the goal is data-level integration into a joint latent space (e.g., Seurat-WNN, TotalVI) rather than decision-level annotation reconciliation
- When annotation sets are already standardized reference labels and no mix-and-match across modalities/algorithms/resolutions is needed

## Inputs

- AnnData object (h5ad) with normalized adata.X (log CPTT RNA, CLR ADT, etc.)
- Two or more conflicting annotation columns in adata.obs (e.g., sctri_rna_leiden_1, sctri_rna_leiden_2, sctri_rna_leiden_3)
- adata.obsm['X_umap'] coordinates for automatic visualization
- Concatenated RNA+other-modality AnnData from scTriangulate.preprocessing.concat_rna_and_other (for multimodal inputs)

## Outputs

- Final reconciled cell-to-cluster annotation assignments (scTriangulate labels) in the output directory
- Per-cluster stability metrics (reassign, TFIDF5, TFIDF10, SCCAF scores) and winning-fraction pruning flags
- UMAP visualizations of triangulated populations
- Per-method Homogeneity, Completeness, and V-Measure tables (e.g., multiome_pbmc_sctriangulate_metrics.csv)

## How to apply

Prepare an AnnData with properly normalized adata.X (log CPTT for RNA, CLR for ADT), at least two conflicting annotation columns in adata.obs, and adata.obsm['X_umap'] (generated via scTriangulate.preprocessing.scanpy_recipe; remove adata.raw if present). Instantiate ScTriangulate with the query annotation columns and call lazy_run: each annotation set is treated as a player, Shapley values determine relative importance, and stability scores (reassign, TFIDF5, TFIDF10, SCCAF) guide selection. Prune unstable clusters using win_fraction_cutoff (default 0.25; 0.35 in the multiome PBMC analysis; add_metrics='tfidf5') and abs_thresh=10 minimum cluster size, then reclassify cells with a nearest-neighbor strategy. When more than 15 annotation sets are provided, Shapley selection is automatically replaced by combined rank-based selection to reduce runtime. For multiome, concatenate normalized RNA and ATAC objects first via scTriangulate.preprocessing.concat_rna_and_other.

## Related tools

- **scTriangulate** (Core framework: Shapley-value decision-level integration of conflicting cluster annotations via lazy_run, stability scoring, and pruning) — https://github.com/frankligy/scTriangulate
- **scanpy** (Preprocessing framework (scanpy_recipe) for normalization, HVG selection, neighbor graphs, PCA, UMAP, and Leiden clustering that generate input annotations)
- **ClusterEnsembles** (Benchmarking comparison methods (HGPA, MCLA, HBGF ensemble clustering) against scTriangulate on Homogeneity/Completeness/V-Measure) — https://github.com/tsano430/ClusterEnsembles

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- Output CSV (e.g., multiome_pbmc_sctriangulate_metrics.csv) exists and contains Homogeneity, Completeness, and V-Measure rows for scTriangulate and comparison methods
- scTriangulate's V-Measure matches or exceeds HGPA, MCLA, and HBGF on the Splatter-simulated ground truth (c1-c5, 3000 cells)
- Recovered cluster count approximates ground truth (5 clusters) without artifact clusters, showing mix-and-match of under- and over-clustered inputs
- Multiome runs retain the reported ~10,991 nuclei after QC and produce clusters corresponding to classical/intermediate/inflammatory CD14 monocytes and CD4 T-cell subsets
- Unstable clusters are correctly flagged: winning fraction wfc < 0.25 (default) labeled unstable; clusters smaller than abs_thresh=10 pruned

## Limitations

- Integration of cluster predictions from dozens of sources significantly increases runtime; over 15 annotation sets triggers automatic rank-based selection instead of Shapley
- Programmatic default analytical settings may not be optimal for all study designs (win_fraction_cutoff and thresholds may need tuning)
- Naming of resulting populations remains largely non-standardized; scTriangulate 0.9.0 does not yet support integration of multiple samples of the same modality or across technologies

## Evidence

- [intro] Shapley value decision-level integration: [evidence span withheld — anchor status: not_found]
- [methods] lazy_run default settings: "We apply the scTriangulate lazy_run function in version 0.9.0 on these four conflicting annotation solutions to derive the final cluster assignment"
- [methods] winning fraction pruning: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [methods] concat multimodal AnnData: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function"
- [readme] README minimum example: "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])
sctri.lazy_run()"
