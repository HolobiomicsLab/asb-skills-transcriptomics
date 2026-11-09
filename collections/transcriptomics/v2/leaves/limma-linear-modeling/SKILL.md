---
name: limma-linear-modeling
description: Use when genome-wide bulk RNA-seq count data must be converted into ranked
  differential-expression gene lists per drug-treatment contrast (e.g., 17-AAG response,
  combination response) to support rank-based gene set analysis.
when_to_use_negative:
- Input is already a differential-expression result table or ranked list — limma modeling
  on raw counts would duplicate the step
- The goal is single-cell (scRNAseq) or single-cell proteomics (CyTOF) analysis rather
  than bulk RNA-seq contrasts
- Sample sizes or the count-matrix/metadata layout do not match the secDrug/CombinationDrugMyeloma
  structure, so AssignGroup-defined contrasts cannot be constructed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Language in which normalization, limma linear modeling, DESeq2/edgeR fits,
    and ranked-list generation are executed
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: R package providing the CombinationDrugMyeloma dataset (`d`) and AssignGroup(x,
    l) helper that define the contrasts modeled with limma
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Packaged dataset object supplying the bulk RNA-seq counts and metadata used
    as limma input
  repo: https://github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:92e72819ac6f06b9dfa66ad0805a64e80ab064828dc263c4a184bcf71d8f0f04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ujjal-Mukherjee/secDrug
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# limma-linear-modeling

## Summary

Fit empirical-Bayes linear models on normalized bulk RNA-seq expression to rank differentially expressed genes across drug-treatment contrasts (e.g., untreated vs 17-AAG + Ixazomib combination in HMCLs), feeding ranked gene lists into gene-set analysis to identify upstream regulators.

## When to use

Use when genome-wide bulk RNA-seq count data must be converted into ranked differential-expression gene lists per drug-treatment contrast (e.g., 17-AAG response, combination response) to support rank-based gene set analysis.

## When NOT to use

- Input is already a differential-expression result table or ranked list — limma modeling on raw counts would duplicate the step
- The goal is single-cell (scRNAseq) or single-cell proteomics (CyTOF) analysis rather than bulk RNA-seq contrasts
- Sample sizes or the count-matrix/metadata layout do not match the secDrug/CombinationDrugMyeloma structure, so AssignGroup-defined contrasts cannot be constructed

## Inputs

- Bulk RNA-seq raw count matrices (untreated baseline, 0.5 µM 17-AAG single-agent, 17-AAG + 15 nM Ixazomib combination, 24 h, HMCL panel)
- Sample metadata / comparison group assignments from CombinationDrugMyeloma via AssignGroup(x, l)

## Outputs

- Ranked gene lists per contrast (ranked_gene_lists.csv)
- Per-contrast DEG count summary table (deg_summary_table.csv) with p < 0.05 and |fold-change| > 2, FDR < 0.05 subsets
- Ranked upstream regulators (CEBPB, ERBB2, CCND1, let-7) from combined GSA/limma output

## How to apply

Load deposited bulk RNA-seq count matrices and sample metadata from the secDrug repository (github.com/Ujjal-Mukherjee/secDrug), using the packaged `d` (CombinationDrugMyeloma) dataset object and the `AssignGroup(x, l)` helper to define sensitive-vs-resistant comparison groups. Normalize raw counts and, alongside DESeq2/edgeR negative-binomial fits, apply limma linear modeling (empirical-Bayes moderated statistics) to generate ranked gene lists for each contrast: 17-AAG-sensitive vs -resistant lines, 17-AAG response, and 17-AAG + 15 nM Ixazomib combination response at 24 h. Report significant genes at p < 0.05, then subset to |fold-change| > 2 with FDR < 0.05 for high-confidence hits. Combine limma ranks with GSA (rank-based gene set analysis) to identify upstream regulators such as CEBPB, ERBB2, CCND1, and let-7, and write per-contrast DEG counts and ranked lists to deg_summary_table.csv and ranked_gene_lists.csv.

## Related tools

- **R** (Language in which normalization, limma linear modeling, DESeq2/edgeR fits, and ranked-list generation are executed) — https://github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (R package providing the CombinationDrugMyeloma dataset (`d`) and AssignGroup(x, l) helper that define the contrasts modeled with limma) — https://github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Packaged dataset object supplying the bulk RNA-seq counts and metadata used as limma input) — https://github.com/Ujjal-Mukherjee/secDrug

## Examples

```
library(secDrug); groups <- AssignGroup(x, l); fit <- eBayes(lmFit(counts_norm, model.matrix(~ groups))); topTable(fit, number=Inf)
```

## Evaluation signals

- deg_summary_table.csv reproduces the reported per-contrast counts: 421 DE genes for 17-AAG-sensitive vs -resistant, 1449 for 17-AAG response, and 3974 for the combination (p < 0.05)
- The combination contrast yields 3974 significant genes (p < 0.05), of which 853 satisfy |fold-change| > 2 with FDR < 0.05
- Top regulators CEBPB, ERBB2, CCND1, and let-7 appear in the ranked outputs (ranked_gene_lists.csv)
- Contrast group labels match AssignGroup(x, l) assignments over the sensitive-vs-resistant HMCL groups
- limma-ranked lists are consistent with the parallel DESeq2/edgeR negative-binomial fits for the same contrasts

## Limitations

- Differential-expression findings are specific to the tested conditions (untreated, 17-AAG single-agent, 17-AAG + Ixazomib at 24 h) in HMCLs and may not transfer to other doses, time points, or primary samples
- Significance at p < 0.05 alone yields thousands of genes (e.g., 3974); the stricter |fold-change| > 2 and FDR < 0.05 cutoffs are needed for high-confidence subsets
- limma linear modeling depends on correct group assignment via AssignGroup; misassignment of sensitive/resistant lines changes all contrasts

## Evidence

- [methods] Apply GSA (rank-based gene set analysis) and limma linear modeling to generate ranked gene lists and identify upstream regulators (e.g., CEBPB, ERBB2, CCND1, let-7).: [evidence span withheld — anchor status: not_found]
- [methods] Normalize raw counts and fit negative-binomial differential-expression models in R with DESeq2 and edgeR for each contrast (17-AAG-sensitive vs -resistant lines, 17-AAG response, combination response).: [evidence span withheld — anchor status: not_found]
- [abstract] we developed a universally applicable R-software package for predicting novel secondary therapies in chemotherapy-resistant cancers: "we developed a universally applicable R-software package for predicting novel secondary therapies in chemotherapy-resistant cancers"
- [results] A total of 3974 genes changed significantly between untreated and 17-AAG + Ixazomib combination-treated samples (p < 0.05), of which 853 showed |fold-change| > 2 with FDR < 0.05.: [evidence span withheld — anchor status: not_found]
