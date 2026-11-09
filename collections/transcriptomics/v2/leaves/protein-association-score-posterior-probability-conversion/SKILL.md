---
name: protein-association-score-posterior-probability-conversion
description: Use when raw correlation-based protein association scores (e.g., FAVA
  PCC scores) must be calibrated into interpretable confidence probabilities using
  a gold-standard benchmark like KEGG, or when STRING-style confidence cutoffs (15%,
  40%, 70%) are needed for a combined network.
when_to_use_negative:
- The input data has not yet been scored by FAVA (i.e., no latent-space PCC network
  exists yet — run the FAVA VAE dimensionality-reduction step first)
- A benchmark of known protein associations (e.g., KEGG) is unavailable, since the
  calibration fit requires local precision estimates against a gold standard
- The goal is to compare against an existing literature-derived network rather than
  calibrate a raw score — FAVA networks are meant to be free of literature bias
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: favapy
  role: Python package providing the pairs_after_cutoff functionality (equivalent
    to the -c cutoff option) for extracting subnetworks at 15%/40%/70% posterior-probability
    cutoffs and handling tsv input via the -t default
  repo: https://github.com/mikelkou/fava
- name: FAVA
  role: VAE-based method that produced the PCC association scores being calibrated;
    compresses high-dimensional omics data into a low-dimensional latent space before
    correlation
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Runtime environment for the favapy fava.py module used to fit and apply the
    calibration function
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:3f4097df27be6255d877143eac051b8f19b806a33f450a5bc544601e1606a0ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - Python
  - favapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# protein-association-score-posterior-probability-conversion

## Summary

Converts raw PCC association scores from FAVA-scored scRNA-seq (Human Protein Atlas) and bulk proteomics (PRIDE) networks into STRING-style posterior confidence probabilities by fitting a calibration function against the KEGG pathway benchmark, then merges the per-data-type networks at 15%/40%/70% cutoffs.

## When to use

Use when raw correlation-based protein association scores (e.g., FAVA PCC scores) must be calibrated into interpretable confidence probabilities using a gold-standard benchmark like KEGG, or when STRING-style confidence cutoffs (15%, 40%, 70%) are needed for a combined network.

## When NOT to use

- The input data has not yet been scored by FAVA (i.e., no latent-space PCC network exists yet — run the FAVA VAE dimensionality-reduction step first)
- A benchmark of known protein associations (e.g., KEGG) is unavailable, since the calibration fit requires local precision estimates against a gold standard
- The goal is to compare against an existing literature-derived network rather than calibrate a raw score — FAVA networks are meant to be free of literature bias

## Inputs

- FAVA-scored PCC network for Human Protein Atlas (scRNA-seq) data as tsv (favapy -t default format)
- FAVA-scored PCC network for PRIDE (bulk proteomics) data as tsv (favapy -t default format)
- KEGG pathway benchmark of known protein associations

## Outputs

- Per-data-type calibrated networks with PCC scores converted to posterior probabilities (parameters a0–a4 per data type)
- Combined union network saved as favapy_combined_network.tsv
- Subnetworks extracted at 15%, 40%, and 70% posterior-probability cutoffs (511048 associations/16790 proteins at 15%; 52953/8191 at 40%; 24182/4166 at 70%)

## How to apply

Load the FAVA-scored PCC networks for each data type (HPA scRNA-seq and PRIDE proteomics) as tsv files (the favapy -t default format). For each data type separately, estimate local precision of the PCC scores by comparing against the KEGG pathway benchmark of known protein associations. Fit the STRING-style calibration function y = a0 + a1*x + a2/(1+exp(a3*(x-a4))) to local precision versus PCC score per data type, yielding parameters a0–a4. Apply the fitted per-data-type calibration functions to convert all PCC scores to posterior probabilities, then merge the HPA and PRIDE networks into a single combined network by union, retaining interactions in both directions (proteinA–proteinB and proteinB–proteinA). Extract subnetworks at the 15%, 40%, and 70% posterior-probability cutoffs using the favapy pairs_after_cutoff functionality (equivalent to the -c cutoff option).

## Related tools

- **favapy** (Python package providing the pairs_after_cutoff functionality (equivalent to the -c cutoff option) for extracting subnetworks at 15%/40%/70% posterior-probability cutoffs and handling tsv input via the -t default) — https://github.com/mikelkou/fava
- **FAVA** (VAE-based method that produced the PCC association scores being calibrated; compresses high-dimensional omics data into a low-dimensional latent space before correlation) — https://github.com/mikelkou/fava
- **Python** (Runtime environment for the favapy fava.py module used to fit and apply the calibration function)

## Examples

```
pip install favapy; favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- The combined network at 15% contains 511048 associations between 16790 proteins
- The combined network at 40% contains 52953 associations between 8191 proteins; at 70%, 24182 associations between 4166 proteins
- The combined calibrated network agrees better with the KEGG benchmark than either the HPA-only or PRIDE-only calibrated network
- Each per-data-type fit of local precision vs. PCC score yields all five calibration parameters a0–a4 without error
- The combined network is a union of HPA and PRIDE calibrated networks, retaining interactions in both directions (proteinA–proteinB and proteinB–proteinA)

## Limitations

- Calibration quality depends entirely on the KEGG pathway benchmark; associations outside KEGG's coverage receive no precision signal
- The reported statistics (511048/16790 at 15%, 52953/8191 at 40%, 24182/4166 at 70%) are specific to the HPA + PRIDE combination and will not transfer to other input data types
- Raw co-expression-style scoring of high-dimensional omics data is unreliable due to sparseness and redundancy, which is why calibration must be applied on FAVA latent-space correlations rather than original-representation correlations

## Evidence

- [abstract] FAVA compresses high-dimensional data into a low-dimensional latent space: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] Correlations in the latent space yield improved networks: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] favapy installation and command-line interface: [evidence span withheld — anchor status: not_found]
- [methods] favapy API members including pairs_after_cutoff: "VAE cook pairs_after_cutoff"
- [readme] FAVA repository availability: "Source code, documentation, and tutorials for FAVA are accessible on GitHub at https://github.com/mikelkou/fava"
