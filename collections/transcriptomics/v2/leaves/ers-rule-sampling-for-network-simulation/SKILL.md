---
name: ers-rule-sampling-for-network-simulation
description: Use when scBONITA Boolean rules and a GRAPHML pathway topology are available
  and cells must be mapped to characteristic pathway-specific signaling states via
  attractor analysis of the learned discrete-state models.
when_to_use_negative:
- You only need static differential-expression or over-representation analysis — the
  article showed conventional ORA was inefficacious here, but attractor simulation
  is the wrong tool if you do not have an executable Boolean model
- The pathway topology or Boolean rules have not been inferred yet (rule inference
  with scBONITA must run first)
- You intend to run attractor analysis on a desktop without the >10G RAM a large scRNA-seq
  dataset requires
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: scBONITA
  role: Core algorithm whose learned models are simulated to reach attractors; provides
    the attractor-analysis functionality that maps cells to distinct signaling modes
  repo: https://github.com/Thakar-Lab/scBONITA
- name: SLURM
  role: Job scheduler used to run the compute-heavy rule inference and attractor analysis
    on high-performance systems
- name: Python 3.6
  role: Runtime required to execute the scBONITA pipeline including attractor analysis
- name: conda
  role: Environment management for the scBonita dependency stack (install from spec-file
    or yml)
- name: GRAPHML
  role: Format of the pathway network topologies simulated with the sampled rules
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:5fb72c5ccbd94348d01a4c3f524b1f631cb6e72c2aef5cb2d57bb3fa4d4dc4b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - GRAPHML
  - Python
  - Python 3.6
  - SLURM
  - conda
  - scBONITA
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ERS rule sampling for network simulation

## Summary

Sample alternative rule sets from the ensemble of rule sets (ERS) inferred by scBONITA from scRNA-seq data, simulate the resulting discrete Boolean networks synchronously to convergence, and assign each sequenced cell to its closest steady-state attractor by Hamming distance.

## When to use

Use when scBONITA Boolean rules and a GRAPHML pathway topology are available and cells must be mapped to characteristic pathway-specific signaling states via attractor analysis of the learned discrete-state models.

## When NOT to use

- You only need static differential-expression or over-representation analysis — the article showed conventional ORA was inefficacious here, but attractor simulation is the wrong tool if you do not have an executable Boolean model
- The pathway topology or Boolean rules have not been inferred yet (rule inference with scBONITA must run first)
- You intend to run attractor analysis on a desktop without the >10G RAM a large scRNA-seq dataset requires

## Inputs

- scBONITA-inferred Boolean regulatory rules (ensemble of rule sets, ERS) from scRNA-seq training data
- GRAPHML pathway network topology (e.g. non-KEGG sources saved in GRAPHML format)
- Binarized single-cell expression states in matrix-style format (genes as rows, cell IDs as columns, log2(TPM+1)-like values)
- Cell metadata/conditions matrix (rows = cells, columns = 0/1 condition labels)

## Outputs

- Steady-state attractors (fixed points) of the simulated Boolean network
- cell_attractor_assignments.csv with one per-cell attractor assignment row for downstream pathway-based characterization of cellular states

## How to apply

Load the scBONITA-inferred Boolean regulatory rules for the target pathway (GRAPHML topology) together with the binarized single-cell expression states. Sample alternative rule sets from the ensemble of rule sets (ERS) learned from the scRNA-seq training data, reflecting the hypothesis that scRNA-seq samples the state space of a dynamic Boolean network. Starting from each observed cell's binarized state, synchronously update the sampled Boolean network and iterate until a steady-state attractor is reached. Compute the Hamming distance between each cell's converged state and each attractor, then assign each cell to its closest attractor; these attractors represent pathway-specific signaling states (e.g. in CD8+ T cells and monocytes). Because rule inference and attractor analysis are RAM-hungry (often >10G for large scRNA-seq datasets), run them as SLURM jobs rather than on a desktop.

## Related tools

- **scBONITA** (Core algorithm whose learned models are simulated to reach attractors; provides the attractor-analysis functionality that maps cells to distinct signaling modes) — https://github.com/Thakar-Lab/scBONITA
- **SLURM** (Job scheduler used to run the compute-heavy rule inference and attractor analysis on high-performance systems)
- **Python 3.6** (Runtime required to execute the scBONITA pipeline including attractor analysis)
- **conda** (Environment management for the scBonita dependency stack (install from spec-file or yml))
- **GRAPHML** (Format of the pathway network topologies simulated with the sampled rules)

## Evaluation signals

- cell_attractor_assignments.csv exists in the output directory
- The file contains exactly one attractor assignment row per sequenced cell
- Every attractor assigned to a cell is a fixed point of the simulated Boolean network (re-updating its state returns the same state)
- Each cell is assigned to the attractor with minimal Hamming distance from its converged state
- Attractor assignments reproduce pathway-based separation of cell states (e.g. CD8+ T cell and monocyte signaling states in the HIV/AS study)

## Limitations

- Rule inference and attractor analysis are not recommended on a desktop computer; RAM needs exceed 10G for large scRNA-seq datasets
- In lieu of time-series data, scBONITA assumes scRNA-seq data samples the state space of a dynamic Boolean network, an approximation inherent to the approach
- scBONITA inference is expected to improve when pure cell populations are sequenced; mixed PBMC subpopulations (as in the HIV/AS data) can limit rule quality
- scBONITA is currently designed for SLURM systems, so transfer to other schedulers requires adaptation

## Evidence

- [abstract] These dynamic models are used for pathway analysis and to map sequenced cells to characteristic signaling states (attractor analysis): "These dynamic models are used for pathway analysis and to map sequenced cells to characteristic signaling states (attractor analysis)"
- [intro] Attractor analysis uses scBONITA-learned models to identify pathway-specific signaling states: "We present a novel method (‘attractor analysis’) that uses the models learned by scBONITA to identify pathway-specific signaling states for CD8+ T cells and monocytes"
- [discussion] scRNA-seq samples the state space of a dynamic Boolean network: "In lieu of using time-series data, scBONITA hypothesizes that scRNA-seq data represents samples in the state space of a dynamic Boolean network."
- [readme] README attractor analysis tutorial reference: "Please refer to the tutorial "Attractor_Analysis_With_scBONITA.ipynb" for a demonstration of attractor analysis with scBONITA using a test dataset packaged with scBONITA"
- [readme] README high-RAM/SLURM requirement: "we hence don't recommend running rule inference and attractor analysis on a desktop computer."
