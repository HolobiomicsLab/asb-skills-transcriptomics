---
name: marker-gene-recovery-validation
description: Use when a redPATH pseudotime ordering exists for single-cell RNA-seq
  data and the goal is to discover and validate genes whose expression significantly
  associates with both cell proliferation and differentiation along the trajectory.
when_to_use_negative:
- When pseudotime results perform poorly — the article notes this gene discovery approach
  may fail when the pseudotime results perform poorly.
- When the goal is to simultaneously model the coupling between proliferation and
  differentiation with an integrative model rather than discover individual correlated
  genes.
- When the input expression matrix has not been GO-subsetted and dispersion-filtered;
  redPATH's preprocessing expects genes reduced via Gene Ontology and a simple dispersion
  threshold.
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: redPATH
  role: Provides calc_correlation (type='dcor' and type='mic'), gene_selection/gene_selection_intersection,
    and the llorens_exprs sample dataset used to discover and validate proliferation/differentiation-coupled
    genes along redPATH pseudotime.
  repo: https://github.com/tinglabs/redPATH
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:ef751cf91b49913d72ff184b773a2d25c5b8a26f39826b587e8cd0e39b709869
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - redPATH
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# marker-gene-recovery-validation

## Summary

Recovers proliferation/differentiation-coupled marker genes (Foxm1, Tubb5, Nek2) and marker genes (Dcx, Sp9) by applying distance correlation (dCor) and Maximal Information Coefficient (MIC) statistics to redPATH pseudotime on the NSC-Llorens neural stem cell dataset, then validating the recovered genes against published results.

## When to use

Use when a redPATH pseudotime ordering exists for single-cell RNA-seq data and the goal is to discover and validate genes whose expression significantly associates with both cell proliferation and differentiation along the trajectory.

## When NOT to use

- When pseudotime results perform poorly — the article notes this gene discovery approach may fail when the pseudotime results perform poorly.
- When the goal is to simultaneously model the coupling between proliferation and differentiation with an integrative model rather than discover individual correlated genes.
- When the input expression matrix has not been GO-subsetted and dispersion-filtered; redPATH's preprocessing expects genes reduced via Gene Ontology and a simple dispersion threshold.

## Inputs

- llorens_exprs expression matrix (m genes x n cells, 145 neural stem cells) from the redPATH package
- GO-subsetted gene matrix from get_GO_exp (mouse organism, gene_type='gene_names')
- Dispersion-filtered gene matrix from filter_exp (dispersion_threshold = 10)
- Precomputed redPATH pseudotime ordering (redpath_pseudotime)

## Outputs

- dcor_result vector of distance correlation scores per gene
- mic_result vector of MIC scores per gene
- selected_result gene-by-cell matrix of union-selected significant genes
- selected_genes_dcor_mic.csv containing the selected gene matrix and correlation scores

## How to apply

Load the llorens_exprs expression matrix from the redPATH package together with the GO-subsetted (Cell Development, Cell Morphology, Cell Differentiation, Cell Fate, Cell Maturation) and dispersion-filtered (dispersion_threshold = 10) gene matrix, plus the precomputed redpath_pseudotime ordering. Run calc_correlation with type='dcor' and again with type='mic' against the redPATH pseudotime for every gene. Apply gene_selection (union selection) on the two correlation vectors with threshold = 0.65 to retain significantly associated genes. Verify that Foxm1, Tubb5, and Nek2 (proliferation/differentiation-coupled) and Dcx and Sp9 (marker genes) appear among the selected genes, and write the selected gene-by-cell matrix plus correlation scores to selected_genes_dcor_mic.csv. The union selection can be contrasted with gene_selection_intersection when only genes passing both statistics are desired.

## Related tools

- **redPATH** (Provides calc_correlation (type='dcor' and type='mic'), gene_selection/gene_selection_intersection, and the llorens_exprs sample dataset used to discover and validate proliferation/differentiation-coupled genes along redPATH pseudotime.) — https://github.com/tinglabs/redPATH

## Examples

```
dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor"); mic_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "mic"); selected_result <- gene_selection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.65)
```

## Evaluation signals

- The output file selected_genes_dcor_mic.csv exists and contains Foxm1, Tubb5, Nek2, Dcx, and Sp9 among the selected genes.
- Every retained gene has a dCor or MIC score above the 0.65 threshold used in gene_selection.
- The selected_result matrix is a genes-by-cells matrix whose gene set matches the published analysis for the NSC-Llorens pseudotime.
- dcor_result and mic_result each contain one score per input gene with no missing values.
- Preprocessing invariants hold: input genes were reduced to GO categories (Cell Development, Cell Morphology, Cell Differentiation, Cell Fate, Cell Maturation) and filtered with dispersion_threshold = 10 before pseudotime inference.

## Limitations

- This approach may fail when the pseudotime results perform poorly, since gene-gene-pseudotime correlations inherit pseudotime errors.
- Integrative models are preferred to analyze proliferation and differentiation processes simultaneously; the underlying mechanism of the coupling remains obscure and requires more integrative computational models.
- The union selection (gene_selection) with threshold = 0.65 retains genes passing either dCor or MIC; use gene_selection_intersection if stricter, both-statistics agreement is required.

## Evidence

- [methods] calc_correlation call for dcor: "dcor_result <- calc_correlation(full_exprs_matrix = input_data, redpath_pseudotime, type = "dcor")"
- [methods] gene_selection call: "selected_result <- gene_selection(full_exprs_matrix = input_data, redpath_pseudotime, dcor_result, mic_result, threshold = 0.5)"
- [discussion] Gene discovery using dCor and MIC: "2) gene discovery using dCor and MIC"
- [readme] README biological analysis marker gene discovery: "Distance Correlation and Maximal Information Coefficient calculation"
- [readme] GO subsetting preprocessing step: "Reduce genes to ones which are related to Cell Development, Cell Morphology, Cell Differentiation, Cell Fate and Cell Maturation (from Gene Ontology)"
- [discussion] pseudotime failure mode limitation: "Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly."
