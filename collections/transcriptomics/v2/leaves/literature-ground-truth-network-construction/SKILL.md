---
name: literature-ground-truth-network-construction
description: Use when benchmarking gene network inference methods and a literature-based
  directed binary ground truth (Omnipath source→target edges) is needed to compute
  AUPRC/EPR per cell type.
when_to_use_negative:
- The goal is to improve method performance using prior knowledge rather than to benchmark
  — the article notes 'general collections like Omnipath did not improve performance
  in most of our tests'
- A causal or perturbation-derived ground truth is required — literature-based ChIP-seq/perturb-seq/literature
  ground truths are 'scarce and ambiguous' per the discussion
- The inferred network's edge orientation cannot be interpreted as regulator→target,
  since the transpose-and-match step assumes regulators align with Omnipath sources
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Omnipath
  role: Source of the literature-curated directed binary ground-truth network via
    the Omnipath Python package
- name: BenGRN
  role: Benchmarking package used to compute prevalence-floored AUPRC and EPR against
    the ground truth
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Gene regulatory network data package used alongside BenGRN for scoring inferred
    networks
  repo: https://github.com/cantinilab/GRnnData
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:a646958d76e353e1efb7ff51128414b95df8160c74fa1f35855a7f0bd914e896
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jkobject/benGRN
  tools:
  - BenGRN
  - DeepSEM
  - GRnnData
  - Geneformer
  - Omnipath
  - Python
  - scGPT
  - scPRINT
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# literature-ground-truth-network-construction

## Summary

Build a literature-curated, directed binary source-to-target gene network (e.g. from the Omnipath Python package) to serve as the ground truth when scoring inferred gene networks with prevalence-floored AUPRC and EPR metrics.

## When to use

Use when benchmarking gene network inference methods and a literature-based directed binary ground truth (Omnipath source→target edges) is needed to compute AUPRC/EPR per cell type.

## When NOT to use

- The goal is to improve method performance using prior knowledge rather than to benchmark — the article notes 'general collections like Omnipath did not improve performance in most of our tests'
- A causal or perturbation-derived ground truth is required — literature-based ChIP-seq/perturb-seq/literature ground truths are 'scarce and ambiguous' per the discussion
- The inferred network's edge orientation cannot be interpreted as regulator→target, since the transpose-and-match step assumes regulators align with Omnipath sources

## Inputs

- Omnipath interactions from the Omnipath Python package (all interactions, directed, HGNC symbols)
- Single-cell RNA-seq expression profiles per cell type (e.g. 1024 cells per kidney, retina, colon cell type)
- List of target genes (e.g. 5000 most differentially expressed genes per cell type)
- Inferred gene network(s) (gene-by-gene weight matrices from scPRINT, scGPT, Geneformer, DeepSEM, GENIE3)

## Outputs

- Directed binary Omnipath ground-truth network (source→target adjacency over HGNC symbols)
- Per-method, per-cell-type AUPRC and EPR values assembled into a results table (e.g. omnipath_benchmark_results.csv)
- Fraction of scPRINT network connections recovered relative to GENIE3

## How to apply

Generate the ground truth using all interactions from the Omnipath Python package, excluding small molecules, lncRNAs, and any element without a unique HGNC symbol, and encode edges as a directed binary source-to-target network. Restrict comparisons to the gene set under study (e.g. the 5000 most differentially expressed genes per cell type). Because Omnipath sources have sparse out-degree, take the transpose of each inferred network so that regulators align with sources and targets align with regulators before comparison. Score with the BenGRN/GRnnData packages: compute EPR as the diagnostic odds ratio (TP×TN)/(FP×FN) at the cutoff giving K equal to the number of ground-truth positives, plus prevalence-floored AUPRC. A predictor must achieve EPR > 1 to beat random (EPR = 1 is random, infinity is perfect).

## Related tools

- **Omnipath** (Source of the literature-curated directed binary ground-truth network via the Omnipath Python package)
- **BenGRN** (Benchmarking package used to compute prevalence-floored AUPRC and EPR against the ground truth) — https://github.com/jkobject/benGRN
- **GRnnData** (Gene regulatory network data package used alongside BenGRN for scoring inferred networks) — https://github.com/cantinilab/GRnnData

## Examples

```
import omnipath; edges = omnipath.database.get_db().interactions.AllInteractions.get(genesymbols=True)
```

## Evaluation signals

- Every ground-truth element maps to a unique HGNC symbol (small molecules, lncRNAs, and non-unique symbols were excluded)
- Edge orientation matches after transposing inferred networks: regulators correspond to Omnipath sources, targets to regulators
- EPR is computed at the cutoff giving K = number of ground-truth positives, and any method scoring EPR ≤ 1 fails the 'beats random' acceptance check
- AUPRC is prevalence-floored so that scores remain interpretable given the low fraction of true Omnipath edges among all gene pairs
- Per-method, per-cell-type values for all 26 cell types are present in the assembled CSV (no missing AUPRC/EPR entries)

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; literature-based ground truths are inherently incomplete and biased toward well-studied interactions
- Omnipath sources have sparse out-degree, requiring a transpose of inferred networks and making orientation errors easy to introduce
- Many GN methods are instead benchmarked on ODE-generated mock-up expression data rather than real ground truths, limiting comparability with other published evaluations

## Evidence

- [methods] Omnipath network build from Python package: "We generate the Omnipath network using all the interactions from the Omnipath Python package"
- [methods] Filtering of non-gene elements: "excluding small molecules, lncRNAs, and any element without a unique HGNC symbol"
- [discussion] No perfect ground truths: "A challenging aspect of GN inference is that no perfect ground truths exist"
- [discussion] Omnipath as prior knowledge caveat: "general collections like Omnipath did not improve performance in most of our tests"
- [discussion] Benchmarking suite used: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
