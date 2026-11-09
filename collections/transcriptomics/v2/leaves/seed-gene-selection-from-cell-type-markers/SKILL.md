---
name: seed-gene-selection-from-cell-type-markers
description: Use when starting CASCC feature selection on a normalized single-cell
  RNA-seq matrix and you need a biologically grounded seed gene (from known cell-type
  markers) to launch findAttractor.adaptive.
when_to_use_negative:
- Input is raw counts or not normalized — run filterGenes and normalization (inputDataType
  = 'well-normalized' for CASCC.featureSelection) before seeding attractors
- You only need a global clustering result without attractor introspection — CASCC::run.CASCC(Data_PDAC_peng_2k)
  wraps the whole pipeline instead of manual seed selection
- No biologically validated marker genes are available for the expected cell types,
  since the attractor signature quality depends on the seed
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CASCC
  role: Provides findAttractor.adaptive and scanSeeds.adaptive, which grow ranked
    co-expression attractor signatures from a seed marker gene
  repo: https://github.com/LingyiC/CASCC
- name: cafr
  role: Dependency implementing the attractor (co-expression) algorithm family underlying
    findAttractor.adaptive
  repo: https://github.com/weiyi-bitw/cafr
- name: Seurat
  role: Version 4.3.0 dependency; hosts the scRNA-seq workflow into which CASCC attractor-based
    feature selection is run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:498b20a442c821240c69de1f2c7da8fdbcb3bf6acd9580371d392bd06e33db6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - R
  - Seurat
  - cafr
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Seed gene selection from cell-type markers

## Summary

Selects known cell-type marker genes (e.g. the macrophage marker AIF1) as seed genes for the CASCC unsupervised adaptive attractor algorithm, so that the converged attractor yields a ranked co-expression signature anchored in biology.

## When to use

Use when starting CASCC feature selection on a normalized single-cell RNA-seq matrix and you need a biologically grounded seed gene (from known cell-type markers) to launch findAttractor.adaptive.

## When NOT to use

- Input is raw counts or not normalized — run filterGenes and normalization (inputDataType = 'well-normalized' for CASCC.featureSelection) before seeding attractors
- You only need a global clustering result without attractor introspection — CASCC::run.CASCC(Data_PDAC_peng_2k) wraps the whole pipeline instead of manual seed selection
- No biologically validated marker genes are available for the expected cell types, since the attractor signature quality depends on the seed

## Inputs

- Normalized, log-transformed single-cell RNA-seq expression matrix (e.g. genes x cells, such as Data_PDAC_peng_2k, 'well-normalized' input)
- Known cell-type marker gene symbols to use as seeds (e.g. AIF1 for macrophages)

## Outputs

- Converged attractor object (attr.res$attractor.final) containing a ranked co-expression gene list
- Ranked gene signature file (e.g. aif1_attractor_signature.csv with gene symbols plus rank order)
- Feature-selected matrix used by predictClusterK for cluster-number prediction

## How to apply

Load the normalized, log-transformed expression matrix (e.g. the package dataset Data_PDAC_peng_2k) in R after installing CASCC with its dependencies Seurat 4.3.0 and cafr. Choose a seed gene from a validated cell-type marker — in the tutorial the macrophage marker AIF1 is used — and call findAttractor.adaptive on the expression matrix with that seed; CASCC.featureSelection exposes exponent.max = 10 and exponent.min = 2 as the adaptive exponent search defaults. Extract the converged attractor from attr.res$attractor.final, preserving the ranked order in which the top-ranked genes are the most representative co-expressed features. For genome-wide marker-driven seeding, scanSeeds.adaptive can iterate seeds over a marker list with parameters epsilon, exponent.max, exponent.min, seedTopN, compareNth, and mc.cores. The resulting ranked attractor list feeds downstream feature selection and clustering (predictClusterK with ward.D2 and silhouette), where each cluster corresponds to a specific attractor.

## Related tools

- **CASCC** (Provides findAttractor.adaptive and scanSeeds.adaptive, which grow ranked co-expression attractor signatures from a seed marker gene) — https://github.com/LingyiC/CASCC
- **cafr** (Dependency implementing the attractor (co-expression) algorithm family underlying findAttractor.adaptive) — https://github.com/weiyi-bitw/cafr
- **Seurat** (Version 4.3.0 dependency; hosts the scRNA-seq workflow into which CASCC attractor-based feature selection is run)

## Examples

```
attr.res <- findAttractor.adaptive(Data_PDAC_peng_2k, "AIF1"); write.csv(data.frame(gene=names(attr.res$attractor.final), rank=seq_along(attr.res$attractor.final)), "aif1_attractor_signature.csv")
```

## Evaluation signals

- attr.res$attractor.final exists and is a ranked gene list whose leading entries include the seed gene AIF1
- Top-ranked attractor genes are consistent with macrophage markers as reported in the tutorial output
- Ranked signature exports successfully (e.g. aif1_attractor_signature.csv exists with gene symbol and rank columns)
- The resulting attractor-based features yield eight distinct clusters in Data_PDAC_peng_2k, each corresponding to a specific attractor
- Adaptive exponent search stays within the package defaults (exponent.max = 10, exponent.min = 2) without convergence failure

## Limitations

- Signatures are seed-dependent: the ranked co-expression signature reflects the chosen marker (AIF1), so a poor or ambiguous marker produces a poorly targeted attractor
- The demonstration dataset Data_PDAC_peng_2k is small (the tutorial notes '# 400 cells'), so attractor behavior may differ on larger or differently normalized matrices
- Requires the pinned Seurat 4.3.0 dependency and the cafr package; CASCC::checkDependencies() should be run after installation

## Evidence

- [methods] Finding co-expression signatures with adaptive attractor algorithm: "attr.res <- findAttractor.adaptive(Data_PDAC_peng_2k, "AIF1")"
- [intro] Adaptive attractor defines the method's co-expression features: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [methods] Attractors correspond to clusters in the benchmark dataset: "The analysis identifies the presence of eight distinct clusters, each corresponding to a specific attractor"
- [readme] CASCC install path and dependency chain for running attractor functions: "remotes::install_version("Seurat", "4.3.0") # dependency"
