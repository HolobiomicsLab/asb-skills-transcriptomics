---
name: boolean-rule-inference-genetic-algorithm
description: Use when you have binarizable single-cell expression data and a pathway
  topology and need executable Boolean rules describing signal integration at each
  network node, with node importance/uncertainty scores.
when_to_use_negative:
- You only need differential expression or conventional ORA — the article shows ORA
  and DE methods were inefficacious at identifying coherent molecular processes, but
  if the goal is purely DE gene lists, rule inference is unnecessary and computationally
  heavy
- The input is bulk RNA-seq intended for the original BONITA method rather than single-cell
  data (scBONITA hypothesizes scRNA-seq samples the state space of a dynamic Boolean
  network)
- Running on a desktop without SLURM/distributed computing — the README warns large
  scRNA-seq datasets need in excess of 10G RAM and rule inference is not recommended
  on a desktop computer
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0797
tools:
- name: scBONITA
  role: 'Primary algorithm: genetic algorithm + node-wise local search inferring per-node
    Boolean rules, ERS, and uncertainty factors from scRNA-seq data over KEGG/GRAPHML
    topologies'
  repo: https://github.com/Thakar-Lab/scBONITA
- name: KEGG
  role: Source of pathway topologies (hsa00010, hsa00020) downloaded for rule inference
- name: SLURM
  role: HPC scheduler that runs the generated sbatch rule-inference jobs
- name: conda
  role: Environment setup for scBonita dependencies via provided yml/spec-file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:b3ab21d8eb18e11c30c65213816d184046ed75366a06a6446ad1a0f4475794d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - KEGG
  - Python
  - SLURM
  - conda
  - scBONITA
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# boolean-rule-inference-genetic-algorithm

## Summary

Infer sign-compatible canalyzing Boolean regulatory rules per node of a prior knowledge network (e.g., KEGG hsa00010/hsa00020 topologies) from single-cell RNA-seq data using scBONITA's genetic algorithm followed by node-wise local search, yielding a minimum-error equivalent rule set (ERS) with per-node uncertainty quantification.

## When to use

Use when you have binarizable single-cell expression data and a pathway topology and need executable Boolean rules describing signal integration at each network node, with node importance/uncertainty scores.

## When NOT to use

- You only need differential expression or conventional ORA — the article shows ORA and DE methods were inefficacious at identifying coherent molecular processes, but if the goal is purely DE gene lists, rule inference is unnecessary and computationally heavy
- The input is bulk RNA-seq intended for the original BONITA method rather than single-cell data (scBONITA hypothesizes scRNA-seq samples the state space of a dynamic Boolean network)
- Running on a desktop without SLURM/distributed computing — the README warns large scRNA-seq datasets need in excess of 10G RAM and rule inference is not recommended on a desktop computer

## Inputs

- Matrix-style scRNA-seq training data (tab or comma-delimited; rows = genes, columns = cells, log2(TPM+1)-like units, first column 'Genes', first row cell IDs)
- Metadata/conditions file (rows = cells, columns = binary 1/0 condition labels, cell IDs matching training data)
- Contrast file (each row a tab-separated pair of conditions from the metadata columns)
- KEGG pathway topologies (e.g., hsa00010, hsa00020) or GRAPHML-format networks from non-KEGG sources

## Outputs

- inferred_rules.txt containing sign-compatible canalyzing Boolean rules for every node of each pathway (the equivalent rule set, ERS)
- Per-node uncertainty factors quantifying rule-inference ambiguity
- Node importance scores ranking genes by impact on signaling
- SLURM sbatch scripts and rule-inference log files

## How to apply

Prepare a matrix-style training dataset (rows as genes/features, columns as cells, expression in log2(TPM+1)-like units, first column labeled 'Genes' or similar, first row holding cell IDs matching the metadata), a binary metadata/conditions file (1s and 0s, rows as cells, columns as conditions), and a contrast file pairing conditions to compare. Run `python3.6 pipeline.py` to set up the pipeline, downloading the target KEGG pathways (e.g., hsa00010, hsa00020) with `--getKEGGPathways 1` and `--organism hsa`; this generates SLURM sbatch scripts. Before that, edit the C sampling code (line 7: NODE = number of nodes, line 8: CELL = number of cells, e.g., 20000 x 10000) and compile with `make`, and create the scBonita conda environment from the provided spec-file. Submit the jobs to the SLURM queue; the genetic algorithm searches for sign-compatible canalyzing rules per node that reproduce observed single-cell expression states, then a node-wise local search refines them. Because multiple rule sets explain the data equally well, scBONITA returns an equivalent rule set (ERS) of discrete-state models per pathway in inferred_rules.txt, plus per-node uncertainty factors (e.g., 0.13 for RHOA in the monocyte leukocyte transendothelial migration network) to prioritize genes by signaling impact.

## Related tools

- **scBONITA** (Primary algorithm: genetic algorithm + node-wise local search inferring per-node Boolean rules, ERS, and uncertainty factors from scRNA-seq data over KEGG/GRAPHML topologies) — https://github.com/Thakar-Lab/scBONITA
- **KEGG** (Source of pathway topologies (hsa00010, hsa00020) downloaded for rule inference)
- **SLURM** (HPC scheduler that runs the generated sbatch rule-inference jobs)
- **conda** (Environment setup for scBonita dependencies via provided yml/spec-file)

## Examples

```
python3.6 pipeline.py --dataFile "example.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa cvThreshold None
```

## Evaluation signals

- Every node of each KEGG network has an inferred rule in inferred_rules.txt
- All inferred rules are sign-compatible and canalyzing
- SLURM log files show rule-inference jobs completed without errors (users should periodically check the slurm queue and log files)
- Per-node uncertainty factors are reported (e.g., 0.13 for RHOA in the monocyte leukocyte transendothelial migration network), enabling gene prioritization
- Multiple equivalent rule sets explain training data equally well — check the ERS rather than a single best rule set

## Limitations

- In lieu of time-series data, scBONITA assumes scRNA-seq data represents samples in the state space of a dynamic Boolean network — an assumption that may not hold for all datasets
- Multiple rule sets can explain the training data equally well, so results are returned as an equivalent rule set (ERS) rather than a unique solution; per-node uncertainty must be considered
- Inference is expected to improve when pure cell populations are sequenced; mixed PBMC subpopulations reduce specificity
- Pre-classified subpopulations were used rather than scBONITA's own clustering to reduce variability
- Large datasets require >10G RAM and a SLURM system; not recommended on desktop computers
- The C code must be recompiled with NODE/CELL constants matching the dataset dimensions

## Evidence

- [methods] genetic algorithm identifies a minimum-error rule set: [evidence span withheld — anchor status: not_found]
- [methods] ERS and uncertainty: [evidence span withheld — anchor status: not_found]
- [readme] workflow data format: "scBONITA needs a training dataset in matrix-style format; this is usually a tab or comma-delimited file with columns as cells and rows as features"
- [readme] KEGG download command: "It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference."
- [discussion] state-space assumption: "In lieu of using time-series data, scBONITA hypothesizes that scRNA-seq data represents samples in the state space of a dynamic Boolean network."
- [readme] C code dimensions: "Modify the C file to reflect the size of the training data set"
- [discussion] inference improves with pure populations: "We expect that scBONITA inference will improve when pure cell populations are sequenced."
