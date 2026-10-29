---
name: cell-type-annotation-marker-genes
description: Use when you have a cell-by-factor or low-dimensional embedding from
  paired multi-omics data (e.g., Mowgli on TEA-seq PBMC) and need per-cell type labels
  grounded in canonical gene/protein markers plus an independent reference annotation.
when_to_use_negative:
- Input is raw counts without normalization (RNA should first be normalized to 10,000
  counts and log1p-transformed; ADT CLR-normalized via Muon)
- The goal is de novo marker discovery rather than annotation of known populations
  against canonical markers and a reference
- Data lack a credible reference (e.g., non-human PBMC) where Azimuth-based cross-validation
  of marker-based labels cannot support the assignment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: kNN graph construction and Leiden clustering of the Mowgli embedding for marker-based
    annotation
  repo: https://github.com/cantinilab/mowgli
- name: Muon
  role: Preprocessing of the multi-modal TEA-seq object (RNA normalization/log1p/HVGs,
    ADT CLR) so markers are comparable
  repo: https://github.com/cantinilab/mowgli
- name: Azimuth
  role: Independent RNA-based reference mapping used to cross-confirm marker-based
    cell-type annotations
- name: Mowgli
  role: Provides the paired multi-omics embedding on which the graph, clustering,
    and annotation are performed
  repo: https://github.com/cantinilab/mowgli
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:9e69aeab8753c25fd1397b1a22a749d057e2ffee75f4d9ad98a7df7c23535a08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Azimuth
  - Human Protein Atlas
  - Mowgli
  - Muon
  - Python
  - Scanpy
  - Signac
  - gProfiler
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cell-type-annotation-marker-genes

## Summary

Annotates cell types in single-cell embeddings by building a kNN graph, clustering with Leiden, and assigning labels from canonical gene and protein markers, cross-confirmed against an independent reference mapping (e.g., Azimuth PBMC reference). Used in the Mowgli paper to obtain cell-type labels on TEA-seq that enable factor-specificity analysis.

## When to use

Use when you have a cell-by-factor or low-dimensional embedding from paired multi-omics data (e.g., Mowgli on TEA-seq PBMC) and need per-cell type labels grounded in canonical gene/protein markers plus an independent reference annotation.

## When NOT to use

- Input is raw counts without normalization (RNA should first be normalized to 10,000 counts and log1p-transformed; ADT CLR-normalized via Muon)
- The goal is de novo marker discovery rather than annotation of known populations against canonical markers and a reference
- Data lack a credible reference (e.g., non-human PBMC) where Azimuth-based cross-validation of marker-based labels cannot support the assignment

## Inputs

- Mowgli cell embedding (shared latent matrix W) from a TEA-seq Muon object
- Normalized RNA expression (log1p, top 2,500 HVGs) and CLR-normalized ADT protein data for marker assessment
- Azimuth PBMC reference for independent RNA-based annotation

## Outputs

- Per-cell Leiden cluster assignments
- Per-cluster cell-type labels (NK, CD8 T, CD4 T, and other PBMC populations) from canonical gene/protein markers
- Reference-confirmed cell-type annotations for use in factor-specificity scoring

## How to apply

Build a kNN graph on the Mowgli embedding (k=20, Euclidean distance) and cluster with the Leiden algorithm at a coarse resolution so clusters correspond to major immune populations. Assign each Leiden cluster a cell type using canonical RNA gene markers and surface protein markers from the ADT modality (TEA-seq profiles scRNA-seq, scATAC-seq, and surface proteins). Cross-confirm the marker-based assignment with an independent RNA-based annotation obtained through Azimuth's PBMC reference mapping. These labels then feed downstream specificity scoring, where for each factor and cell type the proportion of cells with absolute weight > 10^-3, mean weight within (a_i) and outside (b_i) the cell type, and the specificity score (a_i - b_i)/max_j a_j are computed. The same annotation procedure is applied separately to embeddings from comparator methods (Mowgli, integrative NMF, MOFA+) so factor–cell-type associations are comparable.

## Related tools

- **Scanpy** (kNN graph construction and Leiden clustering of the Mowgli embedding for marker-based annotation) — https://github.com/cantinilab/mowgli
- **Muon** (Preprocessing of the multi-modal TEA-seq object (RNA normalization/log1p/HVGs, ADT CLR) so markers are comparable) — https://github.com/cantinilab/mowgli
- **Azimuth** (Independent RNA-based reference mapping used to cross-confirm marker-based cell-type annotations)
- **Mowgli** (Provides the paired multi-omics embedding on which the graph, clustering, and annotation are performed) — https://github.com/cantinilab/mowgli

## Examples

```
import scanpy as sc; sc.pp.neighbors(emb, n_neighbors=20, metric='euclidean'); sc.tl.leiden(emb, resolution=0.5, key_added='leiden')
```

## Evaluation signals

- Leiden clustering at coarse resolution yields clusters that align with canonical marker expression (e.g., NK, CD8 T, CD4 T populations identifiable by gene and protein markers)
- Marker-based labels agree with the independent Azimuth PBMC reference annotation (Supplementary Fig. 3 comparison)
- Annotated cell-type labels support a clean one-to-one factor–cell-type structure in the specificity score (a_i - b_i)/max_j a_j, as reported for Mowgli versus MOFA+/integrative NMF
- The same annotation procedure applied to comparator embeddings (integrative NMF, MOFA+) yields consistent cell-type definitions across methods

## Limitations

- The paper notes the lack of high-quality biological annotation of cells in paired multi-omics benchmark datasets, so reference-based confirmation quality depends on the reference (Azimuth PBMC here)
- Factor-specificity conclusions depend on the annotation: MOFA+ and integrative NMF aggregate information from many factors for the same cell type, so annotation choices influence the specificity comparison
- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); embedding quality, and thus cluster separability, varies with k

## Evidence

- [results] Independent clustering and marker-based annotation of embeddings: "we separately clustered the embeddings obtained from Mowgli, inte- grative NMF, and MOFA+ and annotated them based on gene and protein markers"
- [results] Azimuth cross-confirmation of annotations: "match an independent RNA-based annotation obtained through Azimuth (see Supplementary Fig. 3)"
- [results] Specificity scoring using annotated cell types: [evidence span withheld — anchor status: not_found]
- [other] kNN graph and Leiden parameters from the reproducibility workflow: [evidence span withheld — anchor status: not_found]
- [results] Mowgli defines clear factor–cell-type associations: [evidence span withheld — anchor status: not_found]
