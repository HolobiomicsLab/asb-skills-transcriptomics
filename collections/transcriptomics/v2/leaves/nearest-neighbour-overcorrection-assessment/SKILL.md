---
name: nearest-neighbour-overcorrection-assessment
description: Use when benchmarking cross-species (or multi-dataset) integration strategies
  and you need to check whether integrated embeddings preserve distinguishability
  of cell types present in only a subset of input datasets, as species count increases.
when_to_use_negative:
- When the goal is species mixing rather than biology conservation — iso_F1 measures
  cell type distinguishability, not how well cells from different species mix in the
  embedding
- When cell type annotations are untrusted or cross-species cell type homology is
  unknown — the metric assumes accurate annotations and known one-to-one cell type
  homology from the literature
- When the setting requires non-one-to-one cell type mappings across species — the
  benchmark explicitly assessed only one-to-one cell type mappings
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: fastMNN
  role: Nearest-neighbour-based integration strategy whose embeddings are assessed
    for overcorrection (declining iso_F1)
  repo: https://github.com/LTLA/batchelor
- name: SeuratV4
  role: Provides CCA and RPCA integration variants whose iso_F1 trend is compared
    against fastMNN
  repo: https://github.com/satijalab/seurat-wrappers
- name: BENGAL
  role: Pipeline that generates the integrated embeddings and assessment framework
    in which iso_F1 is computed
- name: scOntoMatch
  role: Unifies cell ontology annotation across datasets (ontoMultiMinimal, ontoMultiMatch)
    to build the shared homology mapping used as iso_F1 reference labels
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
- name: LIGER / LIGER UINMF
  role: iNMF-based integration strategies benchmarked in the same iso_F1 comparison
  repo: https://github.com/welch-lab/liger
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:cb9bccded488c36fb818bb57c57b31425553ac8bf24ed2c84316b145a366bcc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LTLA/batchelor
  tools:
  - BENGAL
  - Harmony
  - LIGER
  - LIGER / LIGER UINMF
  - LIGER UINMF
  - SAMap
  - Scanorama
  - SeuratV4
  - fastMNN
  - scANVI
  - scOntoMatch
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nearest-neighbour-overcorrection-assessment

## Summary

This skill quantifies overcorrection in cross-species scRNA-seq integration by computing the isolated-label F1 (iso_F1) biology-conservation metric per integration strategy and per cumulative number of species, detecting when nearest-neighbour methods (fastMNN, SeuratV4 CCA/RPCA) merge cell types found in only a few datasets.

## When to use

Use when benchmarking cross-species (or multi-dataset) integration strategies and you need to check whether integrated embeddings preserve distinguishability of cell types present in only a subset of input datasets, as species count increases.

## When NOT to use

- When the goal is species mixing rather than biology conservation — iso_F1 measures cell type distinguishability, not how well cells from different species mix in the embedding
- When cell type annotations are untrusted or cross-species cell type homology is unknown — the metric assumes accurate annotations and known one-to-one cell type homology from the literature
- When the setting requires non-one-to-one cell type mappings across species — the benchmark explicitly assessed only one-to-one cell type mappings

## Inputs

- Per-strategy integrated cell embeddings from the BENGAL pipeline (28 gene-homology-mapping x integration-algorithm combinations)
- Four-species heart scRNA-seq datasets with cell type annotations
- Shared cell type homology mapping across species (from scOntoMatch ontoMultiMinimal/ontoMultiMatch)
- List of cell types present in only a subset of input datasets (isolated labels)

## Outputs

- iso_F1 table per integration strategy per number-of-species condition (iso_f1_heart.csv)
- Plot of iso_F1 versus number of species per strategy (e.g. fig6c_iso_f1_trend.png)
- Identification of strategies showing overcorrection (falling iso_F1) versus balanced strategies

## How to apply

Load the per-strategy integrated embeddings produced with the BENGAL pipeline (28 gene-homology-mapping x integration-algorithm combinations, e.g. fastMNN, SeuratV4 CCA/RPCA, scVI, scANVI, Harmony, LIGER, LIGER UINMF, Scanorama, SAMap) together with the shared cell-type homology mapping obtained via scOntoMatch (ontoMultiMinimal then ontoMultiMatch). For each strategy and each cumulative species subset, compute iso_F1 — the biology-conservation metric quantifying maintenance of cell type distinguishability — against the homology-mapped cell type labels (in the heart setting only 8 of 22 cell types are shared by all species, so 14 labels are 'isolated'). Tabulate iso_F1 per strategy per number-of-species condition (e.g. writing iso_f1_heart.csv) and plot iso_F1 versus number of species per strategy. Judge correctness by whether nearest-neighbour-based methods (fastMNN, SeuratV4 CCA, SeuratV4 RPCA) show iso_F1 declining with increasing species count — the overcorrection signature — and whether per-strategy scores match the article's reported ranking (scANVI, scVI and SeuratV4 balance species-mixing and biology conservation).

## Related tools

- **fastMNN** (Nearest-neighbour-based integration strategy whose embeddings are assessed for overcorrection (declining iso_F1)) — https://github.com/LTLA/batchelor
- **SeuratV4** (Provides CCA and RPCA integration variants whose iso_F1 trend is compared against fastMNN) — https://github.com/satijalab/seurat-wrappers
- **BENGAL** (Pipeline that generates the integrated embeddings and assessment framework in which iso_F1 is computed)
- **scOntoMatch** (Unifies cell ontology annotation across datasets (ontoMultiMinimal, ontoMultiMatch) to build the shared homology mapping used as iso_F1 reference labels) — https://github.com/Papatheodorou-Group/scOntoMatch
- **LIGER / LIGER UINMF** (iNMF-based integration strategies benchmarked in the same iso_F1 comparison) — https://github.com/welch-lab/liger

## Evaluation signals

- The reproduced iso_F1 trend qualitatively matches the published Fig. 6c pattern: nearest-neighbour-based methods lose cell type distinguishability with increasing species count
- Per-strategy iso_F1 scores match the article's reported ranking (scANVI, scVI and SeuratV4 balancing species-mixing and biology conservation; SAMap outperforming for whole-body atlas integration with challenging homology annotation)
- The tabulated table contains one iso_F1 value per strategy per cumulative number-of-species condition, with heart-specific context (only 8 of 22 cell types shared by all species)
- SeuratV4 CCA shows a markedly low iso_F1 score for few-species cell types, indicating those cell types were not well-separated after integration
- The overcorrection tendency of fastMNN and SeuratV4 CCA/RPCA increases monotonically as more species are added

## Limitations

- The quantitative scoring assumes accurate cell type annotations and confident known cell type homology from the literature
- The benchmark uses one-to-one cell type mappings only; valid non-one-to-one mappings across species are not assessed, so iso_F1 cannot capture them
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes, which limits the embeddings being scored
- Expression similarities across species may arise from convergence or concerted evolution rather than homology; no algorithm disentangles these sources, so a high iso_F1 does not prove true homology

## Evidence

- [abstract] biology conservation metric: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [abstract] balanced strategies ranking: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [intro] species count effect rationale: "the number of species involved in the integration and the time that species have diverged can impact the output characteristics differently among strategies"
- [discussion] one-to-one mapping limitation: "the different limitations of such approaches, using one-to-one cell type mappings. While valid non-one-to-one cell type mappings across species do exist"
- [methods] ontology matching inputs: "obj_list_matched = scOntoMatch::ontoMultiMatch(obj_list = obj_list_minimal, anno_col = 'cell_ontology_base', onto_id_col = onto_id_col, ont = ont)"
