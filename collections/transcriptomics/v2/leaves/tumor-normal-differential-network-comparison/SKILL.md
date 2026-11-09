---
name: tumor-normal-differential-network-comparison
description: Use when the goal is sample-level comparison of pathway connectivity
  (e.g., EGFR pathway degree density between TCGA-LUAD tumor and adjacent normal samples)
  from RNA-seq expression matrices, requiring one network per sample.
when_to_use_negative:
- The analysis goal is differential gene expression or bulk co-expression correlation
  across groups rather than per-sample direct gene-gene network topology
- Only a single sample is available with no matched/comparable normal cohort, so no
  tumor-vs-normal connectivity contrast is possible
- The input is single-cell scRNA-seq and the goal is clustering or trajectory direction
  via network flow entropy, where the broader c-CSN scRNA-seq workflow (CNDM, NFE)
  rather than this differential-comparison application is appropriate
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: c-CSN
  role: Constructs the conditional cell-specific network (CCSN) for each sample by
    conditional-independence testing, yielding the per-sample degree matrix used for
    EGFR pathway density comparison
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:ea3781c6123ed07a80027d8179b14de4401457d7efa2c55b8a5839bb72750f06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tumor-normal-differential-network-comparison

## Summary

Per-sample differential pathway connectivity analysis using c-CSN: build one conditional cell-specific network (CCSN) per TCGA-LUAD tumor and normal sample, extract EGFR-pathway degree density, and compare tumor vs benign tissues to detect dense pathway rewiring invisible to bulk differential expression.

## When to use

Use when the goal is sample-level comparison of pathway connectivity (e.g., EGFR pathway degree density between TCGA-LUAD tumor and adjacent normal samples) from RNA-seq expression matrices, requiring one network per sample.

## When NOT to use

- The analysis goal is differential gene expression or bulk co-expression correlation across groups rather than per-sample direct gene-gene network topology
- Only a single sample is available with no matched/comparable normal cohort, so no tumor-vs-normal connectivity contrast is possible
- The input is single-cell scRNA-seq and the goal is clustering or trajectory direction via network flow entropy, where the broader c-CSN scRNA-seq workflow (CNDM, NFE) rather than this differential-comparison application is appropriate

## Inputs

- Normalized TCGA LUAD RNA-seq expression matrix (tumor and normal adjacent tissue samples) from the GDC portal
- Gene list for the pathway of interest (e.g., EGFR pathway members including EGF and EGFR)
- Significance level alpha for the conditional-independence hypothesis test

## Outputs

- One direct association network (CCSN) per individual sample
- Conditional network degree matrix (CNDM) integrating per-sample CCSN degrees
- Per-sample EGF and EGFR network degrees
- EGFR pathway connectivity density per sample (tumor vs normal)
- Representative single-sample EGFR network plots and per-sample EGF/EGFR degree plots (Figure S8B/S8C analogues)

## How to apply

Download TCGA LUAD RNA-seq expression data (tumor and normal adjacent tissue) from the GDC portal and normalize gene expression profiles. For each sample, run the c-CSN procedure: select conditional genes as the top-degree hub genes from the CSN (e.g., EGF/EGFR-relevant regulators), estimate the conditional independence statistic rho_{xy|z} via scatter diagram neighborhoods in 3D expression space, and test each gene pair at significance level alpha against the N_alpha quantile of the standard normal distribution to build the CCSN. Integrate the per-conditional-gene CCSNs C_zg(k) into the cell-specific gene-gene interaction degree matrix C-k, then extract the degrees of EGF and EGFR and summarize the connectivity density of the EGFR pathway subnetwork. Compare EGFR-pathway degree density between tumor and normal samples, rendering representative single-sample EGFR networks and per-sample EGF/EGFR degree plots (mirroring Figure S8B and S8C). The rationale is that CSN overestimates associations by including indirect effects, so c-CSN's conditional-independence filtering yields direct, robust per-sample networks suited to differential topology comparison.

## Related tools

- **c-CSN** (Constructs the conditional cell-specific network (CCSN) for each sample by conditional-independence testing, yielding the per-sample degree matrix used for EGFR pathway density comparison) — https://github.com/LinLi-0909/c-CSN

## Examples

```
git clone https://github.com/LinLi-0909/c-CSN && python CCSN.py --expr luad_tcga_tumor_normal.tsv --alpha 0.05 --pathway EGFR --compare tumor_vs_normal
```

## Evaluation signals

- Each sample yields exactly one direct association network: verify 'One direct association network is generated for one cell'
- Per-sample CCSN edge calls correspond to gene pairs whose test statistic falls in the N_alpha quantile rejection region of the standard normal at level alpha
- EGFR pathway subnetwork degree density is consistently higher in tumor samples than in normal/benign samples
- EGF and EGFR CCSN degrees are consistently higher across tumor samples, reproducing the reported Figure S8B/S8C pattern
- Indirect associations are removed: c-CSN networks should be sparser than raw CSN networks, which 'suffer from a problem of overestimation on gene-gene associations'

## Limitations

- c-CSN is not designed to construct causal gene association networks; directions of gene associations cannot be obtained
- No parallel computation scheme is provided despite the G-fold computational cost increase from testing each gene pair against G conditional genes; the authors state a parallel scheme is desired
- Requires conditional gene selection (top-degree hub genes in CSN), which introduces dependence on the initial CSN topology

## Evidence

- [abstract] Per-sample network construction: "One direct association network is generated for one cell."
- [abstract] Direct association rationale: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [methods] LUAD application: "we applied c-CSN to the TCGA lung adenocarcinoma (LUAD) RNA-seq dataset"
- [methods] Conditional gene selection: [evidence span withheld — anchor status: not_found]
- [methods] Per-sample degree integration: [evidence span withheld — anchor status: not_found]
- [intro] CSN overestimation caveat: "CSN suffers from a problem of overestimation on gene–gene associations and includes both direct and indirect associations due to interactive effects from other genes"
- [discussion] No causal direction: "c-CSN is not designed to construct the causal gene association networks, and the directions of the gene associations cannot be obtained."
