---
name: fishers-exact-enrichment-testing
description: Use when two sample groups yield sets of candidate regions (e.g., predicted
  dsRNA regions per sample) and you need to test whether the shared vs group-specific
  overlap fractions differ significantly between groups.
when_to_use_negative:
- Input regions have not yet been classified per sample as candidate dsRNA regions
  — run dsRID prediction first rather than testing raw BAM-derived candidates.
- The comparison involves more than two groups or paired/clustered region-level counts
  where a 2x2 contingency test is inappropriate.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3512
tools:
- name: dsRID
  role: Produces the per-sample predicted dsRNA regions whose overlap fractions are
    tested by the contingency analysis
  repo: https://github.com/gxiaolab/dsRID
- name: Python
  role: Environment (Python 3.x) in which Fisher's exact and Wilcoxon rank-sum tests
    are executed and statistics written to TSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:77073e59260df2f5d5032c72f98a52ae502de179634a08ab093a0676275b4d14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - dsRID
  - extract_whole.py
  - pip
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fishers-exact-enrichment-testing

## Summary

Statistical testing skill for comparing overlap/enrichment of predicted dsRNA regions between sample groups (AD vs control) using Fisher's exact test on overlap contingency counts, with Wilcoxon rank-sum as a complementary test on continuous fractions.

## When to use

Use when two sample groups yield sets of candidate regions (e.g., predicted dsRNA regions per sample) and you need to test whether the shared vs group-specific overlap fractions differ significantly between groups.

## When NOT to use

- Input regions have not yet been classified per sample as candidate dsRNA regions — run dsRID prediction first rather than testing raw BAM-derived candidates.
- The comparison involves more than two groups or paired/clustered region-level counts where a 2x2 contingency test is inappropriate.

## Inputs

- Per-sample candidate dsRNA region calls from dsRID (dsRID_whole.tsv with random-forest predicted probabilities)
- Precomputed group overlap sets from data/Encode_novel_dsRNA_source.tsv (AD-enriched, control-enriched, AD-control shared dsRNA)
- Contingency counts of shared vs AD-specific vs control-specific candidate regions

## Outputs

- Overlap percentages per category (e.g., 76.3% shared, 14.2% AD-specific, 9.3% control-specific)
- Fisher's exact test P-value for the overlap contingency (reported as P < 2.2e-16)
- Wilcoxon rank-sum P-value comparing novel/structured RNA fractions between groups (reported as P = .017)
- dsRID_comparative_stats.tsv containing reproduced percentages and test statistics

## How to apply

After running dsRID on each AD and control brain sample (bash dsRID.sh <input_bam> <output_dir>, or extract_whole.py + model_predict.py to bypass training), classify random-forest predicted regions per sample as candidate dsRNA regions. Build a 2x2 contingency table over all candidate regions tested in both groups: shared (present in both AD and control), AD-specific, and control-specific, then compute overlap fractions. Apply Fisher's exact test to the shared vs group-specific counts and the Wilcoxon rank-sum test to compare novel/structured RNA fractions between groups. Write the reproduced percentages and test statistics to dsRID_comparative_stats.tsv.

## Related tools

- **dsRID** (Produces the per-sample predicted dsRNA regions whose overlap fractions are tested by the contingency analysis) — https://github.com/gxiaolab/dsRID
- **Python** (Environment (Python 3.x) in which Fisher's exact and Wilcoxon rank-sum tests are executed and statistics written to TSV)

## Examples

```
python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv
```

## Evaluation signals

- Reproduced shared fraction equals 76.3% of candidate regions tested in both AD and control samples.
- AD-specific vs control-specific fractions reproduce 14.2% vs 9.3%.
- Fisher's exact test on the overlap contingency yields P < 2.2e-16.
- Wilcoxon rank-sum test on novel/structured RNA fractions yields P = .017, with a higher fraction in AD.
- Output file dsRID_comparative_stats.tsv contains both percentages and test statistics in the expected schema.

## Limitations

- No version of dsRID or specific package versions are stated, so exact reproducibility of test statistics may depend on package versions in requirements.txt.
- Reported significance is based on the ENCODE AD cohort as processed in the paper; novel/structured RNA fraction comparison (Wilcoxon P = .017) is described as a 'potentially distinct' pattern rather than a definitive group difference.
- The comparison is limited to regions tested in both groups; regions called in only one sequencing context are excluded from the contingency.

## Evidence

- [other] AD vs control overlap percentages and tests: "76.3% were identified as long dsRNAs in both groups, whereas 14.2% were specific to AD samples and 9.3% specific to controls."
- [results] AD vs control global profile: "We applied dsRID to an AD cohort sequenced by the ENCODE consortium and characterized the global dsRNA profile with potentially distinct expression patterns between AD and controls."
- [readme] dsRID output probability column: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model"
- [results] Precomputed group-enriched dsRNA sets: "dsRID predicted AD enriched, control enriched and AD contorl shared dsRNA from ENCODE consortium is available under data/Encode_novel_dsRNA_source.tsv."
- [other] Group comparison validation criteria: [evidence span withheld — anchor status: not_found]
