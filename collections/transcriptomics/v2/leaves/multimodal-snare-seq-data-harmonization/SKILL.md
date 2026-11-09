---
name: multimodal-snare-seq-data-harmonization
description: Use when analyzing paired multi-omic SNARE-seq data where cells have
  both a chromatin (cell-by-peak) and expression modality, and cluster-level cell-type
  annotation must be supported by marker genes and GO enrichment.
when_to_use_negative:
- Data are unpaired (e.g., separate scATAC-seq and scRNA-seq experiments) — label
  transfer relies on matching cell barcodes between modalities profiled from the same
  cells
- The goal is purely RNA-based clustering — this skill's clustering operates on the
  chromatin modality's cell-by-topic matrix, not gene expression
- Cellar's reference-annotation mode suffices — for datasets with HuBMAP annotated
  references, direct reference-based label transfer (e.g., scanpy Ingest) may be more
  appropriate
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cellar
  role: 'Interactive tool orchestrating the full joint-modality assignment pipeline:
    cisTopic topics, faiss/Leiden clustering, label transfer, diffxpy DE, and GSEApy
    enrichment'
  repo: https://github.com/euxhenh/cellar
- name: cisTopic
  role: Discover 40 cis-regulatory topics from the SNARE-seq cell-by-peak matrix;
    topic number chosen via log-likelihood model selection
  repo: https://github.com/aertslab/cisTopic
- name: faiss
  role: Approximate nearest-neighbors graph construction on the cell-by-topic matrix
    using 15 neighbors to speed up Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: Leiden
  role: Community-detection clustering of the faiss neighbors graph (resolution 1)
- name: diffxpy
  role: Differential gene expression analysis on transferred clusters with a Welch's
    t-test
  repo: https://github.com/theislab/diffxpy
- name: gseapy
  role: GO term enrichment analysis on the top 500 DE genes by fold-change, yielding
    p-values for cluster annotation
  repo: https://github.com/zqfang/GSEApy
- name: AnnData (Annotated Data object)
  role: In-memory data structure for the paired modalities and cluster labels, used
    in memory-mapping mode for large datasets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:dde39ba33eead68197a73f651f521c29e216f62f6feaa32af12d1fcc20c84509
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData (Annotated Data object)
  - Cellar
  - Leiden
  - cisTopic
  - diffxpy
  - faiss
  - gseapy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multimodal-snare-seq-data-harmonization

## Summary

Jointly cluster paired ATAC/RNA SNARE-seq data by clustering the chromatin modality (cisTopic topics) and transferring cluster labels to the paired expression modality via shared cell barcodes, then annotating clusters with marker and GO enrichment evidence.

## When to use

Use when analyzing paired multi-omic SNARE-seq data where cells have both a chromatin (cell-by-peak) and expression modality, and cluster-level cell-type annotation must be supported by marker genes and GO enrichment.

## When NOT to use

- Data are unpaired (e.g., separate scATAC-seq and scRNA-seq experiments) — label transfer relies on matching cell barcodes between modalities profiled from the same cells
- The goal is purely RNA-based clustering — this skill's clustering operates on the chromatin modality's cell-by-topic matrix, not gene expression
- Cellar's reference-annotation mode suffices — for datasets with HuBMAP annotated references, direct reference-based label transfer (e.g., scanpy Ingest) may be more appropriate

## Inputs

- cell-by-peak chromatin modality matrix from SNARE-seq (e.g., 31,758 cells x 575,265 peaks)
- paired cell-by-gene expression modality matrix from the same SNARE-seq cells (AnnData object)
- four separate kidney SNARE-seq source datasets to be combined
- known cell-type marker gene sets (e.g., SLC5A12 for Proximal Tubule Cells)

## Outputs

- cisTopic topics (40 topics) per cell and cell-by-topic matrix
- Leiden cluster assignments at resolution 1 covering all cells in both modalities
- diffxpy differential expression results per cluster (Welch's t-test, p-values and fold-changes)
- GSEApy GO enrichment p-values per cluster (e.g., 'Apical Plasma Membrane', p value = 1e-4)
- cluster-to-cell-type annotation table (cluster 1 = Proximal Tubule Cells)

## How to apply

Combine the source SNARE-seq datasets (e.g., the kidney SNARE-seq data was formed by combining four separate kidney SNARE-seq datasets) and remove cells for which no annotations were found. Run cisTopic on the cell-by-peak matrix (31,758 cells, 575,265 peaks for the kidney dataset) to discover 40 topics, selecting the topic number via cisTopic's log-likelihood model selection method. Build an approximate nearest-neighbors graph on the cell-by-topic matrix with faiss using 15 neighbors and cluster with Leiden at resolution 1. Because both modalities come from the same cells, copy the chromatin-cluster assignments to the expression modality via cell-ID (barcode)-based label transfer, then run diffxpy (Welch's t-test) on the transferred clusters and select the top 500 DE genes by fold-change for GSEApy GO enrichment; assign cell types by intersecting DE genes with known markers (e.g., SLC5A12 for Proximal Tubule Cells, p-value = 0) and GO terms (e.g., 'Apical Plasma Membrane', p value = 1e-4).

## Related tools

- **Cellar** (Interactive tool orchestrating the full joint-modality assignment pipeline: cisTopic topics, faiss/Leiden clustering, label transfer, diffxpy DE, and GSEApy enrichment) — https://github.com/euxhenh/cellar
- **cisTopic** (Discover 40 cis-regulatory topics from the SNARE-seq cell-by-peak matrix; topic number chosen via log-likelihood model selection) — https://github.com/aertslab/cisTopic
- **faiss** (Approximate nearest-neighbors graph construction on the cell-by-topic matrix using 15 neighbors to speed up Leiden clustering) — https://github.com/facebookresearch/faiss
- **Leiden** (Community-detection clustering of the faiss neighbors graph (resolution 1))
- **diffxpy** (Differential gene expression analysis on transferred clusters with a Welch's t-test) — https://github.com/theislab/diffxpy
- **gseapy** (GO term enrichment analysis on the top 500 DE genes by fold-change, yielding p-values for cluster annotation) — https://github.com/zqfang/GSEApy
- **AnnData (Annotated Data object)** (In-memory data structure for the paired modalities and cluster labels, used in memory-mapping mode for large datasets)

## Examples

```
import faiss; index = faiss.IndexFlatL2(len(topic_matrix[0])); index.add(topic_matrix); D, I = index.search(topic_matrix, 15)  # 15 neighbors for Leiden graph
```

## Evaluation signals

- Cluster-1 evidence table reproduces the known marker SLC5A12 with p-value = 0
- GO enrichment for cluster 1 returns the 'Apical Plasma Membrane' term with p value = 1e-4
- Cluster assignments exist for all cells in both modalities after barcode-based label transfer (no missing labels)
- cisTopic log-likelihood model selection supports the chosen 40 topics
- Four source kidney SNARE-seq datasets are combined and unannotated cells removed before topic modeling

## Limitations

- cisTopic (original R package) is deprecated in favor of pycisTopic per the repository README
- diffxpy is unmaintained; its README recommends PyDESeq2 for pure-python differential expression analysis
- No software versions stated for Cellar, Python, Dash, or faiss, and no DOI provided for archived code/data, complicating exact reproduction
- Label transfer requires cell barcodes to match exactly between the ATAC and RNA modalities; cells without annotations are removed

## Evidence

- [methods] Combined SNARE-seq datasets: "The SNARE-seq data was formed by combining four separate kidney SNARE-seq datasets."
- [methods] cisTopic topic number selection: [evidence span withheld — anchor status: not_found]
- [methods] faiss graph construction: [evidence span withheld — anchor status: not_found]
- [methods] diffxpy for DE: "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [methods] GSEApy for enrichment: "enrichment analysis via the package gseapy (https://github.com/zqfang/GSEApy)"
- [methods] Remove unannotated cells: "We removed cells for which no annotations were found."
- [methods] Top 500 DE genes for enrichment: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [readme] cisTopic deprecated: [evidence span withheld — anchor status: external_doc]
- [readme] diffxpy unmaintained: [evidence span withheld — anchor status: external_doc]
