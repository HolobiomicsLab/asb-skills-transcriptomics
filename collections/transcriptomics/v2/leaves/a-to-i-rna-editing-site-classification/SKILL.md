---
name: a-to-i-rna-editing-site-classification
description: Use when you have tabix-indexed REDItools outTables from RNAseq BAM files
  and need per-site A-to-I vs SNP classification of A-to-G substitutions, benchmarking
  against ground truth sets, or evaluating robustness to missing flanking RNAseq data.
when_to_use_negative:
- The candidate sites are already known RNA editing events (e.g. present in REDIportal)
  and no classification decision is needed
- The substitution is not A-to-G — REDInet is specific to A-to-I editing detection
  from A-to-G substitutions in RNAseq data
- The input is raw BAM data not yet processed into tabix-indexed REDItools outTables;
  the REDItools pileup step must run first
edam_operation: http://edamontology.org/operation_0496
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: REDInet
  role: Pretrained TCN classifier (REDInet.h5, REDIportal model) scoring 8×101 RNAseq
    feature windows as SNP vs editing site
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: REDItools
  role: Generates low-stringency RNAseq pileup outTables (variant and invariant sites)
    that feed REDInet inference
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Faster re-implementation producing complete pileup profiles (<60 min vs ~9
    h for v1)
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: pysam
  role: Python BAM/Tabix interface used to extract 101-nt windows centered on candidate
    sites
  repo: https://github.com/pysam-developers/pysam
- name: scikit-learn
  role: Computes accuracy, specificity, precision, FNR, and ROC-AUC against ground
    truth labels
- name: TensorFlow
  role: Loads and runs the pretrained REDInet.h5 model for inference
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:b5e67f909f6ce5494c5ef0cdef262095d62062dc50d3c918f94e48167e9dbb28
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
  - REDItools3
  - TensorFlow
  - numpy
  - pandas
  - pysam
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0496
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# a-to-i-rna-editing-site-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies A-to-G substitutions in RNAseq data as bona fide A-to-I RNA editing events or non-editing variants (SNPs/artifacts) using the REDInet temporal convolutional network on 8×101 RNAseq nucleotide-frequency windows, without requiring coupled WGS/WES data.

## When to use

Use when you have tabix-indexed REDItools outTables from RNAseq BAM files and need per-site A-to-I vs SNP classification of A-to-G substitutions, benchmarking against ground truth sets, or evaluating robustness to missing flanking RNAseq data.

## When NOT to use

- The candidate sites are already known RNA editing events (e.g. present in REDIportal) and no classification decision is needed
- The substitution is not A-to-G — REDInet is specific to A-to-I editing detection from A-to-G substitutions in RNAseq data
- The input is raw BAM data not yet processed into tabix-indexed REDItools outTables; the REDItools pileup step must run first

## Inputs

- Stranded RNAseq BAM files (e.g. HEK293T KO/WT/OE or A549 samples)
- Tabix-indexed, bgzip-compressed REDItools outTable files (outTable_*.gz indexed with -s 1 -b 2 -e 2)
- GRCh37 or GRCh38 reference genome FASTA (faidx-indexed, Gencode release 46 primaries suggested)
- Ground truth site sets with positive/negative labels per genomic position (e.g. HEK293T KO-WT >20,000 events; KO-OE >35,000 sites)

## Outputs

- Per-site classification labels and softmax probabilities (class 0 = SNP, class 1 = editing site)
- REDInet predictions output file listing classified genomic positions
- Summary benchmark metrics table (accuracy, specificity, precision, FNR, AUC) e.g. ko_wt_metrics.csv
- <.gz name>_discarded.txt listing positions filtered out at the coverage/A-to-G thresholding step

## How to apply

Run REDItools (REDItoolDnaRna.py, v1 or v3) at low stringency (e.g. -t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -n 0.0 -N 0.0 -S) against the GRCh37 or GRCh38 reference to obtain complete pileup intervals, then bgzip-compress and tabix-index each outTable with 'tabix -s 1 -b 2 -e 2'. Launch REDInet_Inference.py (or the light version for HTC clusters) with the matching assembly and default filtering thresholds — minimum coverage 50, minimum 3 A-to-G substitutions, minimum A-to-G substitution frequency 0.01 — to extract 101-nt windows centered on each candidate site and encode them as 8×101 feature matrices. The pretrained REDInet.h5 TCN (trained on ~13 million unique REDIportal/GTEx-derived examples) scores each site via softmax: class 0 = SNP, class 1 = editing site. For validation, join per-site predictions with ground truth labels (e.g. HEK293T KO-WT or KO-OE sets) by genomic position and compute accuracy, specificity, precision, FNR, and AUC with scikit-learn; benchmark acceptance thresholds are accuracy >95%, specificity >93%, precision >96%. For missing-data robustness testing, randomly replace RNAseq-derived nucleotide frequencies with the corresponding genomic reference frequencies at >30% of flanking positions (NumPy random sampling) before inference.

## Related tools

- **REDInet** (Pretrained TCN classifier (REDInet.h5, REDIportal model) scoring 8×101 RNAseq feature windows as SNP vs editing site) — https://github.com/BioinfoUNIBA/REDInet
- **REDItools** (Generates low-stringency RNAseq pileup outTables (variant and invariant sites) that feed REDInet inference) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Faster re-implementation producing complete pileup profiles (<60 min vs ~9 h for v1)) — https://github.com/BioinfoUNIBA/REDItools3
- **pysam** (Python BAM/Tabix interface used to extract 101-nt windows centered on candidate sites) — https://github.com/pysam-developers/pysam
- **scikit-learn** (Computes accuracy, specificity, precision, FNR, and ROC-AUC against ground truth labels)
- **TensorFlow** (Loads and runs the pretrained REDInet.h5 model for inference)

## Examples

```
python3 REDInet_Inference_light_ver.py -r outTable_sample.gz -o results/reds -c 50 -s 3 -f 0.01
```

## Evaluation signals

- On the HEK293T KO-WT ground truth set (>20,000 events), accuracy >95%, specificity >93%, and precision >96% must be reproduced before writing ko_wt_metrics.csv
- On the HEK293T KO-OE ground truth set (>35,000 sites), the same benchmarks (accuracy >95%, specificity >93%, precision >96%) are the acceptance criteria
- On the A549 dataset of >13,000 sites, mean accuracy should exceed 96%
- For the missing-data robustness experiment, false-positive and false-negative rates must both remain below 10% after replacing >30% of flanking RNAseq frequencies with genomic reference frequencies
- The discarded.txt file should account for all positions removed by the MinCov=50 / MinAG=3 / MinAGfreq=0.01 filtering step; it is empty if nothing was filtered

## Limitations

- Candidate positions with insufficient coverage (<50 reads), fewer than 3 A-to-G substitutions, or A-to-G frequency <0.01 are discarded before inference and never scored
- Positions missing nucleotides in the 101-nt window are further filtered; imputation of missing values via the reference genome is a BETA feature in the multiprocessing script
- REDInet accuracy was benchmarked on specific ground truth sets (A549, HEK293T KO-WT/KO-OE); performance depends on RNAseq read quality and coverage of the full 101-base window
- Complete architecture description, feature rescaling, and metric formulas are deferred to the Supplementary Material, which is not included in the main text
- REDItools v1 is ~10x slower than REDItools v3 (~9 h vs <60 min for complete pileup profiles)

## Evidence

- [abstract] REDInet can classify editing events with high accuracy harnessing RNAseq nucleotide frequencies of 101-base windows without the need for coupled genomic data.: "REDInet can classify editing events with high accuracy harnessing RNAseq nucleotide frequencies of 101-base windows without the need for coupled genomic data."
- [results] Each interval, encoded by a matrix of 8 × 101 features (Fig. 1c), was used to feed a TCN model: "Each interval, encoded by a matrix of 8 × 101 features (Fig. 1c), was used to feed a TCN model"
- [results] REDInet achieved an accuracy of >95% on both KO-WT (Fig. 2c) and KO-OE datasets, with a specificity of >93% and precision of >96%: "REDInet achieved an accuracy of >95% on both KO-WT (Fig. 2c) and KO-OE datasets, with a specificity of >93% and precision of >96%"
- [readme] REDInet runs on Tabix indexed REDItools output tables.: "REDInet runs on Tabix indexed REDItools output tables."
- [readme] tabix -s 1 -b 2 -e 2 -c R outTable_<REDItools SRR12492045.SRR12492046 numeric ID>.gz: "tabix -s 1 -b 2 -e 2 -c R outTable_<REDItools SRR12492045.SRR12492046 numeric ID>.gz"
- [results] simulating missing data in flanking regions by randomly replacing the RNAseq data with the corresponding genomic data: "simulating missing data in flanking regions by randomly replacing the RNAseq data with the corresponding genomic data"
