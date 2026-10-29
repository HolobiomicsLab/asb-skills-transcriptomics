---
name: ers-size-quantification
description: 'Use when testing the robustness of Boolean rule inference to training-set
  size: input is a pre-processed scRNA-seq cluster (genes x cells matrix), and the
  goal is to detect at which training fraction the ERS size degrades.'
when_to_use_negative:
- The training fraction is reduced to ~1% of the original cluster — rule determination
  shows a significant performance decline at this level, so results are not reliable
- The goal is pathway analysis or attractor analysis rather than assessing rule-inference
  robustness to training-set size
- Only network topology is available without scRNA-seq training data for the cluster
  — rule inference, and hence ERS quantification, cannot be performed
tools:
- name: scBONITA
  role: Infers Boolean regulatory rules (the ERS) for a fixed KEGG pathway topology
    from each downsampled/augmented training subset
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime required to execute pipeline.py for rule inference on each training
    fraction
- name: conda
  role: Provides the scBonita environment (from yml/spec-file) in which the rule-inference
    pipeline is run
- name: KEGG
  role: Source of the pathway topology (e.g., hsa00010, hsa00020) held fixed across
    training fractions for comparability
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:8c2b825803abb8fdec00e4738c6e76e4f703ecd8ae6b2d3dfb61ec69b0c24488
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - KEGG
  - Python
  - Python 3.6
  - conda
  - scBONITA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ers-size-quantification

## Summary

Quantifies how the size of a scRNA-seq training cluster affects scBONITA Boolean rule inference by tabulating the size of the inferred ensemble of rules (ERS) across downsampled (1%, 50%), full (100%), and augmented (200%) training fractions of the same cell cluster.

## When to use

Use when testing the robustness of Boolean rule inference to training-set size: input is a pre-processed scRNA-seq cluster (genes x cells matrix), and the goal is to detect at which training fraction the ERS size degrades.

## When NOT to use

- The training fraction is reduced to ~1% of the original cluster — rule determination shows a significant performance decline at this level, so results are not reliable
- The goal is pathway analysis or attractor analysis rather than assessing rule-inference robustness to training-set size
- Only network topology is available without scRNA-seq training data for the cluster — rule inference, and hence ERS quantification, cannot be performed

## Inputs

- Pre-processed scRNA-seq expression matrix for one cell cluster (tab/comma-delimited, genes as rows, cells as columns, first column labeled 'Genes', expression as log2(TPM+1)-style values)
- Metadata/conditions matrix (cells as rows, cell labels as 1/0 columns) with matching cell IDs
- Neighboring cell cluster expression data for augmentation to 200%

## Outputs

- ERS (ensemble of rules) size per training fraction
- ers_vs_training_fraction.csv table with ERS sizes at 1%, 50%, 100%, and 200% training fractions

## How to apply

Load the pre-processed expression matrix for one cell cluster from the SCP256 (Kazer et al.) PBMC scRNA-seq dataset in matrix-style format (rows as genes, columns as cells, first column labeled 'Genes'). Generate training subsets by downsampling the cluster to 1% and 50% of its cells and augmenting to 200% by adding cells from a neighboring cluster; per the README, when changing the training size you must also modify the C file (NODE = number of nodes, CELL = number of cells to be sampled) and recompile with 'make'. Run the scBONITA rule-inference pipeline (pipeline.py, Python 3.6+, scBonita conda environment) separately on each subset using the same KEGG pathway topology so the learned rule sets are comparable. Compute the size of the ensemble of rules (ERS) inferred for each fraction, tabulate ERS sizes against training fraction (1%, 50%, 100%, 200%), and save the table as ers_vs_training_fraction.csv.

## Related tools

- **scBONITA** (Infers Boolean regulatory rules (the ERS) for a fixed KEGG pathway topology from each downsampled/augmented training subset) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime required to execute pipeline.py for rule inference on each training fraction)
- **conda** (Provides the scBonita environment (from yml/spec-file) in which the rule-inference pipeline is run)
- **KEGG** (Source of the pathway topology (e.g., hsa00010, hsa00020) held fixed across training fractions for comparability)

## Examples

```
python3.6 pipeline.py --dataFile "cluster_subset.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa
```

## Evaluation signals

- ERS size at the 1% downsampling level decreases markedly relative to the full (100%) cluster, reproducing the reported significant performance decline
- ERS sizes at 50% and 200% training fractions show no significant change relative to the 100% cluster
- The same KEGG pathway topology was used for all fractions, so ERS differences reflect training size and not topology changes
- ers_vs_training_fraction.csv contains one row per fraction (1%, 50%, 100%, 200%) with a computed ERS size for each
- The C file NODE and CELL parameters were updated to match each training subset size and recompiled with 'make' before each run

## Limitations

- Rule determination degrades significantly when training data are downsampled to 1% of the original cluster
- Rule inference and attractor analysis for large scRNA-seq datasets need >10G RAM and are not recommended on a desktop; the pipeline is designed for SLURM systems
- scBONITA inference is expected to improve when pure cell populations are sequenced; mixed PBMC subpopulations limit inference quality
- The C source must be manually adjusted (NODE, CELL) and recompiled with 'make' whenever the training-set size changes

## Evidence

- [readme] scBONITA needs a training dataset in matrix-style format; this is usually a tab or comma-delimited file with columns as cells and rows as features.: "scBONITA needs a training dataset in matrix-style format; this is usually a tab or comma-delimited file with columns as cells and rows as features."
- [readme] CELL is the number of cells to be sampled from the dataset (or larger): "line 8: CELL is the number of cells to be sampled from the dataset (or larger)"
- [methods] scBONITA requires Python 3.6 or newer.: "scBONITA requires Python 3.6 or newer."
- [intro] we used a publicly available dataset of PBMCs from persons before and after HIV infection: "we used a publicly available dataset of PBMCs from persons before and after HIV infection"
