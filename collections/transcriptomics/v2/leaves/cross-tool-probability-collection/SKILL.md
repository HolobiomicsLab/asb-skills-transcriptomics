---
name: cross-tool-probability-collection
description: Use when benchmarking two or more RNA editing detection tools on a shared
  bona fide site set and you need per-site predictions aligned by genomic position
  to compute head-to-head ROC curves, AUC, or FNR.
when_to_use_negative:
- Input is a single tool's raw output with no ground truth labels — probability collection
  only supports comparison once positive/negative labels exist at matched genomic
  positions
- Tools were run on different site sets or with incompatible site definitions — joining
  by position would compare unlike inputs; the article explicitly required 'the same
  set of bona fide sites ... according the guidelines of each specific program'
- A tool does not output per-site probabilities or decisions (only aggregate statistics),
  making per-site ROC-AUC/FNR computation impossible
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: REDInet
  role: Temporal convolutional network classifier supplying baseline per-site editing
    predictions on the shared bona fide site set
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: EditPredict
  role: CNN-based comparator run on the identical site set following its published
    guidelines to yield per-site predictions
- name: RED-ML
  role: Logistic regression comparator run on the identical site set to yield per-site
    predictions
  repo: https://github.com/BGIRED/RED-ML
- name: RDDpred
  role: Comparator specialized in distinguishing sequencing artifacts from genuine
    AG substitutions; per-site decisions used to estimate false negative rate
  repo: https://github.com/vibbits/RDDpred
- name: REDItools
  role: Generates pileup outTables from RNAseq BAMs that are compressed and tabix-indexed
    before feeding the classifiers
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: Python
  role: Used to join per-tool predictions with ground truth labels by genomic position
    and compute ROC curves, ROC-AUC, and FNR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:3f9390315e03f73b94236040e60818c999b5d35730946ccd3f259eb20a0d287f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-tool-probability-collection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Collect per-tool predicted editing probabilities from multiple RNA editing classifiers (REDInet, EditPredict, RED-ML, RDDpred) run on the identical site set, join them with ground-truth positive/negative labels by genomic position, and compute comparative metrics such as ROC-AUC and false negative rate.

## When to use

Use when benchmarking two or more RNA editing detection tools on a shared bona fide site set and you need per-site predictions aligned by genomic position to compute head-to-head ROC curves, AUC, or FNR.

## When NOT to use

- Input is a single tool's raw output with no ground truth labels — probability collection only supports comparison once positive/negative labels exist at matched genomic positions
- Tools were run on different site sets or with incompatible site definitions — joining by position would compare unlike inputs; the article explicitly required 'the same set of bona fide sites ... according the guidelines of each specific program'
- A tool does not output per-site probabilities or decisions (only aggregate statistics), making per-site ROC-AUC/FNR computation impossible

## Inputs

- Ground truth HEK293T KO-WT site set (RNA editing positives and SNP negatives) with genomic positions and labels
- REDItools pileup outTables from sample BAM files, bgzip-compressed and tabix-indexed (tabix -s 1 -b 2 -e 2 -c R)
- Per-tool predicted editing probabilities or per-site decisions from EditPredict, RED-ML, RDDpred, and REDInet

## Outputs

- Position-joined table of per-tool predicted editing probabilities and ground truth labels
- Per-tool ROC curves and ROC-AUC values
- Per-tool false negative rate on true editing positives (e.g. RDDpred FNR)

## How to apply

Run each classifier (EditPredict, a convolutional neural network on sequence contexts; RED-ML, a logistic regression classifier; RDDpred, which separates sequencing artifacts from genuine A-to-G substitutions) on the same bona fide HEK293T KO-WT site set, following each program's own published guidelines rather than a single shared configuration. Record per-site predicted editing probabilities or per-site decisions from every tool. Join each tool's predictions with the ground truth positive/negative labels by genomic position. Compute ROC curves and ROC-AUC per tool, and a false negative rate for tools returning hard decisions (RDDpred FNR on true editing positives was ~7% versus 2% for REDInet). Validate by confirming the reported ordering: REDInet highest AUC (0.98), then EditPredict (0.78), then RED-ML (0.72).

## Related tools

- **REDInet** (Temporal convolutional network classifier supplying baseline per-site editing predictions on the shared bona fide site set) — https://github.com/BioinfoUNIBA/REDInet
- **EditPredict** (CNN-based comparator run on the identical site set following its published guidelines to yield per-site predictions)
- **RED-ML** (Logistic regression comparator run on the identical site set to yield per-site predictions) — https://github.com/BGIRED/RED-ML
- **RDDpred** (Comparator specialized in distinguishing sequencing artifacts from genuine AG substitutions; per-site decisions used to estimate false negative rate) — https://github.com/vibbits/RDDpred
- **REDItools** (Generates pileup outTables from RNAseq BAMs that are compressed and tabix-indexed before feeding the classifiers) — https://github.com/BioinfoUNIBA/REDItools
- **Python** (Used to join per-tool predictions with ground truth labels by genomic position and compute ROC curves, ROC-AUC, and FNR)

## Examples

```
bgzip outTable_<ID> && tabix -s 1 -b 2 -e 2 -c R outTable_<ID>.gz && python3 REDInet_Inference.py --I <outTable_folder> --O <output_folder> --A GRCh37 --N <outTable_name>
```

## Evaluation signals

- Reproduced AUC ordering holds: REDInet (0.98) > EditPredict (0.78) > RED-ML (0.72)
- RDDpred false negative rate on true editing positives is approximately 7%, versus 2% for REDInet
- Every ground truth site has a prediction from each tool after the genomic-position join (no unmatched positions)
- Predicted positives in Alu adenosines increase from ~5% (ADAR1 KO) to ~95% (OE) in HEK293T, confirming biologically coherent directionality of predictions
- All comparisons were carried out on the same set of bona fide HEK293T KO and WT sites, verified against each program's guidelines

## Limitations

- Ground truth datasets are said to be in 'our GitHub repository' without a specific URL or accession, complicating exact reproduction
- Performance metric formulas are deferred to Supplementary Methods not included with the main text
- Each tool must be run under its own guidelines, so input preparation is not uniform across comparators; RDDpred is specialized for A-to-G substitution artifact separation and may behave differently on non-A-to-G events

## Evidence
