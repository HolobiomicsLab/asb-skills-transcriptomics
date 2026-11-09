---
name: cross-method-benchmark-ranking-analysis
description: Use when benchmarking several single-cell integration methods on the
  same dataset collections and you need a reproducible global ranking from batch-mixing
  and bio-conservation metrics computed on each method's latent embedding.
when_to_use_negative:
- Cell type annotations are unavailable for the datasets, since CiLISI and cell type-aware
  metrics require cell type labels and cannot be computed for cell types present in
  only one dataset
- Datasets have highly unequal numbers of cells, where CiLISI (like LISI) is suboptimal
  and batch mixing metrics robust to lopsided cell proportions are still lacking
- Integrating technical replicates where cell type-agnostic batch mixing metrics would
  suffice, making a full semi-supervised benchmark unnecessary
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: STACAS
  role: Semi-supervised and unsupervised integration method under benchmark; anchor-based
    batch correction with optional cell type labels
  repo: https://github.com/carmonalab/STACAS
- name: carmonalab/scib-pipeline
  role: Snakemake pipeline that runs all 11 integration methods and computes benchmark
    metrics
  repo: https://github.com/carmonalab/scib-pipeline
- name: carmonalab/scIntegrationMetrics
  role: R package computing CiLISI and cell type ASW on the integrated latent spaces
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: Harmony
  role: Unsupervised integration baseline compared in the benchmark
- name: FastMNN
  role: Unsupervised integration baseline compared in the benchmark
- name: Seurat v4
  role: Unsupervised integration baseline compared in the benchmark
- name: scVI
  role: Unsupervised integration baseline compared in the benchmark
- name: Scanorama
  role: Unsupervised integration baseline compared in the benchmark
- name: scANVI
  role: Supervised integration baseline compared in the benchmark
- name: scGen
  role: Supervised integration baseline compared in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:38878f88ba5cdb9c839965358ea445fbfc520bc4f36df8c807111b16649caf99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - FastMNN
  - Harmony
  - STACAS
  - Scanorama
  - Seurat v4
  - carmonalab/scIntegrationMetrics
  - carmonalab/scib-pipeline
  - scANVI
  - scGen
  - scVI
  - scib-pipeline
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# cross-method-benchmark-ranking-analysis

## Summary

Ranks multiple scRNA-seq integration tools on cell type-aware batch mixing (CiLISI) and biological variance preservation (cell type ASW) across multiple dataset collections, using a 50-dimensional latent space per method and a global rank aggregation to identify the best-performing method (here, semi-supervised STACAS).

## When to use

Use when benchmarking several single-cell integration methods on the same dataset collections and you need a reproducible global ranking from batch-mixing and bio-conservation metrics computed on each method's latent embedding.

## When NOT to use

- Cell type annotations are unavailable for the datasets, since CiLISI and cell type-aware metrics require cell type labels and cannot be computed for cell types present in only one dataset
- Datasets have highly unequal numbers of cells, where CiLISI (like LISI) is suboptimal and batch mixing metrics robust to lopsided cell proportions are still lacking
- Integrating technical replicates where cell type-agnostic batch mixing metrics would suffice, making a full semi-supervised benchmark unnecessary

## Inputs

- Pancreas, Immune, Lung, and mouse T cell scRNA-seq dataset collections (figshare 10.6084/m9.figshare.12420968.v8 and 10.6084/m9.figshare.12478571)
- Cell type annotations per dataset, optionally perturbed to 15% 'unknown' labels and 20% shuffled labels for noisy supervised benchmarks
- 50-dimensional latent embeddings produced by each integration method

## Outputs

- Per-method CiLISI and celltype_ASW scores for each of the four integration tasks
- Aggregated results table with a global rank per method (e.g., integration_benchmark_ranking.csv)
- Ranked list of integration methods confirming whether semi-supervised STACAS ranks first

## How to apply

Download the benchmark dataset collections (Pancreas, Immune, Lung, mouse T cell scRNA-seq from the figshare DOIs 10.6084/m9.figshare.12420968.v8 and 10.6084/m9.figshare.12478571) and, when evaluating supervised/semi-supervised tools, perturb the cell type annotations to simulate noisy labels (15% set to 'unknown', 20% shuffled). Run all methods (semi-supervised and unsupervised STACAS, Harmony, FastMNN, Seurat v4, scVI, Scanorama, scANVI, scGen, etc.) via the carmonalab/scib-pipeline Snakemake workflow, extracting a 50-dimensional latent embedding for each. Compute CiLISI (cell-type-aware batch mixing) and celltype_ASW (biological variance preservation) on each latent space using the carmonalab/scIntegrationMetrics R package, per dataset collection. Aggregate per-task metric pairs into a results table, compute a global rank across the four integration tasks for each method, and verify the expected ordering (e.g., semi-supervised STACAS first, followed by unsupervised STACAS and scGen). Rationale: focusing on cell type-aware metrics matters because cell type-agnostic batch mixing is arguably only relevant for technical replicates, and benchmarks with supervised methods should account for noisy and partial cell-type information.

## Related tools

- **STACAS** (Semi-supervised and unsupervised integration method under benchmark; anchor-based batch correction with optional cell type labels) — https://github.com/carmonalab/STACAS
- **carmonalab/scib-pipeline** (Snakemake pipeline that runs all 11 integration methods and computes benchmark metrics) — https://github.com/carmonalab/scib-pipeline
- **carmonalab/scIntegrationMetrics** (R package computing CiLISI and cell type ASW on the integrated latent spaces) — https://github.com/carmonalab/scIntegrationMetrics
- **Harmony** (Unsupervised integration baseline compared in the benchmark)
- **FastMNN** (Unsupervised integration baseline compared in the benchmark)
- **Seurat v4** (Unsupervised integration baseline compared in the benchmark)
- **scVI** (Unsupervised integration baseline compared in the benchmark)
- **Scanorama** (Unsupervised integration baseline compared in the benchmark)
- **scANVI** (Supervised integration baseline compared in the benchmark)
- **scGen** (Supervised integration baseline compared in the benchmark)

## Examples

```
git clone https://github.com/carmonalab/scib-pipeline && snakemake --configfile config.yaml   # runs all integration methods and computes CiLISI / celltype_ASW per task
```

## Evaluation signals

- Each integration method yields exactly one 50-dimensional latent embedding per dataset collection
- CiLISI and celltype_ASW are computed for every method-task pair, giving 11 methods x 4 tasks metric pairs
- The aggregated ranking table shows semi-supervised STACAS (ssSTACAS) ranked first globally across the four tasks
- Unsupervised STACAS and scGen occupy the next-best positions in the global ranking, matching the published ordering
- Noisy-label perturbation applied consistently: 15% of labels set to 'unknown' and 20% shuffled for all supervised/semi-supervised methods

## Limitations

- CiLISI and similar cell type-aware metrics require cell type annotations and cannot be calculated for cell types that are only present in one dataset
- CiLISI remains suboptimal when datasets have highly unequal numbers of cells; batch mixing metrics robust to lopsided cell proportions are still lacking
- Benchmarks that include supervised integration methods should account for noisy and partial cell-type information, as done here via label perturbation

## Evidence

- [results] Globally, when considering cell type ASW and CiLISI across the four integration tasks, semi-supervised STACAS was the best-performing method, followed by unsupervised STACAS and scGen.: [evidence span withheld — anchor status: not_found]
- [abstract] semi-supervised STACAS outperforms state-of-the-art unsupervised methods: "Through an open-source benchmark, we show that semi-supervised STACAS outperforms state-of-the-art unsupervised methods, as well as supervised methods such as scANVI and scGen."
- [readme] It includes our new integration metric CiLISI we computed with our scIngrationMetrics R package.: [evidence span withheld — anchor status: external_doc]
- [discussion] One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations.: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [discussion] Benchmarks that include supervised integration methods should account for noisy and partial cell-type information: "We argue that benchmarks that include supervised integration methods should account for noisy and partial cell-type information"
