---
name: string-style-score-calibration-curve-fitting
description: Use when raw association scores (e.g. FAVA PCC scores from scRNA-seq
  or bulk proteomics data) need to be calibrated into interpretable confidence probabilities
  for network filtering at STRING-style confidence cutoffs.
when_to_use_negative:
- When the network scores are already calibrated posterior probabilities (e.g. STRING
  scores) rather than raw PCC values — refitting would double-calibrate
- When no benchmark of known protein associations (e.g. KEGG) exists for the organism,
  since local precision cannot be estimated
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0605
tools:
- name: favapy
  role: Python package used to install FAVA functionality, load scored PCC networks
    (tsv, -t default), and extract pairs_after_cutoff subnetworks at chosen confidence
    cutoffs
  repo: https://github.com/mikelkou/fava
- name: FAVA
  role: Tool that produced the PCC-scored networks (from Human Protein Atlas scRNA-seq
    and PRIDE proteomics data) that serve as calibration input
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Language in which the fava.py module (VAE, cook, pairs_after_cutoff functions)
    is run for calibration and cutoff extraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:00c8e9515e5bd024736537ed2e69dbd6f15a71f649beabbf2a77d886e080b141
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# STRING-style score calibration curve fitting

## Summary

A method for converting raw PCC-derived association scores from FAVA-scored networks into STRING-style posterior confidence probabilities by fitting a five-parameter calibration function against a benchmark of known pathway associations (KEGG), then combining per-data-type calibrated networks and extracting subnetworks at 15%, 40%, and 70% confidence cutoffs.

## When to use

Use when raw association scores (e.g. FAVA PCC scores from scRNA-seq or bulk proteomics data) need to be calibrated into interpretable confidence probabilities for network filtering at STRING-style confidence cutoffs.

## When NOT to use

- When the network scores are already calibrated posterior probabilities (e.g. STRING scores) rather than raw PCC values — refitting would double-calibrate
- When no benchmark of known protein associations (e.g. KEGG) exists for the organism, since local precision cannot be estimated

## Inputs

- FAVA-scored PCC network files in tsv format (the favapy -t default), one per data type (Human Protein Atlas scRNA-seq and PRIDE proteomics)
- KEGG pathway benchmark of known protein associations
- Python environment with the favapy package installed (pip install favapy)

## Outputs

- Per-data-type calibration parameters a0-a4 for the five-parameter STRING-style function
- Calibrated posterior-probability networks for each data type
- favapy_combined_network.tsv: the union of calibrated HPA and PRIDE networks
- Subnetwork sizes at 15%/40%/70% cutoffs (expected: 511048 associations / 16790 proteins at 15%, 52953/8191 at 40%, 24182/4166 at 70%)

## How to apply

First estimate the local precision of PCC scores for each data type separately (e.g. Human Protein Atlas scRNA-seq and PRIDE proteomics networks) by comparison against the KEGG pathway benchmark of known protein associations. Fit the STRING-style calibration function y = a0 + a1*x + a2/(1+exp(a3*(x-a4))) to local precision versus PCC score per data type, obtaining parameters a0-a4; this logistic-plus-linear form captures how precision varies across the score range. Apply each fitted calibration function to convert all PCC scores to posterior probabilities, then merge the per-data-type calibrated networks (HPA and PRIDE) into a single combined network by union. Extract subnetworks at the 15% (low-confidence), 40% (medium), and 70% (high) posterior-probability cutoffs using the favapy pairs_after_cutoff functionality (equivalent to the -c cutoff option), retaining interactions in both directions (proteinA-proteinB and proteinB-proteinA). The rationale is that calibrated probabilities make networks from heterogeneous omics sources comparable and combinable, mirroring the confidence-score semantics of STRING.

## Related tools

- **favapy** (Python package used to install FAVA functionality, load scored PCC networks (tsv, -t default), and extract pairs_after_cutoff subnetworks at chosen confidence cutoffs) — https://github.com/mikelkou/fava
- **FAVA** (Tool that produced the PCC-scored networks (from Human Protein Atlas scRNA-seq and PRIDE proteomics data) that serve as calibration input) — https://github.com/mikelkou/fava
- **Python** (Language in which the fava.py module (VAE, cook, pairs_after_cutoff functions) is run for calibration and cutoff extraction)

## Examples

```
pip install favapy; favapy <path-to-data-file> <path-to-save-output> -c 0.15
```

## Evaluation signals

- The combined calibrated network reproduces the reported sizes: 511048 associations between 16790 proteins at 15%, 52953/8191 at 40%, and 24182/4166 at 70%
- The combined calibrated network agrees better with the KEGG benchmark than either individual (HPA-only or PRIDE-only) calibrated network
- Calibration fitting yields a full set of parameters a0-a4 for each data type separately (HPA and PRIDE)
- Cutoff extraction retains interactions in both directions (proteinA-proteinB and proteinB-proteinA)
- The output network is saved as favapy_combined_network.tsv in tsv format

## Limitations

- Calibration quality depends on the KEGG benchmark, so it is biased toward well-annotated pathways; existing networks are known to be biased by data availability, with well-studied proteins having many more interactions than understudied proteins
- Simple co-expression/correlation analysis of the raw data does not work well due to sparseness and redundancy, so calibration must be applied on FAVA latent-space scores rather than raw PCC
- Reported cutoff statistics correspond to the specific HPA (scRNA-seq) and PRIDE (proteomics) inputs; other data types require their own precision estimation and refitting

## Evidence

- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] You can install FAVA using pip: "You can install FAVA using pip"
- [intro] large-scale scRNA-seq and proteomics data from the Human Protein Atlas, and from PRIDE, respectively: "large-scale scRNA-seq and proteomics data from the Human Protein Atlas, and from PRIDE, respectively"
- [abstract] Source code, documentation, and tutorials for FAVA are accessible on GitHub: "Source code, documentation, and tutorials for FAVA are accessible on GitHub"
