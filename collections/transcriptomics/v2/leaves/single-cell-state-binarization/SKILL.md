---
name: single-cell-state-binarization
description: Use when scRNA-seq data (log2(TPM+1)-style, cells as columns) must be
  converted into Boolean states for simulating a prior-knowledge signaling network
  and mapping each sequenced cell to a characteristic pathway attractor.
when_to_use_negative:
- The analysis goal is differential expression or conventional over-representation
  analysis — the article notes conventional ORA was inefficacious at identifying coherent
  molecular processes here, but attractor/binarized-state modeling is not the tool
  for DE testing itself
- Input lacks a prior-knowledge network topology (KEGG or GRAPHML) — Boolean rules
  cannot be inferred without the network
- Data is bulk RNA-seq with no per-cell structure — use the original bulk BONITA method
  instead of scBONITA's single-cell state mapping
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3308
tools:
- name: scBONITA
  role: Performs the Boolean rule inference (ERS), network simulation to attractors,
    and Hamming-distance attractor assignment of binarized cell states
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime for pipeline.py, pathwayAnalysis.py, and attractor analysis scripts
    (requires Python 3.6 or newer)
  repo: https://github.com/Thakar-Lab/scBONITA
- name: conda
  role: Environment management via the provided spec-file/yml to install scBONITA
    dependencies
- name: SLURM
  role: HPC job scheduler on which scBONITA's generated sbatch rule-inference jobs
    are run
- name: GRAPHML
  role: Format for non-KEGG pathway topologies supplied via --pathwayList
- name: KEGG
  role: Source of pathway topologies (e.g., hsa00010, hsa00020) downloaded via --getKEGGPathways
    for rule inference
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:761db8c1e7c068c9fc0a1a2225d4c7fa03f1aac7c84ce51d1093dd1a86d19a89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - GRAPHML
  - KEGG
  - Python
  - Python 3.6
  - SLURM
  - conda
  - scBONITA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-state-binarization

## Summary

Binarize single-cell RNA-seq expression into discrete Boolean cell states so they can be sampled as initial conditions of scBONITA's learned Boolean networks, then converged to steady-state attractors with per-cell assignment via minimum Hamming distance.

## When to use

Use when scRNA-seq data (log2(TPM+1)-style, cells as columns) must be converted into Boolean states for simulating a prior-knowledge signaling network and mapping each sequenced cell to a characteristic pathway attractor.

## When NOT to use

- The analysis goal is differential expression or conventional over-representation analysis — the article notes conventional ORA was inefficacious at identifying coherent molecular processes here, but attractor/binarized-state modeling is not the tool for DE testing itself
- Input lacks a prior-knowledge network topology (KEGG or GRAPHML) — Boolean rules cannot be inferred without the network
- Data is bulk RNA-seq with no per-cell structure — use the original bulk BONITA method instead of scBONITA's single-cell state mapping

## Inputs

- Training scRNA-seq expression matrix (matrix-style, tab or comma-delimited; rows = features/genes, columns = cells; log2(TPM+1)-like units; first column 'Genes', first row cell IDs)
- Cell metadata/conditions file (rows = cells, columns = 0/1 condition labels, cell IDs matching the training matrix)
- Contrast file (pairwise condition comparisons, tab-separated)
- scBONITA-inferred Boolean rules (ERS) per pathway node (e.g., inferred_rules.txt)
- Pathway topology files (KEGG pathways such as hsa00010/hsa00020, or non-KEGG networks in GRAPHML format)

## Outputs

- Binarized single-cell expression states per network node (Boolean initial conditions)
- Steady-state attractors (fixed points) of the simulated Boolean network per pathway
- Per-cell attractor assignments written to cell_attractor_assignments.csv (one row per sequenced cell)

## How to apply

Load the scBONITA training matrix (features as rows, cells as columns, expression in log2(TPM+1)-like units, first column labeled 'Genes', first row cell IDs matching the metadata file) and binarize the expression of each network node into on/off states consistent with the KEGG or GRAPHML pathway topology. Sample alternative rule sets from the equivalent rule set (ERS) inferred by scBONITA's genetic-algorithm plus node-wise local search stage. Synchronously update the Boolean network starting from each binarized cell's state and iterate until a steady-state attractor (fixed point) is reached. Compute the Hamming distance between each cell's converged state and each attractor, and assign the cell to its closest attractor to characterize pathway-specific signaling states (e.g., in CD8+ T cells and monocytes). This works because scBONITA hypothesizes that scRNA-seq data samples the state space of a dynamic Boolean network in lieu of time-series data.

## Related tools

- **scBONITA** (Performs the Boolean rule inference (ERS), network simulation to attractors, and Hamming-distance attractor assignment of binarized cell states) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime for pipeline.py, pathwayAnalysis.py, and attractor analysis scripts (requires Python 3.6 or newer)) — https://github.com/Thakar-Lab/scBONITA
- **conda** (Environment management via the provided spec-file/yml to install scBONITA dependencies)
- **SLURM** (HPC job scheduler on which scBONITA's generated sbatch rule-inference jobs are run)
- **GRAPHML** (Format for non-KEGG pathway topologies supplied via --pathwayList)
- **KEGG** (Source of pathway topologies (e.g., hsa00010, hsa00020) downloaded via --getKEGGPathways for rule inference)

## Examples

```
python3.6 pipeline.py --dataFile "example.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa
```

## Evaluation signals

- cell_attractor_assignments.csv exists and contains exactly one attractor assignment row per sequenced cell
- Every assigned attractor is a verified fixed point of the simulated Boolean network (converged state is stable under synchronous update)
- Every node of each KEGG network has an inferred rule in inferred_rules.txt and all rules are sign-compatible and canalyzing
- Binarized cell IDs match the training matrix columns and the metadata file cell IDs
- SLURM log files show rule-inference jobs completed without errors before attractor simulation is run

## Limitations

- Rule inference and attractor analysis are not recommended on a desktop computer; large scRNA-seq datasets need in excess of 10G RAM, so a SLURM/HPC system is expected
- Per-node rule uncertainty exists because multiple rule sets can explain the training data equally well (ERS; e.g., uncertainty factor 0.13 for RHOA in the monocyte leukocyte transendothelial migration network)
- scBONITA inference is expected to improve when pure cell populations are sequenced, since current PBMC data are mixed subpopulations; time-series data is rarely available so the method assumes scRNA-seq samples the Boolean network's state space

## Evidence

- [abstract] Attractor analysis with scBONITA facilitated the pathway-based characterization of cellular states in CD8+ T cells and monocytes: "Attractor analysis with scBONITA facilitated the pathway-based characterization of cellular states in CD8+ T cells and monocytes"
- [discussion] the learned discrete-state models are simulated so trajectories fall into steady states (attractors), and each sequenced cell is assigned to the attractor closest to its expression: "To map cells to distinct signaling modes of the pathways described above, we developed scBONITA’s attractor analysis functionalities."
- [discussion] scBONITA hypothesizes that scRNA-seq data represents samples in the state space of a dynamic Boolean network: "In lieu of using time-series data, scBONITA hypothesizes that scRNA-seq data represents samples in the state space of a dynamic Boolean network."
- [readme] training dataset in matrix-style format; columns as cells and rows as features; log2(TPM+1)-like units: "scBONITA needs a training dataset in matrix-style format; this is usually a tab or comma-delimited file with columns as cells and rows as features."
- [readme] the amount of RAM required for large scRNA-seq datasets is in excess of 10G: "we find that the amount of RAM required for large scRNA-seq datasets is in excess of 10G; we hence don't recommend running rule inference and attractor analysis on a desktop computer."
