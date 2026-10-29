---
name: multi-omics-network-union-integration
description: Use when two or more FAVA-scored PCC networks (e.g., scRNA-seq and bulk
  proteomics tsv inputs) need to be converted to calibrated posterior probabilities
  and merged into one combined functional association network with STRING-style confidence
  cutoffs.
when_to_use_negative:
- The input is raw scRNA-seq or proteomics data rather than already FAVA-scored PCC
  networks — run the FAVA VAE dimensionality-reduction and correlation step first.
- Only one data type's network is available — the union integration requires at least
  two calibrated networks (e.g., HPA and PRIDE) to merge.
- No benchmark of known associations (e.g., KEGG) is available — without it, the local
  precision estimates needed to fit the calibration function cannot be computed.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: favapy
  role: Python package used to install FAVA, apply the calibration functions, and
    extract cutoff subnetworks via pairs_after_cutoff
  repo: https://github.com/mikelkou/fava
- name: FAVA
  role: Produces the scored PCC networks from scRNA-seq (Human Protein Atlas) and
    proteomics (PRIDE) inputs that are calibrated and merged
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Language in which the fava.py module runs the calibration and cutoff-extraction
    workflow
  repo: https://fava.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:0a2eba19a10d168fd982929db46e3b9a22c7f6ddae81652e6ae9e67cd8743042
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# multi-omics-network-union-integration

## Summary

Calibrates FAVA-scored PCC networks from two omics data types (Human Protein Atlas scRNA-seq and PRIDE proteomics) into STRING-style posterior probabilities on the KEGG benchmark, then merges them into a single combined functional association network by union and extracts subnetworks at 15%, 40%, and 70% confidence cutoffs.

## When to use

Use when two or more FAVA-scored PCC networks (e.g., scRNA-seq and bulk proteomics tsv inputs) need to be converted to calibrated posterior probabilities and merged into one combined functional association network with STRING-style confidence cutoffs.

## When NOT to use

- The input is raw scRNA-seq or proteomics data rather than already FAVA-scored PCC networks — run the FAVA VAE dimensionality-reduction and correlation step first.
- Only one data type's network is available — the union integration requires at least two calibrated networks (e.g., HPA and PRIDE) to merge.
- No benchmark of known associations (e.g., KEGG) is available — without it, the local precision estimates needed to fit the calibration function cannot be computed.

## Inputs

- FAVA-scored PCC network for Human Protein Atlas scRNA-seq data (tsv, the favapy -t default format)
- FAVA-scored PCC network for PRIDE proteomics data (tsv, the favapy -t default format)
- KEGG pathway benchmark of known protein associations for local precision estimation

## Outputs

- Per-data-type calibration parameters a0–a4 from the fitted STRING-style function
- Calibrated networks of posterior probabilities for each data type
- Combined union network saved as favapy_combined_network.tsv
- Subnetworks extracted at 15%, 40%, and 70% posterior-probability cutoffs

## How to apply

Load the FAVA-scored PCC networks for Human Protein Atlas (scRNA-seq) and PRIDE (proteomics) as tsv files (the favapy -t default). For each data type separately, estimate local precision of the PCC scores against the KEGG pathway benchmark of known protein associations, and fit the STRING-style calibration function y = a0 + a1*x + a2/(1+exp(a3*(x-a4))) to obtain parameters a0–a4. Apply the fitted per-data-type calibration functions to convert all PCC scores to posterior probabilities, then merge the HPA and PRIDE networks into a single combined network by union. Extract subnetworks at the 15%, 40%, and 70% posterior-probability cutoffs using the pairs_after_cutoff functionality (equivalent to the -c cutoff option), retaining interactions in both directions (proteinA–proteinB and proteinB–proteinA). The rationale is that the combined calibrated network integrates complementary omics evidence and agrees better with KEGG than either individual calibrated network.

## Related tools

- **favapy** (Python package used to install FAVA, apply the calibration functions, and extract cutoff subnetworks via pairs_after_cutoff) — https://github.com/mikelkou/fava
- **FAVA** (Produces the scored PCC networks from scRNA-seq (Human Protein Atlas) and proteomics (PRIDE) inputs that are calibrated and merged) — https://github.com/mikelkou/fava
- **Python** (Language in which the fava.py module runs the calibration and cutoff-extraction workflow) — https://fava.readthedocs.io/en/latest/

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Combined network at the 15% cutoff (STRING's low-confidence cutoff) contains 511,048 associations between 16,790 proteins.
- Combined network at the 40% cutoff contains 52,953 associations between 8,191 proteins, and at the 70% cutoff 24,182 associations between 4,166 proteins.
- The combined calibrated network agrees better with the KEGG benchmark than either the HPA-only or PRIDE-only calibrated network.
- Both interaction directions (proteinA–proteinB and proteinB–proteinA) are retained in the cutoff subnetworks.
- Fitted calibration parameters a0–a4 are obtained for each data type separately against the KEGG benchmark.

## Limitations

- Calibration quality depends on the KEGG pathway benchmark, so the resulting posterior probabilities reflect the coverage and biases of KEGG known associations.
- Retention of both interaction directions doubles pairwise records and must be handled consistently when comparing network sizes against reported cutoff statistics.
- The STRING-style calibration function y = a0 + a1*x + a2/(1+exp(a3*(x-a4))) is a fixed functional form; it may not fit local precision curves for data types that deviate from the HPA/PRIDE profiles used here.

## Evidence

- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] favapy command-line invocation: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [methods] pairs_after_cutoff member: ":members:
       VAE
       cook
       pairs_after_cutoff"
- [abstract] FAVA described as a VAE tool: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
