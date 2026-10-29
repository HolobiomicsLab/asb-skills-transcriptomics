---
name: dyngen-ground-truth-network-comparison
description: Use when a quantitative CSNI benchmark with complete cellwise ground-truth
  networks is needed and real single-cell data cannot provide that ground truth —
  i.e. comparing network inference methods on dyngen-simulated transcriptomics.
when_to_use_negative:
- The question concerns trajectory inference, trajectory alignment, or RNA velocity
  benchmarking — dyngen supports those via different ground-truth layers and workflows
  (e.g. analysis/usecase_rna_velocity).
- Only real single-cell data is available and no ground-truth cellwise regulatory
  network exists — AUROC/AUPR against truth cannot be computed.
tools:
- name: dyngen
  role: Simulates the 42 benchmark datasets with cellwise ground-truth regulatory
    networks used as the evaluation reference
  repo: https://github.com/dynverse/dyngen
- name: pySCENIC
  role: 'Benchmarked CSNI method: GRNBoost coexpression modules, top-10 regulator
    selection, AUCell scoring into cell-specific GRNs'
- name: arboreto
  role: Infers coexpression modules for pySCENIC step 1 using stochastic gradient
    boosting machines
- name: AUCell
  role: Computes per-cell module activity scores that are recombined into cell-specific
    (cell, regulator, target, score) tuples
- name: LIONESS
  role: Baseline CSNI method evaluated with the same mean AUROC/AUPR protocol on the
    same dyngen datasets
- name: SSN
  role: Baseline CSNI method evaluated with the same mean AUROC/AUPR protocol on the
    same dyngen datasets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:0ac71b22f66a013c140132007ef52721bbc8ab71b4033bbdcf4821b2a1eaefcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - AUCell
  - LIONESS
  - SSN
  - arboreto
  - dyngen
  - pySCENIC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# dyngen-ground-truth-network-comparison

## Summary

Benchmark cell-specific gene regulatory network inference (CSNI) methods such as pySCENIC, LIONESS, and SSN against dyngen's simulated cellwise ground-truth regulatory networks, scoring each method with mean AUROC and AUPR across cells on 42 dyngen datasets (14 backbones x 3 seeds).

## When to use

Use when a quantitative CSNI benchmark with complete cellwise ground-truth networks is needed and real single-cell data cannot provide that ground truth — i.e. comparing network inference methods on dyngen-simulated transcriptomics.

## When NOT to use

- The question concerns trajectory inference, trajectory alignment, or RNA velocity benchmarking — dyngen supports those via different ground-truth layers and workflows (e.g. analysis/usecase_rna_velocity).
- Only real single-cell data is available and no ground-truth cellwise regulatory network exists — AUROC/AUPR against truth cannot be computed.

## Inputs

- 42 dyngen-simulated datasets (14 predefined backbones x 3 seeds) with transcriptomics profiles
- dyngen ground-truth cell-specific regulatory network per dataset (cell, regulator, target, regulatory effect)
- CSNI method outputs as cell-specific interaction score matrices from pySCENIC (via arboreto modules + AUCell), LIONESS, or SSN

## Outputs

- Per-dataset mean AUROC and mean AUPR values bounded in [0, 1] for each CSNI method
- Ranked comparison of CSNI methods (pySCENIC vs LIONESS vs SSN) with significance of score differences

## How to apply

Load 42 dyngen datasets (14 predefined backbones x 3 seeds), each containing transcriptomics profiles plus the ground-truth cell-specific regulatory network computed from the regulatory effect of each regulator on its target. Run the pySCENIC pipeline: infer coexpression modules with classical network inference via arboreto's stochastic gradient boosting machines, select the top 10 regulators per target gene, skip motif-analysis filtering (dyngen generates no motif data), compute per-cell AUCell activity scores for each module, and recombine modules with scores into a cell-specific GRN of (cell, regulator, target, score) tuples. Retain the top 10,000 interactions per cell and compute AUROC and AUPR against the ground-truth cell-specific GRN, averaging scores across all cells per dataset. Apply the identical protocol to comparison methods LIONESS and SSN on the same normalised dyngen data so AUROC/AUPR distributions are directly comparable across methods.

## Related tools

- **dyngen** (Simulates the 42 benchmark datasets with cellwise ground-truth regulatory networks used as the evaluation reference) — https://github.com/dynverse/dyngen
- **pySCENIC** (Benchmarked CSNI method: GRNBoost coexpression modules, top-10 regulator selection, AUCell scoring into cell-specific GRNs)
- **arboreto** (Infers coexpression modules for pySCENIC step 1 using stochastic gradient boosting machines)
- **AUCell** (Computes per-cell module activity scores that are recombined into cell-specific (cell, regulator, target, score) tuples)
- **LIONESS** (Baseline CSNI method evaluated with the same mean AUROC/AUPR protocol on the same dyngen datasets)
- **SSN** (Baseline CSNI method evaluated with the same mean AUROC/AUPR protocol on the same dyngen datasets)

## Examples

```
git clone https://github.com/dynverse/dyngen_manuscript && cd dyngen_manuscript && Rscript -e 'devtools::install("package/", dependencies = TRUE)'
```

## Evaluation signals

- Each dataset yields a mean AUROC and mean AUPR for the SCENIC condition, both bounded in [0, 1], comparable to the scores reported in the paper's CSNI benchmark.
- The per-cell interaction output is a set of (cell, regulator, target, score) tuples truncated to the top 10,000 interactions per cell.
- The comparison reproduces the ordering pySCENIC > LIONESS > SSN on both mean AUROC and mean AUPR across the 42 dyngen datasets.
- No motif-analysis filtering is applied to pySCENIC (step skipped) because dyngen generates no motif data; presence of a motif step indicates a deviation.
- Scores are averaged across all cells within each dataset before aggregation, not pooled naively across cells.

## Limitations

- A quantitative assessment of CSNI method performance was lacking until dyngen; results are therefore tied to dyngen's simulation assumptions.
- dyngen assumes cells are well-mixed and independent from each other, which may not hold in real tissues.
- dyngen does not generate motif data, so the standard pySCENIC motif-filtering step must be skipped, making the tested pySCENIC configuration non-default.
- For comparison, no extra cell or feature filtering was applied and parameters were matched to tool tutorials; no software version numbers were reported for any tool.

## Evidence

- [results] mean AUROC and AUPR across cells: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [results] lack of quantitative CSNI assessment: "a quantitative assessment of their performance is until now lacking"
- [intro] synthetic ground truth rationale: "Complementary to real biological data, synthetic data provides a valuable alternative where the actual ground truth is completely known and thus can be compared to, in order to make quantitative"
- [readme] manuscript reproduction scripts: [evidence span withheld — anchor status: external_doc]
- [methods] matched parameters, no extra filtering: "For both methods, we used the same normalised data as provided by dyngen, with no extra cell or feature filtering, but otherwise matched the parameters to their respective tutorial vignettes"
