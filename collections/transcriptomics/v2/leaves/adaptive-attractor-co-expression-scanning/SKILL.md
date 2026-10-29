---
name: adaptive-attractor-co-expression-scanning
description: Use when starting from a normalized, log-transformed single-cell expression
  matrix and you need unsupervised, seed-gene-driven co-expression signatures (ranked
  gene lists) to drive feature selection for clustering, as in CASCC.
when_to_use_negative:
- The input is raw (not normalized/log-transformed) counts — run CASCC's filterGenes/normalization
  first; findAttractor.adaptive expects well-normalized data
- You already have a feature table or fixed gene set and only need clustering — go
  straight to predictClusterK/run.kmeans.fixCenters instead of re-deriving attractors
- You want differential expression of finished clusters — use Seurat::FindAllMarkers
  on the feature-selected UMAP object rather than attractor scanning
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CASCC
  role: Provides findAttractor.adaptive, CASCC.featureSelection, scanSeeds.adaptive,
    and collapseAttractorList for adaptive attractor co-expression scanning and downstream
    feature selection
  repo: https://github.com/LingyiC/CASCC
- name: cafr
  role: Dependency package implementing the attractor-finding machinery that CASCC's
    adaptive algorithm builds on
  repo: https://github.com/weiyi-bitw/cafr
- name: Seurat
  role: Required dependency (version 4.3.0) for running CASCC within a Seurat workflow
    and downstream visualization/DEG steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:cecc6e971142b7465129530f960f26906ed3cb0954a7d7ab9324b5ce35c51f6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - R
  - Seurat
  - cafr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# adaptive-attractor-co-expression-scanning

## Summary

Runs CASCC's unsupervised adaptive attractor algorithm (via findAttractor.adaptive, backed by the cafr package) on a normalized single-cell expression matrix to converge on gene co-expression signatures, each a ranked gene list whose top-ranked genes serve as representative features for downstream clustering/feature selection.

## When to use

Use when starting from a normalized, log-transformed single-cell expression matrix and you need unsupervised, seed-gene-driven co-expression signatures (ranked gene lists) to drive feature selection for clustering, as in CASCC.

## When NOT to use

- The input is raw (not normalized/log-transformed) counts — run CASCC's filterGenes/normalization first; findAttractor.adaptive expects well-normalized data
- You already have a feature table or fixed gene set and only need clustering — go straight to predictClusterK/run.kmeans.fixCenters instead of re-deriving attractors
- You want differential expression of finished clusters — use Seurat::FindAllMarkers on the feature-selected UMAP object rather than attractor scanning

## Inputs

- Normalized, log-transformed gene expression matrix (genes x cells), e.g. Data_PDAC_peng_2k shipped with the CASCC R package
- Seed gene symbol with biological meaning (e.g. macrophage marker AIF1)
- Optional: a marker list for scanSeeds.adaptive to scan multiple seeds

## Outputs

- attr.res$attractor.final: converged attractor as a ranked list of co-expressed genes (top-ranked = most representative features)
- Ranked attractor gene list written to file (e.g. aif1_attractor_signature.csv with gene symbols plus rank order)
- Optionally, a collapsed attractor list from collapseAttractorList for feature selection

## How to apply

Load a normalized, log-transformed expression matrix (e.g. the example dataset Data_PDAC_peng_2k shipped with the CASCC package in R). Call CASCC::findAttractor.adaptive on the matrix with a biologically meaningful seed gene — the tutorial uses the macrophage marker 'AIF1' — letting the package defaults handle the adaptive exponent search (CASCC.featureSelection exposes exponent.max = 10, exponent.min = 2 as the search bounds). The algorithm iteratively refines a co-expression signature until convergence; extract the converged attractor from attr.res$attractor.final, preserving the ranked gene order in which top-ranked genes are the most representative features. For a genome-wide view, scanSeeds.adaptive can sweep multiple seed markers (with parameters epsilon, exponent.max/min, seedTopN, compareNth, mc.cores) and collapseAttractorList can merge redundant attractors before clustering. The ranked attractor lists are what CASCC then uses for feature selection (CASCC.featureSelection) prior to predicting cluster number with predictClusterK.

## Related tools

- **CASCC** (Provides findAttractor.adaptive, CASCC.featureSelection, scanSeeds.adaptive, and collapseAttractorList for adaptive attractor co-expression scanning and downstream feature selection) — https://github.com/LingyiC/CASCC
- **cafr** (Dependency package implementing the attractor-finding machinery that CASCC's adaptive algorithm builds on) — https://github.com/weiyi-bitw/cafr
- **Seurat** (Required dependency (version 4.3.0) for running CASCC within a Seurat workflow and downstream visualization/DEG steps)

## Examples

```
attr.res <- findAttractor.adaptive(Data_PDAC_peng_2k, "AIF1")
```

## Evaluation signals

- The output attractor (attr.res$attractor.final) is a non-empty ranked gene list with stable ordering across the converged iteration
- Leading entries of the ranked list are genes co-expressed with the seed; for the AIF1 seed on Data_PDAC_peng_2k they should be consistent with macrophage markers as reported in the tutorial output
- The exported CSV (e.g. aif1_attractor_signature.csv) exists and contains gene symbols plus rank order
- In the tutorial PDAC example, the set of attractors resolves into eight distinct clusters, each corresponding to a specific attractor
- checkDependencies() passes with the pinned dependency versions (Seurat 4.3.0, cafr from GitHub)

## Limitations

- The adaptive exponent search bounds (exponent.min = 2, exponent.max = 10) must bracket an informative exponent; poor normalization of the input matrix undermines the co-expression estimate
- Attractor results depend on the choice of seed gene; redundant or dominant attractors may require collapseAttractorList / attractorModify with overlapN and removeDominant parameters
- The method was demonstrated on a well-normalized ~2k-gene PDAC example (Data_PDAC_peng_2k, 400 cells); behavior on very sparse or differently scaled inputs is not covered by the tutorial

## Evidence

- [readme] CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm.: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [methods] Adaptive attractor call with seed gene: "attr.res <- findAttractor.adaptive(Data_PDAC_peng_2k, "AIF1")"
- [methods] Feature selection exponent bounds: "fs.res <- CASCC.featureSelection(Data_PDAC_peng_2k, inputDataType = "well-normalized", attr.raw = NULL, exponent.max = 10, exponent.min = 2"
- [methods] Seed scanning function signature: "scanSeeds.adaptive(data, markers, epsilon, verbose, exponent.max, exponent.min, seedTopN, compareNth, mc.cores)"
- [methods] Clusters correspond to specific attractors: "The analysis identifies the presence of eight distinct clusters, each corresponding to a specific attractor"
