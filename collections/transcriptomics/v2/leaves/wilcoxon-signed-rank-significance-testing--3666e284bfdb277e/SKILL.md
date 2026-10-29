---
name: wilcoxon-signed-rank-significance-testing
description: Use when two or more classifiers have been run on the identical reference
  and validation datasets and per-sample (paired) performance scores exist, and you
  must test whether the performance gap for one class (e.g., cancer-cell F1) is significant.
when_to_use_negative:
- Scores are not paired — the tools were tested on different samples or datasets,
  so a signed-rank test on differences is invalid
- Only aggregated summary statistics (e.g., a single median F1 per tool) are available
  with no per-biopsy values to pair
- The comparison targets a single proportion or two-group continuous outcome with
  a different design — a t-test or unpaired rank-sum test is the appropriate choice
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Environment in which the paired Wilcoxon test on per-biopsy F1 scores is computed
    for the Fig. 2c significance analysis
- name: scATOMIC
  role: Test classifier whose cancer-cell F1 scores are compared against each of the
    six annotators
  repo: github.com/abelson-lab/scATOMIC
- name: SingleR
  role: Best-performing comparator annotator (cancer-cell F1 0.76) against which scATOMIC's
    gap is tested
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:d9c97e83491ca81be6bdadee69d4ca23c4cbba7824963550d03c91918559c6d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - R
  - Seurat
  - SingleCellNet
  - SingleR
  - scATOMIC
  - scType
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# wilcoxon-signed-rank-significance-testing

## Summary

Applies a paired Wilcoxon signed-rank test to per-sample F1 scores from a controlled classifier benchmark, testing whether scATOMIC's cancer-cell classification advantage over six comparators (SingleR, Seurat, SingleCellNet, scmap-cell, CHETAH, scType) is statistically significant as reported for Fig. 2c.

## When to use

Use when two or more classifiers have been run on the identical reference and validation datasets and per-sample (paired) performance scores exist, and you must test whether the performance gap for one class (e.g., cancer-cell F1) is significant.

## When NOT to use

- Scores are not paired — the tools were tested on different samples or datasets, so a signed-rank test on differences is invalid
- Only aggregated summary statistics (e.g., a single median F1 per tool) are available with no per-biopsy values to pair
- The comparison targets a single proportion or two-group continuous outcome with a different design — a t-test or unpaired rank-sum test is the appropriate choice

## Inputs

- Per-sample F1 scores for each classifier on the same 225-biopsy validation set, stratified by combined class (blood, stromal, cancer)
- Comparator F1 score table (comparator_f1_scores.csv) covering scATOMIC, SingleR, Seurat, SingleCellNet, scmap-cell, CHETAH, scType
- Paired sample identifiers linking each tool's predictions to the same biopsy

## Outputs

- Per-comparator Wilcoxon test statistics and P-values for the cancer-cell F1 gap (Fig. 2c significance calls)
- comparator_f1_scores.csv with per-tool blood/stroma/cancer F1 values
- Significance statement supporting that scATOMIC outperforms current classification methods

## How to apply

After benchmarking each annotator on the same pan-cancer training reference (>300,000 cancer, immune, and stromal cells across 19 cancers) and the external validation set (225 tumour biopsies, >350,000 cells), map every tool's fine-grained predictions onto the three combined categories — blood cell, stromal cell, and cancer cell — and compute combined-class F1 scores. Because each tool is tested on the same biopsies, scores are paired, so use a two-sided or one-sided paired Wilcoxon signed-rank test (not an unpaired rank-sum test) comparing scATOMIC's cancer-cell F1 values against each comparator's. Report the test statistic and P-value per comparator; the paper reports all cancer-cell comparisons at P < 2 × 10−16, supporting that scATOMIC outperforms current classification methods. Write the per-tool F1 table (including scATOMIC at 0.95/0.99/0.99 blood/stroma/cancer and best comparator SingleR at 0.92/0.97/0.76) to comparator_f1_scores.csv so scores and significance are co-reported.

## Related tools

- **R** (Environment in which the paired Wilcoxon test on per-biopsy F1 scores is computed for the Fig. 2c significance analysis)
- **scATOMIC** (Test classifier whose cancer-cell F1 scores are compared against each of the six annotators) — github.com/abelson-lab/scATOMIC
- **SingleR** (Best-performing comparator annotator (cancer-cell F1 0.76) against which scATOMIC's gap is tested)

## Examples

```
wilcox.test(f1_cancer_scATOMIC, f1_cancer_comparator, paired = TRUE, alternative = "greater")
```

## Evaluation signals

- Every cancer-cell F1 comparison yields P < 2 × 10−16 in favour of scATOMIC, matching the Fig. 2c report
- Per-tool blood/stroma/cancer F1 values in the output table match the reported values (scATOMIC 0.95/0.99/0.99; SingleR 0.92/0.97/0.76)
- The test is applied to paired per-biopsy score differences, not pooled or unpaired values
- Direction of the difference is consistent across biopsies (scATOMIC ≥ comparator for the large majority of paired samples)
- The F1 table written to comparator_f1_scores.csv contains one row per tool with all three combined classes populated

## Limitations

- The significance claim in the paper is specific to cancer-cell F1 comparisons; for blood and stromal cells scATOMIC is described as 'comparable to or outperforms' existing annotators, so the test may not show a significant gap for those classes
- Results are contingent on the benchmark design — all tools trained on the same >300,000-cell pan-cancer reference across 19 cancers and tested on the same 225 external biopsies; changing the reference or validation set changes the test outcome
- Wilcoxon tests rank-based differences only and does not quantify effect size of the F1 gap

## Evidence

- [results] scATOMIC achieved median F1 scores of 0.95, 0.99, and 0.99 for blood, stromal, and cancer cells respectively, outperforming all comparators (all cancer-cell comparisons P < 2 × 10−16): [evidence span withheld — anchor status: not_found]
- [other] Wilcoxon test on Fig. 2c: [evidence span withheld — anchor status: not_found]
- [discussion] Comparable or outperforms other annotators: "scATOMIC is comparable to or outperforms other existing automatic cell type annotators when classifying blood and stromal cells using our training reference."
- [abstract] 225-biopsy validation set: "We extensively confirm scATOMIC's accuracy on 225 tumour biopsies encompassing >350,000 cancer and a variety of TME cells."
