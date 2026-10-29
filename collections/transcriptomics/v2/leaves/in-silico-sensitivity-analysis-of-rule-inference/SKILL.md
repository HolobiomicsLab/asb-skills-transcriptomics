---
name: in-silico-sensitivity-analysis-of-rule-inference
description: Use when you need to determine whether scBONITA Boolean rule determination
  is robust to the size of a cluster's scRNA-seq training data, or to choose how many
  cells a training subset must contain before inference degrades.
when_to_use_negative:
- The goal is pathway analysis (scBONITA-PA) or attractor analysis rather than assessing
  the robustness of the rule-inference step itself
- The input data are bulk RNA-seq; the bulk-oriented BONITA method, not this scRNA-seq
  downsampling sensitivity workflow, applies
- The analysis machine lacks >10G RAM, since the README warns rule inference and attractor
  analysis are not recommended on a desktop for large scRNA-seq datasets
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scBONITA
  role: Rule-inference pipeline (pipeline.py) re-run per training subset to infer
    Boolean rule sets from the same pathway topology
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime requirement for pipeline.py and the downsampling/augmentation scripts
    and ERS tabulation
- name: conda
  role: Provides the scBonita conda environment (created from spec-file.txt or yml)
    in which the pipeline is executed
- name: KEGG
  role: Source of pathway topologies (e.g., hsa00010, hsa00020) held fixed across
    training fractions so learned rule sets are comparable
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:3cadfd6a4b923a33af35f4614138cf8d805b989707dd3da2777de16f80d4f438
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# in-silico-sensitivity-analysis-of-rule-inference

## Summary

An in-silico robustness check that quantifies how the size of the ensemble of rules (ERS) inferred by scBONITA depends on the size of the scRNA-seq training data for a cell cluster, by re-running rule inference on downsampled (1%, 50%) and augmented (200%) training subsets against the same pathway topology.

## When to use

Use when you need to determine whether scBONITA Boolean rule determination is robust to the size of a cluster's scRNA-seq training data, or to choose how many cells a training subset must contain before inference degrades.

## When NOT to use

- The goal is pathway analysis (scBONITA-PA) or attractor analysis rather than assessing the robustness of the rule-inference step itself
- The input data are bulk RNA-seq; the bulk-oriented BONITA method, not this scRNA-seq downsampling sensitivity workflow, applies
- The analysis machine lacks >10G RAM, since the README warns rule inference and attractor analysis are not recommended on a desktop for large scRNA-seq datasets

## Inputs

- Pre-processed expression matrix for one cell cluster in matrix-style tab/comma-delimited format (rows = genes, columns = cells, first column labeled 'Genes', e.g. log2(TPM+1)), from the SCP256 (Kazer et al.) PBMC scRNA-seq dataset
- Cells from a neighboring cluster for augmentation to 200% training size
- Pathway topology files used for rule inference (KEGG pathways such as hsa00010, hsa00020 downloaded by the pipeline, or GRAPHML-format networks)
- Compiled scBONITA C sampler configured with NODE and CELL set to the training-data dimensions

## Outputs

- Ensemble of rules (ERS) inferred for each training fraction (1%, 50%, 100%, 200%)
- Table of ERS sizes versus training fraction saved as ers_vs_training_fraction.csv

## How to apply

Load the pre-processed expression matrix for one selected cell cluster from the SCP256 (Kazer et al.) PBMC scRNA-seq dataset in matrix-style format (rows as genes, columns as cells, first column labeled 'Genes'). Generate training subsets by downsampling the cluster to 1% and 50% of its cells and augmenting to 200% by adding cells from a neighboring cluster; because scBONITA's C sampler must know the data dimensions, modify NODE (number of nodes) and CELL (number of cells to be sampled) in the C file and recompile with make before running. Run the scBONITA rule-inference pipeline (pipeline.py, Python 3.6+, scBonita conda environment) separately on each subset using the same pathway topology (e.g., the same KEGG GRAPHML pathways such as hsa00010, hsa00020) so learned rule sets are comparable. Compute the ERS size for each training fraction, tabulate ERS sizes against the training fraction (1%, 50%, 100%, 200%), and save as ers_vs_training_fraction.csv. The rationale is that ERS size is a direct proxy for inference stability: a collapsed ERS at a training fraction indicates the rule-determination performance decline reported at 1% downsampling.

## Related tools

- **scBONITA** (Rule-inference pipeline (pipeline.py) re-run per training subset to infer Boolean rule sets from the same pathway topology) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime requirement for pipeline.py and the downsampling/augmentation scripts and ERS tabulation)
- **conda** (Provides the scBonita conda environment (created from spec-file.txt or yml) in which the pipeline is executed)
- **KEGG** (Source of pathway topologies (e.g., hsa00010, hsa00020) held fixed across training fractions so learned rule sets are comparable)

## Examples

```
python3.6 pipeline.py --dataFile "example.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa
```

## Evaluation signals

- ERS size decreases markedly at the 1% downsampling level relative to the full cluster, reproducing the reported significant performance decline at 1% training fraction
- ERS size shows no significant change once 50% of the cluster's cells are used for training
- Augmenting the training data to 200% with cells from an adjacent cluster does not significantly change the inferred ERS size
- The same pathway topology (identical KEGG/GRAPHML networks) was used for rule inference across all training fractions, so ERS sizes are comparable
- The scBonita conda environment is active and the C sampler was recompiled (NODE/CELL) to accommodate each subset's dimensions before each pipeline run

## Limitations

- Rule determination performance declines significantly when training data are downsampled to 1% of the original cluster, so ERS results below ~50% training fraction are unreliable
- The C sampler's NODE and CELL constants must be manually updated and recompiled for each dataset size, which is error-prone if the training subset changes
- scBONITA is designed for SLURM systems and large scRNA-seq datasets need in excess of 10G RAM; rule inference is not recommended on a desktop computer
- Inference is expected to improve when pure cell populations are sequenced; using mixed PBMC subpopulations (as in the Kazer et al. data) limits achievable robustness

## Evidence

- [readme] Modify the C file to reflect the size of the training data set: "Modify the C file to reflect the size of the training data set"
- [readme] scBONITA needs a training dataset in matrix-style format; this is usually a tab or comma-delimited file with columns as cells and rows as features: "scBONITA needs a training dataset in matrix-style format; this is usually a tab or comma-delimited file with columns as cells and rows as features"
- [methods] scBONITA requires Python 3.6 or newer: "scBONITA requires Python 3.6 or newer."
- [discussion] Although the BONITA algorithm has been rigorously validated in our prior publication, here we wanted to evaluate the scRNA-seq-specific parts of the algorithm: "here we wanted to evaluate the scRNA-seq-specific parts of the algorithm"
- [methods] It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference: "It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference."
