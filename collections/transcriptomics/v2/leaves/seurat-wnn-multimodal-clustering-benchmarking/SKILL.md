---
name: seurat-wnn-multimodal-clustering-benchmarking
description: Use when benchmarking a decision-level or ensemble single-cell integration
  method against a joint-latent-space baseline, especially for RNA plus a second modality
  (splicing PSI, ADT, ATAC) where modality-specific subclusters are expected.
when_to_use_negative:
- When the goal is a final biological clustering rather than a benchmark/baseline
  — WNN is used here as the competing method, not the preferred integration strategy
- When the second modality's signal is sparse or missing for many cells (e.g. PSI
  available in few cells), since joint-embedding integration may dilute modality-specific
  subclusters
- When you need to mix-and-match prior annotations from label transfer or multiple
  algorithms/resolutions — WNN is restricted to supported modalities and does not
  reconcile arbitrary conflicting annotation sets
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat-WNN
  role: Provides the weighted-nearest-neighbor multimodal integration run as the competing/benchmark
    clustering against scTriangulate (e.g. 50 PCs RNA, 30 PCs PSI on AML data)
- name: scTriangulate 0.9.0
  role: Decision-level integration method whose outputs (lazy_run with win_fraction_cutoff=0.25)
    are compared against the WNN benchmark
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy 1.7.2
  role: Preprocessing/QC of the RNA and splicing modalities fed to both integration
    strategies; used within the scTriangulate workflow via scanpy_recipe and concat_rna_and_other
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:e3fec992c55fa03661dcf2b10c2cb04867c401e60352525e1f8903d8af4be0d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - MultiPath-PSI
  - Seurat-WNN
  - scTriangulate
  - scTriangulate 0.9.0
  - scanpy
  - scanpy 1.7.2
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# seurat-wnn-multimodal-clustering-benchmarking

## Summary

Run Seurat's weighted nearest neighbor (WNN) multimodal integration as a competing clustering solution to benchmark decision-level integration methods such as scTriangulate, testing whether modality-defined subpopulations are resolved. In the scTriangulate paper, WNN failed to resolve splicing-defined pediatric AML subclusters that scTriangulate recovered.

## When to use

Use when benchmarking a decision-level or ensemble single-cell integration method against a joint-latent-space baseline, especially for RNA plus a second modality (splicing PSI, ADT, ATAC) where modality-specific subclusters are expected.

## When NOT to use

- When the goal is a final biological clustering rather than a benchmark/baseline — WNN is used here as the competing method, not the preferred integration strategy
- When the second modality's signal is sparse or missing for many cells (e.g. PSI available in few cells), since joint-embedding integration may dilute modality-specific subclusters
- When you need to mix-and-match prior annotations from label transfer or multiple algorithms/resolutions — WNN is restricted to supported modalities and does not reconcile arbitrary conflicting annotation sets

## Inputs

- Normalized single-cell RNA count matrix (AnnData h5ad, log-normalized)
- Second-modality feature matrix in matching cells (e.g. PSI splicing values, ADT CLR-normalized, or ATAC)

## Outputs

- Seurat-WNN integrated cluster assignments per cell
- Joint WNN embedding/UMAP for visualization
- Benchmark comparison against scTriangulate clusters (resolved vs. unresolved subpopulations)

## How to apply

Run Seurat WNN on the same modalities supplied to the competing method, computing modality-specific dimensionality reductions — e.g. 50 PCs for RNA and 30 PCs for splicing (PSI) values in the pediatric AML case — and let WNN weight neighbors across modalities. Then compare the WNN clustering to the alternative (e.g. scTriangulate with win_fraction_cutoff=0.25 on concatenated normalized RNA and pre-imputed PSI values) via UMAP visualization and marker inspection. The acceptance check is qualitative: the alternative solution must retain modality-defined subpopulations distinguishable by reported markers (KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 splicing events in AML), whereas WNN should fail to resolve them. This comparison mirrors the reported Supplementary Fig. 10f,g contrast and illustrates WNN's limitation: it integrates at the data level via a joint embedding/geometric graph, so weak modalities may not drive cluster subdivision.

## Related tools

- **Seurat-WNN** (Provides the weighted-nearest-neighbor multimodal integration run as the competing/benchmark clustering against scTriangulate (e.g. 50 PCs RNA, 30 PCs PSI on AML data))
- **scTriangulate 0.9.0** (Decision-level integration method whose outputs (lazy_run with win_fraction_cutoff=0.25) are compared against the WNN benchmark) — https://github.com/frankligy/scTriangulate
- **scanpy 1.7.2** (Preprocessing/QC of the RNA and splicing modalities fed to both integration strategies; used within the scTriangulate workflow via scanpy_recipe and concat_rna_and_other)

## Examples

```
from sctriangulate import *; sctri = ScTriangulate(dir='./output', adata=adata, query=['rna_leiden','psi_nmf_k3']); sctri.lazy_run()  # compare against Seurat WNN (50 PCs RNA, 30 PCs PSI)
```

## Evaluation signals

- WNN runs on identical input cells and modalities as the competing method, with modality-specific PCs (e.g. 50 for RNA, 30 for PSI) matching the protocol
- UMAP visualization shows whether splicing-defined subpopulations (KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 events in AML blasts) are resolved by WNN vs. scTriangulate
- The qualitative acceptance check: scTriangulate retains the modality-defined subclusters while the WNN solution does not resolve them (mirroring Supplementary Fig. 10f,g)
- Both integrations consume the same normalized inputs — concatenated normalized RNA and pre-imputed PSI AnnData via concat_rna_and_other for scTriangulate
- Comparison is reproducible using the scripts and inputs deposited in the reproduce folder and Synapse storage (syn26320337)

## Limitations

- WNN integrates at the data level via a joint latent space/geometric graph, which can fail to resolve modality-defined subtypes that decision-level integration recovers
- WNN cannot jointly consider prior annotations from label transfer methods and is limited to specific modalities and unsupervised clustering types
- Results depend on the software resolution selected and the number of PCs per modality chosen

## Evidence

- [results] whereas such clusters could not be resolved through multimodal integration using Seurat WNN: [evidence span withheld — anchor status: not_found]
- [intro] While other multimodal methods integrate at the data level: "While other multimodal methods integrate at the data level (through either a low-dimensional latent space6 or through a geometric graph5), scTriangulate integrates results at a decision level to"
- [intro] these and other methods are: (1) limited to specific modalities: "these and other methods are: (1) limited to specific modalities, (2) cannot jointly consider prior annotations from label transfer methods, and (3) are limited to specific types of unsupervised"
- [methods] we concatenate both properly normalized RNA and ADT values: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function."
- [abstract] a computational framework to mix-and-match multiple clustering results: "We describe scTriangulate, a computational framework to mix-and-match multiple clustering results, modalities, associated algorithms, and resolutions to achieve an optimal solution."
