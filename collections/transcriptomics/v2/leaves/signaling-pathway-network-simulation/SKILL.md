---
name: signaling-pathway-network-simulation
description: Use when scRNA-seq-derived Boolean regulatory rules exist for a pathway
  topology (GRAPHML or KEGG) and the goal is to map individual cells to characteristic
  signaling states (attractors) of that pathway.
when_to_use_negative:
- No Boolean regulatory rules have been inferred yet for the pathway — run scBONITA
  rule inference (Step 1–2 of the pipeline) first
- The analysis goal is differential gene expression or conventional over-representation
  analysis (ORA) rather than dynamic modeling — the paper showed ORA was inefficacious
  here, but this skill addresses dynamic signaling states, not DE testing
- Data are too large for available RAM — the README recommends >10G RAM and a SLURM
  system rather than a desktop for attractor analysis
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2259
tools:
- name: scBONITA
  role: 'Performs attractor analysis: simulates the learned Boolean network models
    to reach steady-state attractors and maps cells to closest attractors'
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime environment required by scBONITA (Python 3.6 or newer) for running
    the attractor analysis pipeline
- name: conda
  role: Creates and activates the scBonita environment (from spec-file.txt or yml
    file) in which attractor analysis is run
- name: SLURM
  role: Distributed computing system on which scBONITA attractor analysis jobs (sbatch)
    are executed
- name: GRAPHML
  role: Format of pathway topology networks (non-KEGG sources) used as input for rule
    inference and attractor analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:f5ea905c8230c1a120ed511692440092cc77bdd10f278f302297641820b32ab5
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# signaling-pathway-network-simulation

## Summary

Simulates scBONITA-learned Boolean regulatory rules for a signaling pathway (e.g., KEGG or GRAPHML topology) from each sequenced cell's binarized state until steady-state attractors are reached, then assigns each cell to its closest attractor by Hamming distance. This enables pathway-based characterization of cellular signaling states not apparent from gene expression alone.

## When to use

Use when scRNA-seq-derived Boolean regulatory rules exist for a pathway topology (GRAPHML or KEGG) and the goal is to map individual cells to characteristic signaling states (attractors) of that pathway.

## When NOT to use

- No Boolean regulatory rules have been inferred yet for the pathway — run scBONITA rule inference (Step 1–2 of the pipeline) first
- The analysis goal is differential gene expression or conventional over-representation analysis (ORA) rather than dynamic modeling — the paper showed ORA was inefficacious here, but this skill addresses dynamic signaling states, not DE testing
- Data are too large for available RAM — the README recommends >10G RAM and a SLURM system rather than a desktop for attractor analysis

## Inputs

- scBONITA-inferred Boolean regulatory rules (ensemble of rule sets, ERS) for the target signaling pathway
- Pathway topology in GRAPHML format (or KEGG pathway, e.g., hsa00010, hsa00020)
- Binarized single-cell expression states for sequenced cells (matrix-style file, columns = cells with matching cell IDs)
- Metadata/conditions file with per-cell condition labels

## Outputs

- cell_attractor_assignments.csv with one attractor assignment row per sequenced cell
- Set of steady-state attractors (fixed points) of the simulated Boolean network
- Pathway-specific characterization of cellular states (e.g., attractor memberships for CD8+ T cells and monocytes)

## How to apply

Load the scBONITA-inferred Boolean regulatory rules for the target pathway (GRAPHML topology) together with the binarized single-cell expression states. Sample alternative rule sets from the ensemble of rule sets (ERS) inferred from the scRNA-seq training data. Synchronously update the Boolean network starting from each observed cell's state and iterate until a steady-state attractor is reached. Compute the Hamming distance between each cell's converged state and each attractor, and assign each cell to its closest attractor. Write per-cell attractor assignments to cell_attractor_assignments.csv for downstream pathway-based characterization of cellular states (as demonstrated for CD8+ T cells and monocytes in the HIV-associated atherosclerosis study).

## Related tools

- **scBONITA** (Performs attractor analysis: simulates the learned Boolean network models to reach steady-state attractors and maps cells to closest attractors) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime environment required by scBONITA (Python 3.6 or newer) for running the attractor analysis pipeline)
- **conda** (Creates and activates the scBonita environment (from spec-file.txt or yml file) in which attractor analysis is run)
- **SLURM** (Distributed computing system on which scBONITA attractor analysis jobs (sbatch) are executed)
- **GRAPHML** (Format of pathway topology networks (non-KEGG sources) used as input for rule inference and attractor analysis)

## Examples

```
python3.6 pipeline.py --dataFile "example.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa cvThreshold None
```

## Evaluation signals

- cell_attractor_assignments.csv exists and contains exactly one attractor assignment row per sequenced cell
- Every assigned attractor is a fixed point of the simulated Boolean network (updating from the attractor state returns the same state)
- Each cell's Hamming distance to its assigned attractor is minimal across all identified attractors
- Attractor assignments are consistent with the binarized expression states of cells (cells from the same subpopulation, e.g., CD8+ T cells or monocytes, map coherently to pathway-specific attractors)
- Convergence check: synchronous update from each cell's starting state reaches a steady state within a bounded number of iterations

## Limitations

- Rule inference and attractor analysis require a high-performance SLURM system with >10G RAM for large scRNA-seq datasets; a desktop computer is not recommended
- scRNA-seq data are treated as samples in the state space of a dynamic Boolean network in lieu of time-series data, which is rarely available in human studies
- Inference is expected to improve when pure cell populations are sequenced; mixed PBMC subpopulations add noise
- Pre-classified subpopulations were used rather than scBONITA's own clustering, to reduce variability in downstream pathway analysis
- The number of nodes (NODE) and sampled cells (CELL) must be configured in the C file (line 7/8) and recompiled with make before running

## Evidence

- [abstract] Attractor analysis enabled pathway-based characterization of cellular states: "Attractor analysis with scBONITA facilitated the pathway-based characterization of cellular states in CD8+ T cells and monocytes"
- [intro] Novel attractor analysis method using scBONITA-learned models: "We present a novel method (‘attractor analysis’) that uses the models learned by scBONITA to identify pathway-specific signaling states for CD8+ T cells and monocytes"
- [results] Simulation from cell states to attractors: "These dynamic models are used for pathway analysis and to map sequenced cells to characteristic signaling states (attractor analysis)"
- [methods] Attractor analysis tutorial: "Please refer to the tutorial "Attractor_Analysis_With_scBONITA.ipynb" for a demonstration of attractor analysis with scBONITA using a test dataset packaged with scBONITA"
- [discussion] Hypothesis that scRNA-seq samples the state space: "In lieu of using time-series data, scBONITA hypothesizes that scRNA-seq data represents samples in the state space of a dynamic Boolean network."
