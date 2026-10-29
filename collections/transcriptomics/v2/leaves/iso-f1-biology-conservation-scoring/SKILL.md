---
name: iso-f1-biology-conservation-scoring
description: Use when scoring cross-species scRNA-seq integration strategies (e.g.
  BENGAL outputs) for biology conservation, specifically to detect overcorrection
  where nearest-neighbour methods merge cell types present in only a few species datasets.
when_to_use_negative:
- Input is raw un-integrated scRNA-seq data rather than per-strategy integrated embeddings
  — run an integration algorithm first via the BENGAL pipeline
- Cell type homology across species is non-one-to-one — the article's quantitative
  scoring is limited to one-to-one cell type mappings
- Cell type annotations are uncurated or homology relationships are uncertain — the
  scoring explicitly assumes accurate annotations and confident literature-supported
  homology
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BENGAL
  role: Pipeline that generates the 28 per-strategy integrated embeddings being scored
    and the cross-species integration and assessment framework
- name: fastMNN
  role: Nearest-neighbour integration strategy whose iso_F1 drop with increasing species
    count demonstrates the overcorrection failure mode being quantified
  repo: https://github.com/LTLA/batchelor
- name: SeuratV4
  role: CCA and RPCA integration strategies scored for isolated-label F1; SeuratV4
    CCA showed a low iso_F1 indicating few-species cell types were not well-separated
  repo: https://github.com/satijalab/seurat-wrappers
- name: LIGER
  role: iNMF-based integration strategy among the benchmarked combinations whose biology
    conservation is assessed by iso_F1
  repo: https://github.com/welch-lab/liger
- name: SAMap
  role: Integration strategy for whole-body atlases with challenging gene homology
    annotation, included in the strategy set scored by iso_F1
  repo: https://github.com/atarashansky/SAMap
- name: scOntoMatch
  role: Matches cell ontology labels across datasets (ontoMultiMinimal, ontoMultiMatch)
    to produce the shared cell type homology mapping used as iso_F1 ground truth
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:0204af5638e5aea91cc7baf9d3d323df5037fbaebfc2d8916555e2ced7dd706b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LTLA/batchelor
  tools:
  - BENGAL
  - Harmony
  - LIGER
  - LIGER UINMF
  - SAMap
  - Scanorama
  - SeuratV4
  - fastMNN
  - scANVI
  - scOntoMatch
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# iso-f1-biology-conservation-scoring

## Summary

Computes the isolated-label F1 (iso_F1) biology-conservation score from cross-species integrated embeddings to test whether cell types found in only a few datasets (species-restricted cell types) remain distinguishable after integration. This is the article's new biology conservation metric addressing maintenance of cell type distinguishability alongside 9 established metrics.

## When to use

Use when scoring cross-species scRNA-seq integration strategies (e.g. BENGAL outputs) for biology conservation, specifically to detect overcorrection where nearest-neighbour methods merge cell types present in only a few species datasets.

## When NOT to use

- Input is raw un-integrated scRNA-seq data rather than per-strategy integrated embeddings — run an integration algorithm first via the BENGAL pipeline
- Cell type homology across species is non-one-to-one — the article's quantitative scoring is limited to one-to-one cell type mappings
- Cell type annotations are uncurated or homology relationships are uncertain — the scoring explicitly assumes accurate annotations and confident literature-supported homology

## Inputs

- Per-strategy integrated low-dimensional embeddings from the BENGAL pipeline (28 gene-homology-mapping x integration-algorithm combinations)
- Shared cell type homology mapping across species datasets (one-to-one mapping, e.g. via scOntoMatch ontoMultiMatch)
- Cell type annotation labels per cell, with per-dataset ontology annotation (e.g. heart scRNA-seq datasets with 22 cell types, 8 shared by all species)

## Outputs

- Table of iso_F1 per integration strategy per number-of-species condition (e.g. iso_f1_heart.csv)
- Trend figure of iso_F1 versus number of species per strategy (e.g. fig6c_iso_f1_trend.png)
- Identification of strategies that overcorrect few-species cell types (e.g. low iso_F1 for SeuratV4 CCA)

## How to apply

Load the per-strategy integrated embeddings (28 gene-homology-mapping x integration-algorithm combinations produced by the BENGAL pipeline, e.g. fastMNN, SeuratV4 CCA/RPCA, scVI, scANVI, Harmony, LIGER, LIGER UINMF, Scanorama, SAMap) together with the shared cell type homology mapping. For each strategy and each cumulative species subset, compute the iso_F1 score against the homology mapping — in the heart tasks only 8 of 22 cell types are shared by all species, so 14 are isolated labels. Tabulate iso_F1 per strategy per number-of-species condition (e.g. write to iso_f1_heart.csv) and plot iso_F1 versus number of species (e.g. fig6c_iso_f1_trend.png). Interpretation: nearest-neighbour-based methods (fastMNN, SeuratV4 CCA, SeuratV4 RPCA) tend to overcorrect and merge similar few-species cell types, with iso_F1 dropping as more species are added, whereas scANVI, scVI and SeuratV4 achieve a balance between species-mixing and biology conservation. The scoring assumes cell type annotations are accurate and known cell type homology from literature is correct.

## Related tools

- **BENGAL** (Pipeline that generates the 28 per-strategy integrated embeddings being scored and the cross-species integration and assessment framework)
- **fastMNN** (Nearest-neighbour integration strategy whose iso_F1 drop with increasing species count demonstrates the overcorrection failure mode being quantified) — https://github.com/LTLA/batchelor
- **SeuratV4** (CCA and RPCA integration strategies scored for isolated-label F1; SeuratV4 CCA showed a low iso_F1 indicating few-species cell types were not well-separated) — https://github.com/satijalab/seurat-wrappers
- **LIGER** (iNMF-based integration strategy among the benchmarked combinations whose biology conservation is assessed by iso_F1) — https://github.com/welch-lab/liger
- **SAMap** (Integration strategy for whole-body atlases with challenging gene homology annotation, included in the strategy set scored by iso_F1) — https://github.com/atarashansky/SAMap
- **scOntoMatch** (Matches cell ontology labels across datasets (ontoMultiMinimal, ontoMultiMatch) to produce the shared cell type homology mapping used as iso_F1 ground truth) — https://github.com/Papatheodorou-Group/scOntoMatch

## Evaluation signals

- Reproduced iso_F1 trend qualitatively matches the published Fig. 6c pattern: nearest-neighbour-based methods (fastMNN, SeuratV4 CCA/RPCA) lose cell type distinguishability as species count increases
- Per-strategy iso_F1 scores match the reported ranking, with scANVI, scVI and SeuratV4 balancing species-mixing and biology conservation
- iso_F1 table covers all 28 homology-mapping x integration-algorithm combinations and all cumulative species subsets (8 of 22 heart cell types shared by all species)
- Strategies with high overcorrection (e.g. SeuratV4 CCA) show the expected low iso_F1, confirming few-species cell types are being merged
- Output files iso_f1_heart.csv and fig6c_iso_f1_trend.png are produced with one row/line per strategy-per-species-count condition

## Limitations

- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature
- Benchmark uses one-to-one cell type mappings only; valid non-one-to-one cell type mappings across species are not assessed
- Expression similarities across species may arise from convergence or concerted evolution rather than homology, which iso_F1 cannot disentangle
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes

## Evidence

- [abstract] biology conservation metric: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [discussion] scoring assumptions: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate"
- [readme] LIGER integration: [evidence span withheld — anchor status: external_doc]
- [abstract] benchmark scope: "Here, we benchmark 28 combinations of gene homology mapping methods and data integration algorithms in a variety of biological settings."
