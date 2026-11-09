---
name: simulation-parameter-sweep
description: Use when benchmarking lineage reconstruction quality against simulation
  parameters, e.g., sweeping mutation rate or dropout settings on TedSim-simulated
  barcode data with a known ground-truth tree.
when_to_use_negative:
- Input is real experimental barcode data with no ground-truth tree — Qr/Qh cannot
  be computed against a known lineage
- Goal is to evaluate expression-integrated methods like LinRace's NJ_asym_Dif accuracy
  itself rather than how barcode data quality varies with simulation parameters
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: TedSim
  role: Simulates cell identity fields, ground-truth lineage trees, and mutated barcode
    matrices under varied mu and p_d settings
  repo: https://github.com/ZhangLabGT/LinRace
- name: R
  role: Environment for running the simulation sweep and computing Qr/Qh metrics
- name: LinRace
  role: Package providing the simulation vignette and evaluation framework (RF.dist
    against ground-truth tree) used alongside the sweep
  repo: https://github.com/ZhangLabGT/LinRace
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:ba98ac91e73e243c8a1e437fbb78bcf6cc7ce7d600ee0e098ea9fd6345c0fd54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - LinRace
  - R
  - TedSim
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# simulation-parameter-sweep

## Summary

Systematically vary TedSim simulation parameters (mutation rate mu, dropout probability p_d, ncells, N_char) to generate benchmark datasets with ground-truth lineage trees, and record how downstream metrics such as reconstruction potential (Qr) and homoplasy edge ratio (Qh) respond.

## When to use

Use when benchmarking lineage reconstruction quality against simulation parameters, e.g., sweeping mutation rate or dropout settings on TedSim-simulated barcode data with a known ground-truth tree.

## When NOT to use

- Input is real experimental barcode data with no ground-truth tree — Qr/Qh cannot be computed against a known lineage
- Goal is to evaluate expression-integrated methods like LinRace's NJ_asym_Dif accuracy itself rather than how barcode data quality varies with simulation parameters

## Inputs

- 6-tip cell state tree (phyla) defining the differentiation topology
- TedSim SimulateCIFs parameters (ncells, N_char, max_walk, n_CIF, n_diff, Sigma, mu, p_d)
- Mutation-rate grid (e.g., 0.025, 0.05, 0.075; 0.05–0.4) and dropout on/off conditions

## Outputs

- Per-combination table of reconstruction potential Qr and homoplasy edge ratio Qh (qr_qh_vs_mutation_rate.csv)
- Ground-truth lineage tree (tree_ct with cell_* tip labels) and leaf barcode matrix (muts_leaves) per simulated dataset
- Trend curves of Qr and Qh versus mutation rate, stratified by dropout condition

## How to apply

Simulate ground-truth lineage trees and ancestral barcode matrices with TedSim's SimulateCIFs on a fixed state tree (e.g., ncells=1024, N_char=16, max_walk=8, n_CIF=30, n_diff=20, Sigma=0.5), sweeping mu over a grid such as 0.025–0.4. Run each sweep twice: once with dropout probability p_d=0 (no-dropout condition) and once with dropouts enabled, to isolate their effect. For each dataset, extract tree_ct (with cell_* tip labels) and the leaf barcode matrix muts_leaves (first ncells rows), then compute reconstruction potential Qr (Eq. 13, fraction of recoverable mutated target sites) and homoplasy edge ratio Qh (Eq. 14). Collect Qr and Qh for every mutation-rate × dropout combination, validate expected ranges (e.g., Qh up to ~0.75 with dropouts), and write results to a table such as qr_qh_vs_mutation_rate.csv.

## Related tools

- **TedSim** (Simulates cell identity fields, ground-truth lineage trees, and mutated barcode matrices under varied mu and p_d settings) — https://github.com/ZhangLabGT/LinRace
- **R** (Environment for running the simulation sweep and computing Qr/Qh metrics)
- **LinRace** (Package providing the simulation vignette and evaluation framework (RF.dist against ground-truth tree) used alongside the sweep) — https://github.com/ZhangLabGT/LinRace

## Examples

```
cifs <- SimulateCIFs(ncells=1024, phyla, p_a=p_a, n_CIF=30, n_diff=20, step=cif_step, p_d=p_d, Sigma=0.5, mu=mu, N_char=16, max_walk=8, SIF_res=returnlist, unif_on=FALSE)
```

## Evaluation signals

- Qr and Qh values are produced for every mutation-rate × dropout-setting combination with no missing cells in the output CSV
- Qh with dropouts enabled reaches approximately 0.75 on the 1024-cell, 16-character TedSim configuration
- Qh's trend across mutation rates tracks the performance trend of lineage reconstruction methods (e.g., normalized RF.dist against tree_ct)
- Simulated barcode matrices contain exactly ncells leaf rows after MutsTransform-style extraction of muts_leaves

## Limitations

- Results depend on simulation settings (e.g., max_walk=8, N_char=16); trends may not transfer to other barcode designs or real datasets with different dropout characteristics
- Qh reflects barcode data quality (homoplasy) but does not itself measure a specific reconstruction method's accuracy
- Findings are grounded in TedSim's simulation model; real CRISPR barcode systems may have different mutation and dropout dynamics

## Evidence

- [results] homoplasy edge ratio reaches as high as 0.75 when dropouts are present: [evidence span withheld — anchor status: not_found]
- [methods] SimulateCIFs call: "cifs <- SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu, N_char = N_char, max_walk = max_walk"
- [intro] challenge motivation: "inferring the cell division tree of a massive number of cells is a challenging problem. In addition to the computational complexity of inferring the lineage tree itself"
- [readme] LinRace installation: [evidence span withheld — anchor status: external_doc]
