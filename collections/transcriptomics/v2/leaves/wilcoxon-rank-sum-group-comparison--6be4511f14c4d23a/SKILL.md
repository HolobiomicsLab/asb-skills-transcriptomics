---
name: wilcoxon-rank-sum-group-comparison
description: Use when comparing a continuous per-region metric (e.g. novel/structured
  RNA fraction of dsRID-predicted dsRNA regions) between two independent sample groups
  such as AD and control brain, without assuming normality.
when_to_use_negative:
- When the comparison involves only categorical overlap counts (shared vs group-specific
  regions) — use Fisher's exact test on the contingency table, as the study does
- When the two sample groups are paired or matched rather than independent AD and
  control cohorts — the rank-sum test assumes independent samples
- When input is raw read data rather than per-region dsRID predictions — the test
  operates on computed per-region/per-sample fractions, not on BAM files directly
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: dsRID
  role: Generates the per-sample dsRID_whole.tsv predicted dsRNA probabilities whose
    AD/control overlap and novel/structured RNA fractions are the quantities statistically
    compared
  repo: https://github.com/gxiaolab/dsRID
- name: Python
  role: Runtime (3.x) in which the dsRID pipeline and the Fisher's exact / Wilcoxon
    rank-sum statistical tests on the predicted-region classifications are executed
  repo: https://github.com/gxiaolab/dsRID
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:c1ec65367dc2558628aac2fe6cfd91fc93a8f6291f3427effd618bfa07148b3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - dsRID
  - extract_whole.py
  - pip
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# wilcoxon-rank-sum-group-comparison

## Summary

Non-parametric two-group comparison of region-level continuous metrics — here, the novel/structured RNA fractions of candidate dsRNA regions predicted by dsRID in AD versus control brain samples from the ENCODE consortium PacBio long-read RNA-seq cohort — used alongside Fisher's exact test on the shared/AD-specific/control-specific overlap contingency.

## When to use

Use when comparing a continuous per-region metric (e.g. novel/structured RNA fraction of dsRID-predicted dsRNA regions) between two independent sample groups such as AD and control brain, without assuming normality.

## When NOT to use

- When the comparison involves only categorical overlap counts (shared vs group-specific regions) — use Fisher's exact test on the contingency table, as the study does
- When the two sample groups are paired or matched rather than independent AD and control cohorts — the rank-sum test assumes independent samples
- When input is raw read data rather than per-region dsRID predictions — the test operates on computed per-region/per-sample fractions, not on BAM files directly

## Inputs

- dsRID_whole.tsv per sample (whole-genome scan of candidate dsRNA regions with random-forest predicted probabilities)
- Sample-group labels (AD vs control) for ENCODE consortium PacBio long-read RNA-seq BAM files
- data/Encode_novel_dsRNA_source.tsv (dsRID-predicted AD-enriched, control-enriched, and shared dsRNA regions)

## Outputs

- dsRID_comparative_stats.tsv containing reproduced percentages (76.3% shared, 14.2% AD-specific, 9.3% control-specific), Fisher's exact test statistic (P < 2.2e-16), and Wilcoxon rank-sum statistic (P = .017)

## How to apply

After running dsRID on each ENCODE consortium AD-cohort PacBio BAM (via bash dsRID.sh or the extract_whole.py + model_predict.py shortcut to get dsRID_whole.tsv with random-forest predicted dsRNA probabilities), classify predicted regions per sample as candidate dsRNA regions and compute the overlap fractions between AD and control groups: 76.3% shared, 14.2% AD-specific, 9.3% control-specific. Test the shared vs AD-specific vs control-specific overlap contingency with Fisher's exact test (expected P < 2.2e-16). Separately, compute the novel/structured RNA fraction per sample in each group and compare the AD vs control distributions with a Wilcoxon rank-sum (Mann-Whitney U) test, which is appropriate because fraction values are non-normal continuous scores rather than counts. Write the reproduced percentages and both test statistics to dsRID_comparative_stats.tsv; the validation criterion is a Wilcoxon rank-sum P = .017 with a higher novel/structured RNA fraction in AD.

## Related tools

- **dsRID** (Generates the per-sample dsRID_whole.tsv predicted dsRNA probabilities whose AD/control overlap and novel/structured RNA fractions are the quantities statistically compared) — https://github.com/gxiaolab/dsRID
- **Python** (Runtime (3.x) in which the dsRID pipeline and the Fisher's exact / Wilcoxon rank-sum statistical tests on the predicted-region classifications are executed) — https://github.com/gxiaolab/dsRID

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(ad_fracs, ctrl_fracs, alternative='two-sided')
```

## Evaluation signals

- Reproduced overlap percentages match the reported 76.3% shared, 14.2% AD-specific, and 9.3% control-specific candidate dsRNA regions
- Fisher's exact test on the overlap contingency yields P < 2.2e-16
- Wilcoxon rank-sum test on novel/structured RNA fractions yields P = .017 with the higher fraction in AD samples
- Output file dsRID_comparative_stats.tsv exists and contains both the reproduced percentages and the test statistics
- Per-sample candidate-region classifications derive from dsRID_whole.tsv probability columns (random-forest model) rather than raw read data

## Limitations

- No version of dsRID or specific package versions is stated in the README (required packages only listed in requirements.txt)
- The reported statistics (76.3%/14.2%/9.3%, P < 2.2e-16, P = .017) are specific to the ENCODE consortium AD cohort and PacBio long-read RNA-seq data; other cohorts may differ
- The AD vs control comparison is described as 'potentially distinct expression patterns', i.e. the Wilcoxon result supports a modest difference rather than a definitive biological conclusion
- No discussion section content was available in the source document

## Evidence

- [abstract] AD cohort analysis reveals a global dsRNA profile with potentially distinct expression patterns between AD and controls: "we characterize the global dsRNA profile with potentially distinct expression patterns between AD and controls"
- [readme] Overlap categories of dsRID predictions between AD and control provide the classification used for group comparison statistics.: "dsRID predicted AD enriched, control enriched and AD contorl shared dsRNA from ENCODE consortium is available under data/Encode_novel_dsRNA_source.tsv."
- [readme] dsRID_whole.tsv is the per-region probability table from which candidate dsRNA regions are classified before statistical testing.: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv."
