---
name: paired-wilcoxon-signed-rank-testing
description: Use when comparing a continuous per-sample score (e.g. signalling entropy
  SR from SCENT) between matched patient pairs, such as diagnostic–relapse AML bulk
  samples, and the score distribution is not assumed normal.
when_to_use_negative:
- The samples are not paired/matched (e.g. independent CTC patient groups) — use an
  unpaired Wilcoxon rank-sum test as in the workflow for treated vs untreated CTCs
- The input is raw expression data rather than a computed per-sample/per-cell score
  — SCENT must first be run to estimate signalling entropy before testing
- The data are known to be normally distributed and pairing is informative enough
  that a paired t-test would have substantially more power
edam_operation: http://edamontology.org/operation_3224
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SCENT
  role: Produces the signalling entropy (SR) values per sample/cell that are then
    compared with the paired Wilcoxon signed-rank test
  repo: https://github.com/aet21/SCENT
- name: R
  role: Environment in which the paired Wilcoxon signed-rank test (wilcox.test, paired=TRUE)
    is executed on SR values
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:3194dae1e846c77715f7d3bafa5dfda0193e7f63bbe1151881b69938a62f6c2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_3224
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Paired Wilcoxon signed-rank testing

## Summary

A non-parametric paired statistical test used in this workflow to detect systematic shifts in SCENT-estimated signalling entropy (SR) between matched samples, e.g. relapsed vs diagnostic AML bulk RNA-Seq pairs, where a significant shift (P=0.004) marks drug-resistant, higher-entropy populations.

## When to use

Use when comparing a continuous per-sample score (e.g. signalling entropy SR from SCENT) between matched patient pairs, such as diagnostic–relapse AML bulk samples, and the score distribution is not assumed normal.

## When NOT to use

- The samples are not paired/matched (e.g. independent CTC patient groups) — use an unpaired Wilcoxon rank-sum test as in the workflow for treated vs untreated CTCs
- The input is raw expression data rather than a computed per-sample/per-cell score — SCENT must first be run to estimate signalling entropy before testing
- The data are known to be normally distributed and pairing is informative enough that a paired t-test would have substantially more power

## Inputs

- Per-sample SCENT signalling entropy (SR) values for paired diagnostic–relapse AML bulk RNA-Seq samples, keyed by patient
- Per-cell SR values with CD34/CD96 marker annotations for single-cell AML RNA-Seq data (enrichment testing)
- Per-cell SR values for CTCs from enzalutamide-treated and untreated patients (between-group testing)

## Outputs

- Two-sided Wilcoxon signed-rank test P-value for the paired SR comparison (target P=0.004)
- Wilcoxon P-values for CD34 (P=0.008) and CD96 (P=0.032) enrichment among highest-entropy cells
- Wilcoxon P-value for treated vs untreated CTC SR difference (target P=0.047)
- Compiled comparison table aml_ctc_scent_reproduction.csv

## How to apply

After computing SCENT signalling entropy (SR) for each member of each within-patient pair (here, diagnostic and relapse AML bulk RNA-Seq samples), organise the values as paired vectors keyed by patient. Run a two-sided paired Wilcoxon signed-rank test on the SR differences within pairs using R (wilcox.test with paired=TRUE), which ranks absolute within-pair differences and tests symmetry around zero without assuming normality of SR. The target result from the source workflow is P=0.004 for higher entropy in relapsed vs matched diagnostic AML. The same test framework is used in parallel settings without pairing: a standard (unpaired) Wilcoxon rank-sum test for CD34/CD96 enrichment among highest-entropy cells (expected P=0.008 and P=0.032) and for enzalutamide-treated vs untreated CTCs (expected P=0.047). Compile all reproduced SR comparisons and P-values into the output table aml_ctc_scent_reproduction.csv.

## Related tools

- **SCENT** (Produces the signalling entropy (SR) values per sample/cell that are then compared with the paired Wilcoxon signed-rank test) — https://github.com/aet21/SCENT
- **R** (Environment in which the paired Wilcoxon signed-rank test (wilcox.test, paired=TRUE) is executed on SR values)

## Examples

```
wilcox.test(scent_sr_relapse, scent_sr_diagnostic, paired = TRUE, alternative = "two.sided")
```

## Evaluation signals

- The paired diagnostic–relapse AML SR comparison reproduces the reported P=0.004 (relapsed samples higher entropy)
- CD34 enrichment among highest-entropy cells reproduces P=0.008 and CD96 enrichment P=0.032
- The enzalutamide-treated vs untreated CTC SR comparison reproduces P=0.047
- All tests are correctly paired by patient (paired=TRUE only for diagnostic–relapse pairs, not for between-group comparisons)
- All reproduced SR comparisons and P-values are present in aml_ctc_scent_reproduction.csv

## Limitations

- Signalling entropy comparisons are in silico estimates; no existing assay can measure all single cells in a population, limiting direct validation of bulk-vs-single-cell consistency
- Wilcoxon P-values depend on the SCENT SR estimates, which require an interaction network context; other entropy-based measures (StemID, SLICE) that lack network information give different results
- The paired test assumes within-pair differences are independent and roughly symmetric; heavy outliers or small numbers of pairs (e.g. few AML diagnostic–relapse matches) reduce reliability

## Evidence

- [other] Paired Wilcoxon for SR shift: [evidence span withheld — anchor status: not_found]
- [other] Finding supporting the test: [evidence span withheld — anchor status: not_found]
- [other] Output table: [evidence span withheld — anchor status: not_found]
- [abstract] Signalling entropy identifies drug-resistant phenotypes: "Signalling entropy identifies known cell subpopulations of varying potency and drug resistant cancer stem-cell phenotypes, including those derived from circulating tumour cells"
- [readme] Tool context from README: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge such as marker expression"
