---
name: network-transpose-normalization
description: Use when comparing predicted gene networks from attention- or regression-based
  inference methods to a directed source-to-target ground truth (e.g., Omnipath) whose
  regulators have sparse out-degree, so predicted edge matrices must be transposed
  to align regulator/target conventions.
when_to_use_negative:
- The predicted network and the ground-truth database already share the same regulator/target
  axis convention — transposing would corrupt the comparison
- The ground truth is undirected (e.g., literature co-mention networks) where orientation
  is meaningless
- The inference method already outputs edges in the source-to-target convention of
  the benchmark (e.g., methods explicitly trained to predict regulators of targets)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Omnipath
  role: Provides the directed source-to-target ground-truth interaction database that
    dictates the orientation convention
- name: BenGRN
  role: Benchmarking package used to compute EPR and AUPRC on the orientation-aligned
    networks
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Gene network data structure package used alongside BenGRN for storing and
    evaluating inferred networks
  repo: https://github.com/cantinilab/GRnnData
- name: scPRINT
  role: Source of inferred gene networks (attention-head averages, RidgeClassifier-selected
    heads, genome-wide network) that require transposition before comparison
  repo: https://github.com/cantinilab/scPRINT
- name: GENIE3
  role: Baseline GRN method (default parameters, 100 trees) whose output network is
    transposed for the Omnipath comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:131c64149740a8996834d91a64dad6df363b46d1d2ceda884ddc546155411762
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jkobject/benGRN
  tools:
  - BenGRN
  - DeepSEM
  - GENIE3
  - GRnnData
  - Geneformer
  - Omnipath
  - Python
  - scGPT
  - scPRINT
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# network-transpose-normalization

## Summary

Normalizes the orientation of inferred gene regulatory networks (regulators vs. targets) before benchmarking against a directed ground-truth database such as Omnipath, by taking the transpose of predicted networks when the ground truth's edge direction convention (sparse out-degree of sources) differs from the inference method's convention.

## When to use

Use when comparing predicted gene networks from attention- or regression-based inference methods to a directed source-to-target ground truth (e.g., Omnipath) whose regulators have sparse out-degree, so predicted edge matrices must be transposed to align regulator/target conventions.

## When NOT to use

- The predicted network and the ground-truth database already share the same regulator/target axis convention — transposing would corrupt the comparison
- The ground truth is undirected (e.g., literature co-mention networks) where orientation is meaningless
- The inference method already outputs edges in the source-to-target convention of the benchmark (e.g., methods explicitly trained to predict regulators of targets)

## Inputs

- Inferred gene network adjacency matrices (e.g., scPRINT attention-head averages, scGPT/Geneformer attention, DeepSEM, GENIE3) restricted to the 5000 most differentially expressed genes per cell type
- Omnipath directed binary source-to-target network built from the Omnipath Python package

## Outputs

- Orientation-aligned (transposed) network matrices ready for ground-truth comparison
- Per-method, per-cell-type EPR and prevalence-floored AUPRC values (e.g., assembled into omnipath_benchmark_results.csv)

## How to apply

Build the ground truth as a directed binary source-to-target network from the Omnipath Python package, excluding small molecules, lncRNAs, and elements without a unique HGNC symbol. Because the ground-truth sources have sparse out-degree, take the transpose of each inferred network (e.g., scPRINT's head-averaged attention networks, scGPT's last-layer head-averaged attention, Geneformer's per-cell head means, DeepSEM, and GENIE3 with 100 trees) so rows/columns match regulators-to-sources and targets-to-regulators. Then compute EPR at the cutoff where K equals the number of ground-truth positives and prevalence-floored AUPRC with BenGRN/GRnnData on the oriented networks.

## Related tools

- **Omnipath** (Provides the directed source-to-target ground-truth interaction database that dictates the orientation convention)
- **BenGRN** (Benchmarking package used to compute EPR and AUPRC on the orientation-aligned networks) — https://github.com/jkobject/benGRN
- **GRnnData** (Gene network data structure package used alongside BenGRN for storing and evaluating inferred networks) — https://github.com/cantinilab/GRnnData
- **scPRINT** (Source of inferred gene networks (attention-head averages, RidgeClassifier-selected heads, genome-wide network) that require transposition before comparison) — https://github.com/cantinilab/scPRINT
- **GENIE3** (Baseline GRN method (default parameters, 100 trees) whose output network is transposed for the Omnipath comparison)

## Evaluation signals

- The transposed network's row/column gene order matches the regulator/target convention of the Omnipath ground-truth matrix before computing overlaps
- EPR must exceed 1 for a predictor to beat random (EPR = 1 is random, infinity is perfect) after orientation alignment
- A sanity check: without transposition, edge-overlap statistics against the sparse-out-degree Omnipath sources collapse to near-random levels, while after transposition true-positive recovery is consistent with expected source connectivity
- AUPRC values remain at or above the prevalence floor (fraction of ground-truth positives in the evaluated gene set) across all 26 cell types
- Acceptance requires scPRINT variants to outperform GENIE3 on the Omnipath benchmark after consistent orientation handling across all methods

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; ChIP-seq, perturb-seq, and literature-based ground truths such as Omnipath remain scarce and ambiguous
- Transposition only harmonizes orientation conventions — it cannot correct for genuinely reversed causal directions or ambiguous literature curation in the ground truth
- General collections like Omnipath did not improve performance in most of the paper's tests, so orientation-normalized Omnipath scores may not reflect performance on cell-type-specific causal networks

## Evidence

- [methods] Omnipath ground truth is directed and built from the Omnipath Python package with element filtering: "We generate the Omnipath network using all the interactions from the Omnipath Python package"
- [methods] Ground-truth element filtering excludes small molecules, lncRNAs, and non-unique HGNC symbols: "excluding small molecules, lncRNAs, and any element without a unique HGNC symbol"
- [discussion] BenGRN and GRnnData are the benchmarking suite for evaluating gene networks inferred from scRNA-seq: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [other] Omnipath sources have sparse out-degree, motivating transposing inferred networks so regulators match sources: [evidence span withheld — anchor status: not_found]
