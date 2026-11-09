---
name: ccc-interaction-perturbation-robustness-analysis
description: Use when you need to quantify the stability of ligand-receptor interaction
  rankings inferred from single-cell RNA-seq under perturbed cell composition, shuffled
  cell-type labels, or corrupted prior-knowledge resources.
when_to_use_negative:
- When the goal is to evaluate agreement of CCC predictions with external modalities
  (spatial colocalisation, cytokine activities, receptor protein abundance) rather
  than robustness to noise
- When comparing method-resource combinations on unperturbed data only — this skill
  requires perturbed replicates to compute overlap against the baseline ranking
- When methods that additionally infer intracellular processes (NicheNet, SoptSC)
  are in scope, since the paper states such methods 'are not directly comparable'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LIANA
  role: Framework that orchestrates all CCC methods and resources and enables any
    combination of ligand-receptor methods and resources plus their consensus; used
    to run baselines and perturbed inference
  repo: https://github.com/saezlab/liana
- name: CellChat
  role: CCC inference method whose probability-based predictions are re-run and ranked
    on perturbed inputs
- name: CellPhoneDB
  role: CCC inference method (truncated mean scoring) re-run on each perturbed dataset/resource
    combination
- name: Connectome
  role: CCC inference method (weight_norm scoring) included in the robustness benchmark
- name: NATMI
  role: CCC inference method (mean-expression edge weight) re-run on every perturbed
    combination
  repo: https://github.com/asrhou/NATMI.git
- name: SingleCellSignalR
  role: CCC inference method (LRscore) using a custom version supporting custom ligand-receptor
    resources for resource perturbation
  repo: https://github.com/SCA-IRCM/SingleCellSignalR_v1
- name: iTALK
  role: CCC inference method (logFC means) included in the perturbation benchmark
- name: Cytotalk
  role: CCC inference method (crosstalk score) included in the perturbation benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:f9adac38c60d9644839306a94ca38d23cf2d77cc2cb68f1199d6d5dbd7c19f14
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ccc-interaction-perturbation-robustness-analysis

## Summary

A benchmarking skill that quantifies how robust cell-cell communication (CCC) inference methods are to noise introduced in the input scRNA-seq data (cell subsampling, erroneous cell-type annotation) and in the prior-knowledge resource (replacement of ligand-receptor interactions with spurious ones).

## When to use

Use when you need to quantify the stability of ligand-receptor interaction rankings inferred from single-cell RNA-seq under perturbed cell composition, shuffled cell-type labels, or corrupted prior-knowledge resources.

## When NOT to use

- When the goal is to evaluate agreement of CCC predictions with external modalities (spatial colocalisation, cytokine activities, receptor protein abundance) rather than robustness to noise
- When comparing method-resource combinations on unperturbed data only — this skill requires perturbed replicates to compute overlap against the baseline ranking
- When methods that additionally infer intracellular processes (NicheNet, SoptSC) are in scope, since the paper states such methods 'are not directly comparable'

## Inputs

- Unperturbed single-cell RNA-seq dataset with cell-type annotations (baseline input for CCC inference)
- Ligand-receptor prior-knowledge resources (e.g., CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb)
- Perturbation specification: cell subsampling levels (0-40%), cell-type label reshuffling levels (0-40%), resource interaction replacement levels (0-40%) in selective rank-preserving and non-selective modes, 5 randomisations per level
- Pool of variable genes (2000 most variable genes) from which spurious interactions are drawn

## Outputs

- Ranking of inferred ligand-receptor interactions for each method on every perturbed and unperturbed run
- CSV file of top-250 interaction overlap scores per method as a function of perturbation magnitude, aggregated over 5 randomisations
- PNG figure of overlap vs. perturbation magnitude reproducing Supplementary Fig S13

## How to apply

Orchestrate the CCC methods (CellChat, CellPhoneDB, Connectome, NATMI, SingleCellSignalR, iTALK, Cytotalk) through the LIANA framework and run each on the unperturbed single-cell RNA-seq dataset to obtain the baseline ranking of ligand-receptor interactions. Generate cell-perturbed datasets by subsampling 0-40% of cells and, separately, reshuffling 0-40% of the cell-type labels, with 5 randomisations per perturbation level; generate resource-perturbed priors by replacing 0-40% of resource interactions with spurious interactions drawn from the 2000 most variable genes, in both selective rank-preserving and non-selective modes. Re-run every method on each perturbed combination, rank the inferred interactions, and compute the overlap of the top-250 ranked interactions against the unperturbed ranking per method. Aggregate the overlap scores across the 5 randomisations, plot overlap as a function of perturbation magnitude (reproducing Supplementary Fig S13), and write the scores to a CSV and the figure to a PNG. Rationale: non-selective interaction replacement simulates a resource change and is expected to strongly affect all methods, whereas erroneous cell-type annotation should perturb results more than cell subsampling; all methods, especially CellChat, CellPhoneDB, and SingleCellSignalR, were fairly robust to noise in both data and resource.

## Related tools

- **LIANA** (Framework that orchestrates all CCC methods and resources and enables any combination of ligand-receptor methods and resources plus their consensus; used to run baselines and perturbed inference) — https://github.com/saezlab/liana
- **CellChat** (CCC inference method whose probability-based predictions are re-run and ranked on perturbed inputs)
- **CellPhoneDB** (CCC inference method (truncated mean scoring) re-run on each perturbed dataset/resource combination)
- **Connectome** (CCC inference method (weight_norm scoring) included in the robustness benchmark)
- **NATMI** (CCC inference method (mean-expression edge weight) re-run on every perturbed combination) — https://github.com/asrhou/NATMI.git
- **SingleCellSignalR** (CCC inference method (LRscore) using a custom version supporting custom ligand-receptor resources for resource perturbation) — https://github.com/SCA-IRCM/SingleCellSignalR_v1
- **iTALK** (CCC inference method (logFC means) included in the perturbation benchmark)
- **Cytotalk** (CCC inference method (crosstalk score) included in the perturbation benchmark)

## Evaluation signals

- For each method, overlap of the top-250 ranked interactions of each perturbed run is computed against the unperturbed baseline ranking
- Each perturbation level (0-40% subsampling, 0-40% label reshuffling, 0-40% interaction replacement in selective and non-selective modes) is replicated 5 times and scores are aggregated across randomisations
- Non-selective interaction replacement should produce a strong effect on all methods, larger than selective rank-preserving replacement
- Erroneous cell-type annotation should degrade overlap more than cell subsampling at matched perturbation magnitude
- Results reproduce Supplementary Fig S13 and are persisted as a CSV of scores plus a PNG of overlap-vs-perturbation curves

## Limitations

- No gold standard is available and the biological ground truth for CCC is largely unknown, so the analysis cannot identify a definitively best method
- Such perturbation benchmarks cannot distinguish simple co-expression from actual CCC events, and do not capture complex relationships between CCC events
- Resource perturbation via spurious interactions drawn from the 2000 most variable genes models resource changes only approximately; conclusions are method- and resource-dependent

## Evidence

- [abstract] method-resource influence: "We then examine all possible combinations of methods and resources and show that both strongly influence the predicted intercellular interactions."
- [readme] LIANA as orchestration interface: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
- [discussion] method and resource impact conclusion: "Our results suggest that both the method and resource can considerably impact CCC inference predictions"
- [discussion] no gold standard limitation: "Since a gold standard is currently not available and the biological ground truth is largely unknown2,25, our analyses cannot give a definitive answer of what method is best."
