---
name: biomarker-recovery-verification-against-supplementary-figures
description: Use when a clustering method claims to support downstream DEG analysis
  and you must verify that recovered marker genes match those reported in the paper's
  supplementary figures.
when_to_use_negative:
- The analysis goal is evaluating clustering accuracy (NMI/Purity/ARI) rather than
  verifying that clustering supports downstream DEG recovery
- Input labels are ground-truth annotations rather than SSRE-inferred cluster labels,
  so the validation does not test the method's claimed extension to DEG identification
- The dataset is not the one for which the supplementary marker list was published,
  since the marker set is figure- and dataset-specific
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: R (kruskal.test)
  role: Per gene Kruskal-Wallis rank-sum test with SSRE-inferred labels as the grouping
    factor to obtain DEG p-values
- name: SSRE
  role: Produces the inferred cell type labels whose clustering quality is validated
    via marker-gene recovery; MATLAB and Python implementations
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: One of the SSRE implementations used to generate the inferred labels feeding
    the DEG analysis
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: One of the SSRE implementations used to generate the inferred labels feeding
    the DEG analysis
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:d8599204ecb5c2b0699d927df3ae59ae4028b93d8fccfa14b66366fb16fe7c06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  - R (kruskal.test)
  - SSRE
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Biomarker recovery verification against supplementary figures

## Summary

Validates a downstream differential-expression analysis by checking whether a Kruskal-Wallis test on SSRE-inferred cluster labels recovers the marker genes reported in the paper's supplementary figure (Figure S2: NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, TUBB4Q on the Yan dataset).

## When to use

Use when a clustering method claims to support downstream DEG analysis and you must verify that recovered marker genes match those reported in the paper's supplementary figures.

## When NOT to use

- The analysis goal is evaluating clustering accuracy (NMI/Purity/ARI) rather than verifying that clustering supports downstream DEG recovery
- Input labels are ground-truth annotations rather than SSRE-inferred cluster labels, so the validation does not test the method's claimed extension to DEG identification
- The dataset is not the one for which the supplementary marker list was published, since the marker set is figure- and dataset-specific

## Inputs

- Yan scRNA-seq gene expression matrix (cells x genes)
- SSRE-inferred cell type labels per cell (from the CSUBioGroup/SSRE pipeline)
- Reference marker gene list from Figure S2 of the SSRE paper

## Outputs

- top50_de_genes_yan.csv containing the top-50 differentially expressed genes with Kruskal-Wallis p-values and group assignments
- Validation report listing which Figure S2 markers (NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, TUBB4Q) were recovered at P < 0.01 and which were not

## How to apply

Load the Yan scRNA-seq gene expression matrix and the SSRE-inferred cell type labels, aligning cells between the two tables. For each gene, run the Kruskal-Wallis rank-sum test in R (kruskal.test) using the SSRE-inferred labels as the grouping factor, yielding one p-value per gene. Apply the significance threshold P < 0.01 and rank significant genes by test statistic/p-value to extract the top-50 differentially expressed genes. Write the ranked list with p-values and group assignments to top50_de_genes_yan.csv, then check that the recovered set includes the Figure S2 markers (NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, TUBB4Q) and explicitly report any markers not recovered at P < 0.01.

## Related tools

- **R (kruskal.test)** (Per gene Kruskal-Wallis rank-sum test with SSRE-inferred labels as the grouping factor to obtain DEG p-values)
- **SSRE** (Produces the inferred cell type labels whose clustering quality is validated via marker-gene recovery; MATLAB and Python implementations) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (One of the SSRE implementations used to generate the inferred labels feeding the DEG analysis) — https://github.com/CSUBioGroup/SSRE
- **Python** (One of the SSRE implementations used to generate the inferred labels feeding the DEG analysis) — https://github.com/CSUBioGroup/SSRE

## Examples

```
kw <- apply(X, 1, function(g) kruskal.test(g, factor(ssre_labels))$p.value); top50 <- names(sort(kw[kw < 0.01]))[1:50]; write.csv(data.frame(gene=top50, p=kw[top50]), "top50_de_genes_yan.csv")
```

## Evaluation signals

- Recovered top-50 gene list contains the Figure S2 markers: NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, and TUBB4Q
- All reported DEGs satisfy the significance threshold P < 0.01 under kruskal.test with SSRE-inferred labels as the grouping factor
- Cell count and order in the expression matrix match the label vector after alignment between the two tables
- top50_de_genes_yan.csv has exactly 50 rows with gene, p-value, and group assignment columns
- Any Figure S2 marker absent from the P < 0.01 set is explicitly reported rather than silently omitted

## Limitations

- Validation is specific to the Yan dataset and the marker set reported in Figure S2; markers not recovered at P < 0.01 require explicit reporting rather than assuming method failure or success
- SSRE itself is relatively time-consuming on large-scale datasets, which constrains applying the upstream clustering (and hence this validation) to large scRNA-seq data
- The DEG result depends on the quality of SSRE's inferred labels; erroneous cluster assignments propagate directly into the Kruskal-Wallis grouping

## Evidence

- [abstract] SSRE extension to DEG identification: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes."
- [intro] Two-step procedure rationale: "In order to reduce the effect of irrelevant features and improve the overall accuracy, SSRE designs a two-step procedure, i.e., 1) adaptive gene selection and 2) similarity enhancement."
- [discussion] Future informative-gene extraction context: "more informative genes can be extracted or other biological information, such as gene functions [61] and gene regulatory relationships [62], [63], can be incorporated to distinguish cell types."
- [readme] Reproducibility of clustering results from repository: [evidence span withheld — anchor status: external_doc]
