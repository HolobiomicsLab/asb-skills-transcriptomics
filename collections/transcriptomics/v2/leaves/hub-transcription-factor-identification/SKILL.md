---
name: hub-transcription-factor-identification
description: Use when comparative single-cell regulatory analysis has linked a disease-specific
  subpopulation (e.g., the B7 B cell cluster in a CLL donor) across two conditions
  and you need to rank and validate its candidate key TF regulators.
when_to_use_negative:
- When only a single condition/sample is available — sc-compReg requires four datasets
  (one scRNA-seq and one scATAC-seq from each of two conditions) to identify differential
  regulatory relations
- When the goal is clustering or subpopulation matching itself rather than downstream
  TF ranking — use cnmf and the linked-cluster outputs first
- When differential regulations must be replicated across multiple donors per condition
  — the method does not yet accommodate multiple donors under each condition
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scCompReg
  role: Fits the comparative regulatory model that produces the hub.tf and diff.net
    outputs used for TF identification
  repo: https://github.com/SUwonglab/sc-compReg
- name: R
  role: Runtime environment for loading sample .rds files, calling sc_compreg, and
    exporting tf_i.txt/diff_net_i.txt via write.table
- name: Bedtools
  role: Used in sc_compreg_process_data_.sh to intersect peak-name files against prior_data
    to generate PeakName_intersect.txt and peak_gene_prior_intersect.bed feeding the
    model
- name: Homer
  role: Used with Bedtools in sc_compreg_process_data_.sh to generate MotifTarget.txt
    for motif prior weighting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:8aa9fe9ccafc143d8e1eb6265a478d783fc7b1d2a150083b7ebea5caf91afc66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - Bedtools
  - Homer
  - R
  - R (>=)
  - sc-compReg
  - scCompReg
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hub-transcription-factor-identification

## Summary

Identifies hub transcription factors (e.g., TOX2) driving condition-specific subpopulations by inspecting the hub TF ranking and differential regulatory networks output by sc_compreg after linking subpopulations across two conditions of paired scRNA-seq/scATAC-seq data.

## When to use

Use when comparative single-cell regulatory analysis has linked a disease-specific subpopulation (e.g., the B7 B cell cluster in a CLL donor) across two conditions and you need to rank and validate its candidate key TF regulators.

## When NOT to use

- When only a single condition/sample is available — sc-compReg requires four datasets (one scRNA-seq and one scATAC-seq from each of two conditions) to identify differential regulatory relations
- When the goal is clustering or subpopulation matching itself rather than downstream TF ranking — use cnmf and the linked-cluster outputs first
- When differential regulations must be replicated across multiple donors per condition — the method does not yet accommodate multiple donors under each condition

## Inputs

- compreg.output object from sc_compreg containing hub.tf and diff.net lists for each linked subpopulation
- Paired log2-transformed scRNA-seq (O) and scATAC-seq (E) matrices with cluster indices for two conditions (e.g., sample1.rds, sample2.rds)
- Motif objects from motif_human.rds and MotifTarget.txt via mfbs_load

## Outputs

- Hub TF ranking per subpopulation (compreg.output$hub.tf[[i]], exported as tf_i.txt)
- Condition-specific differential regulatory network per subpopulation (compreg.output$diff.net[[i]], exported as diff_net_i.txt)
- Statistical validation result for the candidate hub TF (e.g., one-tailed two-sample t-test p-value comparing TOX2 expression between conditions)

## How to apply

After running sc_compreg on the O/E gene expression and chromatin accessibility matrices plus cluster indices from both conditions, identify the condition-specific subpopulation from the linked clusters (e.g., the tumor-specific B7 subpopulation in the CLL donor). Inspect the hub TF ranking for each linked population via compreg.output$hub.tf[[i]] and the condition-specific differential network via compreg.output$diff.net[[i]], checking whether a candidate TF such as TOX2 ranks as a hub. Export each population's results via write.table as tf_i.txt and diff_net_i.txt with row.names=FALSE, quote=FALSE, and tab separator. Validate the finding statistically — e.g., confirm TOX2 is significantly higher in the CLL group versus healthy controls using a one-tailed two-sample t-test (p = 0.0013 in this study) — to support calling it a key regulator.

## Related tools

- **scCompReg** (Fits the comparative regulatory model that produces the hub.tf and diff.net outputs used for TF identification) — https://github.com/SUwonglab/sc-compReg
- **R** (Runtime environment for loading sample .rds files, calling sc_compreg, and exporting tf_i.txt/diff_net_i.txt via write.table)
- **Bedtools** (Used in sc_compreg_process_data_.sh to intersect peak-name files against prior_data to generate PeakName_intersect.txt and peak_gene_prior_intersect.bed feeding the model)
- **Homer** (Used with Bedtools in sc_compreg_process_data_.sh to generate MotifTarget.txt for motif prior weighting)

## Examples

```
compreg.output = sc_compreg(sample1$O1, sample1$E1, sample1$O1.idx, sample1$E1.idx, sample1$symbol1, sample1$peak.name1, sample2$O2, sample2$E2, ...); write.table(compreg.output$hub.tf[[i]], 'tf_i.txt', row.names=FALSE, quote=FALSE, sep='\t')
```

## Evaluation signals

- Exported tf_i.txt and diff_net_i.txt files exist and contain the expected columns for each population i
- TOX2 is recovered as a hub regulator for the tumor-specific B7 subpopulation, consistent with the published finding
- One-tailed two-sample t-test on TOX2 expression between CLL and healthy groups yields significant elevation (p = 0.0013)
- The identified subpopulation is correctly linked across the two conditions as CLL-specific B7 by the comparative model

## Limitations

- Differential regulations discovered this way need to be replicated by additional patients and controls
- The method does not yet accommodate multiple donors per condition; extension is pending data availability
- The task here relies on two independent case-control comparisons; single-cohort results should be treated as preliminary

## Evidence

- [abstract] The analysis reveals a tumor-specific B cell subpopulation in the CLL patient and identifies TOX2 as a potential regulator of this subpopulation.: "The analysis reveals a tumor-specific B cell subpopulation in the CLL patient and identifies TOX2 as a potential regulator of this subpopulation."
- [methods] Export hub TF and differential network outputs: [evidence span withheld — anchor status: not_found]
- [results] TOX2 differential expression validation: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion limitation on replication: "the differential regulations discovered this way will need to be replicated by additional patients and controls"
- [readme] README install and usage context: "scCompReg is an R package that provides coupled clustering and joint embedding of scRNA-seq and scATAC-seq on one sample, and performs comparative gene regulatory analysis between two conditions."
