---
name: boolean-network-attractor-analysis
description: Use when you have scBONITA-learned Boolean rules for a pathway (e.g.,
  a GRAPHML KEGG topology like hsa00010) plus binarized single-cell expression states
  and need to map each cell to a characteristic signaling state.
when_to_use_negative:
- The analysis goal is differential gene expression or conventional over-representation
  analysis (ORA) — the article showed ORA was inefficacious at identifying coherent
  molecular processes, but attractor analysis is also not a DE method
- No executable Boolean rules have been inferred for the pathway (run Rule_Inference_With_scBONITA
  first; attractor analysis consumes the learned rules)
- The expression data are not single-cell states to be mapped — bulk state vectors
  are not the intended input for per-cell attractor assignment
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scBONITA
  role: Provides the attractor-analysis functionality that simulates learned Boolean
    models and maps cells to closest attractors by Hamming distance
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime required by scBONITA; attractor analysis is invoked from the scBONITA
    Python pipeline
- name: conda
  role: Environment management for scBONITA dependencies (spec-file.txt / yml environment)
- name: SLURM
  role: Distributed-computing system on which attractor analysis jobs are submitted
    via generated sbatch files
- name: GRAPHML
  role: Format of the pathway network topology supplied as the attractor-analysis
    network
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:1e2e9398bf7886db63c1c284cb2fba130978245164b31d6787ab62a9d85d84fa
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
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# boolean-network-attractor-analysis

## Summary

Simulates scBONITA-inferred discrete (Boolean) state models of signaling pathways until trajectories fall into steady states (attractors), then assigns each sequenced single cell to its closest attractor via Hamming distance, enabling pathway-based characterization of cellular states.

## When to use

Use when you have scBONITA-learned Boolean rules for a pathway (e.g., a GRAPHML KEGG topology like hsa00010) plus binarized single-cell expression states and need to map each cell to a characteristic signaling state.

## When NOT to use

- The analysis goal is differential gene expression or conventional over-representation analysis (ORA) — the article showed ORA was inefficacious at identifying coherent molecular processes, but attractor analysis is also not a DE method
- No executable Boolean rules have been inferred for the pathway (run Rule_Inference_With_scBONITA first; attractor analysis consumes the learned rules)
- The expression data are not single-cell states to be mapped — bulk state vectors are not the intended input for per-cell attractor assignment

## Inputs

- scBONITA-inferred Boolean regulatory rules for a signaling pathway (e.g., KEGG hsa00010 topology as GRAPHML)
- Binarized single-cell expression states from the processed scRNA-seq training matrix (rows = genes, columns = cells, first column labeled 'Genes')
- Ensemble of rule sets (ERS) sampled from scBONITA rule inference

## Outputs

- cell_attractor_assignments.csv — one attractor assignment row per sequenced cell
- Set of steady-state attractors (fixed points) of the simulated Boolean network
- Pathway-based characterization of cellular states (e.g., signaling states of CD8+ T cells and monocytes)

## How to apply

Load the scBONITA-inferred Boolean regulatory rules for the target pathway network (GRAPHML topology) together with the binarized single-cell expression states. Sample rule sets from the ensemble of rule sets (ERS) inferred from the scRNA-seq training data to capture rule-determination uncertainty. Synchronously update the Boolean network starting from each observed cell's state and iterate until a steady-state attractor is reached. Compute the Hamming distance between each cell's converged state and each attractor, and assign the cell to the closest attractor so pathway-specific signaling modes of CD8+ T cells and monocytes can be characterized. Because rule inference and attractor analysis are memory-hungry on large scRNA-seq matrices (>10G RAM), run on a SLURM system rather than a desktop; write per-cell assignments to cell_attractor_assignments.csv.

## Related tools

- **scBONITA** (Provides the attractor-analysis functionality that simulates learned Boolean models and maps cells to closest attractors by Hamming distance) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime required by scBONITA; attractor analysis is invoked from the scBONITA Python pipeline)
- **conda** (Environment management for scBONITA dependencies (spec-file.txt / yml environment))
- **SLURM** (Distributed-computing system on which attractor analysis jobs are submitted via generated sbatch files)
- **GRAPHML** (Format of the pathway network topology supplied as the attractor-analysis network)

## Evaluation signals

- cell_attractor_assignments.csv exists and contains exactly one attractor assignment row per sequenced cell
- Every assigned attractor is a fixed point of the simulated Boolean network (re-simulating from the attractor state leaves it unchanged)
- Each cell's assigned attractor has minimal Hamming distance to the cell's converged state among all attractors
- Simulated trajectories from each cell's starting state converge (no further state changes) before Hamming distances are computed
- Assignments support biologically interpretable grouping — e.g., distinct pathway-specific signaling states recoverable for CD8+ T cells and monocytes as in the HIV-associated atherosclerosis analysis

## Limitations

- Rule inference and attractor analysis require >10G RAM for large scRNA-seq datasets and are not recommended on a desktop computer
- scBONITA assumes scRNA-seq data samples the state space of a dynamic Boolean network since time-series data is rarely available in human studies
- Inference is expected to improve when pure cell populations are sequenced; mixed PBMC subpopulations used in the study may degrade rule quality
- Pre-classified subpopulations were used rather than scBONITA's own clustering to reduce variability and improve specificity

## Evidence

- [abstract] Attractor analysis maps cells to network steady states: "Attractor analysis with scBONITA facilitated the pathway-based characterization of cellular states in CD8+ T cells and monocytes"
- [intro] Novel attractor-analysis method grounded in scBONITA models: [evidence span withheld — anchor status: not_found]
- [discussion] Motivation for attractor analysis functionality: "To map cells to distinct signaling modes of the pathways described above, we developed scBONITA’s attractor analysis functionalities."
- [readme] Tutorial for running attractor analysis: "Please refer to the tutorial "Attractor_Analysis_With_scBONITA.ipynb" for a demonstration of attractor analysis with scBONITA using a test dataset packaged with scBONITA"
- [readme] Memory requirements constrain where attractor analysis can run: "the amount of RAM required for large scRNA-seq datasets is in excess of 10G; we hence don't recommend running rule inference and attractor analysis on a desktop computer."
