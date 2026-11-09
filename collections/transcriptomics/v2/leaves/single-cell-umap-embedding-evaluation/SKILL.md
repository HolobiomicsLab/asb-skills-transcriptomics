---
name: single-cell-umap-embedding-evaluation
description: Use when comparing whether an alternative single-cell feature space (e.g.,
  uTAR or aTAR expression) clusters cell types comparably to annotated gene expression,
  via silhouette coefficients on 2D UMAP embeddings.
when_to_use_negative:
- Inputs are raw FASTQs or Cell Ranger count matrices — this skill evaluates embeddings;
  first run the groHMM TAR-calling workflow to derive TAR expression matrices and
  UMAP coordinates
- You need to identify or annotate the transcriptionally active regions themselves
  rather than assess how well an existing feature space separates cell types
- Cluster labels must first be defined on gene-expression PCs; if no reference cell-type
  labels exist, silhouette comparison against gene-based clustering is not meaningful
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Seurat
  role: Defines reference cell-type labels via KNN graph clustering on the first 10
    principal components of gene expression, and supports UMAP embedding used for
    silhouette scoring
- name: groHMM
  role: 'Upstream: identifies transcriptionally active regions (TARs) in scRNA-seq
    data via hidden Markov model, producing the uTAR/aTAR feature spaces whose embeddings
    are evaluated'
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: rtracklayer
  role: Upstream R package used with groHMM to generate and handle groHMM bed files
    of TARs
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Snakemake
  role: Workflow engine for the TAR-scRNA-seq pipelines (from_fastqs / from_cellranger)
    that generate the TAR expression matrices feeding this evaluation
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:c51a97707452021839def6607ca9ee0fc7861a77cd9f7b91ad7bedd9763a54af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - Snakemake
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-cell-umap-embedding-evaluation

## Summary

Evaluates how well a single-cell feature space (annotated gene expression, aTARs, or uTARs) separates cell types by computing silhouette coefficients on 2D UMAP coordinates, as used in the TAR-scRNA-seq paper to show uTAR reads recapitulate gene-expression clustering (Fig. 2b).

## When to use

Use when comparing whether an alternative single-cell feature space (e.g., uTAR or aTAR expression) clusters cell types comparably to annotated gene expression, via silhouette coefficients on 2D UMAP embeddings.

## When NOT to use

- Inputs are raw FASTQs or Cell Ranger count matrices — this skill evaluates embeddings; first run the groHMM TAR-calling workflow to derive TAR expression matrices and UMAP coordinates
- You need to identify or annotate the transcriptionally active regions themselves rather than assess how well an existing feature space separates cell types
- Cluster labels must first be defined on gene-expression PCs; if no reference cell-type labels exist, silhouette comparison against gene-based clustering is not meaningful

## Inputs

- 2D UMAP coordinates per cell for gene, aTAR, and uTAR feature spaces (one embedding per sample per feature type)
- Cell-type cluster labels from Seurat KNN graph clustering on the first 10 principal components of annotated gene expression
- Sample metadata for the 11 scRNA-seq datasets (human, mouse, chicken, mole rat, lemur, sea urchin)

## Outputs

- Table with one row per sample and silhouette coefficient columns for gene (blue), aTAR (red), and uTAR (maroon) clusterings, matching Fig. 2b

## How to apply

For each sample, load 2D UMAP coordinates computed in each feature space (gene, aTAR, uTAR) together with cell-type labels derived from gene-annotation-based KNN graph clustering on the first 10 PCs in Seurat. Compute scikit-learn silhouette_score with Euclidean distance on the 2D embedding for each feature space, holding the gene-based labels fixed as the reference partition. Assemble a per-sample table with three silhouette columns (gene, aTAR, uTAR) reproducing the Fig. 2b layout across the 11 samples. Judge success by whether uTAR silhouette values closely track the gene-based values per sample (e.g., mouse spleen, lemur lung, naked mole rat spleen, sea urchin embryo), indicating uTAR reads accurately separate cell types.

## Related tools

- **Seurat** (Defines reference cell-type labels via KNN graph clustering on the first 10 principal components of gene expression, and supports UMAP embedding used for silhouette scoring)
- **groHMM** (Upstream: identifies transcriptionally active regions (TARs) in scRNA-seq data via hidden Markov model, producing the uTAR/aTAR feature spaces whose embeddings are evaluated) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **rtracklayer** (Upstream R package used with groHMM to generate and handle groHMM bed files of TARs) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Snakemake** (Workflow engine for the TAR-scRNA-seq pipelines (from_fastqs / from_cellranger) that generate the TAR expression matrices feeding this evaluation) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
from sklearn.metrics import silhouette_score; s = silhouette_score(umap_2d, labels, metric='euclidean')
```

## Evaluation signals

- Table contains silhouette values for all 11 samples across all three feature types (gene, aTAR, uTAR)
- For samples where the paper reports agreement (mouse spleen, lemur lung, naked mole rat spleen, sea urchin embryo), uTAR silhouette values closely track the gene-based silhouette values, qualitatively reproducing Fig. 2b
- aTAR silhouette values show close agreement with gene-expression-based values across samples
- Silhouette scores are computed with scikit-learn silhouette_score using Euclidean distance on the 2D UMAP coordinates, not on high-dimensional PCs

## Limitations

- The degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development, so agreement between uTAR and gene silhouettes is not uniform across samples
- groHMM must be correctly installed and finish running to generate the chr*_HMM.bed files; otherwise downstream TAR embeddings cannot be produced (per README FAQ)
- Comparison relies on cell-type labels from gene-annotation-based clustering, so quality depends on the reference annotation

## Evidence

- [results] Silhouette values computed from 2D UMAP coordinates show close agreement between aTAR- and gene-expression-based clustering, and good agreement between uTAR- and gene-expression-based clustering: [evidence span withheld — anchor status: not_found]
- [intro] groHMM HMM states: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [intro] uTAR dependency on organism/tissue: "we find that the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development"
- [readme] groHMM install FAQ: [evidence span withheld — anchor status: not_found]
