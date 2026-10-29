---
name: lincs-l1000-perturbation-signature-extraction
description: Use when a scRNA-seq-derived tumor sub-clustering (GEP file) must be
  matched to LINCS L1000 perturbation experiments to identify which compounds or combinations
  kill which sub-clusters.
when_to_use_negative:
- The goal is cluster-wise IC50 or drug-kill prediction from screened compounds (GDSC/PRISM
  via CaDRReS-Sc), not LINCS perturbation signature deconvolution for combination
  treatment selection
- No reference cell line correlates with the single-cell cluster GEPs, so LINCS L1000
  perturbation data cannot be meaningfully matched to the tumor sub-clusters
- The input lacks treated/control bulk perturbation samples with concentration metadata,
  so the >90% killed and 75% consistency criteria cannot be evaluated
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDrug
  role: Treatment Selection module that orchestrates LINCS L1000 perturbation matching,
    CIBERSORTx deconvolution, kill/consistency thresholds, and greedy combination
    search
  repo: https://github.com/ailabstw/scDrug
- name: Premnas
  role: Workflow used by scDrug for LINCS L1000-based prediction of minimal combined
    therapies killing tumor sub-clusters
- name: CIBERSORTx
  role: Estimates the proportion of each tumor sub-cluster in every LINCS L1000 bulk
    sample via the cibersortx/fractions Docker
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:6a2e924aa8929166a559b70931ccc22d8e9e0d82d362f754030d0b055a4c2239
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CIBERSORTx
  - LINCS L1000
  - Premnas
  - Python
  - scDrug
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lincs-l1000-perturbation-signature-extraction

## Summary

Extracts perturbation-induced changes in tumor sub-cluster proportions from LINCS L1000 bulk gene expression profiles by deconvolving treated vs control samples with CIBERSORTx, enabling combined-treatment selection that kills the greatest number of single-cell-derived tumor sub-clusters.

## When to use

Use when a scRNA-seq-derived tumor sub-clustering (GEP file) must be matched to LINCS L1000 perturbation experiments to identify which compounds or combinations kill which sub-clusters.

## When NOT to use

- The goal is cluster-wise IC50 or drug-kill prediction from screened compounds (GDSC/PRISM via CaDRReS-Sc), not LINCS perturbation signature deconvolution for combination treatment selection
- No reference cell line correlates with the single-cell cluster GEPs, so LINCS L1000 perturbation data cannot be meaningfully matched to the tumor sub-clusters
- The input lacks treated/control bulk perturbation samples with concentration metadata, so the >90% killed and 75% consistency criteria cannot be evaluated

## Inputs

- Gene expression profile (GEP) file of tumor sub-clusters from scRNA-seq analysis (GEP.txt)
- LINCS L1000 bulk gene expression profiles of perturbation experiments on the selected reference cell line
- Bulk GEPs of the eight reference cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC)
- Treated and control sample expression matrices for CIBERSORTx fraction estimation

## Outputs

- Per-perturbation cell-subset proportion changes across treated vs control samples
- List of 'killed' sub-clusters per perturbation (>90% reduction with 75% consistency at higher concentrations)
- Minimal combined-treatment selection report covering the maximum achievable number of tumor sub-clusters

## How to apply

Compute the Pearson correlation between the single-cell cluster GEP and the bulk GEP of each of the eight reference cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) and select the highest-correlating line. Extract anti-logarithmic bulk GEPs of all perturbation experiments on that cell line from LINCS L1000, then estimate the proportion of each tumor sub-cluster in every bulk sample with CIBERSORTx (cibersortx/fractions Docker). Label a sub-cluster 'killed' when its percentage drops more than 90% after perturbation, applying a consistency check that the same reduction (default threshold 75%) is observed for the same compound at higher concentrations. Finally run a greedy search (sensitivity -0.9, consistency -0.75): iteratively pick the perturbation killing the most subpopulations with the greatest summed reduction, remove killed subpopulations, and repeat until none remain.

## Related tools

- **scDrug** (Treatment Selection module that orchestrates LINCS L1000 perturbation matching, CIBERSORTx deconvolution, kill/consistency thresholds, and greedy combination search) — https://github.com/ailabstw/scDrug
- **Premnas** (Workflow used by scDrug for LINCS L1000-based prediction of minimal combined therapies killing tumor sub-clusters)
- **CIBERSORTx** (Estimates the proportion of each tumor sub-cluster in every LINCS L1000 bulk sample via the cibersortx/fractions Docker)

## Examples

```
docker pull cibersortx/fractions
```

## Evaluation signals

- Selected reference cell line has the highest Pearson correlation against the single-cell cluster GEP among all eight candidates
- Every perturbation in the final report kills only sub-clusters whose percentage is reduced more than 90%
- Each killed subset's reduction (default 75%) is confirmed for the same compound at higher concentrations
- Greedy search terminates only when no more subpopulations can be killed, and the combination covers the maximum achievable number of sub-clusters (e.g. 11 of 14 in the hepatocellular carcinoma case)
- Output report lists treatment combinations only for the given cell clusters with sensitivity threshold -0.9 and consistency threshold -0.75

## Limitations

- Prediction is limited to compounds experimentally screened in LINCS L1000 on the eight available reference cell lines
- A sub-cluster counts as killed only under the strict >90% reduction plus 75% higher-concentration consistency rule, so moderate sensitivities are not captured
- Fraction estimation depends on CIBERSORTx Docker availability; docker-in-docker may fail on some hosts, requiring the CIBERSORTx container to be run outside the scDrug container with a shared mounted folder

## Evidence

- [methods] Compute the Pearson correlation between the single-cell cluster GEP and the bulk GEP of each of the eight reference cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC): [evidence span withheld — anchor status: not_found]
- [methods] killed subset determination: [evidence span withheld — anchor status: not_found]
- [methods] consistency threshold: [evidence span withheld — anchor status: not_found]
- [methods] LINCS L1000 data: "predict potential combined treatment strategies using the LINCS L1000 data [17]"
- [readme] CIBERSORTx fractions Docker: "In the Docker container `scDrug`, pull the Docker image `cibersortx/fractions` used in treatment selection."
