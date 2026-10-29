---
name: transcriptome-wide-pseudouridine-prediction
description: Use when you need transcriptome-wide, base-resolution prediction of novel
  RNA pseudouridine sites in human protein-coding genes using a trained sequence-plus-secondary-structure
  deep learning model.
when_to_use_negative:
- The goal is to re-identify already experimentally validated RMBase Ψ sites — masking
  step removes them, so the scan targets novel sites only
- Input species or genome build is not supported by the trained model reference files
  (hg19 human; mm10 mouse and sacCer3 yeast were used separately in the paper)
- Only a small set of individual RNA fragments needs classification — direct MU-PseUDeep
  prediction is more appropriate than a full transcriptome scan
edam_operation: http://edamontology.org/operation_0431
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: MU-PseUDeep
  role: Trained dual-input CNN (raw RNA sequence + predicted secondary structure)
    scoring uridine-centered 51 bp fragments for Ψ site prediction at >0.99 precision
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: BEDTools
  role: Extracts transcriptome coding sequences from hg19 with GENCODE gtf and masks
    known RMBase Ψ sites with 25 bp flanks
- name: SeqKit
  role: Generates running 51 bp windows across the transcriptome for the uridine-centered
    fragment filter
- name: RMBase v2.0
  role: Source of known Ψ site coordinates used for masking and for cross-checking
    predicted genes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:9aee3e3e63e18cf8eefb0d2b37f0953019b988a4c5bacc433cccca07ac4f20bd
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
  edam_operation: http://edamontology.org/operation_0431
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# transcriptome-wide-pseudouridine-prediction

## Summary

Scans human protein-coding transcriptome sequences with the trained MU-PseUDeep deep learning model to identify pseudouridine (Ψ) sites at a >0.99 precision threshold, yielding per-gene Ψ site predictions with high confidence.

## When to use

Use when you need transcriptome-wide, base-resolution prediction of novel RNA pseudouridine sites in human protein-coding genes using a trained sequence-plus-secondary-structure deep learning model.

## When NOT to use

- The goal is to re-identify already experimentally validated RMBase Ψ sites — masking step removes them, so the scan targets novel sites only
- Input species or genome build is not supported by the trained model reference files (hg19 human; mm10 mouse and sacCer3 yeast were used separately in the paper)
- Only a small set of individual RNA fragments needs classification — direct MU-PseUDeep prediction is more appropriate than a full transcriptome scan

## Inputs

- hg19 human reference genome
- GENCODE gtf annotation file
- RMBase v2.0 known Ψ site coordinates (for masking and validation)
- Trained MU-PseUDeep model (Keras 2.2.2 / TensorFlow 1.10.1)

## Outputs

- List of protein-coding genes containing one or more predicted Ψ sites (2441 genes reported at >0.99 precision)
- Genomic regions of predicted Ψ sites recorded in supplemental table 'Supplemental_table2.csv'
- Count of predicted Ψ-site genes overlapping known RMBase sites (284 genes in the paper)

## How to apply

Extract human transcriptome coding sequences from the hg19 genome using BEDTools with the GENCODE gtf file, converting DNA to RNA according to each gene's strand. Mask known RMBase v2.0 pseudouridine sites plus 25 bases of flanking sequence on both sides so the scan targets novel sites. Generate running 51 bp windows with SeqKit and retain only fragments with a uridine at their center. Score each uridine-centered fragment with the bootstrapped, hyperparameter-optimized MU-PseUDeep model (Keras 2.2.2 / TensorFlow 1.10.1), calling a uridine a potential Ψ site only when model precision exceeds 0.99. Finally, summarize predicted Ψ sites per gene and cross-check the gene set against RMBase documented Ψ sites as a validation step.

## Related tools

- **MU-PseUDeep** (Trained dual-input CNN (raw RNA sequence + predicted secondary structure) scoring uridine-centered 51 bp fragments for Ψ site prediction at >0.99 precision) — https://github.com/smk5g5/MU-PseUDeep
- **BEDTools** (Extracts transcriptome coding sequences from hg19 with GENCODE gtf and masks known RMBase Ψ sites with 25 bp flanks)
- **SeqKit** (Generates running 51 bp windows across the transcriptome for the uridine-centered fragment filter)
- **RMBase v2.0** (Source of known Ψ site coordinates used for masking and for cross-checking predicted genes)

## Examples

```
seqkit locate && bedtools intersect -v -a windows.fasta -b RMBase_psi_sites_plus25bp.bed  # then score with MU-PseUDeep at precision > 0.99
```

## Evaluation signals

- Recover the reported 2441 human protein-coding genes with one or more predicted Ψ site at the >0.99 precision threshold
- Cross-check the predicted gene set against RMBase: 284 genes should already have one or more documented Ψ site
- Verify every scored fragment is 51 bp with a uridine at its center before prediction
- Confirm predicted Ψ sites do not overlap known RMBase Ψ sites or their 25 bp flanks (masking invariant)

## Limitations

- Known RMBase Ψ sites and their 25 bp flanking sequence are masked, so the scan cannot recover already documented sites
- Predictions at the >0.99 precision threshold trade sensitivity for precision; sites below this threshold are discarded
- Transcriptome-wide predictions for mouse (mm10) and yeast (sacCer3) require separately trained/reference-specific runs, not the human scan

## Evidence

- [results] 2441 genes with one or more predicted Ψ site at the >0.99 precision threshold: [evidence span withheld — anchor status: not_found]
- [methods] Fragment generation and filtering: [evidence span withheld — anchor status: not_found]
- [methods] Precision threshold: "A precision threshold of >0.99 was used to predict whether the uridine site is a potential Ψ site"
- [results] RMBase overlap validation: "284 genes already had one or more known Ψ site as documented in the RMBase"
- [abstract] Model availability: "The tool is open source, available at https://github.com/smk5g5/MU-PseUDeep"
