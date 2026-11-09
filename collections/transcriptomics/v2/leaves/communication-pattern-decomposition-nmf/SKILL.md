---
name: communication-pattern-decomposition-nmf
description: Use when a NeuronChat object with inferred interaction networks exists
  and you need to decompose cell groups into outgoing/incoming communication patterns,
  then assign each cell group (e.g., VISp subclass) to a pattern.
when_to_use_negative:
- The input is raw counts or per-pair interaction networks that have not been inferred
  and significance-tested (p<0.05 via permutation with BH correction) by run_NeuronChat
  — pattern decomposition requires the finished NeuronChat network object.
- The biological question is only about total information flow ranking of interaction
  pairs (rankNet_Neuron with measure='weight'/'count'), not about how cell groups
  share combinations of pathways.
- The dataset contains non-neural cell types better modeled by generic paracrine ligand-receptor
  tools (CellChat/CellPhoneDB), since NeuronChatDB is neural-specific (183 mouse entries)
  and centered on neurotransmitter signaling.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuronChat
  role: R package providing identifyCommunicationPatterns_Neuron (NMF-style pattern
    decomposition with parameter k) and netAnalysis_river_Neuron for outgoing/incoming
    pattern visualization
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat whose computational workflow NeuronChat reuses; its
    non-grouped ligand-abundance logic underestimates small-molecule neurotransmitter
    signaling, motivating NeuronChat's functional gene grouping
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:0f780de523efddb6226a87376a520aaea085f0666522e487e19fc86aaa5f036b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# communication-pattern-decomposition-nmf

## Summary

Decompose NeuronChat-inferred neural communication networks into outgoing and incoming communication patterns (e.g., with identifyCommunicationPatterns_Neuron at k=4) and visualize them as river/alluvial plots with netAnalysis_river_Neuron, as done for the 21 VISp cell subclasses in the mouse visual cortex.

## When to use

Use when a NeuronChat object with inferred interaction networks exists and you need to decompose cell groups into outgoing/incoming communication patterns, then assign each cell group (e.g., VISp subclass) to a pattern.

## When NOT to use

- The input is raw counts or per-pair interaction networks that have not been inferred and significance-tested (p<0.05 via permutation with BH correction) by run_NeuronChat — pattern decomposition requires the finished NeuronChat network object.
- The biological question is only about total information flow ranking of interaction pairs (rankNet_Neuron with measure='weight'/'count'), not about how cell groups share combinations of pathways.
- The dataset contains non-neural cell types better modeled by generic paracrine ligand-receptor tools (CellChat/CellPhoneDB), since NeuronChatDB is neural-specific (183 mouse entries) and centered on neurotransmitter signaling.

## Inputs

- NeuronChat object with per-interaction communication networks from run_NeuronChat (e.g., M=100 permutations on the mouse VISp scRNA-seq data)
- Aggregated communication network from net_aggregation(x@net, method='weight')
- Number of patterns k (k=4 in the VISp analysis)

## Outputs

- Outgoing and incoming communication pattern assignments for each cell group/subclass
- Pattern-by-interaction (pathway) contribution matrices
- River/alluvial plot from netAnalysis_river_Neuron (cutoff.1=0.5, cutoff.2=0.5) mapping cell groups to communication patterns

## How to apply

After running run_NeuronChat on the NeuronChat object (e.g., M=100 permutations) and aggregating per-interaction networks with net_aggregation(method='weight'), apply identifyCommunicationPatterns_Neuron separately to outgoing and incoming communication to factorize the group-by-interaction weight matrices into k patterns (k=4 was used for the VISp dataset). Each pattern captures a coherent set of signaling pathways used jointly by a subset of cell groups; the neuron-specific definition of communication strength (built from neurotransmitter signaling entries in NeuronChatDB such as Nrxn1-Nlgn1, Glu-Gria2, GABA-Gabra1) drives the decomposition, with ensemble expression computed via Tukey's trimean. Visualize the pattern-to-group assignments with netAnalysis_river_Neuron using cutoff.1=0.5 and cutoff.2=0.5 to produce alluvial/river plots linking cell groups to patterns and their dominant interaction pairs. Judge correctness by whether pattern assignments reproduce the published Fig. 4 outgoing/incoming pattern structure for the 21 VISp subclasses and whether each pattern is dominated by biologically coherent interactions (e.g., glutamatergic vs GABAergic receptor families).

## Related tools

- **NeuronChat** (R package providing identifyCommunicationPatterns_Neuron (NMF-style pattern decomposition with parameter k) and netAnalysis_river_Neuron for outgoing/incoming pattern visualization) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat whose computational workflow NeuronChat reuses; its non-grouped ligand-abundance logic underestimates small-molecule neurotransmitter signaling, motivating NeuronChat's functional gene grouping) — https://github.com/sqjin/CellChat

## Examples

```
x <- run_NeuronChat(x, M=100); net_aggregated_x <- net_aggregation(x@net, method='weight'); identifyCommunicationPatterns_Neuron(x, k=4); netAnalysis_river_Neuron(x, cutoff.1=0.5, cutoff.2=0.5)
```

## Evaluation signals

- The analysis must identify the expected number of patterns (k=4) for the VISp dataset with stable outgoing/incoming assignments across the 21 subclasses.
- River plot (netAnalysis_river_Neuron with cutoff.1=0.5, cutoff.2=0.5) pattern-to-group assignments should match the reported Fig. 4 results via output_matches_reference.
- Each decomposed pattern should be dominated by a coherent interaction set consistent with the top-10 weight-ranked pairs (Nrxn1-Nlgn1, Nrxn3-Nlgn1, Glu-Gria2, Glu-Grin2b, GABA-Gabra1, etc.).
- Expert review should confirm pattern assignments are biologically sensible (e.g., glutamatergic vs GABAergic signaling patterns).
- Underlying networks must contain only significant links: communications with p >= 0.05 (permutation test, BH-corrected) set to zero before decomposition.

## Limitations

- Ligand/target abundance is estimated from transcriptomics and may be inconsistent with protein or metabolite levels, affecting pattern interpretation.
- NeuronChatDB may contain missing curated interaction entries, leading to bias in the inferred networks that propagate into pattern decomposition.
- Subunit stoichiometries of heteromeric receptors are not contained in NeuronChatDB, and auxiliary proteins and downstream genes are not yet incorporated, which can limit pattern accuracy.
- Permutation-based p-value computation has a trade-off between accuracy and computation speed (O(n) in number of permutations).

## Evidence

- [methods] Run NeuronChat to infer the neural-specific cell-cell communication networks: "x <- run_NeuronChat(x,M=100)"
- [readme] Aggregate networks before pattern analysis: [evidence span withheld — anchor status: external_doc]
- [discussion] Significance filtering before decomposition: "The p-value is corrected for multiple tests by using Benjamini–Hochberg procedure"
- [discussion] Communication strength model underlying patterns: [evidence span withheld — anchor status: not_found]
- [discussion] Permutation-based significance of each communication: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
