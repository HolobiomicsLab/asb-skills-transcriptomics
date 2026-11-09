---
name: classifier-accuracy-specificity-precision-evaluation
description: Use when a per-site classifier output (SNP vs editing probability/label
  for A-to-G candidates) must be scored against a ground truth set such as HEK293T
  KO-WT, KO-OE, or A549 bona fide editing sites.
when_to_use_negative:
- Input lacks a trusted ground truth label set (e.g. sites were not curated by contrasting
  ADAR KO vs WT or KO vs OE samples), so accuracy/specificity/precision cannot be
  meaningfully computed.
- The goal is discovery of new editing events in unannotated samples rather than benchmarking
  a classifier against known bona fide sites.
- Candidate sites violate the inference preconditions (101-base window not fully covered,
  coverage below 50, or A-to-G substitution rate below 1% / fewer than 3 Gs), since
  REDInet predictions on such sites are discarded rather than evaluated.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: REDInet
  role: Pretrained REDInet.h5 temporal convolutional network producing SNP vs editing-site
    probabilities and softmax class labels per candidate site for benchmarking
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: REDItools v1
  role: Generates low-stringency pileup profiles of variant and invariant A-to-G candidate
    sites (REDItoolDnaRna.py) that constitute the prediction input
  repo: https://github.com/BioinfoUNIBA/REDItools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:737722f7894689e6324926673033761ffa5668d1c5e5ca6f8b4604e5205c80ec
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet.git
  tools:
  - Python
  - REDInet
  - REDItools v1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# classifier-accuracy-specificity-precision-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a binary editing-site classifier (e.g. REDInet on REDItools pileup outTables) against ground truth A-to-I labels by computing accuracy, specificity, and precision per site, replicating the HEK293T KO-OE evaluation protocol.

## When to use

Use when a per-site classifier output (SNP vs editing probability/label for A-to-G candidates) must be scored against a ground truth set such as HEK293T KO-WT, KO-OE, or A549 bona fide editing sites.

## When NOT to use

- Input lacks a trusted ground truth label set (e.g. sites were not curated by contrasting ADAR KO vs WT or KO vs OE samples), so accuracy/specificity/precision cannot be meaningfully computed.
- The goal is discovery of new editing events in unannotated samples rather than benchmarking a classifier against known bona fide sites.
- Candidate sites violate the inference preconditions (101-base window not fully covered, coverage below 50, or A-to-G substitution rate below 1% / fewer than 3 Gs), since REDInet predictions on such sites are discarded rather than evaluated.

## Inputs

- Per-site classifier predictions (SNP vs editing-site probabilities and softmax-derived class labels, e.g. from REDInet on tabix-indexed REDItools outTable.gz)
- Ground truth label set, e.g. HEK293T KO-OE (>35,000 sites), KO-WT (>20,000 events), or A549 (>13,000 sites) bona fide A-to-I editing sites from the BioinfoUNIBA/REDInet GitHub repository

## Outputs

- Per-site predictions output file mapping each candidate position to predicted class vs ground truth label
- Summary metrics table reporting accuracy, specificity, and precision

## How to apply

Run the classifier (e.g. REDInet_Inference.py / REDInet_Inference_light_ver.py with MinCov 50, MinAG 3, MinAGfreq 0.01) on tabix-indexed REDItools outTables (-s 1 -b 2 -e 2 -c R) to obtain per-site SNP vs editing-site probabilities and softmax class labels. Align each predicted position with the ground truth label set (positives: A-to-G evidence in RNAseq, no DNAseq change, present in REDIportal; negatives: A-to-G evidence in both RNAseq and DNAseq). Compute accuracy, specificity, and precision using the formulas from the paper's Supplementary Methods, writing per-site predictions to a predictions output file and the benchmark metrics to a summary metrics table. Accept the run only if metrics reach the published benchmarks (accuracy >95%, specificity >93%, precision >96% for KO-OE; accuracy >96% mean for A549).

## Related tools

- **REDInet** (Pretrained REDInet.h5 temporal convolutional network producing SNP vs editing-site probabilities and softmax class labels per candidate site for benchmarking) — https://github.com/BioinfoUNIBA/REDInet.git
- **REDItools v1** (Generates low-stringency pileup profiles of variant and invariant A-to-G candidate sites (REDItoolDnaRna.py) that constitute the prediction input) — https://github.com/BioinfoUNIBA/REDItools

## Examples

```
python3 REDInet_Inference_light_ver.py -r outTable_reditoolsID.gz -o predictions_prefix -c 50 -s 3 -f 0.01
```

## Evaluation signals

- KO-OE benchmark: accuracy >95%, specificity >93%, precision >96% — a run below any of these thresholds fails validation.
- Public HEK293T WT/ADAR KO data cross-check: accuracy >95%, precision >93%, specificity >92% expected on ~69.5 million reads per sample.
- A549 benchmark: mean accuracy >96% on the >13,000-site ground truth set.
- Biology sanity check: predicted positives among Alu adenosines should rise from ~5% (KO) to ~95% (OE) in HEK293T.
- Per-site predictions file is non-empty and every scored position has both a predicted class label and a ground truth label.

## Limitations

- Metric formulas are detailed only in the Supplementary Methods, which are not included in the main text.
- Ground truth datasets are said to be in 'our GitHub repository' without a specific URL or accession, so label provenance must be verified when reproducing.
- Published thresholds are reported as inequality bounds (>95%, >93%, >96%), so exact metric values cannot be reproduced precisely from the main text.

## Evidence
