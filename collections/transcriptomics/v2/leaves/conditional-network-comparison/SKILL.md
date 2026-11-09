---
name: conditional-network-comparison
description: Use when comparing ligand–receptor signaling networks between two experimental
  conditions (e.g., IFN-stimulated vs. control PBMCs) and you need to identify and
  rank strongly perturbed edges, including concordant and discordant ligand/receptor
  changes.
when_to_use_negative:
- Only a single tissue system/condition is available with no matched comparison connectome
  — DifferentialConnectome requires exactly the same edges present in both networks,
  so a single-condition analysis should use CreateConnectome plus FilterConnectome,
  NetworkPlot, or CircosPlot instead
- The question is direct evidence of communication or intracellular transduction —
  paired ligand and receptor expression is not direct evidence of cell–cell communication,
  and spatial proximity cannot be verified from suspension-based single-cell sequencing
- The input data lack a reliable cell-type annotation or a condition subset with fewer
  than ~75 cells per identity, undermining node definition and edge statistics (article
  uses min.cells.per.ident = 75)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: Connectome
  role: 'Core R package: CreateConnectome builds per-condition connectomes, DifferentialConnectome
    computes the differential connectome, and DifferentialScoringPlot/CircosDiff compute
    the always-positive perturbation score'
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Provides SplitObject to split by condition, NormalizeData/ScaleData preprocessing,
    and FindMarkers for per-cell-type differential expression over mapped ligand–receptor
    genes
- name: SeuratData
  role: Distributes the ifnb interferon-stimulated vs. control PBMC example dataset
    used to exercise the differential workflow
- name: R
  role: Implementation language in which the whole differential connectomics workflow
    is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:9aaedfec74a2a5e96f4d62e97dde007d809bcd585a2eb77c04687542c122c4c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# conditional-network-comparison

## Summary

A differential-connectomics skill for comparing cell–cell signaling networks between two conditions using the Connectome R package: it computes condition-specific connectomes from Seurat objects, derives a differential connectome, and scores each edge with an always-positive 'perturbation score' (product of absolute ligand and receptor log-fold changes) to rank strongly perturbed ligand–receptor edges regardless of direction of change.

## When to use

Use when comparing ligand–receptor signaling networks between two experimental conditions (e.g., IFN-stimulated vs. control PBMCs) and you need to identify and rank strongly perturbed edges, including concordant and discordant ligand/receptor changes.

## When NOT to use

- Only a single tissue system/condition is available with no matched comparison connectome — DifferentialConnectome requires exactly the same edges present in both networks, so a single-condition analysis should use CreateConnectome plus FilterConnectome, NetworkPlot, or CircosPlot instead
- The question is direct evidence of communication or intracellular transduction — paired ligand and receptor expression is not direct evidence of cell–cell communication, and spatial proximity cannot be verified from suspension-based single-cell sequencing
- The input data lack a reliable cell-type annotation or a condition subset with fewer than ~75 cells per identity, undermining node definition and edge statistics (article uses min.cells.per.ident = 75)

## Inputs

- Seurat object (v3.0) containing single-cell RNA-seq data from two conditions with a condition metadata column (e.g., 'stim') and cell type annotations (e.g., seurat_annotations)
- Ligand–receptor reference database (FANTOM5 / Ramilowski et al. 2015 ncomms8866 by default, or a user-provided paired ligand–receptor list)

## Outputs

- Differential connectome edge list (DifferentialConnectome output comparing the two condition-specific connectomes)
- Scored and ranked differential edge table (e.g., diff_connectome_scored.csv) with ligand, receptor, source, target, log fold change, and perturbation-score fields

## How to apply

Load a two-condition single-cell dataset (e.g., the 'ifnb' interferon-stimulated vs. control PBMC data from SeuratData) and set cell identities to seurat_annotations. SplitObject by the 'stim' condition, normalize and scale each subset, and run CreateConnectome(species='human', p.values=F) on each to obtain two connectomic networks over the same edges, then compute the differential connectome with DifferentialConnectome(connect.ctrl, connect.stim). For each cell type, run Seurat FindMarkers between stimulated and control identities over the mapped ligand–receptor gene set (FANTOM5 by default) and retain genes with adjusted p-value < 0.05. Filter the differential connectome to edges whose source.ligand and target.receptor cell-gene pairs are both significantly perturbed, then score each edge using DifferentialScoringPlot-style logic (min.score = 1, min.pct = 0.1, infinity.to.max = TRUE), where the perturbation score is the product of the ligand and receptor log fold changes. Rank and export the scored differential edge list; the always-positive score ensures edges are identified strongly perturbed regardless of whether the ligand and receptor changes are concordant or discordant in direction.

## Related tools

- **Connectome** (Core R package: CreateConnectome builds per-condition connectomes, DifferentialConnectome computes the differential connectome, and DifferentialScoringPlot/CircosDiff compute the always-positive perturbation score) — https://github.com/msraredon/Connectome
- **Seurat** (Provides SplitObject to split by condition, NormalizeData/ScaleData preprocessing, and FindMarkers for per-cell-type differential expression over mapped ligand–receptor genes)
- **SeuratData** (Distributes the ifnb interferon-stimulated vs. control PBMC example dataset used to exercise the differential workflow)
- **R** (Implementation language in which the whole differential connectomics workflow is executed)

## Examples

```
ifnb.list <- SplitObject(ifnb, split.by = 'stim'); diff.con <- DifferentialConnectome(CreateConnectome(ifnb.list$CTRL, species='human', p.values=F), CreateConnectome(ifnb.list$STIM, species='human', p.values=F))
```

## Evaluation signals

- Output CSV (diff_connectome_scored.csv) exists with ligand, receptor, source, target, log fold change, and perturbation-score fields all populated
- All retained edges pass the adjusted p < 0.05 significance criterion for both source.ligand and target.receptor cell-gene pairs
- Perturbation scores are positive/monotone with edge magnitude (score is a product of absolute log-fold changes; infinity.to.max = TRUE maps infinite LFCs to the max)
- DifferentialConnectome succeeds without edge-matching errors, confirming both condition connectomes contain exactly the same edge set
- Top-ranked edges correspond to strongly perturbed mechanisms in the stimulated condition (e.g., IFN-responsive ligand–receptor pairs), not solely to concordant up-up changes

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- Output quality is bounded by the ground-truth ligand–receptor database (FANTOM5) against which the original mapping is performed
- Robust claims of signal transduction require extensive wet-lab experimentation not supplied by the computational workflow
- No version or release number for Connectome is stated in the article (README indicates v1.0.1), so exact reproducibility may depend on commit state

## Evidence

- [abstract] It has built-in functionality to facilitate differential and comparative connectomics, in which signaling networks are compared between tissue systems.: "It has built-in functionality to facilitate differential and comparative connectomics, in which signaling networks are compared between tissue systems."
- [other] Connectome assigns each edge a 'perturbation score' defined as the product of the absolute log-fold changes of the ligand and receptor: [evidence span withheld — anchor status: not_found]
- [methods] we will use the interferon-stimulated vs. control PBMC data distributed by SeuratData: "we will use the interferon-stimulated vs. control PBMC data distributed by SeuratData"
- [methods] ifnb.list <- SplitObject(ifnb,split.by = 'stim'): "ifnb.list <- SplitObject(ifnb,split.by = 'stim')"
- [readme] DifferentialConnectome Allows direct quantitative comparison of two connectomes. Requires exactly the same edges to be present: [evidence span withheld — anchor status: external_doc]
- [discussion] strongly paired ligand and receptor expression is not direct evidence of cell–cell communication: "strongly paired ligand and receptor expression is not direct evidence of cell–cell communication"
