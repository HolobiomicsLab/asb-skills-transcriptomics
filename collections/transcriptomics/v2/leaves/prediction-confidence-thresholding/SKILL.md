---
name: prediction-confidence-thresholding
description: Use when classifying genomes (e.g., unknown-lifestyle Burkholderia/Paraburkholderia
  or Pseudomonas strains) with random-forest per-lifestyle probability outputs where
  an assignment cutoff must separate confident calls from uncertain ones.
when_to_use_negative:
- Genomes already carry literature-derived lifestyle labels — these are training/evaluation
  data, not prediction targets for thresholded assignment
- The goal is identifying lifestyle-associated genes (pLAGs via >70% group presence
  and >2 log2 fold change) rather than assigning lifestyles to individual genomes
- The classifier is trained from scratch rather than being the article's trained model
  — the 0.8 cutoff and per-lifestyle binary forests are specific to bacLIFE's setup
  and must be re-justified
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: bacLIFE
  role: Workflow whose lifestyle-prediction module applies the confidence-thresholded
    random forest assignment
  repo: https://github.com/Carrion-lab/bacLIFE
- name: scikit-learn
  role: Trains the per-lifestyle binary random forests (n_estimators=50) and supplies
    predict_proba-style tree-vote confidences that the 0.8 threshold is applied to
- name: Python
  role: Implements the chi2 feature selection, classifier application, thresholding,
    and TSV writing
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:55ae5dc66dc62ca812305739a44020e6b7660ea2e87f0dcd71eb458cc833807b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - Python
  - bacLIFE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# prediction-confidence-thresholding

## Summary

Assigns a predicted lifestyle to a genome only when the binary random forest (scikit-learn, n_estimators=50) adjudication confidence — the fraction of trees voting for a class — exceeds 0.8; genomes below the threshold stay 'unknown'. This prevents overconfident lifestyle calls in bacLIFE's lifestyle-prediction module.

## When to use

Use when classifying genomes (e.g., unknown-lifestyle Burkholderia/Paraburkholderia or Pseudomonas strains) with random-forest per-lifestyle probability outputs where an assignment cutoff must separate confident calls from uncertain ones.

## When NOT to use

- Genomes already carry literature-derived lifestyle labels — these are training/evaluation data, not prediction targets for thresholded assignment
- The goal is identifying lifestyle-associated genes (pLAGs via >70% group presence and >2 log2 fold change) rather than assigning lifestyles to individual genomes
- The classifier is trained from scratch rather than being the article's trained model — the 0.8 cutoff and per-lifestyle binary forests are specific to bacLIFE's setup and must be re-justified

## Inputs

- Gene cluster absence/presence matrix for the genome set (Burkholderia/Paraburkholderia and/or Pseudomonas)
- Trained binary random forest classifiers, one per lifestyle (scikit-learn, n_estimators=50)
- Chi2-selected top-1000 gene-cluster feature vectors (feature selection as applied during training)
- Lifestyle metadata table listing genomes labeled unknown (145 Burkholderia/Paraburkholderia, 976 Pseudomonas)

## Outputs

- Augmented lifestyle metadata table (TSV) with one row per genome: genome identifier, assigned lifestyle (or 'unknown'), and confidence values
- Confidence/probability values (tree-vote fractions) for all genomes, including sub-threshold uncertain scores

## How to apply

Apply chi2 feature selection to restrict each genome's absence/presence gene-cluster vector to the top 1000 informative features, then run the trained per-lifestyle binary random forest classifiers (scikit-learn, n_estimators=50) to obtain, for each lifestyle, the fraction of forest trees voting for the class. Assign a lifestyle only if the adjudication confidence exceeds 0.8; genomes scoring below the threshold (e.g., intermediate splits such as 0.668 vs 0.368) are left as 'unknown' rather than forced to the argmax class. This decision rule reproduces the paper's assignment rates: 137/145 (94.5%) formerly unknown Burkholderia/Paraburkholderia and 740/976 (75.9%) Pseudomonas genomes confidently assigned. Write the augmented metadata table (TSV) with genome identifier, assigned lifestyle (or unknown), and confidence values, keeping confidence values for all genomes including unassigned ones so uncertain calls remain inspectable.

## Related tools

- **bacLIFE** (Workflow whose lifestyle-prediction module applies the confidence-thresholded random forest assignment) — https://github.com/Carrion-lab/bacLIFE
- **scikit-learn** (Trains the per-lifestyle binary random forests (n_estimators=50) and supplies predict_proba-style tree-vote confidences that the 0.8 threshold is applied to)
- **Python** (Implements the chi2 feature selection, classifier application, thresholding, and TSV writing)

## Examples

```
proba = rf.predict_proba(X_chi2_top1000); lifestyle = classes[proba.argmax()] if proba.max() > 0.8 else 'unknown'
```

## Evaluation signals

- Fraction of formerly unknown genomes confidently assigned matches reference rates: 137/145 (94.5%) Burkholderia/Paraburkholderia and 740/976 (75.9%) Pseudomonas
- Every genome in the output with confidence <= 0.8 is labeled 'unknown', not forced to the highest-scoring class (e.g., a 0.668 vs 0.368 split must stay unassigned)
- Output TSV contains genome identifier, predicted-lifestyle, and confidence-value columns, with confidence values recorded for all genomes
- Feature vectors fed to the classifiers contain only the chi2-selected top 1000 gene-cluster features, as used during training
- In leave-one-species-out validation, confident calls match the reported tree-vote behavior (e.g., all 40 B. vietnamiensis genomes as opportunistic human pathogen with 90% of trees voting for the class)

## Limitations

- Genomes below the 0.8 confidence threshold remain unassigned (5.5% of Burkholderia/Paraburkholderia and 24.1% of Pseudomonas formerly unknown genomes in the paper), so no lifestyle call is available for them
- Confidence is a raw tree-vote fraction from n_estimators=50 forests, not a calibrated probability; intermediate values like 0.668 vs 0.368 indicate genuinely ambiguous genomes
- Predictions depend on lifestyle binary classifiers trained on literature-derived metadata, so assignment quality is bounded by the training labels and per-lifestyle classifier availability

## Evidence

- [methods] A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [methods] The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test): "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [results] bacLIFE predicted the formerly unknown lifestyles of 137 out of 145 (94.5%) Burkholderia/Paraburkholderia and 740 out of 976 (75.9%) Pseudomonas strains.: [evidence span withheld — anchor status: not_found]
- [methods] leave genomes below this threshold as unassigned and record their uncertain scores (e.g. intermediate values such as 0.668 vs 0.368): [evidence span withheld — anchor status: not_found]
- [readme] Employs a machine learning model to forecast bacterial lifestyle or other specified metadata: "Employs a machine learning model to forecast bacterial lifestyle or other specified metadata"
