---
name: chemokine-couplet-uniqueness-analysis
description: Use when comparing cell-cell communication predictions across multiple
  patient conditions (e.
when_to_use_negative:
- You have raw scRNA-seq data and have not yet inferred ligand-receptor interactions
  — InterCellar's workflow builds upon pre-computed, inferred ligand-receptor interactions,
  so run CellChat (or another supported tool) first.
- The question is within a single condition rather than a comparison across multiple
  conditions — single-dataset exploration belongs to the cluster-verse/gene-verse/function-verse
  walkthrough, not the multi-condition comparison.
- The analysis requires programming-heavy custom statistics — InterCellar is designed
  for interactive analysis without programming skills, not scripted re-implementations.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellar
  role: Interactive R/Shiny platform used to upload CCI data, filter interactions
    in the gene-verse, run the multi-condition comparison, and generate dot plots
    and pie charts of condition-unique couplets
  repo: https://github.com/martaint/InterCellar
- name: CellChat
  role: Upstream tool used to infer the CCI datasets for critical and non-critical
    COVID-19 patients that are uploaded into InterCellar
  repo: https://github.com/sqjin/CellChat
- name: R/Shiny
  role: Implementation environment for InterCellar; results are reproducible across
    R sessions with reproducible = TRUE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:c65bb39a9970c7cdbc74fc2e7098fc202c190cdb63a26bf0aa9681e839cac98a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellChat
  - InterCellar
  - R
  - R/Shiny
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# chemokine-couplet-uniqueness-analysis

## Summary

Reproduce condition-specific chemokine couplet analysis (Fig. 5b, c of the InterCellar paper) by uploading CellChat-derived CCI data from critical vs. non-critical COVID-19 patients into InterCellar, filtering the gene-verse to CXC- and CC-chemokine ligand-receptor interactions, and computing int-pair/cluster-pair couplets unique to each phenotype in epithelial, macrophage, NK, and T cell clusters.

## When to use

Use when comparing cell-cell communication predictions across multiple patient conditions (e.g. COVID-19 control, moderate, critical) and you need to identify ligand-receptor couplets unique to each condition within specific cell clusters, restricted to a ligand subfamily such as CXCL/CCL chemokines.

## When NOT to use

- You have raw scRNA-seq data and have not yet inferred ligand-receptor interactions — InterCellar's workflow builds upon pre-computed, inferred ligand-receptor interactions, so run CellChat (or another supported tool) first.
- The question is within a single condition rather than a comparison across multiple conditions — single-dataset exploration belongs to the cluster-verse/gene-verse/function-verse walkthrough, not the multi-condition comparison.
- The analysis requires programming-heavy custom statistics — InterCellar is designed for interactive analysis without programming skills, not scripted re-implementations.

## Inputs

- Pre-computed CellChat-derived cell-cell interaction (CCI) datasets for critical and non-critical COVID-19 patients
- Cell cluster annotations (epithelial clusters, macrophages (MoMa), NK cells, T cells)
- Ligand-receptor interaction annotations including chemokine subfamily labels (CXC- and CC-chemokines, e.g. CXCL and CCL genes)

## Outputs

- Per-phenotype lists of unique CXCL and CCL int-pair and cluster-pair couplets per cell cluster
- Dot plots of phenotype-unique chemokine couplets per cell cluster (Fig. 5b-style)
- Pie charts of the proportion of unique couplets favoring each condition (Fig. 5c-style)
- Validated proportions (e.g. 47% epithelial and 57% immune CXCL couplets unique to critical; 56% macrophage, 41% NK, 20% T cell favoring critical)

## How to apply

Load the pre-computed CellChat-derived CCI datasets for critical and non-critical COVID-19 patients into InterCellar as CCI data (step 1 of the workflow). In the gene-verse, use the interactive filtering options to subset interactions to the CXC- and CC-chemokine subfamilies. Run InterCellar's multi-condition comparison functionality, which highlights patterns of communication uniquely found in each condition, to compute int-pair and cluster-pair couplets unique to each phenotype within epithelial and macrophage/NK/T cell clusters. Generate dot plots of phenotype-unique chemokine couplets per cell cluster and pie charts summarizing the proportion of unique couplets favoring each condition. Validate against the reported values: 47% epithelial and 57% immune CXCL couplets unique to critical; macrophages 56%, NK 41%, and T cells 20% favoring critical; confirm CCL4L2 usage in MoMa, CCL7 in nrMa, and absence of rMa CCL3 in COVID-19.

## Related tools

- **InterCellar** (Interactive R/Shiny platform used to upload CCI data, filter interactions in the gene-verse, run the multi-condition comparison, and generate dot plots and pie charts of condition-unique couplets) — https://github.com/martaint/InterCellar
- **CellChat** (Upstream tool used to infer the CCI datasets for critical and non-critical COVID-19 patients that are uploaded into InterCellar) — https://github.com/sqjin/CellChat
- **R/Shiny** (Implementation environment for InterCellar; results are reproducible across R sessions with reproducible = TRUE)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("InterCellar"); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- Reproduced CXCL couplet proportions match the reported values: 47% unique to critical in epithelial cells and 57% in immune cells.
- Macrophage, NK, and T cell unique-interaction proportions favoring the critical phenotype match 56%, 41%, and 20% respectively.
- Ligand usage checks pass: CCL4L2 is used in MoMa, CCL7 in nrMa, and rMa CCL3 is absent in COVID-19.
- Couplets are classified as uniquely found in each condition by InterCellar's multi-condition comparison (int-pair and cluster-pair levels agree with the per-cluster breakdown).
- Downloaded tables/figures from InterCellar's reproducibility options allow diff-comparison against the paper's supplementary results in martaint/InterCellar-reproducibility.

## Limitations

- InterCellar depends on the upstream prediction tool: CellChat (and other supported tools) rely on different reference databases and statistical assumptions, so couplet uniqueness results are method-dependent.
- InterCellar supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); unsupported CCI outputs cannot be loaded.
- Predicted chemokine interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence).

## Evidence

- [abstract] Multi-condition comparison step: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [intro] Universes and interactive subsetting: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [intro] Condition-specific communication patterns: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [discussion] COVID-19 immune discrimination: "an in-depth analysis of cellular interactions in COVID-19 patients confirmed the primary role of immune response in discriminating between different courses of the disease"
- [readme] Reproducible app launch: "The flag `reproducible = TRUE` ensures that your results will be reproducible across R sessions."
- [readme] Reproducibility repository: [evidence span withheld — anchor status: not_found]
