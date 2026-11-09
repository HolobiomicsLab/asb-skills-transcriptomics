---
name: cell-subsampling-and-label-shuffling
description: Use when you need to test how sensitive CCC inference methods (CellChat,
  CellPhoneDB, Connectome, NATMI, SingleCellSignalR, iTALK, Cytotalk via LIANA) are
  to cell-level noise—reduced cell numbers or erroneous cell-type annotation—in a
  scRNA-seq dataset.
when_to_use_negative:
- Your goal is biological CCC prediction itself, not robustness assessment—this skill
  only perturbs inputs and re-scores, it does not yield novel interaction calls.
- You are evaluating methods that additionally infer intracellular processes (NicheNet,
  Cytotalk's full model, SoptSC), which the paper states are 'not directly comparable'
  and were excluded from the comparable framework.
- Your noise concern is in the prior-knowledge resource rather than the data—use the
  selective/non-selective interaction-replacement ablation instead, which perturbs
  the resource, not the cells.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LIANA
  role: Orchestrates all CCC methods and resources, providing the uniform interface
    for re-running inference on each perturbed dataset
  repo: https://github.com/saezlab/liana
- name: CellChat
  role: One of the CCC inference methods re-run on subsampled and label-shuffled datasets
- name: CellPhoneDB
  role: One of the CCC inference methods re-run on perturbed datasets
- name: Connectome
  role: One of the CCC inference methods re-run on perturbed datasets
- name: NATMI
  role: One of the CCC inference methods re-run on perturbed datasets
  repo: https://github.com/asrhou/NATMI.git
- name: SingleCellSignalR
  role: One of the CCC inference methods re-run on perturbed datasets (custom OmniPath-enabled
    build)
  repo: https://github.com/SCA-IRCM/SingleCellSignalR_v1
- name: iTALK
  role: One of the CCC inference methods re-run on perturbed datasets
- name: Cytotalk
  role: One of the CCC inference methods re-run on perturbed datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:d3ad18d4aba5a0983f7ef92a8005a1a3901abf81cf5a6742b8b1219868964111
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - CellChat
  - CellPhoneDB
  - Connectome
  - Cytotalk
  - LIANA
  - NATMI
  - SingleCellSignalR
  - iTALK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cell-subsampling-and-label-shuffling

## Summary

A robustness-to-noise ablation for cell-cell communication (CCC) inference: perturb single-cell RNA-seq inputs by subsampling cells (0-40%) and reshuffling cell-type labels (0-40%), then re-run CCC methods through LIANA and quantify how the top-250 interaction ranking changes relative to the unperturbed baseline.

## When to use

Use when you need to test how sensitive CCC inference methods (CellChat, CellPhoneDB, Connectome, NATMI, SingleCellSignalR, iTALK, Cytotalk via LIANA) are to cell-level noise—reduced cell numbers or erroneous cell-type annotation—in a scRNA-seq dataset.

## When NOT to use

- Your goal is biological CCC prediction itself, not robustness assessment—this skill only perturbs inputs and re-scores, it does not yield novel interaction calls.
- You are evaluating methods that additionally infer intracellular processes (NicheNet, Cytotalk's full model, SoptSC), which the paper states are 'not directly comparable' and were excluded from the comparable framework.
- Your noise concern is in the prior-knowledge resource rather than the data—use the selective/non-selective interaction-replacement ablation instead, which perturbs the resource, not the cells.

## Inputs

- Annotated single-cell RNA-seq dataset (expression matrix plus cell-type labels per cell)
- Ligand-receptor prior-knowledge resource (e.g. CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb)
- Unperturbed baseline ranking of ligand-receptor interactions per CCC method
- Perturbation grid: subsampling levels and label-reshuffling levels from 0 to 40%, with 5 randomisations per level

## Outputs

- Per-method overlap scores of top-250 ranked interactions (perturbed vs unperturbed) for each subsampling and label-reshuffling level
- Aggregated overlap scores across the 5 randomisations, written to a CSV file
- Line plot of overlap as a function of perturbation magnitude (reproducing Supplementary Fig S13), written to a PNG file

## How to apply

First run each CCC method orchestrated through LIANA on the unperturbed dataset to obtain a baseline ranking of ligand-receptor interactions. Then generate two families of cell-level perturbations: (1) subsample 0-40% of cells, and (2) reshuffle 0-40% of cell-type labels, each at multiple levels with 5 randomisations per level. Re-run every method on each perturbed dataset and rank the inferred interactions identically to the baseline. Compute the overlap of the top-250 ranked interactions of each perturbed run against the unperturbed ranking per method, aggregate overlap across the 5 randomisations, and plot overlap versus perturbation magnitude (reproducing Supplementary Fig S13). The rationale is to separate two noise sources the paper found to differ in impact: erroneous cell-type annotation perturbs results more than cell subsampling, so label reshuffling must be tested separately from subsampling.

## Related tools

- **LIANA** (Orchestrates all CCC methods and resources, providing the uniform interface for re-running inference on each perturbed dataset) — https://github.com/saezlab/liana
- **CellChat** (One of the CCC inference methods re-run on subsampled and label-shuffled datasets)
- **CellPhoneDB** (One of the CCC inference methods re-run on perturbed datasets)
- **Connectome** (One of the CCC inference methods re-run on perturbed datasets)
- **NATMI** (One of the CCC inference methods re-run on perturbed datasets) — https://github.com/asrhou/NATMI.git
- **SingleCellSignalR** (One of the CCC inference methods re-run on perturbed datasets (custom OmniPath-enabled build)) — https://github.com/SCA-IRCM/SingleCellSignalR_v1
- **iTALK** (One of the CCC inference methods re-run on perturbed datasets)
- **Cytotalk** (One of the CCC inference methods re-run on perturbed datasets)

## Evaluation signals

- Identity check: at 0% perturbation (no subsampling, no label shuffling) the top-250 overlap against the unperturbed ranking must be maximal for every method.
- Overlap of top-250 ranked interactions should decrease as the perturbation level rises from 0% to 40%; a monotone-decreasing trend indicates correct re-ranking.
- Per-level aggregation across 5 randomisations must be reproducible; missing randomisations change the aggregated overlap.
- Expected qualitative pattern per the paper's finding: label reshuffling (erroneous cell-type annotation) should degrade overlap more than cell subsampling at matched levels.
- Deliverables check: overlap scores present in a CSV file and the perturbation-magnitude plot (Supplementary Fig S13 analogue) saved as PNG.

## Limitations

- No gold standard exists for CCC, so overlap with the unperturbed ranking is a consistency metric, not a correctness metric—the paper explicitly notes it 'cannot give a definitive answer of what method is best'.
- This ablation covers only cell-level noise (subsampling, erroneous annotation); perturbing the resource with spurious interactions is a separate experiment with different dynamics (non-selective replacement had a strong effect on all methods).
- Findings are specific to the compared ligand-receptor methods orchestrated via LIANA; methods with intracellular inference (NicheNet, SoptSC, full Cytotalk) were not directly comparable.
- Results depend on the chosen resource; the paper shows both the method and the resource can considerably impact CCC inference predictions.

## Evidence

- [supplementary] Generate cell-perturbed datasets by subsampling 0-40% of cells and, separately, by reshuffling 0-40% of the cell-type labels, with 5 randomisations per perturbation level.: [evidence span withheld — anchor status: not_found]
- [supplementary] top-250 overlap scoring: [evidence span withheld — anchor status: not_found]
- [readme] LIANA as the orchestration framework: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
- [discussion] no gold standard caveat: "Since a gold standard is currently not available and the biological ground truth is largely unknown2,25, our analyses cannot give a definitive answer of what method is best."
