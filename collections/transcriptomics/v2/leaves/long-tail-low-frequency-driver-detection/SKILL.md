---
name: long-tail-low-frequency-driver-detection
description: Use when you have a list of predicted driver genes for TCGA cancer types
  (e.g., BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA,
  UCEC) and GDC somatic SNV data, and need to classify drivers as low- vs high-frequency
  mutations.
when_to_use_negative:
- When your aim is discovery of the driver genes themselves from RNA-seq (use the
  WMDS.net network-control prediction step instead; this skill only evaluates mutation
  frequency of already-predicted drivers).
- When mutation frequency cannot be computed because no matched tumor-normal somatic
  SNV calls are available for the cancer type (the skill requires sample-level non-synonymous
  SNV calls from GDC).
- When evaluating method performance via precision/recall against a single gold-standard
  list (e.g., CGC overlap counts) rather than characterizing the mutation-frequency
  distribution of predicted drivers.
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: WMDS.net
  role: Source of the predicted cancer driver gene lists whose SNV mutation frequencies
    are classified as low- vs high-frequency across the 14 TCGA cancer types
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Implementation environment of WMDS.net (intlinprog in the Optimization ToolBox);
    used for the driver predictions and the downstream frequency computation/chart
    of Figure 4
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:5141866f2e743bef9f2bf6bbfe75e261845bcd3eb5db5f4ee9c8448ab566f0b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# long-tail-low-frequency-driver-detection

## Summary

Quantifies how many network-predicted cancer driver genes (intersected with the COSMIC Cancer Gene Census, CGC, 616 genes) are rarely mutated in TCGA somatic SNV data, splitting them into low-frequency (<0.05) versus high-frequency (>0.05) classes per cancer type to demonstrate detection of long-tail drivers.

## When to use

Use when you have a list of predicted driver genes for TCGA cancer types (e.g., BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) and GDC somatic SNV data, and need to classify drivers as low- vs high-frequency mutations.

## When NOT to use

- When your aim is discovery of the driver genes themselves from RNA-seq (use the WMDS.net network-control prediction step instead; this skill only evaluates mutation frequency of already-predicted drivers).
- When mutation frequency cannot be computed because no matched tumor-normal somatic SNV calls are available for the cancer type (the skill requires sample-level non-synonymous SNV calls from GDC).
- When evaluating method performance via precision/recall against a single gold-standard list (e.g., CGC overlap counts) rather than characterizing the mutation-frequency distribution of predicted drivers.

## Inputs

- WMDS.net driver gene prediction lists for 14 TCGA cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC)
- COSMIC Cancer Gene Census (CGC) gene list (616 genes)
- TCGA somatic SNV (MAF) data per cancer type from the Genomic Data Commons

## Outputs

- Per-gene, per-cancer-type SNV mutation frequency (fraction of tumor samples with non-synonymous somatic mutations)
- Per-cancer-type fractions of CGC-overlapping predicted drivers in low-frequency (<0.05) vs high-frequency (>0.05) classes (Figure 4 bar chart)
- Case-level confirmation that ERBB3 is a low-frequency (<0.05) predicted driver in LUAD

## How to apply

Load the WMDS.net driver gene predictions for the 14 TCGA cancer types and the CGC gene list of 616 genes from COSMIC (cancer.sanger.ac.uk/cosmic). Load TCGA somatic SNV data for the same cancer types from the Genomic Data Commons (portal.gdc.cancer.gov). For each predicted driver gene in each cancer type, compute the SNV mutation frequency as the fraction of tumor samples carrying non-synonymous somatic mutations in that gene. Intersect predicted drivers with the CGC list per cancer type, split the overlap into low (<0.05) and high (>0.05) mutation-frequency classes, and compute the fraction in each class, reproducing the per-cancer-type bar chart of Figure 4. As a case-level check, confirm that ERBB3 is a WMDS.net-predicted LUAD driver whose LUAD mutation frequency falls in the <0.05 class, consistent with COSMIC records of its rarity in LUAD. The rationale is that a strong driver-discovery method should recover genes that are rarely, rather than frequently, mutated across samples.

## Related tools

- **WMDS.net** (Source of the predicted cancer driver gene lists whose SNV mutation frequencies are classified as low- vs high-frequency across the 14 TCGA cancer types) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Implementation environment of WMDS.net (intlinprog in the Optimization ToolBox); used for the driver predictions and the downstream frequency computation/chart of Figure 4)

## Examples

```
low_frac = {ct: sum(f[g] < 0.05 for g in set(pred[ct]) & cgc) / len(set(pred[ct]) & cgc) for ct in ttypes}
```

## Evaluation signals

- The per-cancer-type low/high-frequency fractions reproduce the Figure 4 pattern: most CGC-overlapping predicted drivers fall in the rarely-mutated (<0.05) class across the 14 TCGA cancer types.
- ERBB3 is present among WMDS.net-predicted drivers in LUAD and its LUAD mutation frequency is <0.05.
- Each gene's mutation frequency equals the fraction of tumor samples with non-synonymous somatic mutations, computed consistently per cancer type over the same sample sets.
- The CGC intersection per cancer type is limited to the 616-gene COSMIC list, and all overlapping genes are assigned to exactly one of the <0.05 or >0.05 classes.

## Limitations

- Mutation frequency is computed only from non-synonymous somatic SNVs; other variant classes are not counted.
- The low/high split uses a fixed threshold (0.05), so genes near the boundary are sensitive to the cutoff choice.
- The evaluation is restricted to CGC-overlapping predicted drivers in the 14 TCGA cancer types; non-CGC predictions are not frequency-classified.

## Evidence

- [other] Most WMDS.net-predicted driver genes overlapping the CGC list were rarely mutated across different cancer types, indicating the method's strong ability to find driver genes with low-frequency mutations.: [evidence span withheld — anchor status: not_found]
- [other] Compute the SNV mutation frequency of each predicted driver gene in each cancer type as the fraction of tumor samples carrying non-synonymous somatic mutations in that gene.: [evidence span withheld — anchor status: not_found]
- [abstract] WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall"
- [readme] In this study, we implement the algorithm using the Matlab program language and solve the binary integer-programming problem using function: "In this study, we implement the algorithm using the Matlab program language and solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox"
