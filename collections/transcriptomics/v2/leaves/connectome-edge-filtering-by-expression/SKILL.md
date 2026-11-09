---
name: connectome-edge-filtering-by-expression
description: Use when a CreateConnectome edgelist from single-cell RNA-seq data contains
  low-confidence ligand-receptor edges (low percent-expressed or weakly scaled expression)
  that must be pruned before centrality, network plotting, or circos visualization.
when_to_use_negative:
- The object is still a raw Seurat object that has not been normalized, scaled on
  ligand/receptor genes, and converted by CreateConnectome — filtering requires the
  edgelist with pct.expressing and z-score attributes
- The goal is differential comparison between two conditions — use DifferentialConnectome
  on matched connectomes instead of expression-based edge filtering
- Cell identities (clusters) contain fewer than ~75 cells, so edge statistics are
  unstable and should be excluded via min.cells.per.ident upstream rather than filtered
  post hoc
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: Connectome
  role: Provides FilterConnectome and CreateConnectome, which generate and prune the
    ligand-receptor edgelist by percent-expressing and z-score thresholds
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Supplies the normalized and scaled Seurat 3.0 object (NormalizeData, ScaleData
    on ligand/receptor genes) from which the connectome edgelist is built
- name: SeuratData
  role: Distributes the panc8 cross-platform pancreas and ifnb PBMC datasets used
    to demonstrate the filtering workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:3ef5adccd1626c6f170ebc73a86b7f310031b672d0c1bf0b1b7a028e5434cef0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# connectome-edge-filtering-by-expression

## Summary

Reduces the large ligand-receptor edgelist produced by CreateConnectome to biologically credible edges by filtering on the fraction of cells expressing ligand and receptor (min.pct), on expression z-scores (min.z), and on minimum cells per cell identity, using the Connectome FilterConnectome function in R.

## When to use

Use when a CreateConnectome edgelist from single-cell RNA-seq data contains low-confidence ligand-receptor edges (low percent-expressed or weakly scaled expression) that must be pruned before centrality, network plotting, or circos visualization.

## When NOT to use

- The object is still a raw Seurat object that has not been normalized, scaled on ligand/receptor genes, and converted by CreateConnectome — filtering requires the edgelist with pct.expressing and z-score attributes
- The goal is differential comparison between two conditions — use DifferentialConnectome on matched connectomes instead of expression-based edge filtering
- Cell identities (clusters) contain fewer than ~75 cells, so edge statistics are unstable and should be excluded via min.cells.per.ident upstream rather than filtered post hoc

## Inputs

- A Connectome edgelist object produced by CreateConnectome from a normalized Seurat 3.0 object with active identities defining the network nodes
- Filtering parameters: min.pct (fraction of cells expressing), min.z (ligand and receptor expression z-score cutoff), remove.na (drop edges with NA statistics)
- min.cells.per.ident threshold applied upstream in CreateConnectome (e.g. 75 cells per identity)

## Outputs

- A reduced connectome edgelist (e.g. panc8.con2) retaining only high-confidence edges with min.pct >= 0.1 and ligand/receptor z-scores > 0.25, suitable for Centrality, NetworkPlot, CircosPlot, and EdgeDotPlot

## How to apply

Run CreateConnectome on the normalized Seurat object (e.g. the panc8 pancreas dataset with species='human', min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE) so that cell identities with fewer than 75 captured cells are already excluded from the node set. Then pass the connectome to FilterConnectome with min.pct=0.1 to require the signaling pair to be detectable in at least 10% of the relevant cell population, min.z=0.25 to retain only edges where both ligand and receptor scaled expression z-scores exceed 0.25, and remove.na=TRUE to drop edges with missing statistics. The rationale is that the raw edgelist is generally very large and includes many low-biologic-relevance connections, and the article states that edges should be kept only when the ligand and receptor z-scores are both above 0.25. FilterConnectome is a streamlined wrapper around the base R subset() function, so identical filtering can be achieved manually, and the same filtration arguments can also be passed directly to downstream visualization functions (NetworkPlot, Centrality, CircosPlot, EdgeDotPlot).

## Related tools

- **Connectome** (Provides FilterConnectome and CreateConnectome, which generate and prune the ligand-receptor edgelist by percent-expressing and z-score thresholds) — https://github.com/msraredon/Connectome
- **Seurat** (Supplies the normalized and scaled Seurat 3.0 object (NormalizeData, ScaleData on ligand/receptor genes) from which the connectome edgelist is built)
- **SeuratData** (Distributes the panc8 cross-platform pancreas and ifnb PBMC datasets used to demonstrate the filtering workflow)

## Examples

```
panc8.con2 <- FilterConnectome(panc8.con, min.pct = 0.1, min.z = 0.25, remove.na = TRUE)
```

## Evaluation signals

- The filtered connectome contains no edges where ligand or receptor scaled expression z-score is <= 0.25
- The filtered connectome contains no edges with missing (NA) statistics when remove.na=TRUE
- All retained edges have min.pct >= 0.1 (signaling detectable in at least 10% of cells)
- Downstream centrality results on the filtered network qualitatively match expected topology (e.g. endothelial cells as dominant VEGF/ANGPT-family receivers in pancreas)
- The same filtering result is reproducible via base R subset() on the unfiltered edgelist, since FilterConnectome is a streamlined wrapper

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- Output quality is bounded by the ground-truth ligand-receptor database (e.g. FANTOM5/ncomms8866) against which the original mapping takes place

## Evidence

- [methods] FilterConnectome retains high-confidence edges: "panc8.con2 <- FilterConnectome(panc8.con,min.pct = 0.1,min.z = 0.25,remove.na = T)"
- [methods] z-score threshold rationale: "filter the data to only include edges with a ligand and receptor z-score above 0.25"
- [methods] minimum cells per identity filter: "We also limit our connectomic analysis to only those clusters with > 75 cells captured"
- [readme] FilterConnectome is a subset wrapper: [evidence span withheld — anchor status: external_doc]
- [methods] CreateConnectome parameters and identity-based nodes: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
