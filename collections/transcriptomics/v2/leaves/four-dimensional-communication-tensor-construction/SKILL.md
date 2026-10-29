---
name: four-dimensional-communication-tensor-construction
description: Use when you have per-sample/per-context LR communication scores from
  one or more CCC tools (e.g., CellChat, CellPhoneDB, NATMI, SingleCellSignalR, cell2cell
  built-in) and need a 4D tensor input for Tensor-cell2cell factorization or cross-tool
  consistency checks.
when_to_use_negative:
- Only a single sample/context is available — use regular cell2cell per-context analysis
  instead of a 4D tensor
- LR pairs or cell types are missing in some contexts and you need to include all
  possible communication members — the implemented decomposition is not optimized
  for missing values, so only shared members can be retained
- Rare cell types present in only some samples are biologically central — the shared-members
  filter excludes them from the tensor
tools:
- name: cell2cell
  role: Provides cell2cell.tensor for building the 4D communication tensor and factor-comparison
    functions (CorrIndex) and cell2cell.preprocessing for per-sample preprocessing
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Consumes the constructed tensor, performs non-negative TCA decomposition,
    and computes 1-CorrIndex similarities between factorizations
  repo: https://github.com/earmingol/cell2cell
- name: LIANA
  role: Runs multiple CCC scoring methods (CellChat, CellPhoneDB, NATMI, SingleCellSignalR)
    per context to generate the score matrices stacked into the tensor
  repo: https://github.com/saezlab/liana-py
- name: CellChat
  role: Supplies the LR database (Human-2020-Jin, Jin et al. 2021) and one of the
    per-context communication scoring methods
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:e95188bfe3482c05ab23d798cb70ea2c3d4028e6becbc832edbae5c0974405fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - LIANA
  - Tensor-cell2cell
  - cell2cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# four-dimensional-communication-tensor-construction

## Summary

Construct a 4D communication tensor (LR pairs × sender cell types × receiver cell types × contexts) from per-context cell-cell communication scores so Tensor-cell2cell can deconvolve context-dependent communication patterns. Retaining only LR pairs and cell types shared across all samples makes tensors from different scoring methods directly comparable.

## When to use

Use when you have per-sample/per-context LR communication scores from one or more CCC tools (e.g., CellChat, CellPhoneDB, NATMI, SingleCellSignalR, cell2cell built-in) and need a 4D tensor input for Tensor-cell2cell factorization or cross-tool consistency checks.

## When NOT to use

- Only a single sample/context is available — use regular cell2cell per-context analysis instead of a 4D tensor
- LR pairs or cell types are missing in some contexts and you need to include all possible communication members — the implemented decomposition is not optimized for missing values, so only shared members can be retained
- Rare cell types present in only some samples are biologically central — the shared-members filter excludes them from the tensor

## Inputs

- Preprocessed single-cell RNA-seq data per sample/context (e.g., .h5ad or similar from cell2cell.preprocessing)
- Per-context LR communication scores from CellChat, CellPhoneDB, NATMI, SingleCellSignalR (via LIANA) or cell2cell built-in scoring
- Ligand-receptor pair list (e.g., Human-2020-Jin-LR-pairs.csv from LewisLabUCSD/Ligand-Receptor-Pairs)
- Sample/context metadata (e.g., 3 control, 3 moderate, 6 severe COVID-19 BALF patients)

## Outputs

- 4D communication tensor (LR pairs × sender cell types × receiver cell types × contexts) built with cell2cell.tensor
- Tensor-cell2cell factorization of the tensor (communication factors)
- Pairwise 1-CorrIndex similarity matrix across decompositions (e.g., 5×5 matrix saved as tensor_similarity_scores.csv)

## How to apply

Load and preprocess the single-cell data per sample with cell2cell.preprocessing (e.g., the BALF COVID-19 dataset of 63k immune and epithelial cells across 3 control, 3 moderate, and 6 severe patients). Compute communication scores per context with the chosen tool(s) via LIANA or cell2cell's built-in rank-based scoring. Build the tensor with cell2cell.tensor, keeping only the LR pairs and cell types shared across all samples — in the BALF case study this yielded 172 shared LR pairs common to all five scoring methods. Decompose the tensor with non-negative TCA; if init='svd' raises memory errors, fall back to init='random' per the documentation. Compare factorizations across tools using 1-CorrIndex similarity (expected mean pairwise similarity ~0.82 in the paper's BALF benchmark).

## Related tools

- **cell2cell** (Provides cell2cell.tensor for building the 4D communication tensor and factor-comparison functions (CorrIndex) and cell2cell.preprocessing for per-sample preprocessing) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Consumes the constructed tensor, performs non-negative TCA decomposition, and computes 1-CorrIndex similarities between factorizations) — https://github.com/earmingol/cell2cell
- **LIANA** (Runs multiple CCC scoring methods (CellChat, CellPhoneDB, NATMI, SingleCellSignalR) per context to generate the score matrices stacked into the tensor) — https://github.com/saezlab/liana-py
- **CellChat** (Supplies the LR database (Human-2020-Jin, Jin et al. 2021) and one of the per-context communication scoring methods) — https://github.com/sqjin/CellChat

## Examples

```
pip install cell2cell
```

## Evaluation signals

- Tensor has exactly four dimensions matching LR pairs × sender cell types × receiver cell types × contexts, containing only members shared across all samples
- In the BALF COVID-19 benchmark, the shared intersection contains 172 LR pairs common to all five scoring methods
- Pairwise 1-CorrIndex similarities between decompositions of tensors built from different scoring methods have a mean of approximately 0.82
- If decomposition raises memory errors with init='svd', rerunning with init='random' completes and yields consistent factors
- The 5×5 pairwise similarity matrix (tensor_similarity_scores.csv) is symmetric with unit diagonal

## Limitations

- Only cell types shared across all samples can be included, potentially missing rare cell types present in a subset of contexts
- The implemented tensor decomposition algorithm is not optimized for missing values; analyses with missing LR pairs or cell types require further developments
- Benchmarking cross-tool consistency is hard because of qualitative differences in CCC tool outputs and lack of ground truth

## Evidence

- [discussion] One can choose any tool of interest, run it on each context separately, and use the resulting communication scores to build and deconvolve a 4D-communication tensor.: "One can choose any tool of interest, run it on each context separately, and use the resulting communication scores to build and deconvolve a 4D-communication tensor."
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [results] we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications): "we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications)"
- [readme] Step 3: Install cell2cell pip install cell2cell: [evidence span withheld — anchor status: not_found]
