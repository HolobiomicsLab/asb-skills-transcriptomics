---
name: rank-overlap-analysis-of-ligand-receptor-interactions
description: Use when you must benchmark or stress-test CCC inference methods (CellChat,
  CellPhoneDB, Connectome, NATMI, SingleCellSignalR, iTALK, Cytotalk) for sensitivity
  to noisy cell data or corrupted ligand-receptor prior-knowledge resources.
when_to_use_negative:
- The CCC method infers intracellular processes (e.g. NicheNet, Cytotalk, SoptSC)
  — the paper states these are 'not directly comparable' with ligand-receptor methods
- The goal is absolute accuracy against a biological ground truth — no gold standard
  for CCC is available, so only perturbation-stability (overlap) can be assessed
- Evaluating agreement with independent modalities (spatial colocalisation, cytokine
  activities, receptor protein abundance) rather than ranking stability under noise
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: LIANA
  role: Orchestrates all CCC methods and resources in one framework so perturbation
    experiments can be re-run per method-resource combination
  repo: https://github.com/saezlab/liana
- name: CellChat
  role: CCC inference method (probability score from differentially expressed transmitter/receiver
    genes) whose ranking stability is evaluated
- name: CellPhoneDB
  role: CCC inference method (truncated mean of transmitter and receiver expression)
    whose ranking stability is evaluated
- name: SingleCellSignalR
  role: CCC inference method (regularised LRscore) whose ranking stability is evaluated;
    uses a custom version allowing custom ligand-receptor resources
  repo: https://github.com/saezlab/SingleCellSignalR_v1
- name: NATMI
  role: CCC inference method (mean-expression edge weight from transmitter/receiver
    gene expression product) whose ranking stability is evaluated
  repo: https://github.com/saezlab/NATMI
- name: Connectome
  role: CCC inference method (weight_norm product of normalised transmitter/receiver
    expression) whose ranking stability is evaluated
- name: iTALK
  role: CCC-inspired method (logFC means of one-versus-all fold change) whose ranking
    stability is evaluated
- name: Cytotalk
  role: CCC-inspired method (crosstalk score from transmitter/receiver expression)
    whose ranking stability is evaluated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:826c4266a4c3a4437fd49e69f90a7700325f1aa77ca809e0d15cc9b2496f55b0
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Rank-overlap analysis of ligand-receptor interactions

## Summary

Quantifies the robustness of cell-cell communication (CCC) inference methods by re-ranking ligand-receptor interactions on perturbed inputs (cell subsampling, cell-type label reshuffling, resource interaction replacement) and computing the overlap of top-250 ranked interactions against the unperturbed ranking, as done in the LIANA benchmark (Dimitrov et al., Nat Commun 2022).

## When to use

Use when you must benchmark or stress-test CCC inference methods (CellChat, CellPhoneDB, Connectome, NATMI, SingleCellSignalR, iTALK, Cytotalk) for sensitivity to noisy cell data or corrupted ligand-receptor prior-knowledge resources.

## When NOT to use

- The CCC method infers intracellular processes (e.g. NicheNet, Cytotalk, SoptSC) — the paper states these are 'not directly comparable' with ligand-receptor methods
- The goal is absolute accuracy against a biological ground truth — no gold standard for CCC is available, so only perturbation-stability (overlap) can be assessed
- Evaluating agreement with independent modalities (spatial colocalisation, cytokine activities, receptor protein abundance) rather than ranking stability under noise

## Inputs

- Unperturbed single-cell RNA-seq dataset with cell-type annotations (e.g. h5ad/Seurat object)
- Ligand-receptor prior-knowledge resources: CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb
- Perturbation specifications: subsampling and label-reshuffling levels (0-40%), interaction-replacement levels (0-40%) with spurious interactions from the 2000 most variable genes, 5 randomisations per level

## Outputs

- CSV file of top-250 rank-overlap scores per method per perturbation type, level, and randomisation
- PNG figure of overlap as a function of perturbation magnitude (reproducing Supplementary Fig S13)
- Baseline (unperturbed) ranking of ligand-receptor interactions per CCC method

## How to apply

Orchestrate all CCC methods through LIANA, which provides an open-source interface decoupling methods from resources, and run each method on the unperturbed single-cell RNA-seq dataset to obtain a baseline ranking of ligand-receptor interactions. Generate cell-perturbed datasets by subsampling 0-40% of cells and, separately, by reshuffling 0-40% of cell-type labels (simulating erroneous annotation), with 5 randomisations per perturbation level. Generate resource-perturbed priors by replacing 0-40% of resource interactions with spurious interactions drawn from the 2000 most variable genes, in both selective rank-preserving and non-selective modes, again with 5 randomisations per level. Re-run every method on each perturbed combination, rank the inferred interactions, and compute the overlap of the top-250 ranked interactions of each perturbed run against the unperturbed ranking. Aggregate overlap scores across the 5 randomisations and plot overlap as a function of perturbation magnitude to reproduce Supplementary Fig S13; interpret a strong overlap drop under non-selective interaction replacement as evidence that a resource change, more than data noise, drives prediction differences.

## Related tools

- **LIANA** (Orchestrates all CCC methods and resources in one framework so perturbation experiments can be re-run per method-resource combination) — https://github.com/saezlab/liana
- **CellChat** (CCC inference method (probability score from differentially expressed transmitter/receiver genes) whose ranking stability is evaluated)
- **CellPhoneDB** (CCC inference method (truncated mean of transmitter and receiver expression) whose ranking stability is evaluated)
- **SingleCellSignalR** (CCC inference method (regularised LRscore) whose ranking stability is evaluated; uses a custom version allowing custom ligand-receptor resources) — https://github.com/saezlab/SingleCellSignalR_v1
- **NATMI** (CCC inference method (mean-expression edge weight from transmitter/receiver gene expression product) whose ranking stability is evaluated) — https://github.com/saezlab/NATMI
- **Connectome** (CCC inference method (weight_norm product of normalised transmitter/receiver expression) whose ranking stability is evaluated)
- **iTALK** (CCC-inspired method (logFC means of one-versus-all fold change) whose ranking stability is evaluated)
- **Cytotalk** (CCC-inspired method (crosstalk score from transmitter/receiver expression) whose ranking stability is evaluated)

## Evaluation signals

- At 0% perturbation the overlap of the top-250 interactions with the unperturbed ranking must be maximal (identity) for every method, serving as a sanity check
- The CSV must contain one overlap score per method x perturbation type x level x randomisation, aggregated over the 5 randomisations per level
- The PNG must show overlap as a monotone function of perturbation magnitude (0-40%) per method, reproducing Supplementary Fig S13
- Non-selective interaction replacement should produce a markedly stronger overlap drop than cell subsampling or label reshuffling, per the reported finding
- Every ranked perturbed run must contain exactly the top-250 interactions compared against the unperturbed ranking

## Limitations

- No gold standard exists and CCC biological ground truth is largely unknown, so this skill measures stability, not correctness
- Methods that additionally infer intracellular processes (NicheNet, Cytotalk, SoptSC) are not directly comparable in this ranking-overlap scheme
- Both the method and the resource can considerably impact CCC inference predictions, so conclusions are conditional on the method-resource combination used
- Non-selective replacement of interactions with spurious pairs from the 2000 most variable genes had a strong effect on all methods — results are sensitive to how the corrupted resource is constructed

## Evidence

- [supplementary] workflow step defining the overlap metric: [evidence span withheld — anchor status: not_found]
- [supplementary] perturbation design: [evidence span withheld — anchor status: not_found]
- [results] key robustness finding: [evidence span withheld — anchor status: not_found]
- [abstract] LIANA as orchestration framework: "we provide LIANA, a LIgand-receptor ANalysis frAmework as an open-source interface to all the resources and methods"
- [readme] LIANA README capability statement: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
- [discussion] no ground truth caveat: "Since a gold standard is currently not available and the biological ground truth is largely unknown2,25, our analyses cannot give a definitive answer of what method is best."
