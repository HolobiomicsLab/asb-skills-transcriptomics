---
name: single-cell-rna-seq-count-matrix-simulation
description: Use when you need synthetic scRNA-seq count matrices (e.g., data1-data5
  scenarios with defined zero-inflation levels) to benchmark dimensionality-reduction
  or clustering methods before running them on real data.
when_to_use_negative:
- Your input is already real scRNA-seq count data (e.g., the HSPCs HRA000084 or pancreatic
  GSE84133 datasets) — preprocessing with scTransform/Seurat is the relevant step,
  not simulation
- You need simulation of single-cell multi-omic modalities beyond gene-expression
  count matrices — Splatter/PROSSTT as configured here simulate counts only
- The target pipeline requires more than 2000 cells or 5000 genes — the parameters
  specified here are fixed to the scGAE benchmark dimensions
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: R package used to generate simulated scRNA-seq scenarios data1, data3, and
    data4 (2000 cells, 5000 genes) and the zero-inflation variants via dropout.shape=-1
    and dropout.mid
- name: PROSSTT
  role: Python simulator used to generate scenarios data2 and data5 (2000 cells, 5000
    genes)
- name: scGAE
  role: Downstream benchmark target whose Run_scGAE.ipynb notebook documents where
    the simulation code is found; simulated matrices serve as its input
  repo: github.com/ZixiangLuo1161/scGAE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:f0f120b272c0ac9dfbd136b3be1a821dc53268904742ee6852ddfcd87ece5fc3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PROSSTT
  - Splatter
  - scGAE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-rna-seq-count-matrix-simulation

## Summary

Generate simulated single-cell RNA-seq count matrices with Splatter (R) and PROSSTT (Python) at a fixed 2000-cell by 5000-gene dimension, including controlled zero-inflation variants, to benchmark methods such as scGAE under different simulated scenarios.

## When to use

Use when you need synthetic scRNA-seq count matrices (e.g., data1-data5 scenarios with defined zero-inflation levels) to benchmark dimensionality-reduction or clustering methods before running them on real data.

## When NOT to use

- Your input is already real scRNA-seq count data (e.g., the HSPCs HRA000084 or pancreatic GSE84133 datasets) — preprocessing with scTransform/Seurat is the relevant step, not simulation
- You need simulation of single-cell multi-omic modalities beyond gene-expression count matrices — Splatter/PROSSTT as configured here simulate counts only
- The target pipeline requires more than 2000 cells or 5000 genes — the parameters specified here are fixed to the scGAE benchmark dimensions

## Inputs

- Simulation parameter scripts from the Run_scGAE.ipynb notebook in the scGAE repository
- Splatter R package simulation parameters (nCells=2000, nGenes=5000, dropout.shape=-1, dropout.mid in {-0.5, 0, 0.5, 1} for zero-inflation variants)
- PROSSTT (Python) simulation parameters (2000 cells, 5000 genes)

## Outputs

- Nine simulated count matrices as CSV or h5ad files: data1, data2, data3, data4, data5, and four scenario4 zero-inflation variants (~12%, 17%, 23%, 30% dropout)
- Each validated matrix containing exactly 2000 rows (cells) and 5000 columns (genes)

## How to apply

Load the simulation parameter scripts referenced by the repository notebook Run_scGAE.ipynb, which documents where the simulation code lives. Generate data1, data3, and data4 with the Splatter R package and data2 and data5 with PROSSTT (Python), each simulated with 2000 cells and 5000 genes. For the scenario4 zero-inflation variants, apply dropout with shape=-1 and dropout.mid set to -0.5, 0, 0.5, and 1, which yields approximately 12%, 17%, 23%, and 30% dropout respectively. Write each of the nine count matrices (data1-data5 plus the four scenario4 variants) to disk as CSV or h5ad files. Validate every output by reloading it and confirming it is a valid count matrix with exactly 2000 rows (cells) and 5000 columns (genes).

## Related tools

- **Splatter** (R package used to generate simulated scRNA-seq scenarios data1, data3, and data4 (2000 cells, 5000 genes) and the zero-inflation variants via dropout.shape=-1 and dropout.mid)
- **PROSSTT** (Python simulator used to generate scenarios data2 and data5 (2000 cells, 5000 genes))
- **scGAE** (Downstream benchmark target whose Run_scGAE.ipynb notebook documents where the simulation code is found; simulated matrices serve as its input) — github.com/ZixiangLuo1161/scGAE

## Examples

```
library(splatter); counts <- splatSimulate(groupCells=2000, nGenes=5000, dropout.shape=-1, dropout.mid=0.5, method='groups')
```

## Evaluation signals

- Every output file loads as a valid CSV or h5ad count matrix
- Each matrix has exactly 2000 rows (cells) and 5000 columns (genes)
- Nine distinct matrices are produced: data1-data5 plus four scenario4 zero-inflation variants
- The scenario4 variants exhibit dropout rates of approximately 12%, 17%, 23%, and 30%, corresponding to dropout.mid values -0.5, 0, 0.5, and 1 with dropout.shape=-1
- data1/data3/data4 are traceable to Splatter and data2/data5 to PROSSTT in the run scripts

## Limitations

- The optimal parameters (and here, even dataset dimensions) are fixed to the scGAE benchmark; the paper notes that 'the optimal K varies among different datasets' and that no random seed or stochastic-variation control was reported for the study
- No software/package versions are specified for the simulation tools themselves in the paper, so exact reproducibility may require the repository scripts
- Only gene-expression count matrices are simulated; the four zero-inflation variants are the only controlled noise axis (dropout 12-30%)

## Evidence

- [other] Generate data1, data3, and data4 with the Splatter R package, each simulated with 2000 cells and 5000 genes.: [evidence span withheld — anchor status: not_found]
- [other] Generate data2 and data5 with PROSSTT (Python), likewise with 2000 cells and 5000 genes.: [evidence span withheld — anchor status: not_found]
- [other] dropout.shape=-1 and dropout.mid in (-0.5, 0, 0.5, 1), producing dropout rates of 12%, 17%, 23%, and 30%: [evidence span withheld — anchor status: not_found]
- [readme] Running example and parameter setting can be found at Run_scGAE.ipynb: [evidence span withheld — anchor status: external_doc]
- [readme] scGAE takes preprocessed data as input.: "scGAE takes preprocessed data as input. Single cell data preprocessing can be done with Seurat R package and scanpy python package."
