---
name: nk-cell-marker-identification
description: Use when a Leiden-clustered scATAC-seq gene activity matrix needs cluster
  annotation, and you must confirm whether a known immune marker (e.g. KLRD1 for NK
  cells) appears among top differentially expressed genes per cluster.
when_to_use_negative:
- Input is raw scATAC-seq cell-by-bin data without a gene activity conversion (use
  BinToGene to first convert to cell-by-gene)
- The task is reference-based cell-type label transfer rather than de novo marker
  discovery (use scanpy Ingest or SingleR instead)
- The dataset is not clustered or lacks per-cluster cell labels, so no one-vs-rest
  contrast can be defined
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: Cellar
  role: 'Interactive platform hosting the full workflow: gene activity loading, PCA,
    faiss/Leiden clustering, and differential expression testing'
  repo: https://github.com/euxhenh/cellar
- name: diffxpy
  role: One-vs-rest differential gene testing per Leiden cluster using Welch's t-test,
    ranked by log fold-change to surface marker genes like KLRD1
  repo: https://github.com/theislab/diffxpy
- name: faiss
  role: Approximate nearest-neighbors graph construction (15 neighbors) that feeds
    Leiden clustering prior to DE testing
  repo: https://github.com/facebookresearch/faiss
- name: scikit-learn
  role: PCA with randomized SVD solver reducing the gene activity matrix to 40 dimensions
    before clustering
- name: scanpy
  role: Preprocessing of the data (CPM normalization to 1e5, log1p, scaling) before
    dimensionality reduction
- name: GSEApy
  role: Optional functional enrichment of the top fold-change DE genes to contextualize
    cluster markers
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:647e8acb23154d84f93caba500fd6b8a3755fdb653a81f563c91f7b16a466f83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - Cellar
  - GSEApy
  - diffxpy
  - faiss
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nk-cell-marker-identification

## Summary

Identifies natural killer (NK) cells as markers from scATAC-seq cell-by-gene activity clusters by running one-vs-rest differential gene testing with diffxpy's Welch's t-test and ranking genes by log fold-change; in the Cellar PBMC 10k analysis KLRD1 emerged as the NK marker for clusters 0 and 4.

## When to use

Use when a Leiden-clustered scATAC-seq gene activity matrix needs cluster annotation, and you must confirm whether a known immune marker (e.g. KLRD1 for NK cells) appears among top differentially expressed genes per cluster.

## When NOT to use

- Input is raw scATAC-seq cell-by-bin data without a gene activity conversion (use BinToGene to first convert to cell-by-gene)
- The task is reference-based cell-type label transfer rather than de novo marker discovery (use scanpy Ingest or SingleR instead)
- The dataset is not clustered or lacks per-cluster cell labels, so no one-vs-rest contrast can be defined

## Inputs

- scATAC-seq cell-by-gene gene activity score matrix (AnnData object, e.g. PBMC 10k)
- Leiden cluster labels for each cell (resolution 1, faiss neighbors graph with 15 neighbors)

## Outputs

- de_genes_per_cluster.csv with per-cluster Welch's t-test DE results ranked by log fold-change
- Marker gene per cluster (e.g. KLRD1 as NK-cell marker for clusters 0 and 4)
- Optionally: top 500 DE genes by fold-change fed to GSEApy enrichment analysis

## How to apply

Load the PBMC scATAC-seq cell-by-gene activity score matrix as an AnnData object, then CPM-normalize to a total count of 1e5, log1p-transform, and scale to zero mean and unit variance. Reduce to 40 dimensions with scikit-learn PCA using a randomized SVD solver, build an approximate neighbors graph with faiss (15 neighbors), and cluster cells with Leiden at resolution 1. For each Leiden cluster, perform one-vs-rest differential gene testing with diffxpy using Welch's t-test and rank genes by log fold-change. Compile the top DE genes per cluster (e.g. into de_genes_per_cluster.csv) and check whether the expected marker (KLRD1 for NK cells) ranks among the top genes for the target clusters (0 and 4 in the Cellar PBMC 10k run), consistent with the NK-cell annotation in Supplementary Fig. 3. Optionally select the top DE genes by fold-change (the paper used the 500 greatest fold-change genes) for downstream enrichment analysis with GSEApy.

## Related tools

- **Cellar** (Interactive platform hosting the full workflow: gene activity loading, PCA, faiss/Leiden clustering, and differential expression testing) — https://github.com/euxhenh/cellar
- **diffxpy** (One-vs-rest differential gene testing per Leiden cluster using Welch's t-test, ranked by log fold-change to surface marker genes like KLRD1) — https://github.com/theislab/diffxpy
- **faiss** (Approximate nearest-neighbors graph construction (15 neighbors) that feeds Leiden clustering prior to DE testing) — https://github.com/facebookresearch/faiss
- **scikit-learn** (PCA with randomized SVD solver reducing the gene activity matrix to 40 dimensions before clustering)
- **scanpy** (Preprocessing of the data (CPM normalization to 1e5, log1p, scaling) before dimensionality reduction)
- **GSEApy** (Optional functional enrichment of the top fold-change DE genes to contextualize cluster markers) — https://github.com/zqfang/GSEApy

## Examples

```
import diffxpy.api as de; test = de.test.t_test(data=adata[adata.obs['leiden']=='0'], grouping='rest', is_logged=True)
```

## Evaluation signals

- Output table de_genes_per_cluster.csv exists and contains per-cluster Welch's t-test results ranked by fold-change
- KLRD1 is reported among the top marker genes for clusters 0 and 4, matching the NK-cell annotation in Supplementary Fig. 3
- Preprocessing invariants hold: CPM normalization to total count 1e5, log1p transformation, and scaling to zero mean and unit variance
- Dimensionality and clustering parameters reproduce the reference run: 40 PCA components (randomized SVD), 15-neighbor faiss graph, Leiden resolution 1
- Cell/gene QC filters applied before DE: cells with fewer than 50 or more than 3000 expressed genes, and genes in fewer than 50 or more than 3000 cells, removed

## Limitations

- diffxpy is unmaintained; the README redirects users interested in pure-Python differential expression to PyDESeq2
- Marker confirmation relies on manual inspection of known marker genes (KLRD1) against annotated figures; the paper notes that annotation often requires domain-knowledge input
- For large datasets, fine-grained per-cluster testing can be costly; Cellar mitigates this with faiss-accelerated neighbors graph construction rather than exact graphs

## Evidence

- [methods] Welch's t-test DE ranking yields KLRD1 NK marker: "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [methods] Preprocessing parameters for gene activity matrix: "The data matrix was then CPM total count normalized (total count = 1e5) and log1p-transformed."
- [methods] PCA solver choice: "We used the PCA implementation of the scikit-learn package with a randomized SVD solver."
- [methods] faiss neighbors graph for Leiden clustering: [evidence span withheld — anchor status: not_found]
- [methods] Cell/gene QC filtering thresholds: "we filtered cells with less than 50 or more than 3000 expressed genes"
- [methods] Top DE genes selected for enrichment: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [readme] diffxpy unmaintained warning: [evidence span withheld — anchor status: external_doc]
- [intro] Semi-automatic annotation requiring user input: "As cell-type assignment often requires user input in the form of domain knowledge, Cellar adopts a semi-automatic solution that permits users to intervene and modify each processing step as needed"
