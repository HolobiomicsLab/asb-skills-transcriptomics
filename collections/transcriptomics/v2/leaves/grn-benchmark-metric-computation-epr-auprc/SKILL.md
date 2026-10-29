---
name: grn-benchmark-metric-computation-epr-auprc
description: Use when comparing inferred gene networks (from scPRINT, GENIE3, scGPT,
  DeepSEM, Geneformer, etc.) against a binary perturb-seq or literature ground truth
  and you need comparable AUPRC/EPR scores.
when_to_use_negative:
- The ground truth is not binary or cannot be thresholded into directed gene-by-gene
  edges (e.g. raw weighted association matrices without a defensible cutoff)
- Inferred networks cannot be aligned to the ground truth gene set and orientation
  (regulators as columns, targets as rows), since mismatched dimensions invalidate
  TP/TN/FP/FN counts
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: BenGRN
  role: Benchmarking suite used to match inferred networks to ground truth and compute
    AUPRC/EPR metrics
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Gene regulatory network data structure package used alongside BenGRN for benchmarking
    inferred networks
  repo: https://github.com/cantinilab/GRnnData
- name: scPRINT
  role: Source of inferred gene networks (averaged Keys/Queries attention embeddings,
    with or without head selection) being scored
  repo: https://github.com/cantinilab/scPRINT
- name: GENIE3
  role: Comparator GRN inference method (default parameters, 100 trees on raw counts)
    scored with the same metrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:bbabd5446d9f3d339c9693efab70982705ebc0c8fc0f008533ecbdda8cc64afa
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
  - Python
  - scGPT
  - scPRINT
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# GRN benchmark metric computation (EPR and AUPRC)

## Summary

Scores inferred gene regulatory networks against a binarized ground truth (e.g. the genome-wide perturb-seq K562 network at a 0.05 cutoff on corrected p-values) using AUPRC above the positive-class prevalence floor and EPR, a diagnostic-odds-ratio-based enrichment metric, via the benGRN and GRnnData packages.

## When to use

Use when comparing inferred gene networks (from scPRINT, GENIE3, scGPT, DeepSEM, Geneformer, etc.) against a binary perturb-seq or literature ground truth and you need comparable AUPRC/EPR scores.

## When NOT to use

- The ground truth is not binary or cannot be thresholded into directed gene-by-gene edges (e.g. raw weighted association matrices without a defensible cutoff)
- Inferred networks cannot be aligned to the ground truth gene set and orientation (regulators as columns, targets as rows), since mismatched dimensions invalidate TP/TN/FP/FN counts

## Inputs

- Inferred gene network score matrices (genes x genes, regulators as columns, targets as rows)
- Binarized directed ground-truth network, e.g. gwps K562 supplementary multiple-hypothesis-testing-corrected p-value matrix thresholded at 0.05
- benGRN and GRnnData Python packages for network matching and metric computation

## Outputs

- Per-method AUPRC (area above prevalence floor) values
- Per-method EPR (diagnostic odds ratio at K-positive cutoff) values
- Aggregated metrics table, e.g. gwps_benchmark_metrics.csv

## How to apply

Match each inferred network to the ground truth with regulators as columns and sparsely regulated targets as rows (no transposition), using benGRN and GRnnData. Compute AUPRC as the area above the positive-class prevalence floor using the composite trapezoidal rule without interpolation to perfect recall, so that a random classifier scores near 0 rather than the prevalence. Compute EPR as the diagnostic odds ratio (TP x TN) / (FP x FN) at the score cutoff that yields K positive predictions, where K equals the number of ground-truth positives, making EPR robust to score scale differences between methods. For the gwps K562 benchmark, ground-truth edges are defined with a 0.05 cutoff on the multiple-hypothesis-testing-corrected p-values over 8000+ expressed genes. Assemble per-method AUPRC and EPR values into gwps_benchmark_metrics.csv and verify ranking claims (e.g. scPRINT with gwps-selected heads outperforming GENIE3).

## Related tools

- **BenGRN** (Benchmarking suite used to match inferred networks to ground truth and compute AUPRC/EPR metrics) — https://github.com/jkobject/benGRN
- **GRnnData** (Gene regulatory network data structure package used alongside BenGRN for benchmarking inferred networks) — https://github.com/cantinilab/GRnnData
- **scPRINT** (Source of inferred gene networks (averaged Keys/Queries attention embeddings, with or without head selection) being scored) — https://github.com/cantinilab/scPRINT
- **GENIE3** (Comparator GRN inference method (default parameters, 100 trees on raw counts) scored with the same metrics)

## Examples

```
from bengrn import BenGRN; grn = GRnnData.load(inferred_matrix, gene_names); b = BenGRN(grn, gt); b.compare_to(other=gt_network)
```

## Evaluation signals

- A random or all-zero network yields AUPRC near 0 because the positive-class prevalence floor is subtracted (no interpolation to perfect recall)
- EPR equals 1 when the inferred network's precision at the K-positive cutoff matches the ground-truth prevalence; values >1 indicate enrichment
- Network orientation check: regulators as columns and sparsely regulated targets as rows, no transposition, before computing TP/TN/FP/FN
- Ground-truth positives derive from the 0.05 cutoff on multiple-hypothesis-testing-corrected gwps p-values over 8000+ expressed genes
- Reproduced ranking should show scPRINT with gwps-selected heads outperforming GENIE3 in the assembled gwps_benchmark_metrics.csv

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; perturb-seq and other ground truths remain scarce and ambiguous
- EPR depends on choosing the cutoff that yields exactly K positive predictions, coupling the metric to the ground-truth positive count
- AUPRC is computed without interpolation to perfect recall, so absolute values are lower than conventionally interpolated AUPRCs and not directly comparable to other papers' numbers

## Evidence

- [methods] We used a cutoff of 0.05 on these values to define the directed binary connection between genes: "We used a cutoff of 0.05 on these values to define the directed binary connection between genes"
- [methods] RidgeClassifier head selection: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [methods] Averaging attention for network extraction: [evidence span withheld — anchor status: not_found]
- [discussion] Ground truth scarcity caveat: "A challenging aspect of GN inference is that no perfect ground truths exist"
- [discussion] Benchmarking tooling: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
