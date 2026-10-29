---
name: single-cell-data-integration-benchmarking
description: Decision support for choosing a homologous-gene mapping and integration
  algorithm when integrating scRNA-seq datasets across species, especially when species
  divergence time, number of species, or one-to-one vs in-paralog homology strategies
  may affect integration quality.
when_to_use_negative:
- When the goal is not cross-species integration but within-species batch correction
  — the pipeline's scoring system assumes accurate cell type annotations and known
  cross-species cell type homology
- When non-one-to-one cell type mappings across species must be assessed — the benchmark
  is limited to one-to-one cell type mappings and lacks specialised assessment metrics
  for such cases
- When species-specific genes must inform the integration and only curated orthology
  is available — the approach relies heavily on curated gene orthology and ignores
  species-specific genes unless using UINMF/SAMap-style unshared-feature methods
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3308
tools:
- name: BENGAL
  role: Benchmarking pipeline that orchestrates homology strategy construction, integration,
    and metric scoring
- name: scVI
  role: Deep generative integration algorithm balancing species-mixing and biology
    conservation
- name: scANVI
  role: Semi-supervised variant of scVI used to embed species jointly while conserving
    cell type identity
- name: SeuratV4
  role: Anchor-based integration algorithm evaluated in the benchmark
  repo: https://github.com/satijalab/seurat-wrappers
- name: SAMap
  role: Heuristic cross-species mapper that outperforms on whole-body atlases with
    challenging gene homology annotation
  repo: https://github.com/atarashansky/SAMap
- name: LIGER
  role: Integrative NMF-based integration algorithm benchmarked via rliger
  repo: https://github.com/welch-lab/liger
- name: fastMNN
  role: Mutual-nearest-neighbour integration algorithm benchmarked via batchelor
  repo: https://github.com/LTLA/batchelor
- name: scOntoMatch
  role: Unifies cell ontology annotations across datasets before scoring biology conservation
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:10ac52caa2d565abbff5e24dcc79892161312fea5577fbf8f15dc2b522570615
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-wrappers
  tools:
  - BENGAL
  - LIGER
  - SAMap
  - SeuratV4
  - fastMNN
  - scANVI
  - scOntoMatch
  - scVI
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-cell-data-integration-benchmarking

## Summary

A skill for benchmarking cross-species integration of single-cell RNA-seq data by comparing combinations of gene homology mapping strategies (e.g., one-to-one orthologs vs in-paralog-including HE/SH mappings) with integration algorithms (scVI, scANVI, SeuratV4, fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SAMap) inside the BENGAL pipeline, scoring results with 9 established metrics plus a biology-conservation metric.

## When to use

Decision support for choosing a homologous-gene mapping and integration algorithm when integrating scRNA-seq datasets across species, especially when species divergence time, number of species, or one-to-one vs in-paralog homology strategies may affect integration quality.

## When NOT to use

- When the goal is not cross-species integration but within-species batch correction — the pipeline's scoring system assumes accurate cell type annotations and known cross-species cell type homology
- When non-one-to-one cell type mappings across species must be assessed — the benchmark is limited to one-to-one cell type mappings and lacks specialised assessment metrics for such cases
- When species-specific genes must inform the integration and only curated orthology is available — the approach relies heavily on curated gene orthology and ignores species-specific genes unless using UINMF/SAMap-style unshared-feature methods

## Inputs

- Species-specific single-cell RNA-seq datasets (e.g., Seurat RDS files, h5ad for SAMap/scVI), such as Tabula Muris and Tabula Sapiens Smartseq-2 lung data or Xenopus tropicalis vs Danio rerio embryo atlases
- Curated sequence-based gene orthology annotations (one-to-one orthologs and one-to-many/many-to-many in-paralogs)
- Cell ontology .obo file (e.g., Cell Ontology cl-basic.obo, Uberon, ZFA) loaded via ontologyIndex::get_OBO
- Metadata table mapping datasets to files, parsed with a tab separator

## Outputs

- Joint latent-space embeddings of cells from multiple species per homology-strategy × algorithm combination
- Aggregated integration and biology-conservation score tables (e.g., homology_strategy_comparison.csv)
- Matched/minimal cell-type ontology annotations across datasets (ontoMultiMatch/ontoMultiMinimal outputs)
- Ranked comparison of homology mapping × integration algorithm combinations (28 combinations benchmarked)

## How to apply

Load species-specific scRNA-seq datasets (e.g., Seurat RDS via getSeuratRds or h5ad) into the BENGAL pipeline and unify cell type annotations with scOntoMatch (ontoMultiMinimal to trim the ontology tree, ontoMultiMatch to match labels, ontoTranslate to map gene/cell-type identifiers). Build homologous-gene feature spaces under each homology strategy — one-to-one orthologs (O2O) versus strategies retaining in-paralogs/co-orthologs (HE/SH) — from curated sequence-based orthology annotations. Run each integration algorithm (scVI, scANVI, SeuratV4, fastMNN, Harmony, LIGER/UINMF, Scanorama, SAMap) on every homology gene set to embed species in a joint latent space. Score each homology-method × algorithm combination with the 9 established integration metrics plus the new biology-conservation metric for maintenance of cell type distinguishability, then aggregate results (e.g., homology_strategy_comparison.csv) to compare strategies. Key decision points: include in-paralogs for evolutionarily distant species pairs (e.g., Xenopus tropicalis vs Danio rerio), and prefer SAMap for whole-body atlases with challenging gene homology annotation.

## Related tools

- **BENGAL** (Benchmarking pipeline that orchestrates homology strategy construction, integration, and metric scoring)
- **scVI** (Deep generative integration algorithm balancing species-mixing and biology conservation)
- **scANVI** (Semi-supervised variant of scVI used to embed species jointly while conserving cell type identity)
- **SeuratV4** (Anchor-based integration algorithm evaluated in the benchmark) — https://github.com/satijalab/seurat-wrappers
- **SAMap** (Heuristic cross-species mapper that outperforms on whole-body atlases with challenging gene homology annotation) — https://github.com/atarashansky/SAMap
- **LIGER** (Integrative NMF-based integration algorithm benchmarked via rliger) — https://github.com/welch-lab/liger
- **fastMNN** (Mutual-nearest-neighbour integration algorithm benchmarked via batchelor) — https://github.com/LTLA/batchelor
- **scOntoMatch** (Unifies cell ontology annotations across datasets before scoring biology conservation) — https://github.com/Papatheodorou-Group/scOntoMatch

## Examples

```
obj_list = getSeuratRds(metadata = metadata, sep = "\t"); obj_list_matched = scOntoMatch::ontoMultiMatch(obj_list = obj_list_minimal, anno_col = 'cell_ontology_base', onto_id_col = onto_id_col, ont = ont)
```

## Evaluation signals

- Reproduce the direction that for evolutionarily distant pairs (e.g., Embryo_dr_xt, Xenopus tropicalis vs Danio rerio), in-paralog-including homology strategies (HE/SH) outperform one-to-one-only (O2O) on the biology-conservation score
- Confirm scVI, scANVI and SeuratV4 achieve a balance between species-mixing and biology conservation across the 9 established metrics
- Verify the aggregated table (homology_strategy_comparison.csv) covers all homology-method × algorithm combinations (28 combinations benchmarked in the paper)
- Check that cell type annotations were unified across datasets via scOntoMatch (minimal ontology tree and matched mappings) before scoring, since quantitative scoring assumes accurate annotations and known cell type homology
- Confirm HVG sets selected under each homology strategy are recorded alongside scores so homology-method effects are attributable to feature space, not algorithm settings

## Limitations

- Quantitative scoring assumes (1) accurate cell type annotations and (2) confident known cell type homology from literature
- The benchmark uses one-to-one cell type mappings; valid non-one-to-one cell type mappings across species are not assessed
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes; flat one-to-one/one-to-many mappings do not comprehensively describe cross-species cell type correspondence
- Functional diversification between orthologs may exceed that between paralogs (out-paralogs can show higher expression similarity than orthologs), and expression similarity may arise from convergence or concerted evolution rather than homology
- SAMap is heuristic and does not explain observed cell type expression similarity

## Evidence

- [abstract] Benchmarking of 28 combinations: "Here, we benchmark 28 combinations of gene homology mapping methods and data integration algorithms in a variety of biological settings."
- [abstract] In-paralogs beneficial for distant species: "For evolutionarily distant species, including in-paralogs is beneficial."
- [abstract] Biology conservation metric: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [intro] BENGAL pipeline development: "we develop the BENchmarking strateGies for cross-species integrAtion of singLe-cell RNA sequencing data (BENGAL) pipeline"
- [discussion] Scoring assumptions: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate; (2) known cell type homology from literature is"
- [methods] getSeuratRds data loading: "obj_list = getSeuratRds(metadata = metadata, sep = "\t")"
- [methods] Ontology tree trimming: "We provide `ontoMultiMinimal` for Merging descendant terms to existing ancestor terms in one dataset, to get a minimum ontology representation of the cell type tree."
- [readme] SAMap installation: [evidence span withheld — anchor status: external_doc]
