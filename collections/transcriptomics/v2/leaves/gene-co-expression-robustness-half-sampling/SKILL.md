---
name: gene-co-expression-robustness-half-sampling
description: Use when comparing gene network inference methods (e.g., scLink, PIDC,
  GENIE3) for edge-stability across cell subsamples and for wall-clock runtime/memory
  on matched scRNA-seq inputs.
when_to_use_negative:
- Input is not a raw single-cell read count matrix — sclink_norm expects counts and
  normalizes to a 10^6 library size, so already-normalized or bulk expression tables
  are inappropriate.
- The goal is to assign edge directionality or distinguish regulatory from co-regulatory
  edges — scLink edges are undirected statistical dependencies and this skill only
  measures stability, not mechanism.
- Gene lists are very large and unstudied — the article recommends first investigating
  correlation structures with sclink_cor (or dividing genes into modules) rather than
  directly benchmarking a full network.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scLink
  role: 'Primary method under test: sclink_norm preprocesses counts and sclink_net
    infers the sparse co-expression network whose half-sample stability is scored'
  repo: https://github.com/Vivianstats/scLink
- name: PIDC
  role: Comparison method using partial information decomposition to quantify statistical
    dependencies between genes on the same inputs
- name: GENIE3
  role: Comparison method that decomposes network prediction between p genes into
    p regression problems using tree-based ensemble methods
- name: R
  role: Implementation environment for the scLink package and the benchmarking scripts
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:ca3acc4631f83d7dea3ef65c9bbadb20ae346977e107ab44f2ba535ca0f3cb6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - GENIE3
  - PIDC
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-co-expression-robustness-half-sampling

## Summary

Benchmark robustness and computational cost of single-cell gene co-expression network inference methods by comparing edges inferred from repeated random half-samplings of cells, as done for scLink versus PIDC and GENIE3 on Tabula Muris Smart-seq2 and 10X Genomics data.

## When to use

Use when comparing gene network inference methods (e.g., scLink, PIDC, GENIE3) for edge-stability across cell subsamples and for wall-clock runtime/memory on matched scRNA-seq inputs.

## When NOT to use

- Input is not a raw single-cell read count matrix — sclink_norm expects counts and normalizes to a 10^6 library size, so already-normalized or bulk expression tables are inappropriate.
- The goal is to assign edge directionality or distinguish regulatory from co-regulatory edges — scLink edges are undirected statistical dependencies and this skill only measures stability, not mechanism.
- Gene lists are very large and unstudied — the article recommends first investigating correlation structures with sclink_cor (or dividing genes into modules) rather than directly benchmarking a full network.

## Inputs

- Smart-seq2 Tabula Muris count matrices for late pro-B, bladder urothelial, myeloid, and microglial cell types (306-4394 cells)
- 10X Genomics count matrix of 10,085 B cells
- Gene sets of 100/200/500 genes (Tabula Muris) and 500/1000 genes (10X) after sclink_norm preprocessing
- Installed R packages: scLink, PIDC, GENIE3

## Outputs

- Robustness scores RS in [0, 1] per (method, dataset, cell type, gene-set size), averaged over 10 half-samplings
- Averaged wall-clock runtime and peak memory per method and condition
- efficiency_robustness_summary.csv with one row per (method, dataset, cell type, gene-set size)

## How to apply

Preprocess each count matrix with scLink's sclink_norm, normalizing to a library size of 10^6 and restricting to gene sets of 100/200/500 genes (Tabula Muris Smart-seq2 cell types) and 500/1000 genes (10X Genomics). For each cell type and gene-set size, run scLink via sclink_net with candidate regularization lda = seq(0.5, 0.1, -0.05), and run PIDC and GENIE3 on identical inputs. Repeat each network inference on 10 random half-samplings of the cells and compute a robustness score RS as the agreement of inferred edges between half-samples. Record averaged wall-clock runtime and peak memory per method, gene-set size, and cell type, including the 1000-gene x 8000-cell 10X run. Aggregate metrics into a single table (one row per method, dataset, cell type, gene-set size) for comparison.

## Related tools

- **scLink** (Primary method under test: sclink_norm preprocesses counts and sclink_net infers the sparse co-expression network whose half-sample stability is scored) — https://github.com/Vivianstats/scLink
- **PIDC** (Comparison method using partial information decomposition to quantify statistical dependencies between genes on the same inputs)
- **GENIE3** (Comparison method that decomposes network prediction between p genes into p regression problems using tree-based ensemble methods)
- **R** (Implementation environment for the scLink package and the benchmarking scripts) — https://github.com/Vivianstats/scLink

## Examples

```
norm <- sclink_norm(counts, genes = gene_set); net <- sclink_net(norm, lda = seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- efficiency_robustness_summary.csv exists with rows for all three methods (scLink, PIDC, GENIE3) and all tested conditions (cell types x gene-set sizes).
- All RS values fall within [0, 1].
- scLink's 1000-gene x 8000-cell 10X Genomics run completes in fewer than 100 seconds.
- scLink shows lower runtime and peak memory than PIDC and GENIE3 across conditions.
- Each network inference is repeated on 10 independent random half-samplings and RS is averaged over replicates.

## Limitations

- No software version is specified for the scLink R package or the comparison tools, so exact reproducibility of runtimes may vary.
- Imputation of gene expression (e.g., via scImpute) can be time-consuming depending on cell number, and imputed data may still be much sparser than bulk data.
- For large gene lists, scLink's penalized inference may need genes first divided into modules based on sclink_cor rather than run on all genes at once.

## Evidence

- [results] scLink achieved higher robustness than PIDC and GENIE3 while requiring much less computation time and memory usage: [evidence span withheld — anchor status: not_found]
- [methods] normalize the read count matrix with a library size of 10^6: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] use the function `sclink_net` to calculate the robust correlation matrix: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [readme] scLink R package is available at github: [evidence span withheld — anchor status: not_found]
- [intro] scLink adapts the Gaussian graphical model to distinguish direct associations: "scLink adapts the Gaussian graphical model [29] to distinguish direct associations between genes from indirect ones and leads to sparse networks"
