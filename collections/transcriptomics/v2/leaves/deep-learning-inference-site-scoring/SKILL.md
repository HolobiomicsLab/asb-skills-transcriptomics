---
name: deep-learning-inference-site-scoring
description: Use when a trained sequence+structure deep learning model must score
  candidate modification sites across a transcriptome and candidate sites must be
  filtered by a strict precision threshold (e.g. >0.99).
when_to_use_negative:
- The task is de novo model training on labeled Ψ/negative fragments rather than inference/scanning
  with an already-trained model
- Sequences are not RNA or lack uridine-centered 51 bp fragment context (the model
  expects uridine at fragment center)
- A lower precision / higher recall operating point is desired — the >0.99 precision
  threshold deliberately prioritizes precision over sensitivity
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0659
tools:
- name: MU-PseUDeep
  role: Trained dual 1D-CNN model that scores uridine-centered fragments using raw
    RNA sequence and predicted secondary structure
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: BEDTools
  role: Extracts coding sequences from hg19 + GENCODE gtf and masks known RMBase Ψ
    sites plus 25 bp flanks
- name: SeqKit
  role: Generates running 51 bp windows across the transcriptome
- name: Hyperas
  role: Hyperparameter optimization wrapper used to train the model applied for inference
  repo: https://github.com/maxpumperla/hyperas
- name: hyperopt
  role: Search backend for the Hyperas hyperparameter optimization
  repo: https://github.com/hyperopt/hyperopt
- name: RMBase v2.0
  role: Source of known Ψ sites for masking and post-hoc validation of predicted genes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:84254f04194bdff381e5be26d31c77dc665bf23ebf537c7da7f7245fc49b140b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - BEDTools
  - Hyperas
  - MU-PseUDeep
  - RMBase v2.0
  - SeqKit
  - hg19
  - hyperopt
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# deep-learning-inference-site-scoring

## Summary

Apply a trained deep learning model (MU-PseUDeep, a dual 1D-CNN on raw RNA sequence and predicted secondary structure) to score candidate uridine-centered fragments transcriptome-wide, calling pseudouridine (Ψ) sites only above a >0.99 precision threshold.

## When to use

Use when a trained sequence+structure deep learning model must score candidate modification sites across a transcriptome and candidate sites must be filtered by a strict precision threshold (e.g. >0.99).

## When NOT to use

- The task is de novo model training on labeled Ψ/negative fragments rather than inference/scanning with an already-trained model
- Sequences are not RNA or lack uridine-centered 51 bp fragment context (the model expects uridine at fragment center)
- A lower precision / higher recall operating point is desired — the >0.99 precision threshold deliberately prioritizes precision over sensitivity

## Inputs

- Reference genome (hg19) and GENCODE gtf annotation for coding sequence extraction
- RMBase v2.0 experimentally validated Ψ site coordinates (for masking and validation)
- Transcriptome FASTA sequences for 51 bp window generation (SeqKit)
- Trained MU-PseUDeep model (Keras 2.2.2 / TensorFlow 1.10.1, hyperparameter-optimized)

## Outputs

- Set of uridines called as predicted Ψ sites at >0.99 precision (Supplemental_table2.csv with genomic regions)
- Per-gene counts of predicted Ψ sites (2441 protein-coding genes with ≥1 predicted site)
- Cross-tabulation of predicted genes against known RMBase Ψ sites (284 overlapping genes)

## How to apply

Extract coding sequences from the reference genome and GTF (hg19 + GENCODE) with BEDTools, converting DNA to RNA according to each gene's strand, and mask known RMBase Ψ sites plus 25 bases of flanking sequence on both sides so the scan targets novel sites. Generate running 51 bp windows with SeqKit and retain only fragments with a uridine at their center, since the model is trained on uridine-centered fragments. Encode each fragment as a one-of-K binary matrix (51 × 4) for sequence and a dot-bracket notation matrix for the RNAshapes-predicted secondary structure, then score both inputs through the paired 1D CNNs of the hyperparameter-optimized model (Keras 2.2.2 / TensorFlow 1.10.1, tuned with Hyperas/Hyperopt). Call a uridine as a potential Ψ site only when the model's prediction precision exceeds 0.99, then summarize predictions per gene and cross-check against known RMBase Ψ sites; this procedure recovered 2441 human protein-coding genes with one or more predicted Ψ site, of which 284 already had documented RMBase sites.

## Related tools

- **MU-PseUDeep** (Trained dual 1D-CNN model that scores uridine-centered fragments using raw RNA sequence and predicted secondary structure) — https://github.com/smk5g5/MU-PseUDeep
- **BEDTools** (Extracts coding sequences from hg19 + GENCODE gtf and masks known RMBase Ψ sites plus 25 bp flanks)
- **SeqKit** (Generates running 51 bp windows across the transcriptome)
- **Hyperas** (Hyperparameter optimization wrapper used to train the model applied for inference) — https://github.com/maxpumperla/hyperas
- **hyperopt** (Search backend for the Hyperas hyperparameter optimization) — https://github.com/hyperopt/hyperopt
- **RMBase v2.0** (Source of known Ψ sites for masking and post-hoc validation of predicted genes)

## Examples

```
seqkit fx2tab --length < transcriptome.fa; bedtools maskfasta -fi coding.fa -bed rmbase_psi_plus25.bed -fo masked.fa  # then score 51nt uridine-centered windows with the trained MU-PseUDeep Keras model and call sites at precision >0.99
```

## Evaluation signals

- Every called site has a uridine at the center of its 51 bp fragment (uridine-centered fragment filter was applied)
- All reported calls meet the >0.99 precision threshold of the trained model
- Per-gene summary reproduces 2441 human protein-coding genes with ≥1 predicted Ψ site
- Cross-check against RMBase recovers 284 genes with one or more documented known Ψ site
- Known RMBase Ψ sites (plus 25 bp flanks) were masked before scanning, so predicted sites represent novel candidates

## Limitations

- The >0.99 precision threshold trades sensitivity for precision; lower-confidence Ψ sites are not reported
- Sites already documented in RMBase are masked with 25 bp flanks, so this scan targets only novel sites
- Only fragments with uridine at their center are scored; non-uridine positions are never candidates
- Model performance depends on the hyperparameter-optimized architecture (Keras 2.2.2 / TensorFlow 1.10.1) as trained; no direct comparison to iPseU-CNN was possible since no source code or webserver was available

## Evidence

- [methods] uridine-centered fragment filter: "Only those fragments with uridine at their center were considered for further prediction"
- [methods] precision threshold >0.99: "A precision threshold of >0.99 was used to predict whether the uridine site is a potential Ψ site"
- [results] transcriptome scan result: "We applied MU-PseUDeep to scan the protein-coding genes in the human transcriptome"
- [results] RMBase overlap validation: "284 genes already had one or more known Ψ site as documented in the RMBase"
- [abstract] model architecture basis: "which inputs the raw RNA sequence and the predicted secondary structure to two sets of convolutional neural networks"
