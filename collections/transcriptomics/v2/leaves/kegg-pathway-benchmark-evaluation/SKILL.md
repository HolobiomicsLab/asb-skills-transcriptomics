---
name: kegg-pathway-benchmark-evaluation
description: Use when a ranked protein–protein association list (e.g. FAVA latent-space
  Pearson correlations or favapy output) must be validated or calibrated against a
  known pathway benchmark such as KEGG co-membership.
when_to_use_negative:
- Input is raw scRNA-seq or proteomics data that has not yet been compressed and scored
  by the FAVA VAE — run FAVA first to obtain latent-space correlations
- The input is already a calibrated posterior-probability network and only cutoff
  extraction is needed — pairs_after_cutoff alone suffices without refitting calibration
edam_operation: http://edamontology.org/operation_2422
tools:
- name: favapy
  role: Runs FAVA and provides pairs_after_cutoff to extract ranked subnetworks for
    KEGG benchmark evaluation
  repo: https://github.com/mikelkou/fava
- name: FAVA
  role: Infers the PCC association networks from scRNA-seq/proteomics data that are
    benchmarked against KEGG
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input format for loading scRNA-seq data (e.g. GSE75748) for train/test split
    evaluation
- name: Python
  role: Scripting environment for the fava.py module and benchmark evaluation
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:baf7890c8d2f82d7fbc0d45cae90beaa23c41f92d14e2e7061b7e45b21bd5f15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# kegg-pathway-benchmark-evaluation

## Summary

Evaluates FAVA protein association networks by benchmarking ranked PCC associations against the KEGG pathway co-membership gold standard, fitting a STRING-style precision calibration, and extracting subnetworks at the 15%/40%/70% confidence cutoffs.

## When to use

Use when a ranked protein–protein association list (e.g. FAVA latent-space Pearson correlations or favapy output) must be validated or calibrated against a known pathway benchmark such as KEGG co-membership.

## When NOT to use

- Input is raw scRNA-seq or proteomics data that has not yet been compressed and scored by the FAVA VAE — run FAVA first to obtain latent-space correlations
- The input is already a calibrated posterior-probability network and only cutoff extraction is needed — pairs_after_cutoff alone suffices without refitting calibration

## Inputs

- Ranked protein association list with PCC scores (tsv, favapy default '-t' input type)
- KEGG pathway co-membership benchmark of known protein associations
- Per-data-type PCC networks (e.g. Human Protein Atlas scRNA-seq and PRIDE proteomics)
- Train/test split of cells (AnnData, e.g. 67%/33% of GSE75748) for overfitting checks

## Outputs

- Local precision estimates of PCC scores against KEGG
- Calibration parameters a0–a4 and posterior probabilities per data type
- Subnetworks at 15%/40%/70% cutoffs (e.g. favapy_combined_network.tsv)
- Training vs test split benchmark performance comparison

## How to apply

Rank protein pairs with favapy's pairs_after_cutoff (default 100000 interactions, retaining both directions proteinA–proteinB and proteinB–proteinA). Estimate local precision of the PCC scores per data type (e.g. Human Protein Atlas scRNA-seq vs PRIDE proteomics) by comparison against the KEGG pathway benchmark of known protein associations. Fit the STRING-style calibration function y = a0 + a1*x + a2/(1+exp(a3*(x-a4))) to local precision versus PCC score per data type, then convert all PCC scores to posterior probabilities and merge data types by union. Extract subnetworks at 15%, 40%, and 70% posterior-probability cutoffs using pairs_after_cutoff (equivalent to the -c option). For generalization checks, train the FAVA VAE on a 67% training split (50 epochs, batch size 32) and compare KEGG benchmark performance against the 33% test split, confirming similarity as in Supplementary Fig. S5.

## Related tools

- **favapy** (Runs FAVA and provides pairs_after_cutoff to extract ranked subnetworks for KEGG benchmark evaluation) — https://github.com/mikelkou/fava
- **FAVA** (Infers the PCC association networks from scRNA-seq/proteomics data that are benchmarked against KEGG) — https://github.com/mikelkou/fava
- **AnnData** (Input format for loading scRNA-seq data (e.g. GSE75748) for train/test split evaluation)
- **Python** (Scripting environment for the fava.py module and benchmark evaluation)

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Combined calibrated network reproduces reported sizes: 511048 associations between 16790 proteins at 15%, 52953/8191 at 40%, and 24182/4166 at 70%
- Training and test split KEGG benchmark performance are similar, reproducing Supplementary Fig. S5 (no overfitting)
- Combined network agrees better with KEGG than either individual (HPA-only or PRIDE-only) calibrated network
- Ranked lists retain interactions in both directions (proteinA–proteinB and proteinB–proteinA)
- Cut-off extraction defaults to 100000 interactions via pairs_after_cutoff unless -c specifies otherwise

## Limitations

- The KEGG benchmark reflects known associations, so comparison inherits the literature bias that affects curated networks
- Benchmark conclusions depend on calibration quality per data type; poorly fit calibration curves (a0–a4) distort posterior probabilities
- Cutoff semantics follow STRING conventions (15% low-confidence, 40% medium, 70% high), so counts are only comparable within that scheme

## Evidence

- [abstract] FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods, across a diverse collection of real as well as simulated datasets: "FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods, across a diverse collection of real as well as simulated datasets"
- [results] 15% cutoff combined network size: [evidence span withheld — anchor status: not_found]
- [supplementary] Train/test generalization finding: "Similar performance was obtained for both training and test sets (Supplementary Fig. S5), suggesting that our models do not overfit the data"
- [methods] favapy CLI usage: "Run FAVA from the command line as follows"
- [methods] pairs_after_cutoff API member: "This section provides documentation for the `fava.py` module."
