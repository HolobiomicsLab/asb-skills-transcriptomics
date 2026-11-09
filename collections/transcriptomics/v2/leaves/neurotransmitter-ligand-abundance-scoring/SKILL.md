---
name: neurotransmitter-ligand-abundance-scoring
description: Use when inferring neuron-neuron communication strength from scRNA-seq
  for neurotransmitter-type entries (glutamate, GABA, etc.) whose 'ligand' is a small
  molecule synthesized and vesicularly transported by the sender group, and you must
  convert gene expression into a ligand abundance score.
when_to_use_negative:
- The interaction entry is a peptide/protein ligand-receptor pair where the ligand
  abundance is directly the ensemble expression of one gene, so no functional-group
  AND/OR logic is needed
- The interaction database lacks curated functional grouping of contributing genes
  (synthetic enzymes vs. vesicular transporters), forcing a degenerate AND-only computation
  known to underestimate neurotransmitter ligand abundance
- 'Abundance estimates must reflect actual metabolite levels: the article warns transcriptomic
  ligand scores can be inconsistent with protein or metabolite levels, so single-cell
  proteomics/metabolomics is the wrong substrate here'
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: NeuronChat
  role: Implements the AND (geometric mean) / OR (arithmetic mean) ligand-abundance
    computation over Tukey-trimean-averaged expression via createNeuronChat and run_NeuronChat
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Alternative workflow whose ligand computation without functional gene grouping
    is the underestimating baseline this skill corrects; NeuronChat's workflow is
    implemented on top of it
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:16bbfe504dca30951ac961835a8e259372f984143c18be142dd8d2d193524d68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# neurotransmitter-ligand-abundance-scoring

## Summary

Computes non-peptide neurotransmitter ligand abundance from single-cell expression by grouping contributing genes (e.g., synthetic enzymes, vesicular transporters) into functional categories and combining them with AND logic (geometric mean) across groups and OR logic (arithmetic mean) within redundant groups. This avoids the dramatic underestimation of small-molecule ligand abundance that generic AND-only ligand-receptor rules produce.

## When to use

Use when inferring neuron-neuron communication strength from scRNA-seq for neurotransmitter-type entries (glutamate, GABA, etc.) whose 'ligand' is a small molecule synthesized and vesicularly transported by the sender group, and you must convert gene expression into a ligand abundance score.

## When NOT to use

- The interaction entry is a peptide/protein ligand-receptor pair where the ligand abundance is directly the ensemble expression of one gene, so no functional-group AND/OR logic is needed
- The interaction database lacks curated functional grouping of contributing genes (synthetic enzymes vs. vesicular transporters), forcing a degenerate AND-only computation known to underestimate neurotransmitter ligand abundance
- Abundance estimates must reflect actual metabolite levels: the article warns transcriptomic ligand scores can be inconsistent with protein or metabolite levels, so single-cell proteomics/metabolomics is the wrong substrate here

## Inputs

- Gene-by-cell normalized count matrix (counts scaled to 10,000 per cell, +1 pseudocount, natural-log transformed), e.g. cortex_data from Yao 2021
- Cell group / subclass label vector (group.by) defining sender and receiver cell groups
- NeuronChatDB interaction database ('mouse' or 'human', 183/190 entries) with contributing-gene functional-group annotations (synthesis, vesicular transport)

## Outputs

- Per-interaction ligand abundance L_i for each sender cell group and target (receptor) abundance T_j for each receiver group
- Per-interaction communication-strength matrices LT_i,j = L_i * T_j (sender-by-receiver)
- Aggregated network weight matrix of dimension (number of cell groups) x (number of cell groups) after net_aggregation and significance filtering

## How to apply

First normalize counts (scale each cell to 10,000, add pseudocount 1, natural-log transform) and, per gene and per cell group, compute the ensemble average with Tukey's trimean, (Q1 + 2*median + Q3)/4, after removing genes expressed in fewer than 25% of cells. For each NeuronChatDB interaction entry, partition the genes contributing to ligand emission into functional groups (e.g., synthesis enzymes vs. vesicular transporters); within a redundant group use the arithmetic mean (OR logic), and across different functional groups use the geometric mean (AND logic). Compute the receptor/target abundance analogously from receptor subunits, then form communication strength LT_i,j as the product of sender ligand abundance L_i and receiver target abundance T_j. Apply AND logic to all contributing genes as a single pool (the CellChat/CellPhoneDB-style fallback) only with caution, since without functional grouping this dramatically underestimates small-molecule neurotransmitter abundance.

## Related tools

- **NeuronChat** (Implements the AND (geometric mean) / OR (arithmetic mean) ligand-abundance computation over Tukey-trimean-averaged expression via createNeuronChat and run_NeuronChat) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Alternative workflow whose ligand computation without functional gene grouping is the underestimating baseline this skill corrects; NeuronChat's workflow is implemented on top of it) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by = cell_group_vector); x <- run_NeuronChat(x, M=100)
```

## Evaluation signals

- Genes expressed in fewer than 25% of cells are excluded before trimean averaging; verify the filtered gene set matches this cutoff
- Trimean values must satisfy (Q1 + 2*median + Q3)/4 per gene per cell group, and the output weight matrix must have dimensions equal to the number of cell groups squared
- Ligand score for a multi-group entry must be below the arithmetic mean across all contributing genes (geometric mean across functional groups penalizes any weak group), while within-group redundant genes use the arithmetic mean
- After run_NeuronChat(x, M=100) with Benjamini-Hochberg correction, entries with p >= 0.05 must be set to zero and significant entries must lie in a valid probability range
- On the ALM benchmark (cortex_data, mouse DB, M=100, 80%-quantile thresholded weight aggregation) the recovered scores must match AUROC 0.764 and AUPRC 0.768 within numerical tolerance

## Limitations

- NeuronChatDB may contain missing curated interaction entries, leading to bias in the inference
- Subunit stoichiometries of heteromeric receptors are not contained in NeuronChatDB and are largely unknown
- The model uses only expression of synthetic enzymes and vesicular transporters; stoichiometric effects of metabolism and pathway dependency are not included, and auxiliary proteins and downstream genes are not yet incorporated
- Permutation-based p-values trade accuracy against computation speed (time complexity O(n) in the number of permutations)

## Evidence

- [discussion] AND/OR logic across functional groups: "the AND logic (i.e., geometric mean) is applied among different groups of genes; since the genes within the same group are redundant for the same function, the OR logic (i.e., arithmetic mean) is"
- [discussion] Tukey trimean ensemble average: [evidence span withheld — anchor status: not_found]
- [discussion] 25% expression filter: "Because the genes expressed in less than 25% of cells are filtered out"
- [discussion] CellChat/CellPhoneDB underestimation baseline: "both of CellChat and CellPhoneDB are implemented without categorizing the contributing genes into different functional groups and thus use AND logic (geometric mean or minimum) for all contributing"
- [readme] NeuronChat object creation from README: [evidence span withheld — anchor status: external_doc]
