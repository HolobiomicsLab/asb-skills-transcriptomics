---
name: proportional-enrichment-testing
description: Use when you have per-cluster expression calls for known drug-target
  genes on scRNA-seq subclonal clusters and must decide which drug combination candidates
  are enriched across resistant and sensitive subclones.
when_to_use_negative:
- Input is bulk RNA-seq or pharmacogenomics data rather than single-cell clusters
  — the greedy set cover secDrug prediction pipeline, not this enrichment test, is
  the correct step
- Drug-target genes are unknown or not derivable from LINCS — no target genes can
  be mapped to clusters, so enrichment testing is uninformative
- More than two secDrugs are to be combined — the article states model systems for
  testing more than two secDrugs are not yet developed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment in which the scRNA-seq screen, Seurat clustering, and proportion
    testing are executed
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: Pipeline package providing the scRNA-seq screening workflow and data access
    for target-gene enrichment
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Provides AssignGroup(x, l) for binning per-cluster target-gene expression
    and PropTest(p1, p2, n) for between-cluster proportion testing
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:25cda1f0f4c12db115437ce898c0feebbeca59bf4f59edd71c93e89f93091248
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# proportional-enrichment-testing

## Summary

A statistical screen that tests whether drug-target genes (e.g., HSP90AA1/HSP90AB1 for 17-AAG, NAMPT for FK866) are enriched across t-SNE-defined single-cell subclones, prioritizing secDrug combinations in drug-resistant myeloma.

## When to use

Use when you have per-cluster expression calls for known drug-target genes on scRNA-seq subclonal clusters and must decide which drug combination candidates are enriched across resistant and sensitive subclones.

## When NOT to use

- Input is bulk RNA-seq or pharmacogenomics data rather than single-cell clusters — the greedy set cover secDrug prediction pipeline, not this enrichment test, is the correct step
- Drug-target genes are unknown or not derivable from LINCS — no target genes can be mapped to clusters, so enrichment testing is uninformative
- More than two secDrugs are to be combined — the article states model systems for testing more than two secDrugs are not yet developed

## Inputs

- scRNA-seq count matrices from HMCLs (e.g., drug-sensitive U266P and resistant U266VR pairs)
- LINCS-derived drug-target gene sets (HSP90AA1, HSP90AB1 for 17-AAG; NAMPT for FK866)
- t-SNE subclonal cluster assignments from Seurat processing

## Outputs

- Per-sample, per-cluster target-gene enrichment scores exported as CSV
- Annotated t-SNE maps showing target-gene enrichment per subclone (reproducing Fig. 2)
- Proportion-test results comparing target-gene enrichment between clusters (PropTest p-values)

## How to apply

Load deposited HMCL scRNA-seq count matrices (e.g., U266P vs U266VR pairs) from github.com/Ujjal-Mukherjee/secDrug into R, preprocess and cluster each sample with Seurat (QC filtering, normalization), and embed subclones via t-SNE. Map the LINCS-derived 17-AAG target genes HSP90AA1 and HSP90AB1 and the FK866 target gene NAMPT onto each t-SNE cluster. Use the CombinationDrugMyeloma package: AssignGroup(x, l) bins per-cluster target-gene expression into enrichment groups, and PropTest(p1, p2, n) performs proportion testing of target-gene-positive cluster fractions between clusters/samples. Accept the screen if the majority of subclonal clusters are enriched for HSP90AA1/HSP90AB1 and NAMPT, matching Fig. 2 of the paper.

## Related tools

- **R** (Environment in which the scRNA-seq screen, Seurat clustering, and proportion testing are executed) — github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (Pipeline package providing the scRNA-seq screening workflow and data access for target-gene enrichment) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Provides AssignGroup(x, l) for binning per-cluster target-gene expression and PropTest(p1, p2, n) for between-cluster proportion testing) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
CombinationDrugMyeloma::AssignGroup(cluster_expr, l); CombinationDrugMyeloma::PropTest(p1, p2, n)
```

## Evaluation signals

- Majority of t-SNE subclonal clusters in both drug-sensitive and drug-resistant samples show enrichment for HSP90AA1/HSP90AB1 and NAMPT
- Enrichment pattern reproduces the per-sample t-SNE cluster map reported in Fig. 2 of the paper
- PropTest output yields significant between-cluster differences in target-gene-positive proportions where expected between resistant and sensitive subclones
- AssignGroup produces coherent enrichment bins (groups) for each cluster that map consistently onto the target-gene-positive/negative calls
- Exported CSV enrichment scores match the annotated t-SNE figure values cluster-for-cluster

## Limitations

- Screen currently supports only two-drug combinations; the article notes no model systems exist for testing more than two secDrugs
- Pre-clinical/clinical validation in xenograft or mouse models has not been performed for some novel secDrugs (e.g., KIN001-102, SB505124), so enrichment alone is not proof of efficacy
- Validation limited to the HMCL panel tested; the article notes a need to test in a wider panel of refractory and clonally derived acquired-resistant lines

## Evidence

- [abstract] scRNAseq screening tool: "we used single-cell RNA sequencing (scRNAseq) as a screening tool to predict top combination candidates based on the enrichment of target genes"
- [results] target-gene enrichment screen finding: [evidence span withheld — anchor status: not_found]
- [discussion] validation in models: "we performed extensively in vitro, ex vivo, and functional validation in research models of refractory and resistant myeloma to validate 17-AAG + FK866, 17AAG + PI, and 17-AAG + IMID as combination"
