---
name: silhouette-coefficient-cluster-evaluation
description: Use when comparing two cell embeddings (e.g., OnClass pred_Y_all score-PCA
  vs Scanorama-integrated expression) on labeled scRNA-seq data and you need a per-dataset,
  statistically testable measure of cluster separation by cell type.
when_to_use_negative:
- Cells lack ground-truth cell type labels — silhouette in this workflow is computed
  against cell type labels, so unlabeled embeddings cannot be scored this way
- You only have a single embedding and no baseline (e.g., the Scanorama expression
  embedding) — the skill is designed for head-to-head comparison and a two-sided significance
  test
- Your goal is annotation accuracy (correct cell type assignment) rather than evaluating
  whether an embedding separates cells by cell type
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Produces the pred_Y_all cell-by-Cell-Ontology-type score matrix whose PCA
    embedding is silhouette-scored
  repo: https://github.com/wangshenguiuc/OnClass
- name: Scanorama
  role: Provides the expression-based integrated embedding (scan_dim=10) used as the
    baseline in the silhouette comparison
  repo: https://github.com/brianhie/scanorama
- name: Python
  role: Implementation language for PCA reduction, silhouette computation, and the
    significance test written to silhouette_scores.csv
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:52e2fead03e4a33c48e3f980921e38d42de8e8fdfc930a794769d3e1ef71d7b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Scanorama
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# silhouette-coefficient-cluster-evaluation

## Summary

Quantify how well cell type labels separate in a low-dimensional single-cell embedding by computing the silhouette coefficient per dataset, as used to show OnClass Cell-Ontology score embeddings integrate the 26 scRNA-seq datasets better than Scanorama expression-based integration.

## When to use

Use when comparing two cell embeddings (e.g., OnClass pred_Y_all score-PCA vs Scanorama-integrated expression) on labeled scRNA-seq data and you need a per-dataset, statistically testable measure of cluster separation by cell type.

## When NOT to use

- Cells lack ground-truth cell type labels — silhouette in this workflow is computed against cell type labels, so unlabeled embeddings cannot be scored this way
- You only have a single embedding and no baseline (e.g., the Scanorama expression embedding) — the skill is designed for head-to-head comparison and a two-sided significance test
- Your goal is annotation accuracy (correct cell type assignment) rather than evaluating whether an embedding separates cells by cell type

## Inputs

- Cell-by-all-Cell-Ontology-type score matrix pred_Y_all (pred_Y_all.released.npy) from OnClassModel.Predict with use_normalize=True
- Scanorama-integrated expression embedding (scan_dim=10) from run_scanorama_multiply_datasets over the 26 datasets
- Per-cell cell type labels for each of the 26 datasets

## Outputs

- Per-dataset silhouette coefficients for the OnClass score-PCA embedding
- Per-dataset silhouette coefficients for the Scanorama expression embedding
- silhouette_scores.csv containing per-dataset silhouette values and the significance test P value

## How to apply

First build the embeddings to be scored: reduce the cell-by-all-Cell-Ontology-type score matrix (pred_Y_all from OnClassModel.Predict with use_normalize=True, saved as pred_Y_all.released.npy) to a low-dimensional embedding with PCA, and build the expression-based comparison embedding by integrating the 26 datasets with Scanorama via run_scanorama_multiply_datasets using scan_dim=10. For each dataset, compute the silhouette coefficient of both embeddings against the cell type labels, so a higher silhouette means cells cluster by cell type rather than by platform artifact. Then test the significance of the silhouette improvement of the OnClass embedding over the Scanorama embedding with a two-sided test, and write the per-dataset silhouette values plus the P value to silhouette_scores.csv. In the reference reproduction, acceptance requires the OnClass score-embedding silhouette to significantly exceed the Scanorama silhouette, matching the reported P-value < 7e-293.

## Related tools

- **OnClass** (Produces the pred_Y_all cell-by-Cell-Ontology-type score matrix whose PCA embedding is silhouette-scored) — https://github.com/wangshenguiuc/OnClass
- **Scanorama** (Provides the expression-based integrated embedding (scan_dim=10) used as the baseline in the silhouette comparison) — https://github.com/brianhie/scanorama
- **Python** (Implementation language for PCA reduction, silhouette computation, and the significance test written to silhouette_scores.csv) — https://github.com/wangshenguiuc/OnClass

## Examples

```
integrated, corrected, genes = scanorama.correct(datasets, genes_list, return_dimred=True)
```

## Evaluation signals

- OnClass score-embedding silhouette significantly exceeds the Scanorama expression-embedding silhouette, reproducing the reported P-value < 7e-293 (two-sided test)
- silhouette_scores.csv exists and contains one silhouette value per dataset for both embeddings plus the P value
- Silhouette is computed on the PCA-reduced pred_Y_all score vectors, not on raw high-dimensional score matrices
- The Scanorama baseline embedding is built with scan_dim=10 via run_scanorama_multiply_datasets
- Higher silhouette corresponds to cells clustering by cell type rather than by platform/dataset artifacts

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so silhouette may not capture separation of cell types absent from the ontology
- OnClass cannot distinguish topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph), which can blur cluster separation for such labels
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed, which can confound embedding comparisons

## Evidence

- [results] silhouette improvement over Scanorama baseline: [evidence span withheld — anchor status: not_found]
- [methods] per-dataset silhouette computation procedure: [evidence span withheld — anchor status: not_found]
- [readme] Scanorama embedding as downstream input: [evidence span withheld — anchor status: not_found]
