---
name: spurious-interaction-generation-from-variable-genes
description: Use when stress-testing the robustness of CCC inference (e.g. CellChat,
  CellPhoneDB, SingleCellSignalR run via LIANA) to corruption of the prior-knowledge
  ligand-receptor resource rather than to noise in the expression data or cell-type
  labels.
when_to_use_negative:
- When the perturbation of interest is cell-level noise (subsampling 0–40% of cells
  or reshuffling 0–40% of cell-type labels) rather than prior-knowledge-resource corruption
  — this is a different ablation axis
- When the goal is to identify a definitively best CCC method or resource — the article
  states no gold standard exists and biological ground truth for CCC is largely unknown
- When genuine large-scale curation of ligand-receptor resources is the goal; spurious
  interactions simulate resource errors and are not biologically validated knowledge
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: LIANA
  role: Framework orchestrating the CCC methods (CellChat, CellPhoneDB, Connectome,
    NATMI, SingleCellSignalR, iTALK, Cytotalk) that are re-run on each spurious-interaction-perturbed
    resource and whose top-250 rankings are compared
  repo: https://github.com/saezlab/liana
- name: NATMI
  role: Representative resource-bound CCC method (connectomeDB2020 prior) of the kind
    whose ligand-receptor list is replaced with spurious interactions in the resource-noise
    ablation
  repo: https://github.com/asrhou/NATMI.git
- name: SingleCellSignalR
  role: LRdb-backed CCC method (custom fork allows custom ligand-receptor resources)
    used to test how spurious resource interactions shift LRscore-based rankings
  repo: https://github.com/SCA-IRCM/SingleCellSignalR_v1
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:7ea7358b6352b3b067b2ec0d0fe476d28e6f224a1f0891506dc782ff8439f555
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# spurious-interaction-generation-from-variable-genes

## Summary

Generates spurious ligand-receptor interactions drawn from the 2000 most variable genes to replace 0–40% of a curated CCC resource's interactions, in both selective (rank-preserving) and non-selective modes. This isolates how much noise in the prior-knowledge resource — as opposed to noise in the scRNA-seq data — destabilises ligand-receptor rankings produced by CCC inference methods.

## When to use

Use when stress-testing the robustness of CCC inference (e.g. CellChat, CellPhoneDB, SingleCellSignalR run via LIANA) to corruption of the prior-knowledge ligand-receptor resource rather than to noise in the expression data or cell-type labels.

## When NOT to use

- When the perturbation of interest is cell-level noise (subsampling 0–40% of cells or reshuffling 0–40% of cell-type labels) rather than prior-knowledge-resource corruption — this is a different ablation axis
- When the goal is to identify a definitively best CCC method or resource — the article states no gold standard exists and biological ground truth for CCC is largely unknown
- When genuine large-scale curation of ligand-receptor resources is the goal; spurious interactions simulate resource errors and are not biologically validated knowledge

## Inputs

- Single-cell RNA-seq expression matrix with cell-type labels (used to identify the 2000 most variable genes)
- Curated ligand-receptor interaction resource (e.g. connectomeDB2020, CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb)
- Baseline ligand-receptor interaction ranking from each CCC method run on the unperturbed dataset

## Outputs

- Perturbed resource variants: replacement fractions 0–40% x 5 randomisations x 2 modes (selective rank-preserving and non-selective)
- Overlap scores of top-250 ranked interactions (perturbed vs unperturbed) per method, aggregated over randomisations, written to a CSV file
- Overlap-vs-perturbation-magnitude plot reproducing Supplementary Fig S13, written to a PNG file

## How to apply

Compute the 2000 most variable genes from the single-cell RNA-seq expression matrix and draw spurious ligand-receptor pairs from that pool only. Replace 0–40% of the interactions in the chosen resource (e.g. CellChatDB, connectomeDB2020, LRdb, Ramilowski) at graded levels (0, 10, 20, 30, 40%) using two modes: selective rank-preserving replacement, which replaces the lowest-ranked interactions, and non-selective replacement, which randomly replaces interactions to simulate a realistic resource change. Generate 5 independent randomisations per perturbation level and mode, then re-run each CCC method (orchestrated through LIANA) on every perturbed resource and rank the inferred interactions. Compute the overlap of each perturbed run's top-250 ranked interactions against the unperturbed ranking, aggregate across the 5 randomisations, and plot overlap versus perturbation magnitude. The paper's key decision point is that non-selective replacement had a strong effect on all methods, so the selective/non-selective contrast is the informative comparison.

## Related tools

- **LIANA** (Framework orchestrating the CCC methods (CellChat, CellPhoneDB, Connectome, NATMI, SingleCellSignalR, iTALK, Cytotalk) that are re-run on each spurious-interaction-perturbed resource and whose top-250 rankings are compared) — https://github.com/saezlab/liana
- **NATMI** (Representative resource-bound CCC method (connectomeDB2020 prior) of the kind whose ligand-receptor list is replaced with spurious interactions in the resource-noise ablation) — https://github.com/asrhou/NATMI.git
- **SingleCellSignalR** (LRdb-backed CCC method (custom fork allows custom ligand-receptor resources) used to test how spurious resource interactions shift LRscore-based rankings) — https://github.com/SCA-IRCM/SingleCellSignalR_v1

## Evaluation signals

- At 0% replacement, overlap with the unperturbed top-250 ranking must be complete (100%) for every method, confirming the perturbation pipeline itself introduces no change
- Both modes must be generated at matched levels (0–40%) with exactly 5 randomisations per level, so selective vs non-selective effects are comparable
- Spurious pairs must be drawn only from the 2000 most variable genes, not from the full gene set
- Non-selective replacement should produce a markedly larger drop in top-250 overlap than selective rank-preserving replacement, reproducing the paper's finding for all methods
- Overlap-vs-perturbation curves are aggregated across the 5 randomisations and exported as a CSV plus a PNG matching Supplementary Fig S13

## Limitations

- No gold standard exists for CCC, so top-250 overlap with the unperturbed ranking measures stability, not biological correctness of the predictions
- Spurious interactions sampled from variable genes model resource corruption abstractly and do not capture the specific curation disagreements or pathway-coverage biases documented among real resources
- Findings are method-and-resource dependent: the article notes both the method and resource can considerably impact CCC inference predictions, so results do not transfer automatically to untested method-resource combinations

## Evidence

- [supplementary] Resource perturbation workflow with graded replacement levels and two modes: [evidence span withheld — anchor status: not_found]
- [results] Non-selective replacement is the dominant resource-noise effect: [evidence span withheld — anchor status: not_found]
- [discussion] Method and resource choice both matter for CCC predictions: "Our results suggest that both the method and resource can considerably impact CCC inference predictions"
- [readme] Resources are manually curated ligand-receptor lists that the spurious interactions stand in for: [evidence span withheld — anchor status: external_doc]
