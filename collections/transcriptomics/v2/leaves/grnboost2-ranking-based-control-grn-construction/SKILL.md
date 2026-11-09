---
name: grnboost2-ranking-based-control-grn-construction
description: Use when building positive/negative control ground-truth GRNs from GRNBoost2
  TF-gene importance rankings to impose on GRouNdGAN-simulated single-cell data for
  AUPRC-based recoverability benchmarking.
when_to_use_negative:
- You want a biologically curated ground-truth GRN rather than synthetic even/odd-parity
  controls — these control GRNs are constructed purely from ranking parity, not from
  known regulation
- You need a multi-layer or non-bipartite GRN — the current GRouNdGAN version only
  supports imposing bipartite GRNs
- You plan to impose your own GRN — in that case skip the --create_grn command entirely
  per the GRouNdGAN documentation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: GRouNdGAN
  role: Framework whose --create_grn command runs GRNBoost2 and formats the rankings
    into a causal graph; control-GRN strategy settings ('pos ctr'/'neg ctr') select
    even/odd index splits for imposition during GAN training
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the ranked TF-gene importance lists (top k = 15 per gene) from the
    preprocessed PBMC-CTL training data from which the parity-split control GRNs are
    built
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:0b5b79d5e906305d176ab5df7ec77aa37c02a491fe8acaa5e4375b19a987b36e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - GENIE3
  - GRNBoost2
  - GRouNdGAN
  - PIDC
  - PPCOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# GRNBoost2 ranking-based control GRN construction

## Summary

Constructs even/odd-parity positive and negative control GRNs by splitting the top-k GRNBoost2 TF-gene rankings from preprocessed PBMC-CTL data into two equal-density bipartite networks, enabling recoverability benchmarking of GRN inference methods on GRouNdGAN-simulated data.

## When to use

Use when building positive/negative control ground-truth GRNs from GRNBoost2 TF-gene importance rankings to impose on GRouNdGAN-simulated single-cell data for AUPRC-based recoverability benchmarking.

## When NOT to use

- You want a biologically curated ground-truth GRN rather than synthetic even/odd-parity controls — these control GRNs are constructed purely from ranking parity, not from known regulation
- You need a multi-layer or non-bipartite GRN — the current GRouNdGAN version only supports imposing bipartite GRNs
- You plan to impose your own GRN — in that case skip the --create_grn command entirely per the GRouNdGAN documentation

## Inputs

- Preprocessed scRNA-seq expression matrix (e.g., PBMC-CTL, 1000 highly variable genes, library-size normalized to 20,000)
- Transcription factor list (Homo_sapiens_TF.csv)
- GRNBoost2 TF-gene importance rankings (k = 15 TFs per gene)
- Control GRN strategy setting ('pos ctr' or 'neg ctr')

## Outputs

- causal_graph.pkl — pickled nested dictionary of gene-index to TF-index sets, one each for the positive and negative control GRNs
- Two equal-density bipartite TF-gene control networks imposed on GRouNdGAN training

## How to apply

Preprocess the reference expression matrix (e.g., CD8+ cytotoxic T-cell PBMC-CTL data from PBMC-All) with GRouNdGAN's --preprocess step, keeping 1000 highly variable genes and library-size normalizing to 20,000. Run --create_grn so GRNBoost2 infers TF-gene rankings using k = 15 top most important TFs per gene and the Homo_sapiens_TF.csv transcription factor list. Split each gene's ranked TF list into two disjoint control GRNs: strategy = 'pos ctr' assigns even indices (0, 2, 4, ...) — i.e., top 1st, 3rd, 5th TFs — and strategy = 'neg ctr' assigns odd indices (1, 3, 5, ...) — i.e., top 2nd, 4th, 6th TFs — yielding two equal-density bipartite networks. Pickle each as a causal_graph.pkl nested dictionary mapping gene indices to sets of TF indices, then impose the positive control during GRouNdGAN training so downstream --benchmark_grn (compute_pr = True) can score reconstructed GRNs against this ground truth.

## Related tools

- **GRouNdGAN** (Framework whose --create_grn command runs GRNBoost2 and formats the rankings into a causal graph; control-GRN strategy settings ('pos ctr'/'neg ctr') select even/odd index splits for imposition during GAN training) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the ranked TF-gene importance lists (top k = 15 per gene) from the preprocessed PBMC-CTL training data from which the parity-split control GRNs are built) — https://github.com/Emad-COMBINE-lab/GRouNdGAN

## Examples

```
groundgan --create_grn --config config.yaml  # then build 'pos ctr'/'neg ctr' parity GRNs and pickle each as causal_graph.pkl
```

## Evaluation signals

- Positive and negative control GRNs have equal edge density (same number of TF-gene edges, split by even/odd index)
- Each control GRN is a valid nested dictionary (gene-index -> set of TF indices) serializable as causal_graph.pkl
- After imposing the positive control and simulating, --benchmark_grn with compute_pr = True yields AUPRC for GRNBoost2, GENIE3, PIDC, and PPCOR well above the random-predictor baseline on imposed edges
- Edges belonging to the unimposed (negative control) GRN score near random in reconstructed GRNs, reproducing the Fig. 3 / Supplementary Data 3 pattern
- Reconstructed imposed-edge AUPRC on simulated data exceeds AUPRC obtained on the reference (unmodified) data

## Limitations

- The causal graph of an observational dataset is not necessarily unique; verifying that the imposed GRN is the most probable one was beyond the study's scope
- Resemblance of simulated data to the reference deteriorates if imposed TF-gene relationships differ significantly from patterns consistent with the reference data
- GRouNdGAN currently only supports bipartite GRNs; multi-layer GRNs are not supported and planned for future versions

## Evidence

- [methods] GRNBoost2 rankings split into parity control GRNs: "This command uses GRNBoost2 (Moerman et al., 2018) to infer a GRN on the preprocessed train set."
- [discussion] Recommended workflow: infer edges then impose subset: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities) and imposing a selected subset of those in the simulated data"
- [methods] Custom GRN imposition skips create_grn: "It is possible to instead impose your own GRN onto GRouNdGAN. If you're opting for this option, skip the ``--create_grn`` command."
- [abstract] Findings: gold standard ground truth GRNs for benchmarking: "GRouNdGAN effectively bridges the existing gap between simulated and biological data benchmarks of GRN inference algorithms, providing gold standard ground truth GRNs"
