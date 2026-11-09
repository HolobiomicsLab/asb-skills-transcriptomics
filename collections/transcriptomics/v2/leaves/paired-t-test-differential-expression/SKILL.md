---
name: paired-t-test-differential-expression
description: Use when you have paired qPCR samples (e.g., 12 tumor/adjacent-normal
  breast cancer pairs) with raw CT values for a target miRNA and 2–3 internal controls,
  and you need to test differential expression under alternative reference-gene normalization
  schemes.
when_to_use_negative:
- Samples are unpaired/independent groups — the paired t test assumes matched tumor/adjacent-normal
  pairs; use an unpaired test instead
- The experiment aggregates more than 5 reference genes, where the article notes geom(sd)
  'does not have the expected benefits compared to the regular geometric mean', making
  the weighted-vs-unweighted comparison uninformative
- You only have normalized high-throughput expression data (e.g., TCGA miRNA-seq)
  rather than raw qPCR CT values for the internal controls
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3519
- http://edamontology.org/topic_0203
tools:
- name: InterOpt
  role: Computes the geom(sd) aggregation weights (calcWeight, ctVal = TRUE) from
    raw CT values of the internal controls to build the weighted geometric-mean virtual
    reference used before the paired t test
  repo: https://github.com/asalimih/InterOpt
- name: R
  role: Runtime environment for the InterOpt package and for running the paired t
    test on the normalized target values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:1f3e72d9d7d2f7576a5066e4fa068a39e85eab9a3fd7465d709c9327a29b5c12
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3519
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# paired-t-test-differential-expression

## Summary

Statistical validation skill for qPCR experiments: after normalizing a target gene (e.g., hsa-miR-21-5p) by a virtual reference gene built from internal controls, run a paired t test between matched tumor/normal pairs under competing normalization schemes (geom(sd)-weighted vs unweighted geometric mean) to test whether the weighting materially changes the differential-expression verdict.

## When to use

Use when you have paired qPCR samples (e.g., 12 tumor/adjacent-normal breast cancer pairs) with raw CT values for a target miRNA and 2–3 internal controls, and you need to test differential expression under alternative reference-gene normalization schemes.

## When NOT to use

- Samples are unpaired/independent groups — the paired t test assumes matched tumor/adjacent-normal pairs; use an unpaired test instead
- The experiment aggregates more than 5 reference genes, where the article notes geom(sd) 'does not have the expected benefits compared to the regular geometric mean', making the weighted-vs-unweighted comparison uninformative
- You only have normalized high-throughput expression data (e.g., TCGA miRNA-seq) rather than raw qPCR CT values for the internal controls

## Inputs

- Raw CT matrix (rows = 3 internal controls miR-16-5p, miR-361-5p, RNU48 + target miR-21-5p; columns = paired tumor/normal samples) from the 12-pair breast cancer tissue qPCR validation experiment
- InterOpt calcWeight output: geom(sd) aggregation weights (ctVal = TRUE) for the internal controls

## Outputs

- Normalized miR-21-5p values (target CT minus virtual-reference CT) per sample under the geom(sd)-weighted scheme
- Normalized miR-21-5p values under the unweighted geometric-mean scheme
- ttest_results.csv containing paired t-test statistics and p-values for both normalization schemes

## How to apply

Load the raw CT matrix containing the internal controls (miR-16-5p, miR-361-5p, RNU48) and the target (miR-21-5p) for each paired tumor/normal sample. Compute geom(sd) aggregation weights with InterOpt's calcWeight (ctVal = TRUE), which minimizes the SD of the log expression of the virtual reference gene, then subtract the weighted geometric-mean reference CT from the target CT per sample. Repeat the normalization using the unweighted geometric mean of the same controls as the comparator scheme. For each scheme, run a paired t test on the normalized miR-21-5p values between tumor and adjacent-normal pairs and write the statistics and p-values to ttest_results.csv. The correct outcome mirrors the paper's 12-pair experimental validation: significant miR-21-5p upregulation under geom(sd)-weighted normalization but no significant change under the unweighted geometric mean.

## Related tools

- **InterOpt** (Computes the geom(sd) aggregation weights (calcWeight, ctVal = TRUE) from raw CT values of the internal controls to build the weighted geometric-mean virtual reference used before the paired t test) — https://github.com/asalimih/InterOpt
- **R** (Runtime environment for the InterOpt package and for running the paired t test on the normalized target values)

## Examples

```
library(InterOpt); w <- calcWeight(x, ctVal=TRUE, weight_method='geom_sd_plus'); new_control <- colSums(w*x); t.test(norm_tumor, norm_normal, paired=TRUE)
```

## Evaluation signals

- Directional expectation: paired t-test p-value is significant (miR-21-5p upregulation) under geom(sd)-weighted normalization but non-significant under the unweighted geometric mean, matching the paper's 12-pair validation
- Weights sanity check: calcWeight weights (ctVal = TRUE) sum to ~1 and the virtual-reference CT equals the weighted sum colSums(w*x) of the control CTs
- Schema check: ttest_results.csv contains one row per normalization scheme with statistic and p-value columns
- Sample integrity: the paired structure (12 tumor/adjacent-normal pairs) is preserved — each pair contributes one difference value to the t test

## Limitations

- The experimental validation of hsa-miR-21-5p used a low number of samples (12 pairs), limiting statistical power
- Aggregating more than 5 reference genes by geom(sd) does not provide expected benefits over the regular geometric mean
- InterOpt weights are optimized for the virtual reference; if control CTs are noisy or one control is unstable, the weighted reference (and hence the t-test verdict) can shift

## Evidence
