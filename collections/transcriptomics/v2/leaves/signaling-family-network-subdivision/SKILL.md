---
name: signaling-family-network-subdivision
description: Use when a filtered ligand-receptor connectome (e.g. from CreateConnectome
  + FilterConnectome) must be broken down by signaling family to identify which cell
  types dominate production or reception of specific modes such as PDGF, EGF, CSF/chemokine,
  or VEGF/ANGPT signaling.
when_to_use_negative:
- The question concerns differential signaling between conditions/time points rather
  than within-tissue topology — use DifferentialConnectome/DifferentialScoringPlot
  instead of per-family centrality
- 'Spatial context matters: suspension-based scRNA-seq connectomes do not preserve
  spatial histologic information, so centrality ranks cannot verify spatial proximity
  of candidate interactions'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Connectome
  role: Provides Centrality, which performs graph-theory-based centrality analysis
    of the input network grouped by signaling family, using weight_sc edgeweights
    and Kleinberg hub/authority scores
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Supplies the normalized, identity-annotated Seurat object (NormalizeData,
    ScaleData, active identity slot) that CreateConnectome consumes to define network
    nodes
- name: SeuratData
  role: Distributes the panc8 cross-platform pancreas dataset used as the demonstration
    input for per-family centrality analysis
- name: R
  role: Language environment in which the Connectome package and the centrality workflow
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:94b1e253557f4f195af3c99db752c4251e133037d26f92087871622cf04e115e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# signaling-family-network-subdivision

## Summary

Subdivides a filtered cell-cell connectome into per-signaling-family (per-mode) sub-networks and ranks cell types by sending vs. receiving centrality, using Connectome's Centrality function with Kleinberg hub/authority scores on cumulative edgeweights.

## When to use

Use when a filtered ligand-receptor connectome (e.g. from CreateConnectome + FilterConnectome) must be broken down by signaling family to identify which cell types dominate production or reception of specific modes such as PDGF, EGF, CSF/chemokine, or VEGF/ANGPT signaling.

## When NOT to use

- The question concerns differential signaling between conditions/time points rather than within-tissue topology — use DifferentialConnectome/DifferentialScoringPlot instead of per-family centrality
- Spatial context matters: suspension-based scRNA-seq connectomes do not preserve spatial histologic information, so centrality ranks cannot verify spatial proximity of candidate interactions

## Inputs

- Seurat 3.0 single-cell RNA-seq object with cell identities set (e.g. panc8 from SeuratData with 'celltype' identity, normalized and scaled on ligand/receptor genes from ncomms8866_human)
- Filtered connectome edgelist produced by FilterConnectome (min.pct=0.1, min.z=0.25, remove.na=TRUE) with edge attributes including 'mode' (signaling family) and 'weight_sc'

## Outputs

- Per-signaling-family centrality table ranking cell types by cumulative incoming/outgoing edgeweight and Kleinberg hub/authority scores (e.g. centrality_table.csv)
- Paired centrality plots per signaling family identifying dominant senders and receivers among cell types

## How to apply

After building the connectome from a Seurat object (NormalizeData, subset to ligand/receptor genes from ncomms8866_human, ScaleData, CreateConnectome with min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE) and filtering high-confidence edges (FilterConnectome with min.pct=0.1, min.z=0.25, remove.na=TRUE), call the Centrality function grouped by signaling family ('mode') with weight.attribute='weight_sc'. This subdivides the network into per-family sub-networks so each ligand-receptor family is analyzed as its own directed, weighted graph. For each sub-network, Centrality ranks each cell type by cumulative incoming and outgoing edgeweight and by Kleinberg hub and authority scores, producing paired centrality plots that identify cell types dominating signal production vs. reception. Assemble the per-family, per-cell-type ranks into a table (e.g. centrality_table.csv) and interpret top receivers and senders family-by-family, as in the pancreas example where mesenchymal cells top-receive PDGF-family, epithelial cells top-receive EGF-family, immune cells top-receive CSF/chemokine-family, and endothelial cells top-receive VEGF/ANGPT-family signals.

## Related tools

- **Connectome** (Provides Centrality, which performs graph-theory-based centrality analysis of the input network grouped by signaling family, using weight_sc edgeweights and Kleinberg hub/authority scores) — https://github.com/msraredon/Connectome
- **Seurat** (Supplies the normalized, identity-annotated Seurat object (NormalizeData, ScaleData, active identity slot) that CreateConnectome consumes to define network nodes)
- **SeuratData** (Distributes the panc8 cross-platform pancreas dataset used as the demonstration input for per-family centrality analysis)
- **R** (Language environment in which the Connectome package and the centrality workflow are executed)

## Examples

```
Centrality(panc8.con2, mode = 'VEGF', weight.attribute = 'weight_sc')
```

## Evaluation signals

- The centrality table exists and is a parseable CSV with one rank entry per (signaling family, cell type) pair
- Per-family rankings qualitatively match the reported pancreas topology, e.g. endothelial cells emerge as dominant VEGF/ANGPT-family signal receivers
- Each cell type is ranked separately for sending (outgoing edgeweight/hub score) and receiving (incoming edgeweight/authority score) positions per family
- Only edges passing the FilterConnectome thresholds (min.pct=0.1, min.z=0.25) contribute to the centrality computation
- Cell types/clusters with fewer than 75 cells (min.cells.per.ident=75) are absent from the centrality output

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; centrality ranks are expression-based inference
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- Output quality is bounded by the ground-truth ligand–receptor database against which mapping is performed (FANTOM5/ncomms8866 by default)
- Any robust claim of signal transduction requires extensive wet-lab experimentation not supplied by the computational workflow

## Evidence

- [methods] Centrality grouped by signaling family subdivides the network: "Built into `Connectome` is a function called `Centrality`, which performs a graph-theory based analysis of the input network."
- [readme] Centrality identifies senders and receivers per signaling mode: [evidence span withheld — anchor status: external_doc]
- [methods] Thresholds for the high-confidence edge filter feeding centrality: "panc8.con2 <- FilterConnectome(panc8.con,min.pct = 0.1,min.z = 0.25,remove.na = T)"
- [methods] CreateConnectome parameters defining network nodes: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [discussion] Limitation: paired expression is not proof of communication: "strongly paired ligand and receptor expression is not direct evidence of cell–cell communication"
