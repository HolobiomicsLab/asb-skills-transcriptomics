---
name: hamming-distance-cell-to-attractor-mapping
description: Use when scBONITA-inferred Boolean rules and binarized single-cell expression
  states exist and you must map sequenced cells to characteristic signaling states
  (attractors) of a pathway.
when_to_use_negative:
- You only have raw unbinarized scRNA-seq counts and have not yet run scBONITA rule
  inference on the pathway topology
- The pathway's Boolean network does not reach fixed-point attractors (e.g., only
  cyclic/oscillatory dynamics), so Hamming-distance assignment to steady states is
  ill-defined
- Conventional differential expression or over-representation analysis suffices for
  the question — the article notes such ORA methods were inefficacious at identifying
  coherent molecular processes, which is precisely when attractor analysis is preferred
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scBONITA
  role: 'Performs the attractor analysis: simulates the learned Boolean models and
    maps sequenced cells to characteristic signaling states'
  repo: https://github.com/Thakar-Lab/scBONITA
- name: conda
  role: Provides the scBonita environment created from the provided yml/spec-file
    in which attractor analysis runs
- name: SLURM
  role: Execution environment scBONITA (including attractor analysis) is designed
    to run on for large scRNA-seq datasets
- name: GRAPHML
  role: Format of the pathway network topologies (KEGG or non-KEGG sources) whose
    Boolean rules drive the attractor simulation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:e2bb6cd32309d43719d13dbfa6583d6fc38f54c768f374cc5603b86685e06878
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - GRAPHML
  - Python
  - SLURM
  - conda
  - scBONITA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hamming-distance-cell-to-attractor-mapping

## Summary

Attractor analysis step of scBONITA that simulates the inferred Boolean network of a signaling pathway (e.g., from a GRAPHML topology) to steady states and assigns each sequenced cell to the attractor closest in Hamming distance, enabling pathway-based characterization of cellular states.

## When to use

Use when scBONITA-inferred Boolean rules and binarized single-cell expression states exist and you must map sequenced cells to characteristic signaling states (attractors) of a pathway.

## When NOT to use

- You only have raw unbinarized scRNA-seq counts and have not yet run scBONITA rule inference on the pathway topology
- The pathway's Boolean network does not reach fixed-point attractors (e.g., only cyclic/oscillatory dynamics), so Hamming-distance assignment to steady states is ill-defined
- Conventional differential expression or over-representation analysis suffices for the question — the article notes such ORA methods were inefficacious at identifying coherent molecular processes, which is precisely when attractor analysis is preferred

## Inputs

- scBONITA-inferred Boolean regulatory rules (ensemble of rule sets, ERS) for the pathway network
- Pathway network topology in GRAPHML format (KEGG or non-KEGG source)
- Binarized single-cell expression states (matrix-style training data: rows = genes, columns = cells, first row = cell IDs, first column labeled 'Genes')

## Outputs

- cell_attractor_assignments.csv with one attractor assignment row per sequenced cell
- Set of steady-state attractors (fixed points) of the simulated Boolean network

## How to apply

Load the scBONITA-inferred Boolean regulatory rules for the target pathway (GRAPHML topology, e.g., a KEGG pathway such as hsa00010) together with the binarized single-cell expression states for each cell. Sample rule sets from the ensemble of rule sets (ERS) learned from the scRNA-seq training data, then synchronously update the Boolean network starting from each observed cell's state, iterating until a steady-state attractor is reached. Compute the Hamming distance between each cell's converged state and every attractor, and assign the cell to its closest attractor. Write the per-cell assignments to cell_attractor_assignments.csv for downstream pathway-based characterization, as demonstrated in the Attractor_Analysis_With_scBONITA.ipynb tutorial. Because scBONITA hypothesizes that scRNA-seq data samples the state space of a dynamic Boolean network, this simulation replaces the need for time-series data.

## Related tools

- **scBONITA** (Performs the attractor analysis: simulates the learned Boolean models and maps sequenced cells to characteristic signaling states) — https://github.com/Thakar-Lab/scBONITA
- **conda** (Provides the scBonita environment created from the provided yml/spec-file in which attractor analysis runs)
- **SLURM** (Execution environment scBONITA (including attractor analysis) is designed to run on for large scRNA-seq datasets)
- **GRAPHML** (Format of the pathway network topologies (KEGG or non-KEGG sources) whose Boolean rules drive the attractor simulation)

## Examples

```
source activate scBonita; jupyter notebook Attractor_Analysis_With_scBONITA.ipynb
```

## Evaluation signals

- cell_attractor_assignments.csv exists and contains exactly one attractor assignment row per sequenced cell (e.g., 9368 rows for the PBMC dataset)
- Every cell is assigned to a state that is a fixed point of the simulated Boolean network
- Each cell's assigned attractor has the minimum Hamming distance among all attractors for that cell's converged state
- Assigned attractors reproduce the pathway-based characterization of cellular states in CD8+ T cells and monocytes reported for the HIV/AS (GSE198339) cohort

## Limitations

- Attractor analysis is compute- and memory-intensive: the README notes RAM requirements exceed 10G for large scRNA-seq datasets and does not recommend running attractor analysis on a desktop computer; scBONITA is designed for SLURM systems
- Inference (and hence attractor assignment) is expected to improve when pure cell populations are sequenced rather than mixed PBMC subpopulations
- In lieu of time-series data, scBONITA assumes scRNA-seq data samples the state space of a dynamic Boolean network — an assumption that may not hold for all datasets

## Evidence

- [abstract] Attractor analysis with scBONITA facilitated the pathway-based characterization of cellular states in CD8+ T cells and monocytes: "Attractor analysis with scBONITA facilitated the pathway-based characterization of cellular states in CD8+ T cells and monocytes"
- [discussion] To map cells to distinct signaling modes of the pathways described above, we developed scBONITA's attractor analysis functionalities.: "To map cells to distinct signaling modes of the pathways described above, we developed scBONITA’s attractor analysis functionalities."
- [methods] Please refer to the tutorial "Attractor_Analysis_With_scBONITA.ipynb" for a demonstration of attractor analysis with scBONITA using a test dataset packaged with scBONITA: "Please refer to the tutorial "Attractor_Analysis_With_scBONITA.ipynb" for a demonstration of attractor analysis with scBONITA using a test dataset packaged with scBONITA"
- [readme] we hence don't recommend running rule inference and attractor analysis on a desktop computer.: "we hence don't recommend running rule inference and attractor analysis on a desktop computer."
