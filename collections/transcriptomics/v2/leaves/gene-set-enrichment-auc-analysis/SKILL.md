---
name: gene-set-enrichment-auc-analysis
description: Use when projecting leukemia cell sub-clusters onto a reference hematopoietic
  atlas and you need the gene-set-enrichment AUC-difference component of the CCI composite
  score to measure similarity between compared sub-clusters.
when_to_use_negative:
- You need to map individual single cells rather than sub-clusters — CCI projects
  cell clusters to reference instead of single cells due to prevalent droplets in
  scRNA-seq
- You intend to identify healthy counterparts based on a single signature — the article
  states this is a big challenge due to the huge differences between leukemia and
  healthy cells, which is why multiple integrated statistics including enrichment
  AUC are needed
- Target cells have not been clustered into sub-clusters and no gene sets for the
  compared sub-clusters can be defined
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CCI (counterpart composite index)
  role: Implements the composite likelihood that integrates the gene-set-enrichment
    AUC-difference statistic together with expression-distance, Spearman-correlation,
    and PCA-space statistics into a Bayes-factor probability mapping leukemia sub-clusters
    to reference hematopoietic cells
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: Runs the CCI pipeline entry point that executes the enrichment and companion
    statistics end-to-end on a sub-cluster input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:cc5c45481405de12c628ad81b2040d52ed249fab1cd48126e9e2deb9a1d228db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - bash
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Gene-set-enrichment AUC analysis

## Summary

Computes a gene-set-enrichment-based similarity statistic — the AUC difference of enrichment scores between the gene sets of a target leukemia sub-cluster and reference sub-clusters — as one of the four component statistics integrated into the CCI composite likelihood (Bayes-factor probability) for identifying healthy counterparts of leukemia cell subpopulations.

## When to use

Use when projecting leukemia cell sub-clusters onto a reference hematopoietic atlas and you need the gene-set-enrichment AUC-difference component of the CCI composite score to measure similarity between compared sub-clusters.

## When NOT to use

- You need to map individual single cells rather than sub-clusters — CCI projects cell clusters to reference instead of single cells due to prevalent droplets in scRNA-seq
- You intend to identify healthy counterparts based on a single signature — the article states this is a big challenge due to the huge differences between leukemia and healthy cells, which is why multiple integrated statistics including enrichment AUC are needed
- Target cells have not been clustered into sub-clusters and no gene sets for the compared sub-clusters can be defined

## Inputs

- Expression matrix of the target sub-cluster (cells x genes, e.g. AML1_cluster1 toy data: 100 cells, 4K genes) in data_obj/
- Reference normal-cell atlas in data_ref/ (expression matrix with cells as rows and genes as columns, gene list, cell types, cell identity of each cell)
- Gene sets defined for the compared sub-clusters

## Outputs

- AUC-difference enrichment-score statistic per leukemia-vs-reference sub-cluster pair
- Composite CCI BF-probability scores per cell/sub-cluster pair written to the result/ folder

## How to apply

Prepare the target expression matrix of the sub-cluster (e.g. 100 cells, 4000 genes in data_obj/) and the reference atlas in data_ref/ containing four files: expression matrix (cells as rows, genes as columns), gene list, cell types, and cell identity. For each leukemia-vs-reference sub-cluster pair, compute enrichment scores over the gene sets of the compared sub-clusters and take the AUC difference of these enrichment scores as the enrichment component statistic. Combine this statistic with the other three CCI components — Euclidean distance of expression profiles, Spearman rank correlation of expression, and Euclidean distance in the top-30 principal component space — into the composite likelihood expressed as a Bayes factor (BF) probability, following the formula in the released cci package source. Because droplet scRNA-seq data is prevalent, CCI projects cell clusters rather than single cells, so cluster the target cells before running. Validate by comparing reimplemented BF probabilities against the released tool's scores in result/ using rank correlation and maximum absolute deviation within tight tolerance.

## Related tools

- **CCI (counterpart composite index)** (Implements the composite likelihood that integrates the gene-set-enrichment AUC-difference statistic together with expression-distance, Spearman-correlation, and PCA-space statistics into a Bayes-factor probability mapping leukemia sub-clusters to reference hematopoietic cells) — https://github.com/pengfeeei/cci
- **bash** (Runs the CCI pipeline entry point that executes the enrichment and companion statistics end-to-end on a sub-cluster input)

## Examples

```
bash run_cci.sh AML1_cluster1
```

## Evaluation signals

- Reimplemented BF-probability scores match the released tool's scores in result/ within tight numerical tolerance (high rank agreement, near-zero maximum deviation across cell/sub-cluster pairs)
- All BF probabilities fall in the valid [0,1] probability range for every leukemia-vs-reference sub-cluster pair
- Enrichment AUC differences are finite and computed for every compared sub-cluster gene-set pair, with no missing pairs in the output matrix
- Running 'bash run_cci.sh AML1_cluster1' on the 100-cell/4K-gene toy data completes within minutes on one normal CPU and reproduces the result/ folder outputs
- Output schema matches the released tool: per-pair BF probability scores plus example PDF plots in result/

## Limitations

- CCI requires properly clustered target cells before running; it projects cell clusters to reference instead of single cells due to prevalent droplets in scRNA-seq
- Based on a single signature, identifying healthy counterparts is unreliable due to the huge differences between leukemia and healthy cells — the enrichment AUC statistic alone is insufficient and must be combined with the other CCI statistics
- Computation requires compiling C++ code (bash makefile.sh) and R > 3.4 with Seurat, ggplot2 and RColorBrewer installed

## Evidence

- [abstract] We developed counterpart composite index (CCI; available at GitHub: https://github.com/pengfeeei/cci) to search for the healthy counterpart of each leukemia cell subpopulation: "We developed counterpart composite index (CCI; available at GitHub: https://github.com/pengfeeei/cci) to search for the healthy counterpart of each leukemia cell subpopulation"
- [discussion] single-signature challenge: "it is a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature due to the huge differences"
- [readme] cluster-level projection: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq."
- [readme] score location: "CCI score (BF probability) could be found in folder result/."
- [readme] run command: "bash run_cci.sh AML1_cluster1 (toy data in data_obj/)"
