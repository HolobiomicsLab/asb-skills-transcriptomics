---
name: barretts-esophagus-biopsy-heterogeneity-assessment
description: Use when evaluating MaCroDNA cell-to-cell assignments on BE biopsy scDNA-seq/scRNA-seq
  data in the absence of ground truth, to test statistical significance and quantify
  assignment stability across resampled runs.
when_to_use_negative:
- Ground-truth cell correspondence is available (e.g., the CRC scTrio-seq dataset
  with partially known ground truth) — direct assignment-accuracy evaluation is then
  the right approach instead of resampling/stability proxies
- You need probabilistic confidence measures per assignment — MaCroDNA does not provide
  these (unlike clonealign); only the 'step' column of cell2cell_step offers an ordinal
  confidence ranking
- The sample is biologically homogeneous (e.g., healthy/NDBE) and the goal is confident
  cell-to-cell matching — the article notes matching cells is distinctively challenging
  in homogeneous samples
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: MaCroDNA
  role: Performs cell-to-cell assignment between scRNA-seq and scDNA-seq cells via
    maximum weighted bipartite matching of per-gene read counts, iteratively refined
    (cell2cell, cell2cell_step outputs); the core method whose assignments are being
    tested for significance and stability
  repo: github.com/NakhlehLab/MaCroDNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:e922e09b28f59176bceb489435427348fc19afbb7c21fa6d85cdaf6611f40adf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# barretts-esophagus-biopsy-heterogeneity-assessment

## Summary

Assess the reliability of MaCroDNA cell assignments on Barrett's esophagus (BE) biopsies — where no ground-truth scRNA-seq to scDNA-seq correspondence exists — via random-assignment null tests and resampling-based stability analyses, and relate stability to biopsy heterogeneity (homogeneous healthy/NDBE vs heterogeneous HGD/EAC).

## When to use

Use when evaluating MaCroDNA cell-to-cell assignments on BE biopsy scDNA-seq/scRNA-seq data in the absence of ground truth, to test statistical significance and quantify assignment stability across resampled runs.

## When NOT to use

- Ground-truth cell correspondence is available (e.g., the CRC scTrio-seq dataset with partially known ground truth) — direct assignment-accuracy evaluation is then the right approach instead of resampling/stability proxies
- You need probabilistic confidence measures per assignment — MaCroDNA does not provide these (unlike clonealign); only the 'step' column of cell2cell_step offers an ordinal confidence ranking
- The sample is biologically homogeneous (e.g., healthy/NDBE) and the goal is confident cell-to-cell matching — the article notes matching cells is distinctively challenging in homogeneous samples

## Inputs

- BE scDNA-seq copy-number matrix as dna_df (genes as rows, cells as columns, pandas dataframe)
- BE scRNA-seq gene expression matrix as rna_df (genes as rows, cells as columns, pandas dataframe)
- Per-biopsy sample labels classifying biopsies as homogeneous (healthy/NDBE) or heterogeneous (HGD/EAC)

## Outputs

- Per-biopsy significance results comparing observed matching objectives against random-assignment null distributions
- Per-assignment stability and definiteness metrics across resampled MaCroDNA runs
- Combined results table saved as be_stability_results.csv

## How to apply

Load the BE scDNA-seq copy-number matrix and scRNA-seq expression matrix as dna_df and rna_df dataframes with genes as rows and cells as columns (processed with pandas). For each biopsy, run cell2cell_assignment via maximum weighted bipartite matching of per-gene read counts, and use the cell2cell_step output where a smaller 'step' value indicates a more confident assignment. Build a null distribution by randomly permuting RNA-to-DNA assignments and compare the observed matching objective against this null to establish statistical significance per biopsy. Subsample (resample) cells from each biopsy, repeat the assignment, and aggregate stability and definiteness metrics per biopsy, grouping biopsies into homogeneous healthy/NDBE versus heterogeneous HGD/EAC. Validation succeeds if homogeneous healthy/NDBE biopsies show lower stability than heterogeneous HGD/EAC biopsies and observed assignment scores exceed the random-assignment null.

## Related tools

- **MaCroDNA** (Performs cell-to-cell assignment between scRNA-seq and scDNA-seq cells via maximum weighted bipartite matching of per-gene read counts, iteratively refined (cell2cell, cell2cell_step outputs); the core method whose assignments are being tested for significance and stability) — github.com/NakhlehLab/MaCroDNA

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- Observed matching objective for each biopsy exceeds the random-assignment null distribution, giving statistical significance for that biopsy's assignments
- Resampling-based stability metrics for homogeneous healthy/NDBE biopsies are lower than those for heterogeneous HGD/EAC biopsies, matching the paper's reported finding
- cell2cell_step output is present and every assignment has a valid 'step' value, with smaller steps denoting more confident assignments
- be_stability_results.csv contains aggregated per-biopsy stability, definiteness, and significance results with biopsies correctly grouped into healthy/NDBE vs HGD/EAC

## Limitations

- No ground truth information about the correspondence between cells exists for the BE dataset, so reliability must be inferred indirectly via resampling and stability analyses
- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign
- Matching cells is distinctively challenging in homogeneous samples, a biologically driven issue that may arise when utilizing MaCroDNA in such cases

## Evidence

- [discussion] Resampling and stability analyses on the BE dataset: "we performed comprehensive resampling and stability analyses"
- [discussion] MaCroDNA lacks probabilistic confidence measures: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
- [discussion] Homogeneous samples are challenging for matching: "This suggests that matching cells is distinctively challenging in homogeneous samples and highlights a biologically driven issue that may arise"
- [readme] cell2cell assignment API and confidence step semantics: "cell2cell, cell2cell_step = macrodna.cell2cell_assignment()"
- [readme] Step column encodes assignment confidence: ""step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is."
- [readme] Input data orientation for MaCroDNA: "the input should be two dataframes `dna_df` and `rna_df`. In these two dataframes, the row ids should be the genes and the column ids should be the cells."
