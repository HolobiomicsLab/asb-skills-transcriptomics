---
name: random-forest-classifier-inference
description: Use when you have a gene-cluster absence/presence matrix for genomes
  labeled 'unknown' and literature-derived lifestyle metadata plus trained binary
  random forest classifiers (scikit-learn, n_estimators = 50) to predict a lifestyle
  per genome at >0.8 confidence.
when_to_use_negative:
- Input is raw genomes without an annotated, clustered gene-cluster absence/presence
  matrix — run the bacLIFE clustering module (Prokka, linclust, MCL) first
- No trained classifiers or literature-derived lifestyle labels exist for the target
  genus — train/fit the model on labeled genomes before inference
- Genomes already carry a literature-derived lifestyle label — inference is only for
  formerly unknown lifestyles
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides the binary random forest classifiers (n_estimators = 50, one per
    lifestyle) whose predict probabilities, computed as the fraction of trees voting
    for the class, are thresholded at >0.8
- name: bacLIFE
  role: Lifestyle-prediction module that supplies the trained classifiers, chi2 feature
    selection, and the unknown-lifestyle genomes being adjudicated
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:e0b87f48658b5a3feeea344cdec5bf936d3a16aa13e63ac2b74c11b54d52ea59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - Python
  - bacLIFE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# random-forest-classifier-inference

## Summary

Assigns bacterial lifestyles to genomes with formerly unknown lifestyles by running trained scikit-learn random forest binary classifiers (one per lifestyle, n_estimators = 50) over chi2-selected gene-cluster presence/absence features, in the bacLIFE lifestyle-prediction module.

## When to use

Use when you have a gene-cluster absence/presence matrix for genomes labeled 'unknown' and literature-derived lifestyle metadata plus trained binary random forest classifiers (scikit-learn, n_estimators = 50) to predict a lifestyle per genome at >0.8 confidence.

## When NOT to use

- Input is raw genomes without an annotated, clustered gene-cluster absence/presence matrix — run the bacLIFE clustering module (Prokka, linclust, MCL) first
- No trained classifiers or literature-derived lifestyle labels exist for the target genus — train/fit the model on labeled genomes before inference
- Genomes already carry a literature-derived lifestyle label — inference is only for formerly unknown lifestyles

## Inputs

- Gene cluster absence/presence matrix (rows = genomes, columns = MCL gene clusters)
- Literature-derived lifestyle metadata labels for the genomes
- Trained random forest binary classifiers per lifestyle (scikit-learn, n_estimators = 50; e.g. Supplementary Datasets 10)
- Chi2 feature-selection transformation fitted during training (top 1000 features)

## Outputs

- Augmented lifestyle metadata table (TSV) with one row per genome: genome identifier, assigned lifestyle (or unknown/uncertain), and confidence values for all genomes
- Confidence values for all genomes (Supplementary Datasets 2)

## How to apply

Load the gene cluster absence/presence matrix, literature-derived lifestyle labels, and the trained random forest binary classifiers (one per lifestyle, scikit-learn, n_estimators = 50) for the Burkholderia/Paraburkholderia or Pseudomonas dataset. Apply the chi2 feature selection used during training to restrict each genome's feature vector to the top 1000 informative gene-cluster features. For each unknown-lifestyle genome (145 Burkholderia/Paraburkholderia, 976 Pseudomonas in the paper), run the corresponding binary classifiers and compute the confidence as the fraction of trees voting for the lifestyle class. Assign a lifestyle only when the random forest adjudication probability exceeds 0.8; leave genomes below the threshold unassigned while recording their intermediate scores (e.g. 0.668 vs 0.368) in the augmented metadata table.

## Related tools

- **scikit-learn** (Provides the binary random forest classifiers (n_estimators = 50, one per lifestyle) whose predict probabilities, computed as the fraction of trees voting for the class, are thresholded at >0.8)
- **bacLIFE** (Lifestyle-prediction module that supplies the trained classifiers, chi2 feature selection, and the unknown-lifestyle genomes being adjudicated) — https://github.com/Carrion-lab/bacLIFE

## Examples

```
proba = rf_lifestyle.predict_proba(X_chi2_top1000)[:, 1]; assigned = 'phytopathogenic' if proba > 0.8 else 'unknown'
```

## Evaluation signals

- Output TSV contains genome, predicted-lifestyle, and confidence-value columns for every genome
- Fraction of confident assignments is within tolerance of the reported 137/145 (94.5%) for Burkholderia/Paraburkholderia and 740/976 (75.9%) for Pseudomonas
- Genomes with adjudication confidence ≤ 0.8 are left unassigned rather than force-labeled
- Confidence values lie in [0, 1] and equal the fraction of trees voting for the class (uncertain cases show intermediate splits such as 0.668 vs 0.368)

## Limitations

- Genomes whose random forest adjudication falls at or below the 0.8 probability threshold remain unassigned with uncertain scores (e.g. 0.668 vs 0.368); ~24% of Pseudomonas strains (236/976) stayed unknown
- Prediction is restricted to the lifestyles present in the literature-derived training metadata, with one binary classifier per lifestyle
- Feature space is limited to the top 1000 chi2-selected gene-cluster features from the genus-specific clustering, so classification depends on the upstream MCL/linclust clustering consistency

## Evidence

- [methods] A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [methods] The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test): "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [readme] Employs a machine learning model to forecast bacterial lifestyle or other specified metadata: "Employs a machine learning model to forecast bacterial lifestyle or other specified metadata"
