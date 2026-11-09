---
name: false-negative-rate-estimation
description: Use when evaluating an RNA editing predictor (e.g. RDDpred) against a
  curated set of validated editing positives and negatives, and you need per-tool
  missed-site rates or ROC-AUC to rank competing classifiers.
when_to_use_negative:
- The input is raw RNAseq BAM data without a curated bona fide ground truth site set
  — FNR cannot be estimated without labeled positives
- The goal is de novo genome-wide editing discovery rather than benchmarking a predictor
  against known sites
- Sites lack the required support (≥50 RNAseq and ≥10 DNAseq reads, fully covered
  101-base window) and would distort the FNR denominator
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RDDpred
  role: Tool whose per-site decisions on true editing positives are used to estimate
    the false negative rate
  repo: https://github.com/vibbits/RDDpred
- name: REDInet
  role: Temporal convolutional network classifier whose FNR (~2%) and ROC-AUC (0.98)
    serve as the reference benchmark values
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: RED-ML
  role: Logistic regression comparator run on the identical site set to produce ROC-AUC
    baselines
  repo: https://github.com/BGIRED/RED-ML
- name: REDItools
  role: Generates the RNAseq pileup outTables (compressed and tabix-indexed) that
    feed classifier predictions on the ground truth sites
  repo: https://github.com/BioinfoUNIBA/REDItools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:c851c79866c2378ac9dfd2206112e474040e588c11e1bdcc4d6a1be087af5332
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/vibbits/RDDpred
  tools:
  - EditPredict
  - Python
  - RDDpred
  - RED-ML
  - REDInet
  - REDItools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# false-negative-rate-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates the false negative rate (FNR) of an RNA editing detection tool by running it on a bona fide ground-truth set of true A-to-I editing positives and counting missed sites, as done when benchmarking RDDpred (~7% FNR) against REDInet (2%) on the HEK293T KO-WT dataset.

## When to use

Use when evaluating an RNA editing predictor (e.g. RDDpred) against a curated set of validated editing positives and negatives, and you need per-tool missed-site rates or ROC-AUC to rank competing classifiers.

## When NOT to use

- The input is raw RNAseq BAM data without a curated bona fide ground truth site set — FNR cannot be estimated without labeled positives
- The goal is de novo genome-wide editing discovery rather than benchmarking a predictor against known sites
- Sites lack the required support (≥50 RNAseq and ≥10 DNAseq reads, fully covered 101-base window) and would distort the FNR denominator

## Inputs

- Ground truth HEK293T KO-WT site set: positives (A-to-G in RNAseq, no DNAseq change, present in REDIportal) and negatives (A-to-G in both RNAseq and DNAseq), with genomic positions
- Per-tool predicted editing probabilities / per-site editing calls on the same site set (EditPredict, RDDpred, RED-ML, REDInet)
- REDItools outTable (bgzip/tabix-indexed) pileups from the sample BAM files for classifier input

## Outputs

- False negative rate per tool on true editing positives (e.g. RDDpred ~7% vs REDInet 2%)
- ROC curves and ROC-AUC per tool joined to ground truth labels by genomic position (REDInet 0.98 > EditPredict 0.78 > RED-ML 0.72)
- Per-site decision table mapping each tool's call to the bona fide positive/negative label

## How to apply

Run each tool on the identical set of bona fide HEK293T KO-WT sites following each program's own published guidelines, recording per-site decisions. For FNR estimation, count true editing positives that the tool fails to call as edited and divide by the total number of ground-truth positives — in the REDInet benchmark this yielded RDDpred FNR ~7% versus 2% for REDInet. In parallel, join each tool's per-site predicted editing probabilities with the ground truth positive/negative labels by genomic position to compute ROC curves and ROC-AUC (REDInet 0.98, EditPredict 0.78, RED-ML 0.72). The ground truth positives are sites with A-to-G evidence in RNAseq, no base change in DNAseq (WGS/WES), presence in REDIportal, and ≥50 RNAseq reads support; negatives have A-to-G evidence in both RNAseq and DNAseq (RNA variants not due to editing). Because FNR is computed on curated positives only, sites must pass the full filtering cascade (≥10 DNAseq reads, 101-base window fully covered, ≥3 Gs or substitution rate ≥1%) before evaluation.

## Related tools

- **RDDpred** (Tool whose per-site decisions on true editing positives are used to estimate the false negative rate) — https://github.com/vibbits/RDDpred
- **REDInet** (Temporal convolutional network classifier whose FNR (~2%) and ROC-AUC (0.98) serve as the reference benchmark values) — https://github.com/BioinfoUNIBA/REDInet
- **RED-ML** (Logistic regression comparator run on the identical site set to produce ROC-AUC baselines) — https://github.com/BGIRED/RED-ML
- **REDItools** (Generates the RNAseq pileup outTables (compressed and tabix-indexed) that feed classifier predictions on the ground truth sites) — https://github.com/BioinfoUNIBA/REDItools

## Examples

```
python3 REDInet_Inference_light_ver.py -r outTable.gz -o redinet_pred -c 50 -s 3 -f 0.01; python -m reditools analyze sample.bam  # then join per-tool calls to ground truth labels by position
```

## Evaluation signals

- Reproduced AUC ordering holds: REDInet (0.98) > EditPredict (0.78) > RED-ML (0.72)
- RDDpred false negative rate on true editing positives is approximately 7%, versus 2% for REDInet
- Every tool's per-site predictions join to ground truth labels one-to-one by genomic position with no unmatched sites
- All evaluated sites satisfy the filtering cascade: ≥50 RNAseq and ≥10 DNAseq reads, RNAseq A-to-G rate ≥0.4 (negatives), 101-base window fully covered

## Limitations

- FNR depends entirely on the quality and completeness of the curated ground truth positives; mislabeled sites bias the estimate
- Each tool must be run 'according the guidelines of each specific program', so configuration differences may confound cross-tool comparison
- Only sites with the 101-base window fully covered by RNAseq reads were maintained, excluding partially covered regions from the FNR denominator

## Evidence

- [other] Run RDDpred, specialized in separating sequencing artifacts from genuine A-to-G substitutions, on the same sites and record per-site decisions to estimate the false negative rate on true editing positives.: [evidence span withheld — anchor status: not_found]
- [results] RDDpred obtained a false negative rate of ~7% versus 2% for REDInet.: [evidence span withheld — anchor status: not_found]
