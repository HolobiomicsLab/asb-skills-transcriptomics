---
name: biology-conservation-metric-scoring
description: Use when you have joint latent embeddings of two or more species' scRNA-seq
  data from an integration algorithm (e.g. scVI, scANVI, SeuratV4) and need to score
  species mixing versus biology conservation in the BENGAL benchmark.
when_to_use_negative:
- The cross-species cell type correspondence involves valid non-one-to-one cell type
  mappings — the benchmark's quantitative scoring assumes one-to-one cell type mappings
  and known cell type homology
- Cell type annotations are unreliable or literature-derived cell type homology is
  uncertain, since the scoring system assumes accurate annotations and confident homology
- The task is integrative analysis of a single species' batches rather than cross-species
  integration — species mixing is undefined in that setting
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BENGAL
  role: Benchmarking pipeline that computes the 9 established metrics plus the new
    biology conservation metric on each homology-method x algorithm combination
- name: scVI
  role: Reference integration algorithm whose embeddings are scored; balances species-mixing
    and biology conservation
- name: scANVI
  role: Reference integration algorithm whose embeddings are scored; balances species-mixing
    and biology conservation
- name: SeuratV4
  role: Reference integration algorithm whose embeddings are scored; balances species-mixing
    and biology conservation
  repo: https://github.com/satijalab/seurat-wrappers
- name: scOntoMatch
  role: Unifies cell ontology annotations across datasets (ontoMultiMinimal, ontoMultiMatch,
    ontoTranslate) so cell type distinguishability can be compared across species
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:3667e7818890d23f2d5f39002304de7d3a40890f1e7518d19b358143c9f14ceb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-wrappers
  tools:
  - BENGAL
  - SeuratV4
  - scANVI
  - scOntoMatch
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# biology-conservation-metric-scoring

## Summary

Scores cross-species single-cell RNA-seq integrations with the 9 established integration metrics plus a new biology conservation metric that quantifies maintenance of cell type distinguishability, as implemented in the BENGAL benchmarking pipeline. It lets a practitioner judge whether an integration algorithm preserved species-specific biology rather than over-mixing cells.

## When to use

Use when you have joint latent embeddings of two or more species' scRNA-seq data from an integration algorithm (e.g. scVI, scANVI, SeuratV4) and need to score species mixing versus biology conservation in the BENGAL benchmark.

## When NOT to use

- The cross-species cell type correspondence involves valid non-one-to-one cell type mappings — the benchmark's quantitative scoring assumes one-to-one cell type mappings and known cell type homology
- Cell type annotations are unreliable or literature-derived cell type homology is uncertain, since the scoring system assumes accurate annotations and confident homology
- The task is integrative analysis of a single species' batches rather than cross-species integration — species mixing is undefined in that setting

## Inputs

- Joint latent-space embeddings of cross-species scRNA-seq datasets produced by integration algorithms (scVI, scANVI, SeuratV4, fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SAMap)
- Cell type annotations unified across datasets via cell ontology matching (scOntoMatch ontoMultiMinimal / ontoMultiMatch)
- Homologous-gene feature spaces built under different homology strategies (O2O one-to-one orthologs vs HE/SH strategies retaining in-paralogs), from curated sequence-based orthology annotations

## Outputs

- Per-combination scores from the 9 established integration metrics plus the biology conservation metric (maintenance of cell type distinguishability)
- homology_strategy_comparison.csv aggregating integrated and biology-conservation scores by homology method
- Ranking of homology-method x algorithm combinations, e.g. HE/SH > O2O on biology conservation for the distant Embryo_xt_dr task

## How to apply

Within the BENGAL pipeline, load the species-pair task (e.g. Embryo_xt_dr, Xenopus tropicalis vs Danio rerio) and, for each homology-method x algorithm combination (28 combinations benchmarked), embed both species in a joint latent space. Score every combination with the 9 established integration metrics plus the new biology conservation metric, which assesses maintenance of cell type distinguishability using curated cell ontology annotations matched across datasets with scOntoMatch. Aggregate integrated and biology-conservation scores by homology method into homology_strategy_comparison.csv and inspect the direction of the comparison — for evolutionarily distant tasks, in-paralog-including homology strategies (HE/SH) should outperform one-to-one orthologs (O2O) on biology conservation. Trust scANVI, scVI and SeuratV4 as reference algorithms because they achieve a balance between species-mixing and biology conservation.

## Related tools

- **BENGAL** (Benchmarking pipeline that computes the 9 established metrics plus the new biology conservation metric on each homology-method x algorithm combination)
- **scVI** (Reference integration algorithm whose embeddings are scored; balances species-mixing and biology conservation)
- **scANVI** (Reference integration algorithm whose embeddings are scored; balances species-mixing and biology conservation)
- **SeuratV4** (Reference integration algorithm whose embeddings are scored; balances species-mixing and biology conservation) — https://github.com/satijalab/seurat-wrappers
- **scOntoMatch** (Unifies cell ontology annotations across datasets (ontoMultiMinimal, ontoMultiMatch, ontoTranslate) so cell type distinguishability can be compared across species) — https://github.com/Papatheodorou-Group/scOntoMatch

## Evaluation signals

- homology_strategy_comparison.csv reproduces the reported direction: for the evolutionarily distant Embryo_xt_dr task, in-paralog-including strategies (HE/SH) outperform O2O on the biology-conservation score
- Reference algorithms scANVI, scVI and SeuratV4 show balanced species-mixing and biology conservation scores rather than extreme over-mixing
- All 28 homology-method x integration-algorithm combinations receive scores from each of the 9 established metrics plus the biology conservation metric (no missing scoring cells)
- Cell type labels used in scoring are ontology-matched across datasets (post ontoMultiMatch), so cell type distinguishability is comparable between species
- Scored datasets pass the assumption checks: accurate cell type annotations and known cell type homology from literature

## Limitations

- Quantitative scoring of species mixing and biology conservation rests on two assumptions: (1) cell type annotations are accurate; (2) known cell type homology from literature is correct
- The benchmark uses one-to-one cell type mappings; valid non-one-to-one cell type mappings across species exist and are not assessed
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes
- Functional diversification between orthologs can exceed that between paralogs, so orthology-based matching may mislead conservation scoring

## Evidence

- [abstract] New biology conservation metric definition: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [abstract] Reference algorithms balance the two score families: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [abstract] Expected direction for distant species: "For evolutionarily distant species, including in-paralogs is beneficial."
- [discussion] Assumptions underlying the scoring system: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate;"
- [discussion] One-to-one cell type mapping restriction: "the different limitations of such approaches, using one-to-one cell type mappings. While valid non-one-to-one cell type mappings across species do exist"
- [readme] Ontology annotation unification supporting cross-dataset scoring: "scOntoMatch is an R package which unifies ontology annotation of scRNA-seq datasets to make them comparable across studies."
