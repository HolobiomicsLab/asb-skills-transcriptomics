---
name: auprc-epr-network-metrics-computation
description: Use when comparing inferred gene regulatory networks (e.g., from scPRINT,
  scGPT, DeepSEM, Geneformer, GENIE3) against a binary ground-truth network and you
  need EPR and AUPRC scores to rank methods.
when_to_use_negative:
- When no cell-type-specific gold-standard edges exist for the genes in the inferred
  network — the article notes 'no perfect ground truths exist' and ChIP-seq/perturb-seq
  ground truths remain scarce and ambiguous.
- When the ground truth is literature-based rather than ChIP-seq/perturb-seq, if the
  transposition convention (rows=targets, columns=regulators) specific to ChIP-seq/perturb-seq
  would then be misapplied.
- When benchmarking on ODE-generated mock-up expression data rather than real ground
  truths, which the authors criticize as uninformative.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: BenGRN
  role: Package used to compute EPR and AUPRC benchmarking metrics against ground-truth
    networks
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Gene regulatory network data structure package used together with BenGRN for
    metric computation
  repo: https://github.com/cantinilab/GRnnData
- name: scPRINT
  role: Produces the inferred gene networks (averaged Keys/Queries attention matrices)
    that are scored with EPR/AUPRC
  repo: https://github.com/cantinilab/scPRINT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:bab0d4de2973705aa92a3bbe4743d938651a64551a7b0c487af099e5f8ac4d8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jkobject/benGRN
  tools:
  - BenGRN
  - DeepSEM
  - GRnnData
  - Geneformer
  - Python
  - scGPT
  - scPRINT
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# AUPRC-EPR network metrics computation

## Summary

Compute EPR (early precision ratio) and AUPRC to benchmark inferred gene networks against ChIP-seq/perturb-seq gold-standard directed binary graphs, using the BenGRN/GRnnData packages as done in the scPRINT benchmarks on hESC/mESC datasets.

## When to use

Use when comparing inferred gene regulatory networks (e.g., from scPRINT, scGPT, DeepSEM, Geneformer, GENIE3) against a binary ground-truth network and you need EPR and AUPRC scores to rank methods.

## When NOT to use

- When no cell-type-specific gold-standard edges exist for the genes in the inferred network — the article notes 'no perfect ground truths exist' and ChIP-seq/perturb-seq ground truths remain scarce and ambiguous.
- When the ground truth is literature-based rather than ChIP-seq/perturb-seq, if the transposition convention (rows=targets, columns=regulators) specific to ChIP-seq/perturb-seq would then be misapplied.
- When benchmarking on ODE-generated mock-up expression data rather than real ground truths, which the authors criticize as uninformative.

## Inputs

- Inferred gene network score matrices (e.g., averaged attention matrices from scPRINT/scGPT, DeepSEM/Geneformer/GENIE3 networks)
- Gold-standard directed binary ground-truth networks (source-to-target lists converted to a gene-by-gene binary graph)
- Gene name ordering / network dimension mapping between inferred and ground-truth networks

## Outputs

- EPR value per method-dataset pair (odds-ratio at K = number of ground-truth positives)
- AUPRC value per method-dataset pair (floored at positive-class prevalence)
- Results table (e.g., esc_benchmark_results.csv) with AUPRC and EPR entries for every method-dataset combination

## How to apply

Match each inferred network to the ground truth respecting the convention that row elements are targets and column elements are regulators, using the non-transposed orientation for ChIP-seq/perturb-seq ground truths. Compute EPR as the diagnostic odds ratio (TP x TN) / (FP x FN) at the score cutoff yielding K positive predictions, where K equals the number of ground-truth positives; EPR near 1 indicates random performance and values above 1 indicate better-than-random edge recovery. Compute AUPRC with the area floored at the positive-class prevalence and no interpolation to perfect recall, using the BenGRN/GRnnData packages. Assemble metric values for every method-dataset combination (e.g., the two human and two mouse ESC datasets) into a results table such as esc_benchmark_results.csv.

## Related tools

- **BenGRN** (Package used to compute EPR and AUPRC benchmarking metrics against ground-truth networks) — https://github.com/jkobject/benGRN
- **GRnnData** (Gene regulatory network data structure package used together with BenGRN for metric computation) — https://github.com/cantinilab/GRnnData
- **scPRINT** (Produces the inferred gene networks (averaged Keys/Queries attention matrices) that are scored with EPR/AUPRC) — https://github.com/cantinilab/scPRINT

## Examples

```
from bengrn import BenGRN; grn = GRnnData.GRnnData.from_scanpy(adata); ben = BenGRN(grn); ben.compare_to(other_net, true_net=gt_net, type='chip')
```

## Evaluation signals

- EPR near 1 indicates random edge recovery; verify EPR > 1 for methods claimed to outperform random.
- Verify the results table contains an AUPRC and EPR entry for every method-dataset combination (e.g., all methods on both human and both mouse ESC datasets).
- Confirm network orientation: row elements are targets and column elements are regulators; use the non-transposed orientation for ChIP-seq/perturb-seq ground truths.
- Check AUPRC was computed with the area floored at the positive-class prevalence and without interpolation to perfect recall.
- Confirm EPR cutoff uses K equal to the number of ground-truth positive predictions.

## Limitations

- No perfect ground truths exist for gene network inference benchmarking; ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous.
- EPR and AUPRC are sensitive to the transposition/orientation convention of the ground-truth network; mismatched row/column roles invalidate the scores.
- AUPRC floor at positive-class prevalence means reported values do not reach the theoretical 1.0 for perfect recovery.

## Evidence

- [methods] Compute EPR as the diagnostic odds ratio: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [intro] Benchmarks like BeeLine and MCalla et al. have shown GN inference remains challenging: "Benchmarks like BeeLine27 and MCalla et al.28 have shown that despite the existence of many methods, GN inference remains a challenging problem."
- [discussion] With BenGRN and GRnnData, our suite of tools for benchmarking: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [discussion] no perfect ground truths exist for gene network inference: "A challenging aspect of GN inference is that no perfect ground truths exist"
- [methods] attention-based network extraction averaged over cells: [evidence span withheld — anchor status: not_found]
