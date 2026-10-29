---
name: pearson-correlation-cutoff-network-filtering
description: Use when you have a PCC-scored functional association network (e.g. from
  FAVA on scRNA-seq or proteomics data) and need to extract a high-confidence subnetwork
  at STRING-style probability cutoffs.
when_to_use_negative:
- The scores are uncalibrated raw PCC values — cutoffs are only meaningful after converting
  scores to posterior probabilities via the STRING-style calibration function
- No benchmark of known associations (e.g. KEGG) is available to estimate local precision,
  so the calibration fit a0–a4 cannot be established
- The goal is de novo network inference rather than filtering an already-scored network
  — use the FAVA VAE inference step instead
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3523
tools:
- name: favapy
  role: Provides the pairs_after_cutoff functionality (equivalent to the -c cut-off
    option) used to extract subnetworks at the 15%/40%/70% confidence cutoffs; installed
    via pip install favapy
  repo: https://github.com/mikelkou/fava
- name: FAVA
  role: Produces the PCC-scored association networks (from Human Protein Atlas scRNA-seq
    and PRIDE proteomics data) that serve as input for cutoff filtering
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Execution environment for the favapy module and its pairs_after_cutoff function
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:4a7dcc70caf1bc7af5d38215fff99bc66e07fc32766d540babd06db5def24b24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pearson-correlation-cutoff-network-filtering

## Summary

Filter PCC-scored protein association networks — after STRING-style calibration of scores into posterior probabilities — at defined confidence cutoffs (15%, 40%, 70%) using favapy's pairs_after_cutoff to produce usable subnetworks.

## When to use

Use when you have a PCC-scored functional association network (e.g. from FAVA on scRNA-seq or proteomics data) and need to extract a high-confidence subnetwork at STRING-style probability cutoffs.

## When NOT to use

- The scores are uncalibrated raw PCC values — cutoffs are only meaningful after converting scores to posterior probabilities via the STRING-style calibration function
- No benchmark of known associations (e.g. KEGG) is available to estimate local precision, so the calibration fit a0–a4 cannot be established
- The goal is de novo network inference rather than filtering an already-scored network — use the FAVA VAE inference step instead

## Inputs

- FAVA-scored PCC networks for Human Protein Atlas (scRNA-seq) data in tsv format (the -t default)
- FAVA-scored PCC networks for PRIDE (proteomics) data in tsv format
- KEGG pathway benchmark of known protein associations for calibration
- Fitted calibration parameters a0–a4 per data type
- Posterior-probability cutoff values (15%, 40%, 70%)

## Outputs

- Subnetworks at 15%, 40%, and 70% posterior-probability cutoffs (e.g. 511048 associations / 16790 proteins at 15%; 52953/8191 at 40%; 24182/4166 at 70%)
- Combined calibrated network saved as favapy_combined_network.tsv

## How to apply

First calibrate raw PCC scores into posterior probabilities by fitting the STRING-style function y = a0 + a1*x + a2/(1+exp(a3*(x-a4))) to local precision versus PCC score, benchmarked against KEGG pathway associations per data type (e.g. Human Protein Atlas scRNA-seq and PRIDE proteomics separately). Apply each per-data-type calibration function to convert all PCC scores, then merge networks (e.g. HPA and PRIDE) by union. Extract subnetworks with favapy's pairs_after_cutoff (equivalent to the -c CLI option) at 15%, 40%, and 70% posterior-probability cutoffs, retaining interactions in both directions (proteinA-proteinB and proteinB-proteinA). The 15% cutoff corresponds to STRING's low-confidence level; the 70% cutoff yields the highest-confidence associations. Save the combined network as favapy_combined_network.tsv.

## Related tools

- **favapy** (Provides the pairs_after_cutoff functionality (equivalent to the -c cut-off option) used to extract subnetworks at the 15%/40%/70% confidence cutoffs; installed via pip install favapy) — https://github.com/mikelkou/fava
- **FAVA** (Produces the PCC-scored association networks (from Human Protein Atlas scRNA-seq and PRIDE proteomics data) that serve as input for cutoff filtering) — https://github.com/mikelkou/fava
- **Python** (Execution environment for the favapy module and its pairs_after_cutoff function)

## Examples

```
favapy <path-to-data-file> <path-to-save-output> -c 0.15
```

## Evaluation signals

- The combined network at 15% contains 511048 associations between 16790 proteins
- The 40% subnetwork contains 52953 associations between 8191 proteins, and the 70% subnetwork 24182 associations between 4166 proteins
- Interactions are retained in both directions (proteinA-proteinB and proteinB-proteinA) in the extracted subnetworks
- The combined network agrees better with the KEGG benchmark than either individual (HPA-only or PRIDE-only) calibrated network
- Output file favapy_combined_network.tsv is produced in tsv format

## Limitations

- Cutoff statistics depend on prior calibration against the KEGG pathway benchmark; skipping or mis-fitting the calibration function (parameters a0–a4) invalidates the posterior-probability interpretation of the cutoffs
- Networks inferred from omics data via simple co-expression analysis on the original (non-latent) representation perform poorly due to high sparseness and redundancy, so PCC scores must come from the FAVA latent space
- The combined network size depends on union-merging of HPA and PRIDE networks; changing the input data types changes the cutoff statistics

## Evidence

- [methods] pairs_after_cutoff: ":members:
       VAE
       cook
       pairs_after_cutoff"
- [full_text] workflow cutoffs: [evidence span withheld — anchor status: not_found]
- [full_text] combined network size: [evidence span withheld — anchor status: not_found]
- [methods] favapy installation: "You can install FAVA using pip:

.. code-block:: bash

   pip install favapy"
- [intro] raw co-expression limitation: "this approach does not work well due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy (many similar cells/samples are analyzed) of such"
