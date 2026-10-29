---
name: ligand-receptor-pair-filtering
description: Use when building a multi-method or multi-context cell-cell communication
  tensor and you must restrict inputs to LR pairs and cell types shared across all
  samples, e.g. the 172 shared LR pairs common to all five scoring methods on the
  BALF COVID-19 data.
when_to_use_negative:
- The analysis targets a single context/sample, where per-tool output already defines
  the LR space and no cross-method intersection is needed
- The tensor decomposition implementation supports missing values, so dropping LR
  pairs or cell types absent in some contexts is unnecessary (the implemented Tensor-cell2cell
  algorithm is not optimized for missing values, but future variants may be)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tensor-cell2cell
  role: Consumes the filtered shared LR-pair and cell-type space to build and deconvolve
    the 4D communication tensor
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Provides cell2cell.tensor for assembling the filtered LR pairs and cell types
    into the 4D tensor and cell2cell.preprocessing for per-sample preprocessing
  repo: https://github.com/earmingol/cell2cell
- name: LIANA
  role: Runs CellChat, CellPhoneDB, NATMI, and SingleCellSignalR per sample so their
    LR outputs can be intersected
  repo: https://github.com/saezlab/liana-py
- name: CellChat
  role: Source of the Human-2020-Jin curated LR database used to define the candidate
    pair universe
  repo: https://github.com/sqjin/CellChat
- name: Ligand-Receptor Pairs compendium
  role: Curated lists of literature-supported LR pairs, including the Jin et al. 2021
    Human list
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:6d7fc75700b8b23fb51096c7626695e02f30cad03f5a5527413fecfe659c23ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - LIANA
  - Ligand-Receptor Pairs compendium
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ligand-receptor-pair-filtering

## Summary

Filter and harmonize ligand-receptor (LR) pair inventories so that communication tensors built from multiple scoring methods (CellChat, CellPhoneDB, NATMI, SingleCellSignalR, Tensor-cell2cell built-in) share a common LR and cell-type space before tensor decomposition.

## When to use

Use when building a multi-method or multi-context cell-cell communication tensor and you must restrict inputs to LR pairs and cell types shared across all samples, e.g. the 172 shared LR pairs common to all five scoring methods on the BALF COVID-19 data.

## When NOT to use

- The analysis targets a single context/sample, where per-tool output already defines the LR space and no cross-method intersection is needed
- The tensor decomposition implementation supports missing values, so dropping LR pairs or cell types absent in some contexts is unnecessary (the implemented Tensor-cell2cell algorithm is not optimized for missing values, but future variants may be)

## Inputs

- Per-sample communication scores from CellChat, CellPhoneDB, NATMI, SingleCellSignalR, and Tensor-cell2cell built-in rank-based scoring (via LIANA)
- Curated LR pair list (Human-2020-Jin-LR-pairs.csv from LewisLabUCSD/Ligand-Receptor-Pairs)
- BALF COVID-19 single-cell RNA-seq dataset (63k immune and epithelial cells; 12 samples)

## Outputs

- Intersection of LR pairs shared across all methods and samples (172 shared LR pairs in the reference workflow)
- 4D communication tensor (LR pairs × sender cell types × receiver cell types × contexts) built with cell2cell.tensor from the filtered subset

## How to apply

Start from a curated LR list appropriate to the study — here the Human-2020-Jin LR pairs published with CellChat (Jin et al. 2021, Nature Communications), available from the LewisLabUCSD/Ligand-Receptor-Pairs compendium. Run each communication scoring method per sample (12 BALF samples: 3 control, 3 moderate, 6 severe), then intersect the resulting LR pair lists across methods and samples, retaining only the LR pairs present in every context (172 pairs in the reference workflow). Analogously, restrict the cell-type dimension to cell types shared across all samples, because the Tensor-cell2cell algorithm cannot handle missing tensor entries. Build the 4D tensor (LR pairs × sender cells × receiver cells × contexts) from this filtered subset using cell2cell.tensor. The rationale is comparability: decomposition consistency metrics such as 1-CorrIndex are only meaningful when all factorizations operate on identical dimensions.

## Related tools

- **Tensor-cell2cell** (Consumes the filtered shared LR-pair and cell-type space to build and deconvolve the 4D communication tensor) — https://github.com/earmingol/cell2cell
- **cell2cell** (Provides cell2cell.tensor for assembling the filtered LR pairs and cell types into the 4D tensor and cell2cell.preprocessing for per-sample preprocessing) — https://github.com/earmingol/cell2cell
- **LIANA** (Runs CellChat, CellPhoneDB, NATMI, and SingleCellSignalR per sample so their LR outputs can be intersected) — https://github.com/saezlab/liana-py
- **CellChat** (Source of the Human-2020-Jin curated LR database used to define the candidate pair universe) — https://github.com/sqjin/CellChat
- **Ligand-Receptor Pairs compendium** (Curated lists of literature-supported LR pairs, including the Jin et al. 2021 Human list) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
import cell2cell; tensor = cell2cell.tensor.BaseTensor(); tensor.interaction_subset = sorted(set(chat_lrs) & set(ccdb_lrs) & set(natmi_lrs) & set(scsr_lrs) & set(c2c_lrs))  # 172 shared LR pairs
```

## Evaluation signals

- The intersection retains exactly the LR pairs present in every method and every sample (172 pairs for the BALF five-method workflow); recompute set intersections independently to verify
- The cell-type dimension contains only cell types observed across all 12 BALF samples; any context-specific cell type signals a filtering bug
- All five resulting tensors have identical shape along the LR-pair and cell-type axes, a precondition for pairwise 1-CorrIndex comparison
- After decomposition of the five filtered tensors, mean pairwise decomposition similarity (1-CorrIndex) should be approximately 0.82, per the reference finding
- The 5×5 pairwise similarity matrix saved as tensor_similarity_scores.csv is symmetric with unit diagonal

## Limitations

- Only cell types shared across all samples are included, so rare context-specific cell types are excluded from the COVID-19 and ASD case-study tensors
- The implemented tensor decomposition algorithm is not optimized for missing values, which is why aggressive filtering (dropping non-shared LR pairs and cell types) is required rather than optional
- It is hard to compare and evaluate CCC tools because of the qualitative differences in their outputs; filtering to a common LR space mitigates but does not remove score-comparability issues
- Benchmarking CCC prediction tools is challenging due to the lack of a ground truth, so filtered-subset results cannot be validated against an absolute reference

## Evidence

- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [full_text] retaining only LR pairs and cell types shared across all samples (172 shared LR pairs common to all methods): [evidence span withheld — anchor status: not_found]
- [results] we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications): "we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications)"
- [readme] Find a curated list of ligand-receptor pairs for your analyses at our GitHub Repository: "Find a curated list of ligand-receptor pairs for your analyses at our GitHub Repository"
