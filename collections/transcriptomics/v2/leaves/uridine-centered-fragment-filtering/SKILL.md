---
name: uridine-centered-fragment-filtering
description: Use when scanning transcriptome-wide RNA sequences for candidate pseudouridine
  (Ψ) sites and you need to restrict deep-learning scoring to uridine-centered 51-base
  fragments.
when_to_use_negative:
- When the goal is to re-predict already experimentally documented RMBase Ψ sites
  rather than discover novel ones (known sites plus ±25 flanking bases should be masked
  first)
- When scoring full-length sequences without center alignment — the model expects
  a uridine at the fragment center for the prediction to be interpretable
edam_operation: http://edamontology.org/operation_3695
tools:
- name: SeqKit
  role: Generates the running 51-bp windows across the transcriptome before uridine-center
    filtering
- name: BEDTools
  role: Masks known RMBase Ψ sites plus 25 flanking bases and extracts transcriptome
    coding sequences from hg19/GENCODE
- name: MU-PseUDeep
  role: Scores each uridine-centered 51-nt fragment (raw sequence + predicted secondary
    structure) and calls Ψ sites at >0.99 precision
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: RMBase v2.0
  role: Source of known Ψ site coordinates used for masking and validation cross-check
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:659d1e4bdaa960c5ba824dbf4194affc4606d91cd623c6e914cc84afa21b6450
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - BEDTools
  - MU-PseUDeep
  - RMBase v2.0
  - SeqKit
  - hg19
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Uridine-centered fragment filtering for pseudouridine site prediction

## Summary

Generates 51-nt running windows across RNA transcript sequences and retains only fragments with a uridine at their center so that MU-PseUDeep scores every candidate Ψ site, with a >0.99 precision threshold used to call predicted pseudouridines.

## When to use

Use when scanning transcriptome-wide RNA sequences for candidate pseudouridine (Ψ) sites and you need to restrict deep-learning scoring to uridine-centered 51-base fragments.

## When NOT to use

- When the goal is to re-predict already experimentally documented RMBase Ψ sites rather than discover novel ones (known sites plus ±25 flanking bases should be masked first)
- When scoring full-length sequences without center alignment — the model expects a uridine at the fragment center for the prediction to be interpretable

## Inputs

- Human transcriptome protein-coding sequences (hg19 + GENCODE gtf, DNA converted to RNA by strand)
- Known Ψ site coordinates from RMBase v2.0
- Trained MU-PseUDeep model (Keras 2.2.2 / TensorFlow 1.10.1)

## Outputs

- Set of uridine-centered 51-nt RNA fragments
- Predicted Ψ sites passing the >0.99 precision threshold
- Per-gene counts of predicted Ψ sites (e.g., 2441 predicted genes, of which 284 have documented RMBase sites)
- Genomic regions of predictions recorded in Supplemental_table2.csv

## How to apply

First mask known RMBase v2.0 pseudouridine sites together with 25 bases of flanking sequence on both sides using BEDTools, so the scan targets novel sites rather than re-predicting documented ones. Then generate running windows of 51 base pair fragments across the transcriptome (extracted from hg19 and the GENCODE gtf file, converted DNA-to-RNA per gene strand) using SeqKit. Retain only those fragments with a uridine at their center, since pseudouridines can only occur at uridine positions and the model consumes 51 × 4 one-of-K encoded matrices centered on the candidate site. Score each uridine-centered fragment with the bootstrapped, hyperparameter-optimized MU-PseUDeep model and call the center uridine as a potential Ψ site only when the prediction precision exceeds 0.99. Summarize predicted Ψ sites per gene and cross-check the predicted gene set against RMBase to estimate how many predictions correspond to already documented sites.

## Related tools

- **SeqKit** (Generates the running 51-bp windows across the transcriptome before uridine-center filtering)
- **BEDTools** (Masks known RMBase Ψ sites plus 25 flanking bases and extracts transcriptome coding sequences from hg19/GENCODE)
- **MU-PseUDeep** (Scores each uridine-centered 51-nt fragment (raw sequence + predicted secondary structure) and calls Ψ sites at >0.99 precision) — https://github.com/smk5g5/MU-PseUDeep
- **RMBase v2.0** (Source of known Ψ site coordinates used for masking and validation cross-check)

## Evaluation signals

- Every scored fragment must be 51 nt long with a uridine at position 26 (the center)
- No scored fragment should overlap a known RMBase Ψ site ±25 flanking bases (masking invariant)
- Only calls with model precision >0.99 should be retained; the article recovered 2441 human protein-coding genes with ≥1 predicted Ψ site under this threshold
- Of the predicted genes, 284 should have one or more already documented RMBase Ψ site, serving as a sanity cross-check

## Limitations

- The uridine-only restriction means non-U positions are never evaluated, which is correct for Ψ but assumes the transcript sequence and strand conversion were performed accurately
- The >0.99 precision threshold trades sensitivity for precision; sites below this threshold are discarded
- Known RMBase sites (plus 25-nt flanks) are masked, so this skill is not suited to reproducing predictions on already documented sites

## Evidence

- [methods] Uridine-centered fragments are the scoring unit: "Only those fragments with uridine at their center were considered for further prediction"
- [methods] Precision threshold governs Ψ calls: "A precision threshold of >0.99 was used to predict whether the uridine site is a potential Ψ site"
- [methods] Known sites masked with 25-bp flanks: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
