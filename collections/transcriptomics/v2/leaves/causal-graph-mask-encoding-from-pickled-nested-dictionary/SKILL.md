---
name: causal-graph-mask-encoding-from-pickled-nested-dictionary
description: 'Use when you need to impose a GRN on GRouNdGAN training: convert TF→gene
  edges into the pickled causal graph (nested dict of gene index -> set of regulating
  TF indices) that fixes the masked causal generator''s per-target-gene connectivity.'
when_to_use_negative:
- Your GRN is multi-layer (TF→TF chains or TF-gene-phenotype causal graphs) — the
  current version of GRouNdGAN only allows imposing a bipartite GRN.
- The imposed TF-gene relationships differ significantly from patterns in the reference
  dataset — resemblance of simulated data to the reference deteriorates in that case.
- You want unnormalized count simulation — the model generates expression values library-size
  normalized (library size = 20000) across the considered genes and TFs.
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0091
tools:
- name: GRouNdGAN
  role: Consumes the pickled causal graph to mask the connectivity of its per-target-gene
    causal generators, enforcing the user-defined GRN during simulation
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the GRN from the preprocessed train set via the --create_grn command
    before it is formatted as the causal graph
- name: CausalGAN
  role: 'Architectural basis: GRouNdGAN''s controller and target generators follow
    the causal GAN design whose mask structure the causal graph defines'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:075582448ea8490da1d05877c81f8f75ee3522a5ef4081cdde4e13e1152934e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - CausalGAN
  - GRNBoost2
  - GRouNdGAN
  - scGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# causal-graph-mask-encoding-from-pickled-nested-dictionary

## Summary

Encodes a user-defined or GRNBoost2-inferred gene regulatory network into GRouNdGAN's pickled causal graph format — a nested dict mapping each gene index to its set of regulating TF indices — so the masked connectivity of the causal generator forces simulated scRNA-seq data to respect the imposed regulatory structure.

## When to use

Use when you need to impose a GRN on GRouNdGAN training: convert TF→gene edges into the pickled causal graph (nested dict of gene index -> set of regulating TF indices) that fixes the masked causal generator's per-target-gene connectivity.

## When NOT to use

- Your GRN is multi-layer (TF→TF chains or TF-gene-phenotype causal graphs) — the current version of GRouNdGAN only allows imposing a bipartite GRN.
- The imposed TF-gene relationships differ significantly from patterns in the reference dataset — resemblance of simulated data to the reference deteriorates in that case.
- You want unnormalized count simulation — the model generates expression values library-size normalized (library size = 20000) across the considered genes and TFs.

## Inputs

- User-provided or GRNBoost2-inferred GRN edges (TF→gene, bipartite)
- Preprocessed scRNA-seq reference dataset for GAN training
- GRouNdGAN config file specifying generator architecture (noise per gene = 1, depth per gene = 3, width per gene = 2)

## Outputs

- Pickled causal graph (nested dict of gene index -> set of regulating TF indices)
- Masked causal generator whose per-target-gene connectivity is fixed by the imposed GRN
- Trained GRouNdGAN model (e.g., checkpoint step_1000000.pth) embodying the imposed causal structure

## How to apply

Either run GRouNdGAN's --create_grn command, which uses GRNBoost2 (Moerman et al., 2018) to infer a GRN on the preprocessed train set and appropriately format it as a causal graph, or skip --create_grn and impose your own GRN directly. Format the network as a pickled nested dictionary keyed by target-gene index with values that are sets of regulating TF indices; only bipartite TF→gene edges are accepted, so TF nodes must never appear as target keys. Use this graph to instantiate the masked causal generator with one generator per target gene (noise per gene = 1, depth per gene = 3, width per gene = 2), where each target generator receives only a noise value and the simulated expression of its regulating TFs. Then train the causal GAN (type = causal GAN, max steps = 1,000,000, batch size = 1024, critic iterations = 5, gradient-penalty lambda = 10) so the imposed masks define the ground-truth GRN for later benchmarking or in silico TF perturbation.

## Related tools

- **GRouNdGAN** (Consumes the pickled causal graph to mask the connectivity of its per-target-gene causal generators, enforcing the user-defined GRN during simulation) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the GRN from the preprocessed train set via the --create_grn command before it is formatted as the causal graph)
- **CausalGAN** (Architectural basis: GRouNdGAN's controller and target generators follow the causal GAN design whose mask structure the causal graph defines)

## Examples

```
import pickle; causal_graph = pickle.load(open('causal_graph.pkl', 'rb'))  # nested dict: gene index -> set of regulating TF indices
```

## Evaluation signals

- The pickled causal graph loads as a nested dict mapping gene indices to sets of regulating TF indices, and every referenced TF index is a valid column index of the preprocessed dataset.
- The encoded graph is strictly bipartite: no TF index appears as a target-gene key, satisfying GRouNdGAN's bipartite-only constraint.
- The masked causal generator instantiates with one generator per target gene (noise per gene = 1, depth per gene = 3, width per gene = 2) and each target generator's input is exactly its noise value plus the simulated expression of its masked regulating TFs.
- Training with the imposed graph completes with the message 'Finished training' and the final checkpoint step_1000000.pth is saved to the checkpoints/ directory.
- The imposed GRN can be exported as a ground-truth CSV via the [GRN Benchmarking]/ground truth save path setting and matches the TF→gene edges of the original network.

## Limitations

- The current version only allows imposing a bipartite GRN; multi-layer GRNs and TF-gene-phenotype causal graphs are not supported and are planned for future versions.
- The underlying causal graph of an observational dataset is not necessarily unique; verifying that the imposed GRN is the most probable one is beyond the study's scope.
- Resemblance of simulated data to the reference dataset deteriorates if the imposed TF-gene relationships are significantly different from those consistent with the reference data patterns.
- When trained on a new dataset with significantly different numbers of genes and TFs, hyperparameters may need re-tuning using a validation set.

## Evidence

- [full_text] Masked connectivity fixes per-target-generator inputs: [evidence span withheld — anchor status: not_found]
- [methods] GRN inference and formatting into causal graph: "--create_grn     Infer a GRN from preprocessed data using GRNBoost2 and appropriately format as causal graph"
- [methods] Custom GRN imposition path: "It is possible to instead impose your own GRN onto GRouNdGAN. If you're opting for this option, skip the ``--create_grn`` command."
- [discussion] Bipartite-only constraint: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN, an assumption that we plan to relax with the future versions of the model."
- [intro] Target generators read only their masked TF inputs: [evidence span withheld — anchor status: not_found]
