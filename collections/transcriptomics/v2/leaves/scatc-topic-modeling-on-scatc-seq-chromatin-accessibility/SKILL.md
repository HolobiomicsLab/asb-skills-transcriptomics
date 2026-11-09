---
name: scatc-topic-modeling-on-scatc-seq-chromatin-accessibility
description: Use when analyzing scATAC/SNARE-seq chromatin accessibility as a cell-by-peak
  matrix (e.g. ~31,758 cells x 575,265 peaks) and you need unsupervised topics and
  clusters in the chromatin modality before assigning cell types.
when_to_use_negative:
- The input is already a gene-expression matrix with no chromatin-accessibility (cell-by-peak)
  modality — cisTopic topic modeling over peaks does not apply; use expression-based
  dimensionality reduction instead.
- The scATAC data was converted to a gene activity score matrix (e.g. via BinToGene
  against GENCODE v35) and expression tools are intended — topic modeling on raw peaks
  is unnecessary.
- Cell barcodes are not shared between the ATAC and RNA modalities, so cell-ID-based
  label transfer of cluster assignments is impossible.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: cisTopic
  role: Probabilistic modeling of cis-regulatory topics on the cell-by-peak SNARE-seq
    matrix; topic number selected via its log-likelihood model selection method
  repo: https://github.com/aertslab/cisTopic
- name: faiss
  role: Approximate nearest-neighbors graph construction on the cell-by-topic matrix
    with 15 neighbors, feeding Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: Cellar
  role: Interactive framework orchestrating topic modeling, clustering, and label
    transfer across the paired modalities
  repo: https://github.com/euxhenh/cellar
- name: AnnData (Annotated Data object)
  role: In-memory/memory-mapped data container holding the cell-by-peak matrix and
    derived topics and clusters
- name: Leiden
  role: Community detection clustering of the faiss neighbors graph at resolution
    1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:525e050dd6ef72dfac32d95b585990269d5dc24562018484c9c973ca888a5537
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aertslab/cisTopic
  tools:
  - AnnData (Annotated Data object)
  - Cellar
  - Leiden
  - cisTopic
  - diffxpy
  - faiss
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# scatc-topic-modeling-on-scatc-seq-chromatin-accessibility

## Summary

Run cisTopic probabilistic topic modeling on a single-cell multiome (SNARE-seq) chromatin-accessibility cell-by-peak matrix to discover cis-regulatory topics, then cluster the cell-by-topic matrix to derive joint-modality cell clusters. This is the Cellar approach for extracting structure from sparse scATAC data so labels can be transferred to the paired expression modality.

## When to use

Use when analyzing scATAC/SNARE-seq chromatin accessibility as a cell-by-peak matrix (e.g. ~31,758 cells x 575,265 peaks) and you need unsupervised topics and clusters in the chromatin modality before assigning cell types.

## When NOT to use

- The input is already a gene-expression matrix with no chromatin-accessibility (cell-by-peak) modality — cisTopic topic modeling over peaks does not apply; use expression-based dimensionality reduction instead.
- The scATAC data was converted to a gene activity score matrix (e.g. via BinToGene against GENCODE v35) and expression tools are intended — topic modeling on raw peaks is unnecessary.
- Cell barcodes are not shared between the ATAC and RNA modalities, so cell-ID-based label transfer of cluster assignments is impossible.

## Inputs

- Cell-by-peak chromatin accessibility matrix from SNARE-seq / scATAC-seq (e.g. 31,758 cells x 575,265 peaks), combining multiple source datasets
- Cell barcode IDs shared between the chromatin and paired expression modalities (for cell-ID-based label transfer)
- AnnData (Annotated Data object), optionally in memory-mapping mode for large matrices

## Outputs

- Cell-by-topic matrix (40 cisTopic topics for the kidney SNARE-seq analysis)
- Leiden cluster assignments per cell in the chromatin modality (resolution 1)
- Cluster labels transferred to all cells of the paired RNA expression modality via cell-ID matching

## How to apply

Load the chromatin modality as a cell-by-peak matrix (kidney SNARE-seq combined from four source datasets; 31,758 cells x 575,265 peaks), first removing cells for which no annotations were found. Run cisTopic on the cell-by-peak matrix to discover cis-regulatory topics, selecting the topic number (40 in the kidney analysis) via cisTopic's log-likelihood model selection method rather than arbitrarily. Build an approximate nearest-neighbors graph on the resulting cell-by-topic matrix using faiss with 15 neighbors, and cluster it with the Leiden community detection algorithm at resolution 1. Because SNARE-seq profiles both modalities from the same cells, copy the chromatin-derived cluster assignments to the paired RNA modality via cell-ID-based label transfer (matching cell barcodes), after which differential expression (diffxpy Welch's t-test, top 500 DE genes by fold-change) and GSEApy GO enrichment can be performed on the transferred clusters. Keep objects in AnnData format; Cellar uses memory-mapping so large peak matrices (hundreds of thousands of peaks) remain tractable.

## Related tools

- **cisTopic** (Probabilistic modeling of cis-regulatory topics on the cell-by-peak SNARE-seq matrix; topic number selected via its log-likelihood model selection method) — https://github.com/aertslab/cisTopic
- **faiss** (Approximate nearest-neighbors graph construction on the cell-by-topic matrix with 15 neighbors, feeding Leiden clustering) — https://github.com/facebookresearch/faiss
- **Cellar** (Interactive framework orchestrating topic modeling, clustering, and label transfer across the paired modalities) — https://github.com/euxhenh/cellar
- **AnnData (Annotated Data object)** (In-memory/memory-mapped data container holding the cell-by-peak matrix and derived topics and clusters)
- **Leiden** (Community detection clustering of the faiss neighbors graph at resolution 1)

## Examples

```
import cisTopic; # run on kidney SNARE-seq cell-by-peak matrix (31,758 cells x 575,265 peaks), 40 topics via log-likelihood selection, then faiss ANN (15 neighbors) + Leiden (resolution 1)
```

## Evaluation signals

- The cisTopic topic count is justified by the log-likelihood model selection curve (40 topics for the kidney SNARE-seq data), not chosen arbitrarily.
- Cluster assignments from the chromatin modality exist for all cells in both modalities after cell-ID-based label transfer.
- Downstream differential expression on a transferred cluster yields biologically interpretable markers (cluster 1: SLC5A12, p-value = 0) supporting Proximal Tubule Cells.
- GSEApy GO enrichment on the top 500 DE genes by fold-change returns coherent terms for the cluster (cluster 1: 'Apical Plasma Membrane', p-value = 1e-4).
- Cells without annotations were removed before topic modeling, and the combined dataset spans all four kidney SNARE-seq source datasets.

## Limitations

- cisTopic (the R package) is deprecated in favor of pycisTopic, per its README warning.
- No software versions are stated for Cellar, cisTopic, faiss, or Python in the article context, complicating exact reproduction.
- Reproduction of the interactive analysis depends on an external web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact, though the MIT-licensed repo exists.
- Topics and clusters derived in the chromatin modality depend on the number of neighbors (15) and Leiden resolution (1); different parameters will change cluster boundaries and downstream DE/enrichment results.

## Evidence

- [methods] cisTopic topic count and selection method: [evidence span withheld — anchor status: not_found]
- [methods] faiss neighbors graph for Leiden clustering: [evidence span withheld — anchor status: not_found]
- [discussion] faiss purpose for Leiden: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [methods] Removal of unannotated cells before analysis: "We removed cells for which no annotations were found."
- [methods] SNARE-seq combination of sources: "The SNARE-seq data was formed by combining four separate kidney SNARE-seq datasets."
- [readme] cisTopic repo and deprecation notice: [evidence span withheld — anchor status: not_found]
