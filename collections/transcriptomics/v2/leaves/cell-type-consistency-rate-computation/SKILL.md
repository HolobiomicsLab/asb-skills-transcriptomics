---
name: cell-type-consistency-rate-computation
description: Use when evaluating how accurately an automatic cell-type annotation
  method labels clusters of scRNA-seq data against manually curated cell types, and
  a comparable consistent-rate table across multiple methods is needed.
when_to_use_negative:
- The goal is per-cell annotation accuracy rather than cluster-level consistency with
  manual labels — the scCATCH benchmark scores agreement at the cluster level
- The datasets or tissue types are not covered by the CellMatch reference database,
  since scCATCH's performance depends heavily on database coverage and a limited reference
  can cause incorrect annotation
- Cluster assignments are themselves unreliable or absent — inadequate clustering
  analysis introduces errors into cell type annotation per the article's discussion
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCATCH
  role: Cluster-based annotation method under evaluation; run via createscCATCH, findmarkergene,
    findcelltype against CellMatch
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: One-against-all marker-based cluster annotation baseline compared against
    scCATCH
- name: SingleR
  role: Cell-based annotation baseline run on the same cluster assignments
- name: CellAssign
  role: Cell-based annotation baseline run on the same cluster assignments
- name: Garnett
  role: Cell-based annotation baseline run on the same cluster assignments
- name: scMap
  role: Cell-based annotation baseline run on the same cluster assignments
- name: CHETAH
  role: Cell-based annotation baseline run on the same cluster assignments
- name: Matrix
  role: R package used to load the processed benchmark scRNA-seq datasets
- name: devtools
  role: Used to install scCATCH from GitHub (devtools::install_github)
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:6e39ced6818aba95d199d7a24c08524ba6f7729196dca9047648068d22b1321a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scCATCH
  repo_url: https://github.com/ZJUFanLab/scCATCH
  tools:
  - CHETAH
  - CellAssign
  - Garnett
  - Matrix
  - Seurat
  - SingleR
  - devtools
  - scCATCH
  - scMap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cell-type-consistency-rate-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark methodology from the scCATCH paper for quantifying automatic cell-type annotation accuracy as per-cluster and per-dataset consistent rates, comparing cluster-based scCATCH and Seurat against cell-based methods (CellAssign, SingleR, Garnett, scMap, CHETAH) on the Chen, Xin, and Gierahn scRNA-seq datasets.

## When to use

Use when evaluating how accurately an automatic cell-type annotation method labels clusters of scRNA-seq data against manually curated cell types, and a comparable consistent-rate table across multiple methods is needed.

## When NOT to use

- The goal is per-cell annotation accuracy rather than cluster-level consistency with manual labels — the scCATCH benchmark scores agreement at the cluster level
- The datasets or tissue types are not covered by the CellMatch reference database, since scCATCH's performance depends heavily on database coverage and a limited reference can cause incorrect annotation
- Cluster assignments are themselves unreliable or absent — inadequate clustering analysis introduces errors into cell type annotation per the article's discussion

## Inputs

- Processed scRNA-seq count/expression matrices for the Chen, Xin, and Gierahn benchmark datasets
- Published/manual cluster assignments (character vector of cluster labels per dataset)
- CellMatch tissue-specific reference database (shipped with scCATCH)
- Trained or configured baseline annotators: SingleR, CellAssign, Garnett, scMap, CHETAH, Seurat

## Outputs

- Per-cluster consistent rates for each method and each dataset
- Per-dataset consistent rates for each method and each dataset
- Aggregated comparison table (e.g., table1_consistency_rates.csv) matching Table 1 of the scCATCH article

## How to apply

Load the processed Chen, Xin, and Gierahn scRNA-seq benchmark matrices together with their published cluster assignments using the Matrix package in R. For scCATCH, build objects with createscCATCH(data, cluster), run findmarkergene with the paper's marker criteria (log10 fold change >= 0.25, Wilcoxon p < 0.05, expressed in >= 25% of cells) against the CellMatch reference database (353 cell types, 686 subtypes, 184 tissue types), then run findcelltype. Run the cell-based baselines (SingleR, CellAssign, Garnett, scMap, CHETAH) and Seurat's one-against-all marker-based annotation on the same cluster assignments. Define the consistent-rate for each method on each dataset as the fraction of clusters whose automatic annotation agrees with the manually assigned cell type. Assemble per-cluster and per-dataset consistent rates into a single table (e.g., table1_consistency_rates.csv) and compare against Table 1 of the article to confirm agreement.

## Related tools

- **scCATCH** (Cluster-based annotation method under evaluation; run via createscCATCH, findmarkergene, findcelltype against CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (One-against-all marker-based cluster annotation baseline compared against scCATCH)
- **SingleR** (Cell-based annotation baseline run on the same cluster assignments)
- **CellAssign** (Cell-based annotation baseline run on the same cluster assignments)
- **Garnett** (Cell-based annotation baseline run on the same cluster assignments)
- **scMap** (Cell-based annotation baseline run on the same cluster assignments)
- **CHETAH** (Cell-based annotation baseline run on the same cluster assignments)
- **Matrix** (R package used to load the processed benchmark scRNA-seq datasets)
- **devtools** (Used to install scCATCH from GitHub (devtools::install_github)) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Reproduced per-dataset consistent rates match the values reported in Table 1 of the scCATCH article
- Consistent-rate is computed as the fraction of clusters whose automatic annotation agrees with the manual cell type, per method per dataset
- Garnett and CHETAH show consistent rates as low as 0% on the Gierahn dataset, matching the article's reported failure mode
- scCATCH findmarkergene was run with the paper's thresholds: log10 fold change >= 0.25, Wilcoxon p < 0.05, expression in >= 25% of cells
- All methods (scCATCH, Seurat, CellAssign, SingleR, Garnett, scMap, CHETAH) are evaluated on identical cluster assignments for a fair comparison

## Limitations

- scCATCH performance depends majorly on the reference database; a limited CellMatch reference can lead to incorrect annotation of cell types
- Inadequate clustering analysis (too many or too few clusters) introduces errors into cell type annotation
- Cell-based methods can fail completely on some datasets (Garnett and CHETAH scored 0% on the Gierahn dataset)
- Effects of multiple clustering algorithms on cell type annotations were not evaluated

## Evidence

- [readme] Performance comparison repo: [evidence span withheld — anchor status: external_doc]
