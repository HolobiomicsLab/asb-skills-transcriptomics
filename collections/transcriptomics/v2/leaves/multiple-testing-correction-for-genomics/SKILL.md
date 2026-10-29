---
name: multiple-testing-correction-for-genomics
description: Use when a per-gene statistical test (e.g., Wilcoxon rank-sum between
  cell clusters) returns thousands of raw p-values that must be multiple-testing corrected
  before declaring significant DE genes or enrichment terms.
when_to_use_negative:
- The hypothesis space is a single pre-registered test (no multiplicity), so uncorrected
  p-values are already valid
- Input is a ranked gene list without per-gene p-values (e.g., pure fold-change ranking),
  where correction of p-values is not applicable
- The p-values already come from a procedure that internally controls FDR, and re-correction
  would be over-conservative
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scEFSC
  role: Produces the cluster labels whose pairwise differential expression tests generate
    the p-value family that requires correction
  repo: https://github.com/Conan-Bian/scEFSC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:6c33b555bc379aa42a866894e5ed080e1a2dc6b20abaf30408b87e06c4ba826c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Conan-Bian/scEFSC
  repo_url: https://github.com/Conan-Bian/scEFSC
  tools:
  - scEFSC
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multiple-testing-correction-for-genomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Adjusts per-gene p-values from high-dimensional genomic tests (e.g., differential expression between scEFSC clusters) for multiple testing so that only statistically significant genes/pathways are retained for downstream GO and KEGG enrichment analysis.

## When to use

Use when a per-gene statistical test (e.g., Wilcoxon rank-sum between cell clusters) returns thousands of raw p-values that must be multiple-testing corrected before declaring significant DE genes or enrichment terms.

## When NOT to use

- The hypothesis space is a single pre-registered test (no multiplicity), so uncorrected p-values are already valid
- Input is a ranked gene list without per-gene p-values (e.g., pure fold-change ranking), where correction of p-values is not applicable
- The p-values already come from a procedure that internally controls FDR, and re-correction would be over-conservative

## Inputs

- Vector/table of raw per-gene p-values from pairwise cluster differential expression testing (e.g., Wilcoxon rank-sum on the log2-normalized GSE83139 expression matrix)
- Cluster labels from the scEFSC wMetaC ensemble step defining which cell groups are compared

## Outputs

- dge_results.csv: per-cluster-comparison differentially expressed genes with raw and multiple-testing-adjusted p-values
- Filtered significant gene lists per cluster comparison feeding GO and KEGG enrichment

## How to apply

After running a per-gene test (the scEFSC workflow ranks genes by Wilcoxon rank-sum between pairs of clusters from the log2-normalized expression matrix), apply a family-wise or FDR correction to the vector of raw p-values so the adjusted values control false discoveries across all genes tested. Store the corrected values alongside raw p-values per cluster comparison so significance can be re-thresholded (e.g., adjusted p-value cutoffs) without re-running the tests. Only genes passing the correction enter the downstream gene ontology and KEGG pathway enrichment, mirroring the paper's analysis of the top differentially expressed genes across its nine clusters (533 enriched GO terms and 9 KEGG pathways with 70 genes annotated). Validation hinges on the output table carrying adjusted p-values, not raw ones, for every gene-comparison row.

## Related tools

- **scEFSC** (Produces the cluster labels whose pairwise differential expression tests generate the p-value family that requires correction) — https://github.com/Conan-Bian/scEFSC

## Examples

```
dge$p_adj <- p.adjust(dge$p_value, method = "BH")
```

## Evaluation signals

- dge_results.csv exists and every DE gene row carries an adjusted p-value column, not only raw p-values
- Adjusted p-values are monotonically non-decreasing relative to raw p-values for the same gene-comparison pair
- All adjusted p-values fall within [0,1] and the count of genes passing the significance cutoff is far smaller than the total genes tested
- Downstream go_kegg_enrichment.csv contains GO terms and KEGG pathways with enrichment statistics (term, gene count, p-value, adjusted p-value) consistent with the paper's reported 533 GO terms and 9 KEGG pathways

## Limitations

- The article does not specify which exact correction method (e.g., Benjamini-Hochberg vs Bonferroni) was applied to the DE p-values
- Correction stringency directly reduces the gene lists fed to GO/KEGG enrichment, so overly strict thresholds can strip biological signal from rare cell-type comparisons

## Evidence

- [other] Workflow step requiring multiple-testing correction: [evidence span withheld — anchor status: not_found]
- [other] Enrichment statistics schema expected in outputs: [evidence span withheld — anchor status: not_found]
