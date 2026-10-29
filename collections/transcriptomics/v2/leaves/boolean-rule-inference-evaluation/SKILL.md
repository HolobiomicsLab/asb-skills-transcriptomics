---
name: boolean-rule-inference-evaluation
description: Use when you need to test whether Boolean rule sets inferred from a scRNA-seq
  cell cluster are stable to the number of training cells, i.e. whether performance
  degrades below a critical training fraction (e.g. after downsampling to 1%).
when_to_use_negative:
- When the goal is pathway analysis or attractor/state mapping rather than assessing
  the robustness of rule inference itself — scBONITA-PA and attractor analysis are
  separate steps with separate skills
- When the training clusters are already very small or impure (mixed PBMC subpopulations);
  the article notes inference is expected to improve with pure cell populations, so
  downsampling from a small impure cluster may not yield interpretable ERS trends
- When only bulk RNA-seq data are available — this skill depends on scBONITA's scRNA-seq-specific
  rule inference
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scBONITA
  role: Runs the Boolean rule-inference pipeline (pipeline.py) on each downsampled/augmented
    training subset under a fixed pathway topology so ERS sizes are comparable
  repo: https://github.com/Thakar-Lab/scBONITA
- name: conda
  role: Provides the scBonita environment (created from spec-file.txt or yml) required
    to execute pipeline.py
- name: Python 3.6
  role: Runtime for pipeline.py and the downsampling/augmentation scripts generating
    training subsets
- name: KEGG
  role: Source of pathway topologies (e.g. hsa00010, hsa00020) held constant across
    all training fractions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:6cd32735d6da950d1b35f5a0725173fdfd38c9a8271ba4900caa3d924e568163
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# boolean-rule-inference-evaluation

## Summary

A robustness-evaluation skill for scBONITA Boolean rule inference: it quantifies how the size of the inferred ensemble of rules (ERS) changes as the scRNA-seq training set for a cell cluster is downsampled (1%, 50%) or augmented (200%) with cells from adjacent clusters, revealing the training fraction at which rule determination degrades.

## When to use

Use when you need to test whether Boolean rule sets inferred from a scRNA-seq cell cluster are stable to the number of training cells, i.e. whether performance degrades below a critical training fraction (e.g. after downsampling to 1%).

## When NOT to use

- When the goal is pathway analysis or attractor/state mapping rather than assessing the robustness of rule inference itself — scBONITA-PA and attractor analysis are separate steps with separate skills
- When the training clusters are already very small or impure (mixed PBMC subpopulations); the article notes inference is expected to improve with pure cell populations, so downsampling from a small impure cluster may not yield interpretable ERS trends
- When only bulk RNA-seq data are available — this skill depends on scBONITA's scRNA-seq-specific rule inference

## Inputs

- Pre-processed expression matrix for a single cell cluster (tab/comma-delimited, rows=genes with first column labeled 'Genes', columns=cells with first row of cell IDs, log2(TPM+1)-style values)
- Metadata/conditions file (rows=cells, 0/1 condition labels, cell IDs matching training data columns)
- Contrast file listing pairs of conditions to compare
- KEGG pathway IDs (e.g. hsa00010, hsa00020) or GRAPHML pathway topology files

## Outputs

- Ensemble-of-rules (ERS) size for each training fraction (1%, 50%, 100%, 200%)
- Tabulated comparison saved as ers_vs_training_fraction.csv
- Per-subset scBONITA rule-inference outputs (inferred Boolean rules per pathway node)

## How to apply

Load one pre-processed cell cluster from the scRNA-seq matrix (e.g. a cluster from the SCP256 Kazer et al. PBMC dataset, rows=genes, columns=cells, first column labeled 'Genes') and generate training subsets by downsampling to 1% and 50% of cells and augmenting to 200% with cells from a neighboring cluster. Run the scBONITA pipeline (pipeline.py, Python 3.6+, scBonita conda environment) separately on each subset with the SAME KEGG pathway topology so the learned Boolean rule sets are directly comparable across training fractions. For each training fraction (1%, 50%, 100%, 200%), compute the size of the inferred ensemble of rules (ERS) and tabulate ERS sizes against the fraction, saving as ers_vs_training_fraction.csv. Before modifying the pipeline, adjust the C code constants for the training-set size (line 7: NODE = number of nodes; line 8: CELL = number of cells sampled), then recompile with make. Judge success by confirming the ERS size drops markedly at 1% downsampling (the reported significant performance decline) while remaining stable at ≥50% and at 200% augmentation.

## Related tools

- **scBONITA** (Runs the Boolean rule-inference pipeline (pipeline.py) on each downsampled/augmented training subset under a fixed pathway topology so ERS sizes are comparable) — https://github.com/Thakar-Lab/scBONITA
- **conda** (Provides the scBonita environment (created from spec-file.txt or yml) required to execute pipeline.py)
- **Python 3.6** (Runtime for pipeline.py and the downsampling/augmentation scripts generating training subsets)
- **KEGG** (Source of pathway topologies (e.g. hsa00010, hsa00020) held constant across all training fractions)

## Examples

```
python3.6 pipeline.py --dataFile "example.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa cvThreshold None
```

## Evaluation signals

- ERS size decreases markedly at the 1% downsampling level relative to the full 100% cluster (reproduces the reported significant decline)
- ERS size shows no significant change between the 50%, 100%, and 200% training fractions (stability plateau at ≥50%)
- The same KEGG/GRAPHML topology was used for every training fraction, so rule sets are directly comparable
- The output table ers_vs_training_fraction.csv contains all four fractions (1%, 50%, 100%, 200%) with non-empty ERS entries
- C-code constants NODE (nodes) and CELL (cells sampled) were adjusted and the make file recompiled before running the largest augmented (200%) subset

## Limitations

- Rule determination degrades significantly when training data are reduced to 1% of the cluster, so ERS comparisons below ~50% training fraction are unreliable
- scBONITA was validated on mixed PBMC subpopulations rather than pure sorted populations; inference quality (and thus ERS behavior) is expected to improve with pure cell populations
- Rule inference and attractor analysis require >10G RAM for large scRNA-seq datasets and are not recommended on desktop machines; scBONITA is designed for SLURM systems
- Subpopulations must be pre-classified; this evaluation depends on the supplied cluster labels rather than scBONITA's own clustering

## Evidence

- [results] Rule determination showed a significant performance decline when data were downsampled to 1% of the original cluster, but no significant change once 50% of cells were used or when the dataset was augmented to 200%: [evidence span withheld — anchor status: not_found]
- [readme] README training data format: "scBONITA needs a training dataset in matrix-style format; this is usually a tab or comma-delimited file with columns as cells and rows as features."
- [readme] README C-code modification: "line 7: NODE is the number of nodes in the dataset (or larger) - line 8: CELL is the number of cells to be sampled from the dataset (or larger)"
- [readme] README KEGG download command: "It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference."
- [methods] Article methods environment requirements: "scBONITA requires Python 3.6 or newer."
