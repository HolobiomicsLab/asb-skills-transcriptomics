---
name: genome-metadata-table-augmentation
description: Use when you have a gene cluster absence/presence matrix plus literature-derived
  lifestyle labels, and genomes labeled 'unknown' need a predicted lifestyle appended
  to the metadata table.
when_to_use_negative:
- Genomes already carry literature-derived lifestyle labels — this skill only adjudicates
  genomes labeled 'unknown'
- You have raw assemblies but no gene cluster absence/presence matrix — run bacLIFE's
  clustering module (linclust + MCL) first
- The goal is identifying lifestyle-associated genes (pLAGs, >70% group presence and
  >2 log2 fold change) rather than assigning lifestyles to genomes
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: bacLIFE
  role: Workflow whose lifestyle-prediction module supplies the trained per-lifestyle
    random forest models and metadata adjudication scheme
  repo: https://github.com/Carrion-lab/bacLIFE
- name: scikit-learn
  role: Implements the binary random forest classifiers (n_estimators=50) run per
    genome to produce lifestyle probability confidences
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:38cbeda8115429f163b214371715adf6b73e464affa0e7a64cb80a37118dece5
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# genome-metadata-table-augmentation

## Summary

Augments a genome metadata table by predicting lifestyles for genomes labeled 'unknown' with bacLIFE's trained per-lifestyle binary random forest classifiers, keeping only assignments with adjudication confidence >0.8 and recording confidence values for all genomes.

## When to use

Use when you have a gene cluster absence/presence matrix plus literature-derived lifestyle labels, and genomes labeled 'unknown' need a predicted lifestyle appended to the metadata table.

## When NOT to use

- Genomes already carry literature-derived lifestyle labels — this skill only adjudicates genomes labeled 'unknown'
- You have raw assemblies but no gene cluster absence/presence matrix — run bacLIFE's clustering module (linclust + MCL) first
- The goal is identifying lifestyle-associated genes (pLAGs, >70% group presence and >2 log2 fold change) rather than assigning lifestyles to genomes

## Inputs

- Gene cluster absence/presence matrix across genomes (from bacLIFE clustering module)
- Literature-derived lifestyle metadata labels per genome (TSV)
- Trained random forest classifiers, one binary classifier per lifestyle (scikit-learn, n_estimators=50; Supplementary Datasets 10)
- chi2 feature-selection state restricting to top 1000 gene-cluster features

## Outputs

- Augmented lifestyle metadata table (TSV) with one row per genome: genome identifier, assigned lifestyle (or unknown), and confidence values
- Confidence values for all genomes (as distributed in Supplementary Datasets 2)

## How to apply

Load the gene cluster absence/presence matrix, literature-derived lifestyle labels, and the trained random forest classifiers (scikit-learn, n_estimators=50, one binary classifier per lifestyle) for the Burkholderia/Paraburkholderia and Pseudomonas datasets. Reapply the chi2 feature selection used during training so each genome's feature vector is restricted to the top 1000 informative gene-cluster features. For each unknown-lifestyle genome, run the per-lifestyle binary random forests to obtain a probability confidence value (fraction of trees voting for the class). Assign a lifestyle only when the random forest adjudication confidence exceeds 0.8; leave lower-confidence genomes as unassigned while recording their intermediate scores (e.g. 0.668 vs 0.368). Write the augmented lifestyle metadata TSV with one row per genome containing genome identifier, assigned lifestyle (or unknown), and confidence values, and validate the confident-assignment fraction against the reported rates of 137/145 (94.5%) for Burkholderia/Paraburkholderia and 740/976 (75.9%) for Pseudomonas.

## Related tools

- **bacLIFE** (Workflow whose lifestyle-prediction module supplies the trained per-lifestyle random forest models and metadata adjudication scheme) — https://github.com/Carrion-lab/bacLIFE
- **scikit-learn** (Implements the binary random forest classifiers (n_estimators=50) run per genome to produce lifestyle probability confidences)

## Examples

```
proba = rf.predict_proba(X_top1000)[:, 1]; lifestyle = LIFESTYLE if proba > 0.8 else 'unknown'
```

## Evaluation signals

- Confident-assignment fraction within tolerance of 137/145 (94.5%) Burkholderia/Paraburkholderia and 740/976 (75.9%) Pseudomonas
- Every assigned lifestyle has random forest adjudication confidence >0.8; unassigned genomes retain recorded intermediate scores (e.g. 0.668 vs 0.368)
- Output TSV contains genome, predicted-lifestyle, and confidence-value columns with one row per input genome
- Feature vectors passed to the classifiers are restricted to the chi2 top 1000 gene-cluster features, matching the training-time feature selection
- Confidence values for all genomes (assigned and unassigned) are written out, as in Supplementary Datasets 2

## Limitations

- Genomes whose maximum adjudication confidence is at or below 0.8 remain unassigned — about 24% of Pseudomonas genomes and 5.5% of Burkholderia/Paraburkholderia genomes in the study
- Ambiguous cases yield intermediate split probabilities (e.g. 0.668 vs 0.368) and cannot be reliably resolved by the model
- Prediction quality depends on the upstream gene-cluster absence/presence matrix and on lifestyle labels derived from literature metadata

## Evidence

- [methods] A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [methods] The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test): "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [methods] Confidence values for all genomes can be found in Supplementary Datasets 2: "Confidence values for all genomes can be found in Supplementary Datasets 2"
- [readme] Employs a machine learning model to forecast bacterial lifestyle or other specified metadata: "Employs a machine learning model to forecast bacterial lifestyle or other specified metadata"
