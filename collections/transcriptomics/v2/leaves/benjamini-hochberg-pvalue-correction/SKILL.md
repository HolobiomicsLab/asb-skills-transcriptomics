---
name: benjamini-hochberg-pvalue-correction
description: Use when many per-gene hypothesis tests (e.g. Fisher's exact tests on
  gene-cluster absence/presence per lifestyle) yield a vector of p-values that must
  be FDR-corrected before ranking candidate lifestyle-associated genes.
when_to_use_negative:
- The analysis calls only a handful of pre-specified tests rather than screening all
  gene clusters genome-wide, where FDR correction is not the intended multiplicity
  framework
- The downstream pLAG decision relies solely on the prevalence (>70%) and log2 fold
  change (>2) thresholds and does not use statistical ranking, making correction unnecessary
- Input is already a table of adjusted p-values — re-applying p.adjust() would double-correct
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: bacLIFE
  role: Analytical module whose pLAG detection rule combines Fisher's exact test p-values
    with BH correction and the >70% prevalence / >2 log2 fold change thresholds
  repo: https://github.com/Carrion-lab/bacLIFE
- name: R fisher.test
  role: Base R function producing the raw per-gene-cluster p-values that are subsequently
    BH-corrected with p.adjust()
- name: gtools
  role: Provides foldchange() and foldchange2logratio() to compute the log2 fold changes
    stored alongside corrected p-values in the pLAG table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:ed5c8a8e472f583f45f9eaabc1b90033230b73d549c66f4f90dee35689905506
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - R fisher.test
  - bacLIFE
  - gtools
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# benjamini-hochberg-pvalue-correction

## Summary

Apply Benjamini-Hochberg (BH) FDR correction to p-values from per-gene-cluster Fisher's exact tests when detecting lifestyle-associated genes (pLAGs) with bacLIFE's analytical module, so that thousands of simultaneous gene-cluster tests across lifestyles can be ranked under a controlled false discovery rate.

## When to use

Use when many per-gene hypothesis tests (e.g. Fisher's exact tests on gene-cluster absence/presence per lifestyle) yield a vector of p-values that must be FDR-corrected before ranking candidate lifestyle-associated genes.

## When NOT to use

- The analysis calls only a handful of pre-specified tests rather than screening all gene clusters genome-wide, where FDR correction is not the intended multiplicity framework
- The downstream pLAG decision relies solely on the prevalence (>70%) and log2 fold change (>2) thresholds and does not use statistical ranking, making correction unnecessary
- Input is already a table of adjusted p-values — re-applying p.adjust() would double-correct

## Inputs

- Vector of raw p-values from per-gene-cluster Fisher's exact tests (one test per gene cluster per lifestyle)
- Gene cluster absence/presence matrix for the dataset (e.g. Burkholderia/Paraburkholderia and Pseudomonas)
- Per-genome lifestyle labels from metadata and bacLIFE lifestyle predictions

## Outputs

- Vector of BH-adjusted p-values aligned to the input p-value vector
- pLAG table (e.g. plags_table.csv) with gene cluster ID, lifestyle, group prevalence, log2 fold change, p-value, and p-adjusted

## How to apply

In R, run fisher.test() on the 2x2 contingency table for each gene cluster x lifestyle comparison (present in >=70% of the lifestyle group's genomes and >2 log2 fold change versus remaining bacteria), collecting raw p-values into a vector. Pass this vector to p.adjust() with the Benjamini-Hochberg method to obtain adjusted p-values for ranking. Rank pLAG candidates by the BH-adjusted p-value while retaining the concrete membership thresholds (>70% group prevalence, >2 log2 fold change) as the actual pLAG definition. Export gene cluster ID, lifestyle, group prevalence, log2 fold change, raw p-value, and p-adjusted together (e.g. plags_table.csv) so corrected statistics stay traceable to each candidate. The rationale is that bacLIFE screens thousands of gene clusters simultaneously per lifestyle, so uncorrected per-test p-values would inflate false positives.

## Related tools

- **bacLIFE** (Analytical module whose pLAG detection rule combines Fisher's exact test p-values with BH correction and the >70% prevalence / >2 log2 fold change thresholds) — https://github.com/Carrion-lab/bacLIFE
- **R fisher.test** (Base R function producing the raw per-gene-cluster p-values that are subsequently BH-corrected with p.adjust())
- **gtools** (Provides foldchange() and foldchange2logratio() to compute the log2 fold changes stored alongside corrected p-values in the pLAG table)

## Examples

```
padj <- p.adjust(pvals, method = "BH")
```

## Evaluation signals

- Per-lifestyle pLAG counts after BH-corrected ranking match reported values: 382/478/786 (environmental/opportunistic/plant pathogen) for Burkholderia/Paraburkholderia and 48/74/377 for Pseudomonas
- Row-count check (row_count_equals) on plags_table.csv reproduces the expected pLAG counts per lifestyle
- Every exported pLAG satisfies both membership thresholds: >70% presence within the lifestyle group and >2 log2 fold change versus remaining bacteria
- Length and ordering of the p-adjusted vector match the raw p-value vector (1:1 mapping per gene cluster x lifestyle)
- All BH-adjusted p-values lie within [0,1] and are >= the corresponding raw p-values

## Limitations

- BH correction controls false discovery rate, not family-wise error rate, so some flagged candidates may be false positives (hence bacLIFE's experimental validation of 14 predicted LAGs, of which 6 were confirmed)
- The BH-corrected ranking alone does not define pLAGs — the >70% group prevalence and >2 log2 fold change thresholds must also be satisfied
- Genome sets must first be filtered to high-quality genomes (>90% completeness, <5% contamination) via CheckM, otherwise spurious gene clusters distort the tested contingency tables

## Evidence

- [methods] pLAG definition (>70% group presence, >2 log2 fold change): "we used the threshold of 70% of the genomes belonging to a lifestyle/group and more than a 2 log2 fold change"
- [other] BH correction in the pLAG workflow: [evidence span withheld — anchor status: not_found]
- [intro] Predicted pLAG counts: "For Burkholderia/Paraburkholderia and Pseudomonas, we obtained a total of 786 and 377 predicted lifestyle-associated genes (pLAGs) for a phytopathogenic lifestyle, respectively"
- [readme] bacLIFE role in LAG prediction: "performs large-scale comparative genomics to predict bacterial lifestyles and to pinpoint candidate genes, denominated  **lifestyle-associated genes (LAGs)**"
