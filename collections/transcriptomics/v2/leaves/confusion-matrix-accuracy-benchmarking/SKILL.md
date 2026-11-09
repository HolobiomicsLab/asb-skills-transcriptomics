---
name: confusion-matrix-accuracy-benchmarking
description: Use when predicted cell type labels from marker-based scoring (e.g.,
  GeneVector phenotype_probability) must be validated against reference annotations
  via a confusion matrix with per-type accuracy and overall misclassification rate.
when_to_use_negative:
- No reference or ground-truth cell type annotations exist for the dataset — per-type
  accuracy cannot be computed without a published label to compare against.
- The goal is assessing clustering or embedding quality rather than label accuracy
  — the article instead benchmarks those with ARI, kBET, cLISI, and silhouette score
  (Supplementary Fig. 6A–L).
- Fine-grained subtype annotation is required — the benchmarked marker sets cover
  only the three lineage-level categories T cell, B/Plasma, and Myeloid.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GeneVector
  role: Produces SoftMax pseudo-probabilities and argmax 'genevector' cell labels
    via phenotype_probability, which are cross-tabulated in the confusion matrix
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads and holds the TICA AnnData subset (raw counts in .X) whose obs columns
    feed the predicted-vs-reference cross-tabulation
- name: AnnData
  role: Stores the 'Pseudo-probability' columns, predicted 'genevector' labels, and
    reference cell type annotations in .obs for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:47f931be9ab248d3372af21cdf8843be77f8789f5d5296c8bcc776a5b84d6c54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - PyTorch
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# confusion-matrix-accuracy-benchmarking

## Summary

Benchmarks marker-based cell type predictions by cross-tabulating GeneVector's argmax 'genevector' labels against CellTypist-derived reference annotations on the TICA subset, producing a confusion matrix with per-type accuracies (97.1% T cell, 95% myeloid, 94.3% B/Plasma) and a 3.977% overall misclassification rate.

## When to use

Use when predicted cell type labels from marker-based scoring (e.g., GeneVector phenotype_probability) must be validated against reference annotations via a confusion matrix with per-type accuracy and overall misclassification rate.

## When NOT to use

- No reference or ground-truth cell type annotations exist for the dataset — per-type accuracy cannot be computed without a published label to compare against.
- The goal is assessing clustering or embedding quality rather than label accuracy — the article instead benchmarks those with ARI, kBET, cLISI, and silhouette score (Supplementary Fig. 6A–L).
- Fine-grained subtype annotation is required — the benchmarked marker sets cover only the three lineage-level categories T cell, B/Plasma, and Myeloid.

## Inputs

- TICA subset AnnData (.h5ad) with raw counts in .X (23,764 cells, 2,000 highly variable genes) carrying CellTypist-derived reference annotations in .obs
- CellEmbedding built from genes.vec (emb_dimension=100, vector='average') supporting phenotype_probability
- Marker gene dictionaries per cell type: T cell (CD3D, CD3G, CD3E, TRAC, IL32, CD2), B/Plasma (CD79A, CD79B, MZB1, CD19, BANK1), Myeloid (LYZ, CST3, AIF1, CD68, C1QA, C1QB, C1QC)

## Outputs

- Confusion matrix cross-tabulating predicted 'genevector' labels vs CellTypist-derived reference annotations
- Per-type accuracies (T cell, B/Plasma, Myeloid) and overall misclassification rate (expected 3.977%)

## How to apply

First obtain predictions by running cell_embed.phenotype_probability with the lineage marker dictionaries — T cell (CD3D/CD3G/CD3E/TRAC/IL32/CD2), B/Plasma (CD79A/CD79B/MZB1/CD19/BANK1), myeloid (LYZ/CST3/AIF1/CD68/C1QA/C1QB/C1QC) — which yields SoftMax pseudo-probability columns and argmax labels in the 'genevector' obs column. Cross-tabulate these predicted labels against the CellTypist-derived reference annotations to build the confusion matrix, computing per-type accuracy as the diagonal fraction for each lineage. The predictions derive from a GeneVector model trained on the TICA subset (23,764 cells, 2,000 highly variable genes, raw counts in .X) with emb_dimension=100 for 1000 iterations at threshold 1e-6, embedded with GeneEmbedding(vector='average') and aggregated into a CellEmbedding. Report per-type accuracies against the published values (97.1% T, 95% myeloid, 94.3% B/Plasma) and the overall misclassification rate (3.977% expected), alongside the CellAssign comparator values (85.6% myeloid / 7.54%). Decision point: if a predicted type maps predominantly off-diagonal, revisit the marker set or the upstream entropy-based gene QC before retraining.

## Related tools

- **GeneVector** (Produces SoftMax pseudo-probabilities and argmax 'genevector' cell labels via phenotype_probability, which are cross-tabulated in the confusion matrix) — https://github.com/nceglia/genevector
- **Scanpy** (Loads and holds the TICA AnnData subset (raw counts in .X) whose obs columns feed the predicted-vs-reference cross-tabulation)
- **AnnData** (Stores the 'Pseudo-probability' columns, predicted 'genevector' labels, and reference cell type annotations in .obs for comparison)

## Examples

```
annotated_adata = cell_embed.phenotype_probability(adata_embedded, markers); pd.crosstab(annotated_adata.obs['genevector'], reference_labels)
```

## Evaluation signals

- Per-type accuracies reproduce the reported values: 97.1% T cells, 95% myeloid, and 94.3% B/Plasma relative to CellTypist-derived annotations.
- Overall misclassification rate matches the reported 3.977% on the TICA subset.
- Confusion matrix is diagonally dominant — each predicted label's largest count coincides with its matching reference type.
- Myeloid misclassification remains far below the CellAssign comparator values (85.6% myeloid / 7.54%) reported for the same data.
- Every cell receives exactly one argmax 'genevector' label and its SoftMax pseudo-probabilities sum to ~1.

## Limitations

- Accuracies are defined only relative to CellTypist-derived reference annotations, not an independent ground truth; a different reference taxonomy would change the labels and scores.
- Only the three lineage-level categories (T cell, B/Plasma, Myeloid) are benchmarked; subtype-level assignments are not evaluated.
- Prediction quality inherits limitations of empirical MI estimation, which is limited by the available number of cells, fidelity of counts, and discretization strategy.

## Evidence

- [intro] Cell type predictions in TICA are validated against published annotations: [evidence span withheld — anchor status: not_found]
- [methods] phenotype_probability generates the pseudo-probabilities and labels to be benchmarked: "adata = cell_embed.phenotype_probability(adata, markers)"
- [readme] README documents the marker-based automated cell type assignment used before cross-tabulation: [evidence span withheld — anchor status: external_doc]
- [abstract] Latent space arithmetic is the basis for the cell type classification being benchmarked: "GeneVector uses latent space arithmetic in a lower dimensional gene embedding to identify transcriptional programs and classify cell types."
