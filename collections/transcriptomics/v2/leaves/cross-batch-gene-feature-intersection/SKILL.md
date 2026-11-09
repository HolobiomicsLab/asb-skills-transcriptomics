---
name: cross-batch-gene-feature-intersection
description: Use when integrating scRNA-seq count tables from multiple platforms/batches
  whose gene sets differ, and a shared gene universe must be established before multi-table
  CA (corralm) or before comparing against PCA/Harmony baselines.
when_to_use_negative:
- Analyzing a single table — use single-table corral for dimensionality reduction
  instead of intersecting features for multi-table CA
- Batches assayed such divergent gene sets that the intersection is too small to be
  a meaningful feature space
- Inputs are already batch-corrected embeddings rather than raw count matrices — feature
  intersection at the count level is no longer applicable
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: corralm
  role: Consumes the gene-intersected multi-table data to compute integrated cell
    embeddings via multi-table correspondence analysis across the three SCMixology
    batches
  repo: github.com/laurenhsu1/corral
- name: corral
  role: R/Bioconductor package providing the corral/corralm calls that interface with
    SingleCellExperiment objects after feature intersection
  repo: github.com/laurenhsu1/corral
- name: Harmony
  role: Run on both corralm and PCA embeddings of the feature-intersected data to
    obtain batch-corrected representations for the SV comparison baseline
- name: Bioconductor (SingleCellExperiment)
  role: Holds the per-batch count tables whose rownames are intersected and subsetted
    into a multi-table structure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:d0b454e06f0bc42017724fe39d8ab60168cc4120cc3dcae195fb1e9444d405b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Bioconductor (SingleCellExperiment)
  - Harmony
  - R
  - corral
  - corralm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-batch-gene-feature-intersection

## Summary

Restricts multiple scRNA-seq batches (e.g., 10X, Celseq2, Dropseq in SCMixology) to their intersecting set of genes so all tables share a common feature space, a prerequisite for multi-table correspondence analysis with corralm.

## When to use

Use when integrating scRNA-seq count tables from multiple platforms/batches whose gene sets differ, and a shared gene universe must be established before multi-table CA (corralm) or before comparing against PCA/Harmony baselines.

## When NOT to use

- Analyzing a single table — use single-table corral for dimensionality reduction instead of intersecting features for multi-table CA
- Batches assayed such divergent gene sets that the intersection is too small to be a meaningful feature space
- Inputs are already batch-corrected embeddings rather than raw count matrices — feature intersection at the count level is no longer applicable

## Inputs

- Per-batch SingleCellExperiment scRNA-seq count matrices (SCMixology 10X, Celseq2, Dropseq) with gene identifiers as rownames
- Gene identifier sets for each batch used to compute the intersection

## Outputs

- Feature-intersected multi-table SingleCellExperiment structure with identical gene rownames across all batches
- Common gene list serving as the shared feature space for corralm and PCA baselines
- Downstream SV comparison table (method, SV score per batch/condition) written to sv_integration_results.csv

## How to apply

Load each batch (10X, Celseq2, Dropseq) as a SingleCellExperiment object with gene-level rownames and combine them into a multi-table structure. Compute the intersecting gene set across all batches (e.g., Reduce(intersect, ...) over rownames) and subset every batch to exactly that common feature space, preserving identical rownames and order across tables — this is the decision point: intersect before any embedding, because multi-table CA requires matched features. Run corralm on the intersected multi-table data to compute integrated cell embeddings across the three batches. As a baseline, compute PCA embeddings of the same feature-intersected data and run Harmony on both corralm and PCA embeddings, then quantify batch mixing with the scaled variance (SV) metric, where SV values close to 1 indicate successful cross-batch integration and values far from 1 indicate unsuccessful integration. Write the SV comparison (corralm, corralm+Harmony, PCA+Harmony) to sv_integration_results.csv.

## Related tools

- **corralm** (Consumes the gene-intersected multi-table data to compute integrated cell embeddings via multi-table correspondence analysis across the three SCMixology batches) — github.com/laurenhsu1/corral
- **corral** (R/Bioconductor package providing the corral/corralm calls that interface with SingleCellExperiment objects after feature intersection) — github.com/laurenhsu1/corral
- **Harmony** (Run on both corralm and PCA embeddings of the feature-intersected data to obtain batch-corrected representations for the SV comparison baseline)
- **Bioconductor (SingleCellExperiment)** (Holds the per-batch count tables whose rownames are intersected and subsetted into a multi-table structure)

## Examples

```
common <- Reduce(intersect, lapply(sce_batches, rownames)); sce_batches <- lapply(sce_batches, function(s) s[common,]); corralm(sce_batches)
```

## Evaluation signals

- After subsetting, all batches have identical rownames (same genes, same order) and equal numbers of features, i.e., a nonempty common feature space
- No batch retains genes absent from any other batch; each batch's rownames equals the intersection set exactly
- corralm runs without dimension mismatch errors on the intersected multi-table structure
- Scaled variance (SV) for corralm and corralm+Harmony is close to 1, while PCA+Harmony SV values are far from 1, matching the reported Fig. 4A-4B behavior on SCMixology
- sv_integration_results.csv contains one SV score per method (corralm, corralm+Harmony, PCA+Harmony) per batch/condition

## Limitations

- No accession IDs or links for the SCMixology and Zhengmix benchmarking datasets are provided in the article's discussion, so dataset provenance must be resolved separately
- Benchmarking relies on ground-truth labels from the original dataset publications that were not independently validated; SCMixology is a simple platform-mixture design rather than a complex tissue
- The article reports no tool or package versions for corral or its dependencies, so reproduction may require resolving package versions yourself

## Evidence

- [other] Restrict all batches to the intersecting set of genes so the batches share a common feature space for multi-table analysis.: [evidence span withheld — anchor status: not_found]
- [abstract] corralm introduced for multi-table dimension reduction: "we introduce corralm for integrative multi-table dimension reduction of scRNAseq data"
- [intro] corral interfaces with Bioconductor classes including SingleCellExperiment: "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
- [readme] README on single-table vs multi-table use of corral: "It can be used on a single table for dimensionality reduction, or it can be used for integration across batches, samples, and modes."
