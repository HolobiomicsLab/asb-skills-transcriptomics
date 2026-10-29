---
name: causal-graph-pickled-dictionary-formatting
description: 'Use when preparing a TF-gene regulatory network to impose on GRouNdGAN
  training: you have TF-gene rankings (e.g., GRNBoost2 output) or a custom GRN and
  must convert it to the causal_graph.pkl nested-dictionary format.'
when_to_use_negative:
- Your GRN is multi-layer or includes TF-gene-phenotype edges — the current GRouNdGAN
  version only allows imposing a bipartite GRN
- You only need simulated data quality evaluation (--evaluate) or a perturbation study
  (--perturb) with no new GRN to impose
- The imposed TF-gene relationships differ significantly from patterns consistent
  with the reference dataset, which degrades resemblance to the reference data
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: GRouNdGAN
  role: Provides --create_grn and the causal_graph.pkl format; its training step consumes
    the pickled causal graph to impose the GRN on simulated data
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers TF-gene importance rankings (k = 15 top TFs per gene) from the preprocessed
    train set; these rankings are subset into the causal graph dictionary
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:966537298cd4914ae4da80abe02eab2d42012096cd4673beb4fb570bb4efe8a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GENIE3
  - GRNBoost2
  - GRouNdGAN
  - PIDC
  - PPCOR
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Causal graph pickled dictionary formatting

## Summary

Format an inferred or user-provided GRN as a pickled nested dictionary (causal_graph.pkl) mapping each gene's index to the set of TF indices that regulate it, so GRouNdGAN can impose the bipartite causal graph during GAN training.

## When to use

Use when preparing a TF-gene regulatory network to impose on GRouNdGAN training: you have TF-gene rankings (e.g., GRNBoost2 output) or a custom GRN and must convert it to the causal_graph.pkl nested-dictionary format.

## When NOT to use

- Your GRN is multi-layer or includes TF-gene-phenotype edges — the current GRouNdGAN version only allows imposing a bipartite GRN
- You only need simulated data quality evaluation (--evaluate) or a perturbation study (--perturb) with no new GRN to impose
- The imposed TF-gene relationships differ significantly from patterns consistent with the reference dataset, which degrades resemblance to the reference data

## Inputs

- Preprocessed scRNA-seq expression matrix (e.g., preprocessed.h5ad with 1000 highly variable genes, library-size normalized to 20,000)
- TF-gene importance rankings from GRNBoost2 (k = 15 top TFs per gene)
- Transcription factor list (Homo_sapiens_TF.csv)
- Optionally, a user-provided custom GRN to impose (skipping --create_grn)

## Outputs

- causal_graph.pkl — pickled nested dictionary mapping gene index → set of regulating TF indices
- Optionally two control graphs: positive ('pos ctr', even TF ranks) and negative ('neg ctr', odd TF ranks) bipartite networks of equal density

## How to apply

First preprocess the expression matrix with GRouNdGAN's --preprocess step (e.g., PBMC-CTL with 1000 highly variable genes and library-size normalization to 20,000), since gene and TF indices in the dictionary refer to rows of that preprocessed matrix. To derive a GRN, run --create_grn, which uses GRNBoost2 with k = 15 top most important TFs per gene against the Homo_sapiens_TF.csv transcription factor list. Then build the causal graph by assigning, for every gene index, the set of indices of its selected regulating TFs; for the even/odd-parity control scheme, 'pos ctr' takes even-ranked TF indices (0, 2, 4...) and 'neg ctr' takes odd-ranked TF indices (1, 3, 5...) per gene, yielding two equal-density bipartite networks. Finally pickle the nested dictionary as causal_graph.pkl and point the [EXPERIMENT] causal graph field at it before --train; if using your own GRN, skip --create_grn entirely and supply the pickle directly. The graph must be bipartite TF→gene, as the current GRouNdGAN version does not support multi-layer GRNs.

## Related tools

- **GRouNdGAN** (Provides --create_grn and the causal_graph.pkl format; its training step consumes the pickled causal graph to impose the GRN on simulated data) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers TF-gene importance rankings (k = 15 top TFs per gene) from the preprocessed train set; these rankings are subset into the causal graph dictionary)

## Examples

```
python groundgan.py --create_grn  # then pickle {gene_idx: {tf_idx, ...}} as causal_graph.pkl with strategy='pos ctr'
```

## Evaluation signals

- causal_graph.pkl unpickles to a nested dict whose keys are gene indices into the preprocessed matrix and whose values are sets of TF indices
- Positive and negative control graphs ('pos ctr' even indices, 'neg ctr' odd indices) have equal edge density and disjoint edge sets
- All TF indices correspond to genes present in the Homo_sapiens_TF.csv transcription factor list
- Downstream --benchmark_grn with compute_pr = True shows high AUPRC for imposed edges and near-random AUPRC for unimposed (negative control) edges, matching the Fig. 3 pattern

## Limitations

- The current version only supports bipartite TF→gene GRNs; multi-layer GRNs and TF-gene-phenotype causal graphs are planned for future versions
- The underlying causal graph of an observational dataset is not necessarily unique, and verifying the imposed GRN is the most probable one is beyond the study's scope
- Resemblance to the reference dataset deteriorates if imposed TF-gene relationships differ significantly from reference data patterns

## Evidence

- [methods] create_grn infers GRN via GRNBoost2: "--create_grn     Infer a GRN from preprocessed data using GRNBoost2 and appropriately format as causal graph"
- [methods] custom GRN imposition skips create_grn: "It is possible to instead impose your own GRN onto GRouNdGAN. If you're opting for this option, skip the ``--create_grn`` command."
- [discussion] bipartite-only limitation: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN"
- [discussion] recommended edge-subset workflow: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities) and imposing a selected subset of those in the simulated data"
- [other] GRNBoost2 uses k=15 top TFs per gene: "This command uses GRNBoost2 (Moerman et al., 2018) to infer a GRN on the preprocessed train set."
