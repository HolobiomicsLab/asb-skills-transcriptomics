---
name: benjamini-hochberg-multiple-testing-correction
description: Use when many gene-set (GO_BP/KEGG) enrichment tests are run in parallel
  — e.g., one test per detected RGM or SCENIC regulon on the hESC, mESC, or mHSC lineage
  datasets — and per-module significance must be assessed with a threshold that accounts
  for multiple testing.
when_to_use_negative:
- Only a single enrichment test is performed — no multiplicity exists, so a plain
  p-value cutoff suffices.
- The goal is per-gene differential-expression testing rather than per-module gene-set
  enrichment significance; other correction contexts may need different family-wise
  structures.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: Detects the regulon-like gene modules whose enrichment p-values are BH-corrected
    to compute enrichment precision
  repo: https://github.com/WPZgithub/CEFCON
- name: pyscenic
  role: Baseline regulon-detection method whose modules receive identical BH correction
    and precision scoring for comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:1bc3cbe9c3f04cdb2902115e9da8d770af50634603c2355f78a9e6b5395c5a32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - pyscenic
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Apply Benjamini-Hochberg (BH) false-discovery-rate correction to per-module GO_BP and KEGG enrichment p-values when benchmarking regulon-like gene modules (RGMs) against SCENIC regulons, so that significance calls across many tested modules are controlled and enrichment precision (fraction of modules with BH-adjusted p <= 0.05) is computed fairly per dataset.

## When to use

Use when many gene-set (GO_BP/KEGG) enrichment tests are run in parallel — e.g., one test per detected RGM or SCENIC regulon on the hESC, mESC, or mHSC lineage datasets — and per-module significance must be assessed with a threshold that accounts for multiple testing.

## When NOT to use

- Only a single enrichment test is performed — no multiplicity exists, so a plain p-value cutoff suffices.
- The goal is per-gene differential-expression testing rather than per-module gene-set enrichment significance; other correction contexts may need different family-wise structures.

## Inputs

- Per-module raw (uncorrected) enrichment p-values from GO_BP functional enrichment of CEFCON RGMs
- Per-module raw enrichment p-values from KEGG pathway enrichment of CEFCON RGMs
- Per-module raw enrichment p-values from GO_BP/KEGG enrichment of pySCENIC regulons (baseline comparison)
- Module membership lists per lineage (e.g., RGMs.csv) from cefcon_results.RGM_activity()

## Outputs

- BH-adjusted p-value per gene module per enrichment database (GO_BP, KEGG)
- Binary significance calls per module at BH-adjusted p <= 0.05
- Per-dataset enrichment precision = fraction of modules significant at BH-adjusted p <= 0.05, for CEFCON vs. SCENIC (Fig. 5b)

## How to apply

For each of the five tested lineages (hESC, mESC, and three mHSC lineages), first run GO_BP and KEGG enrichment on every detected regulon-like gene module obtained via cefcon_results.RGM_activity() (or on the pySCENIC regulons for the baseline). Collect the raw enrichment p-values across all modules in the dataset and apply the Benjamini-Hochberg FDR procedure to adjust them within each dataset/tool comparison. Call a module 'significant' when its BH-adjusted p <= 0.05 and compute enrichment precision as the fraction of significant modules; the same BH-corrected thresholding is applied identically to CEFCON RGMs and SCENIC regulons so the comparison in Fig. 5b is apples-to-apples. Rationale: enrichment precision is the primary metric for judging whether CEFCON captures 'more biologically reasonable and meaningful gene modules', and without BH correction the fraction-significant score would be inflated by the large number of modules tested.

## Related tools

- **CEFCON** (Detects the regulon-like gene modules whose enrichment p-values are BH-corrected to compute enrichment precision) — https://github.com/WPZgithub/CEFCON
- **pyscenic** (Baseline regulon-detection method whose modules receive identical BH correction and precision scoring for comparison)

## Examples

```
cefcon_results.RGM_activity()  # detect RGMs per lineage, then BH-correct per-module GO_BP/KEGG p-values at p <= 0.05
```

## Evaluation signals

- Every module in a dataset has a non-missing BH-adjusted p-value in [0,1] for both GO_BP and KEGG enrichment.
- For each dataset, BH-adjusted p-values are monotonically consistent with raw p-values (adjusted >= raw) and the BH rank/step-up procedure was applied within a dataset-tool family, not pooled across datasets.
- Significance is declared strictly at BH-adjusted p <= 0.05; enrichment precision equals (# significant modules)/(# tested modules) per dataset.
- The same BH correction and 0.05 cutoff are applied to both CEFCON RGMs and pySCENIC regulons before the Fig. 5b comparison table is compiled.
- Compiled table reproduces the paper's qualitative result: CEFCON attains higher GO_BP/KEGG enrichment precision than SCENIC across all five tested datasets.

## Limitations

- The finding that CEFCON outperforms SCENIC holds only for the five tested datasets (hESC, mESC, three mHSC lineages); BH-corrected precision should be recomputed for new datasets.
- Precision at BH-adjusted p <= 0.05 measures module-level enrichment significance, not biological correctness of the module itself.
- Significance calls depend on the upstream module set (GRN edges selected with edge_threshold_avgDegree=8), which changes the size of the testing family.

## Evidence

- [methods] Perform GO_BP and KEGG enrichment on each module and compute the precision as the fraction of modules significant at BH-adjusted p <= 0.05.: [evidence span withheld — anchor status: not_found]
- [intro] In addition, CEFCON detects the regulon-like gene modules (RGMs) involving the identified driver regulators.: "In addition, CEFCON detects the regulon-like gene modules (RGMs) involving the identified driver regulators."
- [readme] CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using AUCell: "CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using AUCell"
- [results] In functional enrichment precision evaluation, CEFCON substantially outperformed SCENIC on all five tested datasets: [evidence span withheld — anchor status: not_found]
