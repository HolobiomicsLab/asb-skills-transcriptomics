---
name: macrophage-marker-gene-interpretation
description: Use when scNovel's query stage has flagged cells as novel (score > lambda
  = 0.5) and Leiden clusters of those novel cells need to be named by checking marker
  expression such as CD163.
when_to_use_negative:
- The query cells have not yet passed scNovel's novel-detection stage (novelty score
  below lambda = 0.5), so there are no novel clusters to interpret
- The goal is reference-based annotation of known cell types (B cell, T cell, etc.)
  rather than naming novel clusters — scNovel's classification output, not marker
  interpretation, is the right step
- The dataset is a bulk or non-single-cell expression matrix, where per-cluster marker
  interpretation is not applicable
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scNovel
  role: Upstream detector whose novel-cell flags and novelty score threshold (lambda
    = 0.5) define the clusters whose markers are interpreted
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Performs Leiden clustering of novel cells and marker-expression visualization
    (UMAP/dot plots) used to assign cluster identity
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:82823d01682fa00e2ec0fd34f02650e3db434305e728dd477a525e5d78636855
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# macrophage-marker-gene-interpretation

## Summary

Interprets the marker-expression profiles of novel cell clusters detected by scNovel (e.g., Leiden clusters from the GSE171524 COVID-19 query set) to assign biological identity, such as labeling CD163-high clusters 3, 5 and 12 as candidate Macrophage subtypes. It converts unsupervised cluster IDs into evidence-based cell-type annotations.

## When to use

Use when scNovel's query stage has flagged cells as novel (score > lambda = 0.5) and Leiden clusters of those novel cells need to be named by checking marker expression such as CD163.

## When NOT to use

- The query cells have not yet passed scNovel's novel-detection stage (novelty score below lambda = 0.5), so there are no novel clusters to interpret
- The goal is reference-based annotation of known cell types (B cell, T cell, etc.) rather than naming novel clusters — scNovel's classification output, not marker interpretation, is the right step
- The dataset is a bulk or non-single-cell expression matrix, where per-cluster marker interpretation is not applicable

## Inputs

- AnnData object (scanpy) containing the scNovel-detected novel cells with Leiden cluster assignments
- Normalized, log1p-transformed expression matrix (normalize_total with target_sum=1e4 followed by log1p)
- List of candidate marker genes (e.g., CD163) to evaluate across novel clusters

## Outputs

- Cluster-level marker expression summary (e.g., dot plot or UMAP panel) identifying CD163-high clusters
- Cell-type annotation for novel clusters (e.g., clusters 3, 5, 12 labeled as candidate Macrophage subtypes)
- Saved interpretation figure (e.g., figure6_novel_detection.png)

## How to apply

After scNovel's query stage flags novel cells (novelty score exceeding lambda = 0.5) and Leiden clustering in scanpy partitions them, inspect expression of canonical markers across the resulting novel clusters. In the GSE171524 COVID-19 analysis (116,314 cells from 7 healthy controls and 20 COVID-19 samples vs. a 1.5-million-cell immune atlas reference), the decision rule was: clusters with high CD163 expression are candidate Macrophage subtypes — clusters 3, 5 and 12 met this criterion. Confirm the annotation by verifying that disease-related differential genes show consistent expression patterns across the proposed subtypes, as the paper did for its three novel Macrophage subtypes. Render marker/annotation UMAPs (Figure 6A-D style) to document the interpretation.

## Related tools

- **scNovel** (Upstream detector whose novel-cell flags and novelty score threshold (lambda = 0.5) define the clusters whose markers are interpreted) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Performs Leiden clustering of novel cells and marker-expression visualization (UMAP/dot plots) used to assign cluster identity)

## Examples

```
sc.pp.normalize_total(novel_adata, target_sum=1e4); sc.pp.log1p(novel_adata); sc.tl.leiden(novel_adata); sc.pl.dotplot(novel_adata, var_names=['CD163'], groupby='leiden')
```

## Evaluation signals

- Clusters 3, 5 and 12 in the GSE171524 novel-cell set show high CD163 expression, matching the published candidate Macrophage subtype assignment
- B cell and T cell are rarely predicted as novel while AT1 and AT2 are successfully detected as novel in the UMAP panel (Figure 6A-D)
- COVID-related differential genes show consistent expression patterns across the proposed novel Macrophage subtypes
- The novel cells were flagged using the barcode-preprocessed scoring mode with threshold lambda = 0.5 before clustering

## Limitations

- A better score function could improve scNovel performance, so novel-cluster membership (and hence marker-based interpretation) may change with improved detection
- The current method does not incorporate other biology information, e.g., single-cell chromatin accessibility, so marker-based identity assignments rest on transcriptomic evidence alone
- Marker interpretation is within the context of a profoundly imbalanced clinical COVID-19 dataset, which can affect which subtypes are surfaced

## Evidence

- [abstract] three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis: "three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis"
- [discussion] Within the context of the profoundly imbalanced clinical COVID-19 dataset, a comprehensive analysis has unveiled the existence of three promising novel subtypes of macrophages.: "Within the context of the profoundly imbalanced clinical COVID-19 dataset, a comprehensive analysis has unveiled the existence of three promising novel subtypes of macrophages."
- [other] Check the marker expression of the resulting novel clusters, confirming clusters 3, 5 and 12 show high CD163 expression and are therefore candidate Macrophage subtypes: [evidence span withheld — anchor status: not_found]
