---
name: gsva-family-sample-level-geneset-scoring
description: Use when benchmarking sample-wise pathway quantification methods (GSVA,
  PLAGE, ssGSEA, z-score) against rROMA module activity on a normalized RNA-seq matrix
  to test which method recovers an expected treatment- or disease-driven pathway shift.
when_to_use_negative:
- The expression matrix is not normalized (rRoma/GSVA comparisons assume normalized
  data such as edgeR cpm or DESeq2 vst)
- The goal is the rROMA module-activity computation itself rather than benchmarking
  the GSVA-family sample-wise scoring methods against it
- The analysis requires a pathway database other than the 50 Hallmark gene sets used
  in the rROMA benchmark
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSVA
  role: R/Bioconductor package providing the four compared sample-wise gene set scoring
    methods (GSVA, PLAGE, ssGSEA, z-score) applied to the identical normalized matrix
- name: edgeR
  role: cpm normalization of the GSE176121 count matrix prior to scoring
- name: rRoma_comp
  role: Repository containing data and R scripts to reproduce the comparison of sample-wise
    methods in the rRoma article
  repo: https://github.com/sysbio-curie/rRoma_comp
- name: rROMA
  role: Reference method whose PC1-based module activity and shift p/q values benchmark
    the GSVA-family scores
  repo: https://github.com/sysbio-curie/rROMA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:95455d7da8bb22db7febdd13c93a69c2c563b062317a9e9fca2e25f643ac5130
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rRoma_comp
  tools:
  - GSVA
  - R
  - edgeR
  - rROMA
  - rRoma_comp
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# GSVA-family sample-level gene set scoring

## Summary

Compute per-sample pathway activity scores for the same 50 MSigDB Hallmark gene sets using the four sample-wise methods GSVA, PLAGE, ssGSEA and z-score, so their results can be benchmarked against rROMA's PC1-based module activity on identical normalized expression matrices (edgeR cpm) from the GSE176121 cystic fibrosis dataset.

## When to use

Use when benchmarking sample-wise pathway quantification methods (GSVA, PLAGE, ssGSEA, z-score) against rROMA module activity on a normalized RNA-seq matrix to test which method best recovers an expected treatment- or disease-driven pathway shift.

## When NOT to use

- The expression matrix is not normalized (rRoma/GSVA comparisons assume normalized data such as edgeR cpm or DESeq2 vst)
- The goal is the rROMA module-activity computation itself rather than benchmarking the GSVA-family sample-wise scoring methods against it
- The analysis requires a pathway database other than the 50 Hallmark gene sets used in the rROMA benchmark

## Inputs

- edgeR cpm-normalized RNA-seq expression matrix with genes as rows and samples as columns (e.g. Rehman_CF_NCF_RNA_exp_matrix_norm.rda, Rehman_IL17_TNFa_RNA_exp_matrix_norm.rda)
- Sample label table defining group comparisons (e.g. Rehman_CF_NCF_sample_labels.rda, Rehman_IL17_TNFa_sample_labels.rda)
- Gene set list: 50 MSigDB Hallmark gene sets from h.all GMT (MSigDB v6.2, HALLMARK_ prefix removed)

## Outputs

- Sample-wise gene set score matrices per method (GSVA, PLAGE, ssGSEA, z-score) for the 50 Hallmark sets
- Scenario-1 comparison table: number of dysregulated pathways per method (expected: 0 for GSVA/ssGSEA/z-score, 45/50 for PLAGE, 3 for rROMA)
- Scenario-2 ranking of the TNFα signaling pathway by each method's significance/score (expected: GSVA 5, PLAGE 5, ssGSEA 7, z-score 18 vs top rank for rROMA)

## How to apply

Start from the same edgeR-cpm-normalized expression matrix (genes as rows, samples as columns) and the same gene set list used for rROMA — here the 50 MSigDB Hallmark sets extracted with SelectFromMSIGdb('HALLMARK') from MSigDB v6.2, with the 'HALLMARK_' prefix stripped. Compute sample-wise scores for each gene set with the GSVA package under its four methods (GSVA, PLAGE, ssGSEA, z-score), reproducing the article's comparison via the companion scripts in sysbio-curie/rRoma_comp (e.g. Rehman_TNF_IL17_other_methods.R). For the CF-vs-non-CF scenario (scenario 1), test differential pathway activity between CF and healthy groups per method and count dysregulated pathways; for the IL17+TNFα-treated-vs-control scenario (scenario 2), rank the TNFα signaling pathway among all tested Hallmark sets by each method's significance/score. Call pathways shifted in rROMA at 'ppv Median Exp' <= 0.05 so both method families are judged by comparable significance criteria.

## Related tools

- **GSVA** (R/Bioconductor package providing the four compared sample-wise gene set scoring methods (GSVA, PLAGE, ssGSEA, z-score) applied to the identical normalized matrix)
- **edgeR** (cpm normalization of the GSE176121 count matrix prior to scoring)
- **rRoma_comp** (Repository containing data and R scripts to reproduce the comparison of sample-wise methods in the rRoma article) — https://github.com/sysbio-curie/rRoma_comp
- **rROMA** (Reference method whose PC1-based module activity and shift p/q values benchmark the GSVA-family scores) — https://github.com/sysbio-curie/rROMA

## Examples

```
Rscript Rehman_TNF_IL17_other_methods.R
```

## Evaluation signals

- Scenario-1 counts reproduce the article benchmark: 0 dysregulated pathways for GSVA, ssGSEA and z-score; 45/50 for PLAGE; 3/50 for rROMA (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION)
- Scenario-2 rankings reproduce the benchmark: TNFα signaling ranked 5/24 by GSVA, 5/27 by PLAGE, 7/17 by ssGSEA, 18/45 by z-score, versus top rank for rROMA
- All methods were run on the identical edgeR-cpm matrix and the same 50 Hallmark gene sets as the rROMA run
- rROMA modules called shifted at 'ppv Median Exp' <= 0.05 match the article's reported shifted set

## Limitations

- Authors state the CF case study lacks statistical power and would require additional transcriptional datasets and testing of several gene set reference databases
- PLAGE flags 45/50 pathways as dysregulated in scenario 1, illustrating inflated false-positive shift detection relative to GSVA/ssGSEA/z-score (0) and rROMA (3)
- The benchmark is specific to GSE176121 airway epithelial cultures and the Hallmark (MSigDB v6.2) database; conclusions may not transfer to other datasets or gene set collections

## Evidence

- [readme] Repository contaning data and codes to reproduce the comparison of sample-wise methods in the rRoma article.: [evidence span withheld — anchor status: external_doc]
- [readme] comparison scripts listed in rRoma_comp README: [evidence span withheld — anchor status: external_doc]
- [methods] rRoma requires normalized input data: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [discussion] rROMA found 3 significantly active Hallmark pathways in the CF dataset: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [readme] input matrix format for gene set scoring: "rRoma requires a gene expression matrix, with column names indicating samples and row names indicating gene names."
