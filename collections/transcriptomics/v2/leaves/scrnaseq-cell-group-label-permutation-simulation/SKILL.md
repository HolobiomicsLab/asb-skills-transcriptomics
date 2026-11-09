---
name: scrnaseq-cell-group-label-permutation-simulation
description: Use when running NeuronChat (or its shared CellChat/CellPhoneDB workflow)
  on scRNA-seq or spatial transcriptomics and you must decide which inferred ligand-receptor
  communication links are statistically significant, or when testing p-value stability
  across permutation counts (M=100 vs M=1000).
when_to_use_negative:
- For spatially constrained networks, where the relevant test is the spatial proximity
  enrichment score obtained by permuting cell/spot labels (1000 permutations by default),
  not group-label permutation of communication strength
- When the goal is ligand-abundance estimation itself (trimean, AND/OR logic) rather
  than significance testing of the resulting links
- When input expression data have not been normalized (10,000 scale, pseudocount 1,
  natural-log), since permutation p-values depend on the recalculation pipeline
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: NeuronChat
  role: Provides createNeuronChat and run_NeuronChat(x, M=...), which implement the
    group-label permutation test, BH correction, and zeroing of non-significant communication
    strengths
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Shares the same computational workflow for permutation-based significance;
    its implementation without functional gene grouping (plain AND logic) is the comparison
    case for ligand-abundance behavior
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:042ecb9b1df659ada1d585a0083f231ad255621b8eb26f9bf27fd9f7442c80e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# scrnaseq-cell-group-label-permutation-simulation

## Summary

NeuronChat's permutation-test module for assessing statistical significance of inferred neural communication links: cell-group labels are randomly permuted, communication strength is recalculated, and Benjamini–Hochberg-corrected p<0.05 links are retained while non-significant ones are set to zero.

## When to use

Use when running NeuronChat (or its shared CellChat/CellPhoneDB workflow) on scRNA-seq or spatial transcriptomics and you must decide which inferred ligand-receptor communication links are statistically significant, or when testing p-value stability across permutation counts (M=100 vs M=1000).

## When NOT to use

- For spatially constrained networks, where the relevant test is the spatial proximity enrichment score obtained by permuting cell/spot labels (1000 permutations by default), not group-label permutation of communication strength
- When the goal is ligand-abundance estimation itself (trimean, AND/OR logic) rather than significance testing of the resulting links
- When input expression data have not been normalized (10,000 scale, pseudocount 1, natural-log), since permutation p-values depend on the recalculation pipeline

## Inputs

- Normalized scRNA-seq or spatial transcriptomics count matrix (10,000 scale, pseudocount 1, natural-log transformed)
- Cell-group label vector (e.g., 'subclass') for group.by
- NeuronChatDB interaction database ('mouse' or 'human'; 183 mouse / 190 human entries)
- Permutation count M (e.g., 100 or 1000)

## Outputs

- Communication probabilities and permutation p-values for all interaction pairs (stored on the NeuronChat object, e.g., x@net)
- Benjamini–Hochberg-corrected p-values per communication link
- Thresholded communication-strength matrix with non-significant links set to zero
- Comparison table (e.g., CSV) of significant-link overlap between M=100 and M=1000 runs

## How to apply

Build a NeuronChat object from the normalized count matrix (counts scaled to 10,000, plus pseudocount of 1, natural-log transformed) with createNeuronChat using DB='mouse' or 'human' and a group.by cell-group vector (e.g., subclass). Call run_NeuronChat(x, M=100) (or M=1000 for finer resolution): for each of the M permutations, group labels of cells are randomly permuted and communication strength — the product of ligand and target abundance, computed with trimean ensemble averaging, AND logic (geometric mean) across functional gene groups and OR logic (arithmetic mean) within redundant gene groups — is recalculated to yield a null distribution and a p-value per interaction pair. Correct the p-values for multiple testing with the Benjamini–Hochberg procedure, call links significant at p<0.05, and set the communication strength of non-significant links to zero before downstream aggregation (e.g., net_aggregation with method='weight'). To validate stability, rerun with M=1000 and compare significant-link sets (overlap counts and per-link p-value agreement), as in Supplementary Fig. 3.

## Related tools

- **NeuronChat** (Provides createNeuronChat and run_NeuronChat(x, M=...), which implement the group-label permutation test, BH correction, and zeroing of non-significant communication strengths) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Shares the same computational workflow for permutation-based significance; its implementation without functional gene grouping (plain AND logic) is the comparison case for ligand-abundance behavior) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by=cell_group_vector); x <- run_NeuronChat(x, M=1000)
```

## Evaluation signals

- Significant-link sets from M=100 and M=1000 runs show high overlap, matching the stability reported in Supplementary Fig. 3 of the paper
- All communication strengths for links with BH-corrected p >= 0.05 are exactly zero in the output network matrix
- All reported p-values lie in [0,1] and are BH-corrected for the number of interaction tests
- Rerunning with the same M reproduces comparable significant-link counts (permutation p-value granularity is 1/(M+1), so p-values should quantize accordingly)
- The thresholded output aggregates sensibly via net_aggregation(x@net, method='weight') with no negative or NA weights

## Limitations

- There is a trade-off between accuracy and computation speed for permutation-based p-value calculations (time complexity O(n) in the number of permutations), so small M gives coarser p-value resolution
- NeuronChat estimates ligand/target abundance from transcriptomics, which may be inconsistent with protein or metabolite levels, affecting the communication strengths being tested
- NeuronChatDB may contain missing curated interaction entries, leading to bias in the inference underlying the tested links

## Evidence

- [discussion] Permutation test definition: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] Significance threshold and zeroing: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [discussion] Multiple-testing correction: "The p-value is corrected for multiple tests by using Benjamini–Hochberg procedure"
- [methods] Permutation count in the run call: "x <- run_NeuronChat(x,M=100)"
- [readme] Package installation for the tool used: "You can install the latest version of NeuronChat like so:

``` r
devtools::install_github("Wei-BioMath/NeuronChat")"
