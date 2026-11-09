---
name: dot-plot-and-pie-chart-visualization
description: Use when int-pair/cluster-pair couplets from a multi-condition CCI comparison
  (e.g., critical vs non-critical COVID-19 CellChat output subset to a ligand subfamily
  in InterCellar's gene-verse) must be visualized as per-cluster dot plots and per-condition
  proportion pie charts.
when_to_use_negative:
- When interactions have not yet been inferred from scRNA-seq data — InterCellar builds
  upon pre-computed ligand-receptor interactions, not raw count matrices
- When a ligand subfamily other than the one under study is needed but cannot be isolated
  — the gene-verse filtering must first subset the data interactively
- When a remote web-based tool is preferred despite patient-data privacy concerns
  — InterCellar deliberately runs locally, so static remote platforms are not a substitute
  here
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: InterCellar
  role: R/Shiny app in which the interactive filtering, multi-condition comparison,
    and dot-plot/pie-chart generation of condition-unique couplets are performed
  repo: https://github.com/martaint/InterCellar
- name: CellChat
  role: Upstream tool whose inferred ligand-receptor interactions serve as the CCI
    data input rendered as dot plots and pie charts
  repo: https://github.com/sqjin/CellChat
- name: R/Shiny
  role: Implementation framework providing the interactive visualization interface
    and reproducible=TRUE session flag
  repo: https://github.com/martaint/InterCellar
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:1e7e55898b6b9368224d08bd85def4f107d2b9efcf3aefafa300a6afb938bee9
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Dot-plot and pie-chart visualization

## Summary

Visualization skill within InterCellar's interactive R/Shiny workflow for rendering condition-specific cell-cell communication results: dot plots display phenotype-unique ligand-receptor couplets per cell cluster (e.g., CXC/CC-chemokine couplets in epithelial, macrophage, NK and T cell clusters across COVID-19 phenotypes), while pie charts summarize the proportion of unique couplets favoring each condition.

## When to use

Use when int-pair/cluster-pair couplets from a multi-condition CCI comparison (e.g., critical vs non-critical COVID-19 CellChat output subset to a ligand subfamily in InterCellar's gene-verse) must be visualized as per-cluster dot plots and per-condition proportion pie charts.

## When NOT to use

- When interactions have not yet been inferred from scRNA-seq data — InterCellar builds upon pre-computed ligand-receptor interactions, not raw count matrices
- When a ligand subfamily other than the one under study is needed but cannot be isolated — the gene-verse filtering must first subset the data interactively
- When a remote web-based tool is preferred despite patient-data privacy concerns — InterCellar deliberately runs locally, so static remote platforms are not a substitute here

## Inputs

- Pre-computed, inferred ligand-receptor interactions uploaded as CCI data (e.g., CellChat output for critical and non-critical COVID-19 patients)
- Filtered interaction subsets restricted to a ligand subfamily (e.g., CXC- and CC-chemokines) via the gene-verse
- Phenotype-unique int-pair/cluster-pair couplets per cell cluster from InterCellar's multi-condition comparison

## Outputs

- Dot plots of phenotype-unique chemokine couplets per cell cluster (epithelial, macrophage, NK, T cell)
- Pie charts summarizing the proportion of unique couplets favoring each condition (e.g., critical vs control vs moderate)
- Downloadable tables and figures supporting reproducibility

## How to apply

Upload the pre-computed CCI datasets (e.g., CellChat-derived results for critical and non-critical COVID-19 patients) into InterCellar, then in the gene-verse use the interactive filtering options to subset interactions to the ligand subfamily of interest such as CXC- and CC-chemokines. Run the multi-condition comparison functionality to compute int-pair and cluster-pair couplets uniquely found in each condition within the relevant cell clusters. Generate dot plots of the phenotype-unique chemokine couplets per cell cluster and pie charts summarizing the proportion of unique couplets favoring each condition. Validate the reproduced proportions against reported values (e.g., 47% epithelial and 57% immune CXCL couplets unique to critical; 56% macrophage, 41% NK, 20% T cell favoring critical) and use InterCellar's download options to export tables and figures for reproducibility. The analysis is fully interactive in R/Shiny, requiring no programming skills, and the reproducible=TRUE flag ensures results are reproducible across R sessions.

## Related tools

- **InterCellar** (R/Shiny app in which the interactive filtering, multi-condition comparison, and dot-plot/pie-chart generation of condition-unique couplets are performed) — https://github.com/martaint/InterCellar
- **CellChat** (Upstream tool whose inferred ligand-receptor interactions serve as the CCI data input rendered as dot plots and pie charts) — https://github.com/sqjin/CellChat
- **R/Shiny** (Implementation framework providing the interactive visualization interface and reproducible=TRUE session flag) — https://github.com/martaint/InterCellar

## Examples

```
library(InterCellar); InterCellar::run_app( reproducible = TRUE )
```

## Evaluation signals

- Reproduced CXCL couplet percentages match the reported values: 47% unique to critical in epithelial cells and 57% in immune cells
- Macrophage, NK and T cell proportions of unique interactions favoring the critical phenotype reproduce 56%, 41% and 20% respectively
- Confirmed ligand assignments: CCL4L2 usage in MoMa, CCL7 usage in nrMa, and absence of rMa CCL3 in COVID-19
- Dot plots contain only couplets flagged as uniquely found in each condition by the multi-condition comparison (no shared couplets)
- Pie chart proportions within each cell cluster sum to 100% across conditions, and all figures/tables are downloadable for result collection

## Limitations

- InterCellar supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); CCI data from other tools must be converted or cannot be used
- Predicted interactions and their visualized couplets await experimental validation, e.g., by smFISH or flow cytometry co-occurrence
- Prediction methods rely on different reference databases and statistical assumptions, so the plotted couplets depend on the upstream tool chosen

## Evidence

- [abstract] InterCellar guides the biological interpretation through customized analysis steps, multiple visualization options, and the possibility to link biological pathways to ligand-receptor interactions.: "InterCellar guides the biological interpretation through customized analysis steps, multiple visualization options, and the possibility to link"
- [intro] In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data: "In each universe (cluster-verse, gene-verse, and function-verse), different filtering and visualization options are provided, giving the opportunity to interactively subset the data"
- [intro] InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures.: "InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures."
- [readme] InterCellar allows researchers to interactively analyze the results of cell-cell communication from scRNA-seq data.: [evidence span withheld — anchor status: external_doc]
