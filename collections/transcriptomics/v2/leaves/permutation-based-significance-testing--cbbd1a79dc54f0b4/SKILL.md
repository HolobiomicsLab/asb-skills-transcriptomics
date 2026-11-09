---
name: permutation-based-significance-testing
description: Use when benchmarking GRN inference methods (e.g., SCING, GRNBOOST2,
  PIDC, ppcor) on Perturb-seq data where significant downstream targets of guides
  must be identified to compute AUROC and TPR at FPR 0.05.
when_to_use_negative:
- The dataset contains no Perturb-seq perturbation guides or ground-truth perturbed
  gene labels, so the permutation null has no meaningful labels to compare against
- The goal is module discovery, pathway enrichment (enrichr), or AUCell module scoring
  from a SCING GRN rather than perturbation target prediction benchmarking
- Applied to spatial transcriptomics GRNs, where the article notes SCING does not
  utilize spatial information during network construction and no perturbation truth
  exists
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: GRN inference method under evaluation; its merged, edge-pruned network supplies
    edges for perturbation target prediction with directed and undirected evaluation
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Gradient-boosting baseline GRN inference method whose network edges are scored
    with the same permutation-tested perturbation prediction pipeline
- name: PIDC
  role: Baseline GRN inference method benchmarked on the same Perturb-seq datasets
    via perturbation target prediction
- name: ppcor
  role: Partial correlation baseline GRN inference method scored alongside SCING in
    the AUROC/TPR-at-FPR-0.05 comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:ed825f752ec1e63b3c7d4e44d373626f4a0842442998ee336c8f1cb3b0f94b3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# permutation-based-significance-testing

## Summary

Permutation-based significance testing in the SCING benchmark is used to call significantly perturbed downstream targets when evaluating GRN inference methods, by permuting perturbation effects to establish a null against elastic net predictions on Perturb-seq data.

## When to use

Use when benchmarking GRN inference methods (e.g., SCING, GRNBOOST2, PIDC, ppcor) on Perturb-seq data where significant downstream targets of guides must be identified to compute AUROC and TPR at FPR 0.05.

## When NOT to use

- The dataset contains no Perturb-seq perturbation guides or ground-truth perturbed gene labels, so the permutation null has no meaningful labels to compare against
- The goal is module discovery, pathway enrichment (enrichr), or AUCell module scoring from a SCING GRN rather than perturbation target prediction benchmarking
- Applied to spatial transcriptomics GRNs, where the article notes SCING does not utilize spatial information during network construction and no perturbation truth exists

## Inputs

- Perturb-seq count matrices from GEO (GSE90063, Dixit et al.; GSE153056, Papalexi et al.), preprocessed for GRN inference
- Inferred GRNs per method (SCING, GRNBOOST2, PIDC, ppcor), with edges treated as undirected (SCING also evaluated with directed edges)

## Outputs

- Significantly perturbed downstream target calls per guide (permutation-tested elastic net predictions)
- Per-method, per-dataset AUROC and TPR-at-FPR-0.05 values compiled in perturbseq_benchmark_results.csv

## How to apply

For each method's GRN, fit an elastic net regression model to predict the perturbation effect on downstream genes of a perturbed guide, then use permutation testing to determine which downstream targets are significantly perturbed, producing the label sets needed for target-prediction scoring. Perturbed downstream genes of guides for specific genes are identified, and perturbed genes are predicted at each depth in the network from the perturbed gene, with true positive rate and false positive rate determined at each depth. Score each method on each dataset by computing AUROC and TPR at an FPR of 0.05, treating edges as undirected for all methods and additionally evaluating SCING with directed edges. Compile the AUROC and TPR-at-FPR-0.05 values into a table keyed by method and dataset (perturbseq_benchmark_results.csv), accepting it only if it has one AUROC and one TPR-at-FPR-0.05 entry per method per Perturb-seq dataset (Dixit et al. GSE90063 and Papalexi et al. GSE153056), with SCING directed results reported separately.

## Related tools

- **SCING** (GRN inference method under evaluation; its merged, edge-pruned network supplies edges for perturbation target prediction with directed and undirected evaluation) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Gradient-boosting baseline GRN inference method whose network edges are scored with the same permutation-tested perturbation prediction pipeline)
- **PIDC** (Baseline GRN inference method benchmarked on the same Perturb-seq datasets via perturbation target prediction)
- **ppcor** (Partial correlation baseline GRN inference method scored alongside SCING in the AUROC/TPR-at-FPR-0.05 comparison)

## Evaluation signals

- The output table contains exactly one AUROC and one TPR-at-FPR-0.05 entry for each of the four methods on each of the two Perturb-seq datasets
- SCING directed-edge evaluation is reported separately from the undirected edges used for all methods
- Each perturbation call is supported by a permutation test null rather than raw elastic net coefficients alone
- Metrics (AUROC, TPR at FPR 0.05) are computed from network depth traversals from the perturbed gene, so TPR/FPR pairs exist at each depth
- Cross-method comparison uses the same two datasets (GSE90063 and GSE153056) and identical target definitions, so differences reflect inference quality

## Limitations

- Inferred GRN links are not necessarily causal or directional, limiting interpretation of downstream prediction directionality
- Many hyperparameters require selection and selection can be dataset dependent; optimization of hyperparameters is needed per dataset
- SCING's benchmark advantage holds when sparsity is adjusted; it outperforms only select methods when all cells are used

## Evidence

- [other] For each method's GRN, fit an elastic net regression model to predict the perturbation effect on downstream genes and use permutation testing to identify significantly perturbed downstream targets.: [evidence span withheld — anchor status: not_found]
- [readme] We identified downstream perturbed genes of guides for specific genes. We then predict perturbed genes at each depth in the network from the perturbed gene.: "We identified downstream perturbed genes of guides for specific genes. We then predict perturbed genes at each depth in the network from the perturbed gene."
- [readme] We utilize AUROC and TPR at FPR 0.05 as metrics for evaluation.: "True positive rate, and false positive rate are determined at each depth in the network. We utilize AUROC and TPR at FPR 0.05 as metrics for evaluation."
- [intro] We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy.: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data"
