---
name: personalized-differential-coexpression-analysis
description: Use when the goal is to find driver nodes/genes specific to individual
  patients from matched tumor-normal RNA-seq matrices (e.g., TCGA), and to assess
  patient-level tumor heterogeneity via driver-gene recurrence frequencies.
when_to_use_negative:
- When a single population-level driver-gene list is sufficient — use the global WMDS.net
  dominating set on the pooled differential network instead of per-patient SSNs
- When input is already a ranked driver-gene list or a feature table rather than raw
  tumor/normal expression matrices — WMDS.net requires expression matrices to recompute
  co-expression
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0602
tools:
- name: WMDS.net
  role: Weighted minimum dominating set framework used to identify driver nodes of
    patient-specific differential gene co-expression networks
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Solves the binary integer-programming problem of the weighted MDS with the
    intlinprog function from the Optimization ToolBox
- name: DawnRank
  role: Source of the gene interaction network intersected with TCGA genes to build
    Gene_Net
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:a3367d24eca3c5a7f820142da4815e18dd3a1b4cede373c174d5f34676a3f48f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# personalized-differential-coexpression-analysis

## Summary

Builds patient-specific differential co-expression networks and identifies personalized driver genes by solving a weighted minimum dominating set (WMDS.net) on each network, then quantifies recurrence frequency of driver genes across patients within each cancer type.

## When to use

Use when the goal is to find driver nodes/genes specific to individual patients from matched tumor-normal RNA-seq matrices (e.g., TCGA), and to assess patient-level tumor heterogeneity via driver-gene recurrence frequencies.

## When NOT to use

- When a single population-level driver-gene list is sufficient — use the global WMDS.net dominating set on the pooled differential network instead of per-patient SSNs
- When input is already a ranked driver-gene list or a feature table rather than raw tumor/normal expression matrices — WMDS.net requires expression matrices to recompute co-expression

## Inputs

- Matched tumor and normal RNA-seq expression matrices (genes as rows, sample IDs as columns), e.g. TCGA_BLCAtumor and TCGA_BLCAnormal
- Gene interaction hub network (Gene_Net), the intersection of the DawnRank gene interaction network and genes present in TCGA
- Per-patient tumor sample expression vectors for SSN construction across 14 cancer types

## Outputs

- Per-patient sample-specific networks (SSNs) with edges retained at Delta-r Z-test P < 0.05
- Per-patient lists of personalized driver genes from the weighted minimum dominating set (WMDS.netP solution)
- Frequency table of personalized driver genes per cancer type binned as high (>=0.6), medium (0.3–0.6), low (<0.3), e.g. personalized_driver_frequency_table.csv

## How to apply

For each patient, add that patient's tumor sample to the TCGA normal-sample reference co-expression network, recalculate Pearson correlations, and retain edges whose Delta-r Z-test P-value is < 0.05, yielding a sample-specific network (SSN). On each SSN, solve the weighted minimum dominating set as a binary integer program using intlinprog in Matlab R2020b, with node weight wi = (sum over adjacent j of -log10(p_ij))^(-gamma) and gamma = 0.01, integrating node degree and significance of co-expression difference into node controllability. Run this on 14 TCGA cancer types (the bundled test data are TCGA_BLCAtumor and TCGA_BLCAnormal expression matrices with genes as rows and sample IDs as columns). Then compute, per cancer type, the recurrence frequency of each personalized driver gene across patients and bin genes into high-frequency (>= 0.6), medium-frequency (0.3 <= f < 0.6), and low-frequency (< 0.3) classes. The low-frequency class dominating in every cancer type is the paper's key tumor-heterogeneity finding; a sanity check is that CDKN2A appears in the LUAD WMDS.netP personalized list while absent from the global WMDS.net driver list.

## Related tools

- **WMDS.net** (Weighted minimum dominating set framework used to identify driver nodes of patient-specific differential gene co-expression networks) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Solves the binary integer-programming problem of the weighted MDS with the intlinprog function from the Optimization ToolBox)
- **DawnRank** (Source of the gene interaction network intersected with TCGA genes to build Gene_Net)

## Examples

```
matlab -batch "wi=(sum(-log10(p_ij),2)).^(-0.01); drivers=intlinprog(wi,A,b,Aeq,beq,lb,ub,intcon)"
```

## Evaluation signals

- Low-frequency (<0.3) personalized driver genes account for the largest proportion in every cancer type, reproducing Figure 5 of the paper
- CDKN2A is present in the LUAD personalized WMDS.netP driver list but absent from the global WMDS.net driver list
- Retained SSN edges all pass the Delta-r Z-test threshold P < 0.05
- The dominating-set solution is obtained via intlinprog with node weight exponent gamma = 0.01
- WMDS.net shows a better precision-recall balance than other top-tier tools on TCGA cancer datasets

## Limitations

- Gene_Net restricts the analysis to the intersection of the DawnRank gene interaction network and TCGA genes, so drivers outside that hub network cannot be recovered
- Requires matched normal reference data (e.g., TCGA normal samples) to compute differential co-expression; unpaired or single-condition data are not supported
- Runtime scales with the number of patients since one SSN and one integer program must be solved per patient

## Evidence

- [intro] We developed a weighted MDS network model (WMDS.net) to find the drive nodes of differential gene co-expression networks.: "We developed a weighted MDS network model (WMDS.net) to find the drive nodes of differential gene co-expression networks."
- [intro] intlinprog in Matlab R2020b: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [intro] Gene_Net definition: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
- [abstract] Precision-recall balance: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall."
- [readme] README input/output spec: "The inputs to the main program are the expression matrices of the two conditions (which are TCGA_BLCAtumor and TCGA_BLCAnormal in the tested data), and the output is the gene list of key nodes in the"
