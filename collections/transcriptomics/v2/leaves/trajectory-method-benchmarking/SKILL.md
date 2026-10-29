---
name: trajectory-method-benchmarking
description: Use when comparing trajectory/pseudotime inference tools on scRNA-seq
  data with known cell type labels, to quantify which method recovers developmental
  ordering on neuronal or hematopoietic datasets.
when_to_use_negative:
- Input is already a filtered/normalized feature table that has not undergone the
  same GO-subsetting (get_GO_exp) and dispersion filtering (dispersion_threshold =
  10) as the method being benchmarked — comparisons on differently preprocessed matrices
  are not valid
- Datasets with no known cell type labels, since CI, BI, KC, and POS are computed
  against known labels
- Non-developmental scRNA-seq analyses (e.g. differential expression or clustering-only
  goals) where pseudotime ordering is not the target output
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: redPATH
  role: Primary method under test; run via redpath(filtered_GO_data, threadnum = 4,
    base_path_range = c(4:8)) after GO subsetting and dispersion filtering
  repo: https://github.com/tinglabs/redPATH
- name: SCORPIUS
  role: Baseline pseudotime inference method run on identical filtered matrices for
    comparison
- name: TSCAN
  role: Baseline pseudotime inference method run on identical filtered matrices for
    comparison
- name: Monocle2
  role: Baseline pseudotime inference method run on identical filtered matrices for
    comparison
- name: DPT
  role: Baseline diffusion pseudotime method run on identical filtered matrices for
    comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:c6487ac34655942c96b3b7c38bb339f3fc5ff696478b459dd6147c5162817573
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - DPT
  - Monocle2
  - R
  - SCORPIUS
  - TSCAN
  - redPATH
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# trajectory-method-benchmarking

## Summary

Benchmark pseudotime-inference methods (redPATH vs SCORPIUS, TSCAN, Monocle2, DPT) on developmental single-cell RNA-seq datasets using ordering-agreement metrics CI, BI, KC, and POS against known cell type labels. Grounded in the redPATH paper's Figure 2b comparison workflow.

## When to use

Use when comparing trajectory/pseudotime inference tools on scRNA-seq data with known cell type labels, to quantify which method best recovers developmental ordering on neuronal or hematopoietic datasets.

## When NOT to use

- Input is already a filtered/normalized feature table that has not undergone the same GO-subsetting (get_GO_exp) and dispersion filtering (dispersion_threshold = 10) as the method being benchmarked — comparisons on differently preprocessed matrices are not valid
- Datasets with no known cell type labels, since CI, BI, KC, and POS are computed against known labels
- Non-developmental scRNA-seq analyses (e.g. differential expression or clustering-only goals) where pseudotime ordering is not the target output

## Inputs

- Raw single-cell expression matrix (genes x cells), e.g. llorens_exprs neural stem cell dataset (145 cells) or hematopoietic dataset
- Known cell type labels per cell (e.g. llorens_labels) for metric computation
- Estimated number of cell types k0 per dataset for setting base_path_range

## Outputs

- Pseudotime orderings from redPATH, SCORPIUS, TSCAN, Monocle2, and DPT per dataset
- CI, BI, KC, and POS scores per method-dataset pair
- Consolidated benchmark comparison table (e.g. figure2b_benchmark_scores.csv)

## How to apply

Standardize preprocessing across all methods: subset each raw expression matrix to development-related genes with redPATH's get_GO_exp (gene_type = 'gene_names', organism = 'mouse', takelog = F), then filter with filter_exp using dispersion_threshold = 10 and threads = 2. Run redpath() on each filtered matrix with threadnum = 4 and base_path_range = c(4:8), where the range spans the estimated number of cell types k0 to k0 + 4. Run SCORPIUS, TSCAN, Monocle2, and DPT on the identical filtered matrices so orderings are comparable. Compute CI, BI, KC, and POS scores for every method-dataset pair against known labels (e.g. llorens_labels), then assemble one comparison table (e.g. figure2b_benchmark_scores.csv) and check that redPATH attains the best scores across metrics on the three neuronal and one hematopoietic datasets.

## Related tools

- **redPATH** (Primary method under test; run via redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8)) after GO subsetting and dispersion filtering) — https://github.com/tinglabs/redPATH
- **SCORPIUS** (Baseline pseudotime inference method run on identical filtered matrices for comparison)
- **TSCAN** (Baseline pseudotime inference method run on identical filtered matrices for comparison)
- **Monocle2** (Baseline pseudotime inference method run on identical filtered matrices for comparison)
- **DPT** (Baseline diffusion pseudotime method run on identical filtered matrices for comparison)

## Examples

```
subset_GO_data <- get_GO_exp(input_data, gene_type = "gene_names", organism = "mouse", takelog = F); filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2); redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8))
```

## Evaluation signals

- Every method-dataset pair was run on the SAME filtered matrix (identical get_GO_exp subset and filter_exp dispersion_threshold = 10 output), not method-specific preprocessing
- base_path_range for redpath() equals k0 to k0 + 4, where k0 is the estimated number of cell types for that dataset
- CI, BI, KC, and POS scores are computed for all 4 methods on all 4 datasets (3 neuronal + 1 hematopoietic) against known labels such as llorens_labels
- The assembled benchmark table shows redPATH attaining the best (top) score across all evaluation metrics, reproducing the paper's Figure 2b result
- Output table saved as figure2b_benchmark_scores.csv contains scores for every method-dataset-metric combination without missing values

## Limitations

- The benchmark relies on known cell type labels; the article notes that approaches that avoid labeling 'may fail when the pseudotime results perform poorly'
- Baselines are limited to SCORPIUS, TSCAN, Monocle2, and DPT; other popular tools mentioned (Seurat, Wishbone, reCAT, CycleX) are not part of this comparison
- GO subsetting assumes mouse organism gene names; other organisms or gene identifier types require adapting get_GO_exp parameters

## Evidence

- [methods] GO subsetting rationale: "Reduce genes to ones which are related to Cell Development, Cell Morphology, Cell Differentiation, Cell Fate and Cell Maturation (from Gene Ontology)"
- [readme] filter_exp parameters: "filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2)"
- [readme] redpath call and base_path_range rule: "redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8))
# base_path_range is set to the estimated number of cell types, k0, and k0 +4."
- [intro] baselines cited in intro: "including Monocle2 [17], TSCAN [18], and SCORPIUS [19], and for cell cycle processes such as reCAT [20]"
- [results] benchmark outcome finding: [evidence span withheld — anchor status: not_found]
