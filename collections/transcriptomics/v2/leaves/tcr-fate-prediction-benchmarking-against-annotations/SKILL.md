---
name: tcr-fate-prediction-benchmarking-against-annotations
description: Use when a T-cell trajectory method (e.g., VDJ feature space vs expression-only
  PCs) must be validated by comparing predicted cell fates against existing cell-type
  annotations such as CD8+T, CD4+T, DP(P)_T and DP(Q)_T labels.
when_to_use_negative:
- The dataset lacks paired single-cell VDJ/TCR data with complete, productive TCR
  information (e.g., datasets with missing BCR/TCR contigs or too few cells, such
  as Tonglin et al.'s 972 healthy B cells, insufficient even for miloR neighbourhood
  construction)
- The trajectory topology is cyclic or non-bifurcating (e.g., B cell receptor editing),
  since absorbing Markov chains and the destiny diffusion map cannot represent cyclic
  trajectories
- No trusted cell-type annotations exist to serve as ground truth for scoring fate
  assignments
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_2830
tools:
- name: dandelionR
  role: R package providing vdjPseudobulk VDJ feature space construction and projectPseudotimeToCell
    for projecting pseudobulk branch probabilities back to cells
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Builds the neighbourhood graph (buildGraph k = 30, d = 20 on X_scvi; makeNhoods
    prop = 0.3) used for VDJ pseudobulking
- name: destiny
  role: DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT (w_width = 0.1) used for trajectory
    inference on both VDJ feature space and expression-only PCs
- name: Dandelion
  role: Original Python implementation whose VDJ feature space improved DP-to-CD4/CD8
    trajectory prediction; the R workflow is benchmarked against its reported improvement
  repo: https://www.github.com/zktuong/dandelion
- name: scRepertoire
  role: loadContigs (AIRR format), combineTCR (removeNA = TRUE, removeMulti = FALSE,
    filterMulti = TRUE) and combineExpression to attach TCR data to the SingleCellExperiment
  repo: https://github.com/ncborcherding/scRepertoire
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:b4c76a7ee05ff600bcf37e35b8d85a51458c5a85c0fcbb538b238b210782e51c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - destiny
  - miloR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tcr-fate-prediction-benchmarking-against-annotations

## Summary

Benchmarks T-cell fate prediction from trajectory analysis (VDJ feature space vs gene-expression-only baseline) by projecting pseudotime and branch probabilities back to cells and scoring predicted CD4/CD8 fate assignments against annotated cell-type labels, as in the dandelionR reproduction of Dandelion's DP-to-CD4/CD8 trajectory improvement.

## When to use

Use when a T-cell trajectory method (e.g., VDJ feature space vs expression-only PCs) must be validated by comparing predicted cell fates against existing cell-type annotations such as CD8+T, CD4+T, DP(P)_T and DP(Q)_T labels.

## When NOT to use

- The dataset lacks paired single-cell VDJ/TCR data with complete, productive TCR information (e.g., datasets with missing BCR/TCR contigs or too few cells, such as Tonglin et al.'s 972 healthy B cells, insufficient even for miloR neighbourhood construction)
- The trajectory topology is cyclic or non-bifurcating (e.g., B cell receptor editing), since absorbing Markov chains and the destiny diffusion map cannot represent cyclic trajectories
- No trusted cell-type annotations exist to serve as ground truth for scoring fate assignments

## Inputs

- SingleCellExperiment object with VDJ-informed trajectory results (pseudobulk feature space, diffusion map, branch probabilities), e.g. sce_vdj / demo_sce + demo_airr from the fetal immune atlas lymphoid cells
- Annotated cell-type labels per cell (CD8+T, CD4+T, ABT(ENTRY), DP(P)_T, DP(Q)_T populations)
- Pseudobulk-level pseudotime and branch probabilities from both VDJ-based and expression-only trajectory runs

## Outputs

- Cell-level projected pseudotime and CD4/CD8 branch probabilities (projectPseudotimeToCell output)
- Fate prediction accuracy for the VDJ feature space run
- Fate prediction accuracy for the gene-expression-only baseline
- trajectory_accuracy_comparison.csv containing both accuracy values

## How to apply

After running trajectory inference on both the VDJ feature space (built with dandelionR::vdjPseudobulk on a miloR neighbourhood graph over X_scvi) and a matched gene-expression-only baseline, project pseudotime and branch probabilities from pseudobulks back to individual cells with projectPseudotimeToCell. Compute branch probabilities with markovProbability (knn = 30), defining the root as the right-most pseudobulk along PC1 and CD8/CD4 branch tips as the extremes along PC2 of the diffusion map (destiny::DiffusionMap with n_pcs = 50, n_eigs = 10; DPT with w_width = 0.1). For each run, score predicted CD4/CD8 fate assignments against the annotated cell-type labels (e.g., fraction of cells whose highest branch probability matches the annotated CD4+T or CD8+T fate) and write both accuracies to trajectory_accuracy_comparison.csv. The rationale is that Dandelion's VDJ feature space, which integrates VDJ recombination information with gene expression, should improve fate prediction from double-positive T cells relative to expression-only tools, and dandelionR's workflow must reproduce that improvement in R.

## Related tools

- **dandelionR** (R package providing vdjPseudobulk VDJ feature space construction and projectPseudotimeToCell for projecting pseudobulk branch probabilities back to cells) — https://github.com/tuonglab/dandelionR
- **miloR** (Builds the neighbourhood graph (buildGraph k = 30, d = 20 on X_scvi; makeNhoods prop = 0.3) used for VDJ pseudobulking)
- **destiny** (DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT (w_width = 0.1) used for trajectory inference on both VDJ feature space and expression-only PCs)
- **Dandelion** (Original Python implementation whose VDJ feature space improved DP-to-CD4/CD8 trajectory prediction; the R workflow is benchmarked against its reported improvement) — https://www.github.com/zktuong/dandelion
- **scRepertoire** (loadContigs (AIRR format), combineTCR (removeNA = TRUE, removeMulti = FALSE, filterMulti = TRUE) and combineExpression to attach TCR data to the SingleCellExperiment) — https://github.com/ncborcherding/scRepertoire

## Examples

```
sce <- projectPseudotimeToCell(sce, pb.milo, dm, dif.pse, knn = 30); write.csv(trajectory_accuracy_comparison, "trajectory_accuracy_comparison.csv")
```

## Evaluation signals

- The VDJ feature space trajectory achieves higher CD4/CD8 fate prediction accuracy than the gene-expression-only baseline, consistent with the paper's reported improvement
- Both accuracy values are written to trajectory_accuracy_comparison.csv with the VDJ-based run exceeding the expression-only run
- Projected cell-level branch probabilities correctly assign the majority of CD8+T-annotated cells to the CD8 branch tip and CD4+T-annotated cells to the CD4 branch tip
- The root is defined at the right-most pseudobulk along PC1 and branch tips at the extremes along PC2, matching the DP(P)_T/DP(Q)_T to CD4/CD8 topology
- The retained cell set is consistent with filtering for complete TCR information (Suo et al. data retained 17,308 of 65,102 cells)

## Limitations

- destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies, so benchmarking beyond two fates is not supported
- Cyclic trajectories (e.g. B cell receptor editing) cannot be represented by absorbing Markov chains, so the benchmark cannot cover such data
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder exact reproduction of the original Dandelion; Lin's CCC for CD4+ probability (0.850) falls below the commonly accepted threshold of 0.9
- Lin's CCC comparison shows only approximate agreement between dandelionR and original Dandelion outputs (17,281 vs 17,234 cells, 17,208 shared)

## Evidence

- [intro] Dandelion's VDJ feature space strategy improved the prediction accuracy of trajectories from double-positive T cells to CD4/CD8 T cells: "which improved the prediction accuracy of trajectories from double-positive T cells to CD4/CD8 T cells, demonstrating significant potential for future applications"
- [methods] vdjPseudobulk builds the feature space from the miloR pseudobulk object: "pb.milo <- vdjPseudobulk(milo_object, mode_option = "abT","
- [methods] Diffusion map parameters for trajectory inference: "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [methods] DPT branch tips and width parameter: "dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1)"
- [intro] dandelionR implements absorbing Markov chain pseudotime projection: "dandelionR enables the construction of the VDJ feature space to perform trajectory analysis using diffusion maps and absorbing Markov chains, with seamless interaction with scRepertoire."
- [discussion] Discussion: destiny limits topology: "destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR’s trajectory inference function"
- [readme] README: markov chain pseudotime inspired by Palantir: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the [palantir](https://github.com/dpeerlab/Palantir) python package."
- [results] Results: Lin's CCC below threshold: "Lin's CCC for CD4 + probability (0.850) falls below the commonly accepted threshold of 0.9"
