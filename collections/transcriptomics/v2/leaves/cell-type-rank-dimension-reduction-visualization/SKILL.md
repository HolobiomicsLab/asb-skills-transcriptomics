---
name: cell-type-rank-dimension-reduction-visualization
description: Use when a scRank analysis has produced a ranked list of drug-responsive
  cell types (obj@cell_type_rank) and you need to check or display how the ranking
  distributes across single cells on a dimension-reduction embedding.
when_to_use_negative:
- The scRank object has not yet been scored with rank_celltype, so no cell-type ranking
  exists to visualize.
- The input requires paired disease-vs-treated data or differential expression alone,
  since scRank is designed for untreated single-cell data only.
- The drug is an agonist or multi-target compound, where scRank's perturbation model
  (target-gene node deletion for inhibitors) is only preliminary and the ranking visualization
  would be unreliable.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: Provides the plot_dim function to visualize cell-type ranks in dimension-reduction
    space, plus rank_celltype and plot_net used upstream/complementarily
  repo: https://github.com/ZJUFanLab/scRank
- name: Seurat
  role: Supplies the Seurat object input and dimension-reduction embedding on which
    the ranking is visualized
- name: R
  role: Runtime environment for the scRank pipeline and plot export
- name: rTensor
  role: Dependency for scRank's network reconstruction (Constr_net) feeding the ranked
    cell types
  repo: https://github.com/rikenbit/rTensor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:2b236e2905af28806e71e9cfa7b7bf035b521dd0b676947f37967450ee6a9308
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - Seurat
  - rTensor
  - scRank
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-type-rank-dimension-reduction-visualization

## Summary

Visualize the scRank drug-responsive cell-type ranking in dimension-reduction space using the plot_dim function, so that per-cell in silico perturbation scores from rank_celltype can be inspected on a UMAP/t-SNE embedding alongside the exported cell_type_rank table.

## When to use

Use when a scRank analysis has produced a ranked list of drug-responsive cell types (obj@cell_type_rank) and you need to check or display how the ranking distributes across single cells on a dimension-reduction embedding.

## When NOT to use

- The scRank object has not yet been scored with rank_celltype, so no cell-type ranking exists to visualize.
- The input requires paired disease-vs-treated data or differential expression alone, since scRank is designed for untreated single-cell data only.
- The drug is an agonist or multi-target compound, where scRank's perturbation model (target-gene node deletion for inhibitors) is only preliminary and the ranking visualization would be unreliable.

## Inputs

- scRank S4 object with completed rank_celltype scoring stored in obj@cell_type_rank
- Untreated disease-condition single-cell (snRNA-seq/scRNA-seq) count matrix with cell-type metadata (e.g., GEO GSE144136 MDD dlPFC)
- Dimension-reduction embedding (UMAP/t-SNE) computed on the Seurat object

## Outputs

- Dimension-reduction plot (PNG) coloring cells/cell types by drug-perturbation rank
- Ranking table export (e.g., scRank_MDD_fluoxetine_ranking.csv) from obj@cell_type_rank
- Complementary module heatmap PNGs from plot_net(mode='heatmap', celltype=...)

## How to apply

After running CreateScRank, Constr_net, and rank_celltype on a Seurat-derived scRank S4 object, call plot_dim to overlay the per-cell-type perturbation ranking onto the dimension-reduction space of the single-cell dataset. Confirm the top-ranked cell type (e.g., excitatory cluster Ex_9 for fluoxetine targeting SLC6A4 in untreated MDD dlPFC snRNA-seq) visually dominates the embedding, and that the bottom-ranked type (e.g., Inhib_5) shows minimal perturbation signal. Complement the embedding plot with plot_net heatmaps of the drug-target-gene module (initialized via init_mod) to verify module-2 activity differs between top- and bottom-ranked cell types. Export the ranking table (e.g., scRank_MDD_fluoxetine_ranking.csv) and plots as PNG files for reproducibility.

## Related tools

- **scRank** (Provides the plot_dim function to visualize cell-type ranks in dimension-reduction space, plus rank_celltype and plot_net used upstream/complementarily) — https://github.com/ZJUFanLab/scRank
- **Seurat** (Supplies the Seurat object input and dimension-reduction embedding on which the ranking is visualized)
- **R** (Runtime environment for the scRank pipeline and plot export)
- **rTensor** (Dependency for scRank's network reconstruction (Constr_net) feeding the ranked cell types) — https://github.com/rikenbit/rTensor

## Examples

```
R> obj <- rank_celltype(obj); plot_dim(obj); plot_net(obj, mode='heatmap', celltype='Ex_9'); write.csv(obj@cell_type_rank, 'scRank_MDD_fluoxetine_ranking.csv')
```

## Evaluation signals

- obj@cell_type_rank is populated and its top entry matches the expected drug-responsive type (Ex_9 for fluoxetine/SLC6A4 in the MDD dataset).
- plot_dim output shows the top-ranked cell type's cells distinctly highlighted in the embedding while the bottom-ranked type (Inhib_5) shows minimal signal.
- Ranking table exports as a valid CSV (scRank_MDD_fluoxetine_ranking.csv) with all cell types and their perturbation scores.
- Module-2 heatmap (plot_net mode='heatmap') shows higher drug-target-gene module activity in the top-ranked cell type than the bottom-ranked one.
- All requested figures are exported as PNG files without plotting errors.

## Limitations

- scRank primarily focuses on inhibitor drugs; agonist modeling uses a simplifying assumption (setting out-edge weights of the target gene node to their maximum value) and is only preliminarily validated.
- Multi-target drug perturbation requires tailored methodologies not yet fully developed; only preliminary results exist (Figure S17).
- Adaptability across clustering methodologies is only demonstrated in supplementary benchmark tests (Figure S15).

## Evidence
