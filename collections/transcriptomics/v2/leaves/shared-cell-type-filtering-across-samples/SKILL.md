---
name: shared-cell-type-filtering-across-samples
description: Use when building a 4D-communication tensor from single-cell data spanning
  multiple samples or contexts and the cell-type annotations differ between samples,
  so dimensions must match before Tensor-cell2cell decomposition.
when_to_use_negative:
- When the downstream decomposition algorithm supports missing values — the article
  notes the implemented algorithm is not optimized for missing values, but this filtering
  can drop rare cell types present in only some samples
- When analyzing a single sample/context, where no cross-sample dimension alignment
  is needed
- When the goal is to preserve context-specific rare cell types rather than produce
  a factorizable tensor with consistent dimensions
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: cell2cell
  role: Provides the preprocessing (cell2cell.preprocessing) and tensor assembly (cell2cell.tensor)
    modules used to aggregate scores, filter to shared cell types, and stack 3D context
    tensors into the 4D tensor
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Downstream consumer of the shared-cell-type-filtered 4D tensor; its TCA-based
    decomposition requires consistent sender/receiver dimensions across contexts
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:553da5d70178c2ec85fc19948df83dd40220e43c0fc465e42a52d2c776bb71a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Shared cell-type filtering across samples

## Summary

Restricts a cell-cell communication tensor to only the cell types that are present (annotated) across every sample/context before tensor factorization, because the tensor decomposition requires consistent dimensions (sender/receiver axes) across the contexts axis.

## When to use

Use when building a 4D-communication tensor from single-cell data spanning multiple samples or contexts and the cell-type annotations differ between samples, so dimensions must match before Tensor-cell2cell decomposition.

## When NOT to use

- When the downstream decomposition algorithm supports missing values — the article notes the implemented algorithm is not optimized for missing values, but this filtering can drop rare cell types present in only some samples
- When analyzing a single sample/context, where no cross-sample dimension alignment is needed
- When the goal is to preserve context-specific rare cell types rather than produce a factorizable tensor with consistent dimensions

## Inputs

- Per-context cell-type-level communication matrices (LR pairs x sender cell types x receiver cell types)
- Cell-type annotations for each sample/context (e.g., from single-cell expression data with annotations)
- Curated ligand-receptor pair list (e.g., Human-2020-Jin-LR-pairs.csv)

## Outputs

- 4D-communication tensor whose sender/receiver axes contain only cell types shared across all samples
- Serialized tensor object (e.g., communication_tensor.pkl) ready for Tensor-cell2cell factorization

## How to apply

After aggregating single-cell expression to cell-type-level communication scores for each context (e.g., using the non-zero fraction aggregation in cell2cell.preprocessing), compare the cell-type annotation sets of all samples and retain only cell types shared across all of them. Assemble each context's scores into a 3D tensor (LR pairs x senders x receivers) and stack contexts into a 4D tensor via the cell2cell.tensor module with the shared cell types as the sender and receiver axes. The rationale is that the TCA-based decomposition of Tensor-cell2cell requires identical sender and receiver dimensions along the contexts axis; in the COVID-19 and ASD case studies the authors explicitly built tensors containing only cell types shared across all samples. Serialize the filtered tensor (e.g., communication_tensor.pkl) and validate that reloading it reproduces the communication scores and that its dimensions correspond to contexts, LR pairs, sender cell types, and receiver cell types.

## Related tools

- **cell2cell** (Provides the preprocessing (cell2cell.preprocessing) and tensor assembly (cell2cell.tensor) modules used to aggregate scores, filter to shared cell types, and stack 3D context tensors into the 4D tensor) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Downstream consumer of the shared-cell-type-filtered 4D tensor; its TCA-based decomposition requires consistent sender/receiver dimensions across contexts) — https://github.com/earmingol/cell2cell

## Examples

```
pip install cell2cell
```

## Evaluation signals

- The saved tensor is a valid 4D object whose dimensions correspond to contexts, LR pairs, sender cell types, and receiver cell types
- Every cell type appearing on the sender and receiver axes is annotated in every sample/context included in the tensor
- Reloading the serialized pickle (communication_tensor.pkl) reproduces the original communication scores
- Each context's 3D slice has identical sender and receiver axis labels, so all context slices are dimensionally compatible for stacking

## Limitations

- Restricting to shared cell types can miss rare cell types that appear in only some samples — the authors note this was done in the COVID-19 and ASD case studies
- Further developments are needed to enable analyses with missing values that would include all possible communication members (LR pairs and cell types missing in some contexts)

## Evidence

- [discussion] Only cell types shared across all samples were included in the COVID-19 and ASD case-study tensors, potentially missing rare cell types: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [methods] Data preprocessing module: "cell2cell.preprocessing : Includes functions for manipulating data and variables (e.g. data preprocessing, integration, permutation, among others)."
- [discussion] Further developments needed for missing values: "Thus, further developments will facilitate analyses with missing values to include all possible members of communication"
