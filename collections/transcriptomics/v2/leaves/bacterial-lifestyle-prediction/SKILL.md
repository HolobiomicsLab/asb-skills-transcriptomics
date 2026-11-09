---
name: bacterial-lifestyle-prediction
description: Use when you have a gene cluster absence/presence matrix for genomes
  with literature-derived lifestyle labels and need to (a) quantify lifestyle predictability
  via cross-validated AUC or (b) assign lifestyles to unknown-lifestyle genomes.
when_to_use_negative:
- Input is a raw genome set without a gene cluster absence/presence matrix — run the
  bacLIFE clustering module (Prokka, linclust/MCL) first
- Lifestyle labels are not grounded in literature-derived metadata, since the classifiers
  are binary one-vs-rest models trained per labeled lifestyle group
- Analysis goal is identifying individual lifestyle-associated genes (LAGs) rather
  than classifying whole-genome lifestyles — use the pLAG enrichment thresholds (>70%
  group presence, >2 log2 fold change) instead
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bacLIFE
  role: Workflow that clusters genes into gene clusters and hosts the lifestyle prediction
    module with the machine learning model; results embedded in a Shiny app
  repo: https://github.com/Carrion-lab/bacLIFE
- name: scikit-learn
  role: Chi2 feature selection (top 1000 features) and binary random forest classifiers
    (n_estimators=50) with 5-fold cross-validation and AUC computation
- name: Python
  role: Implementation language of the Snakemake-based first module that organizes
    and executes the pipeline rules
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:8295ba9aa72b546d2068eccbe9f8fe42d49a67974d0ba5cce5bced1b22b20305
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bacterial-lifestyle-prediction

## Summary

Predict bacterial lifestyles (e.g. environmental, opportunistic animal pathogen, plant pathogen) from gene cluster absence/presence matrices using binary random forest classifiers trained per lifestyle, as implemented in the bacLIFE workflow. Assigns lifestyles to unknown genomes via confidence-thresholded adjudication.

## When to use

Use when you have a gene cluster absence/presence matrix for genomes with literature-derived lifestyle labels and need to (a) quantify lifestyle predictability via cross-validated AUC or (b) assign lifestyles to unknown-lifestyle genomes.

## When NOT to use

- Input is a raw genome set without a gene cluster absence/presence matrix — run the bacLIFE clustering module (Prokka, linclust/MCL) first
- Lifestyle labels are not grounded in literature-derived metadata, since the classifiers are binary one-vs-rest models trained per labeled lifestyle group
- Analysis goal is identifying individual lifestyle-associated genes (LAGs) rather than classifying whole-genome lifestyles — use the pLAG enrichment thresholds (>70% group presence, >2 log2 fold change) instead

## Inputs

- Gene cluster absence/presence matrix (genome x gene-cluster binary matrix from bacLIFE clustering module)
- Lifestyle metadata labels (environmental, opportunistic animal pathogen, plant pathogen) assigned by literature review per genome
- Trained random forest classifiers (scikit-learn, n_estimators=50, one binary classifier per lifestyle; Supplementary Datasets 10)
- Chi2 feature-selection model used during training (top 1000 features)

## Outputs

- Per-lifestyle, per-genus mean 5-fold cross-validated AUC values (lifestyle_prediction_auc.csv)
- Augmented lifestyle metadata TSV with genome identifier, assigned lifestyle (or unknown), and confidence values for all genomes (cf. Supplementary Datasets 2)
- Counts of confident assignments among formerly unknown-lifestyle genomes (e.g. 137/145 Burkholderia/Paraburkholderia; 740/976 Pseudomonas)

## How to apply

Load the gene cluster absence/presence matrix and literature-derived lifestyle metadata, then apply chi2 feature selection to retain the top 1000 informative features. Train one binary random forest classifier per lifestyle (scikit-learn, n_estimators=50) in a 5-fold cross-validation scheme (70% training, 30% test), computing mean AUC per lifestyle and genus. For unknown-lifestyle genomes, run the trained classifiers to obtain probability confidence values (fraction of trees voting for the class) and assign a lifestyle only when confidence exceeds 0.8, leaving the rest unassigned. High AUCs reflect strong genomic signatures of lifestyle in the clustered gene content; genomes with intermediate scores (e.g. 0.668 vs 0.368) should be kept as unknown rather than force-assigned.

## Related tools

- **bacLIFE** (Workflow that clusters genes into gene clusters and hosts the lifestyle prediction module with the machine learning model; results embedded in a Shiny app) — https://github.com/Carrion-lab/bacLIFE
- **scikit-learn** (Chi2 feature selection (top 1000 features) and binary random forest classifiers (n_estimators=50) with 5-fold cross-validation and AUC computation)
- **Python** (Implementation language of the Snakemake-based first module that organizes and executes the pipeline rules)

## Examples

```
from sklearn.feature_selection import SelectKBest, chi2; from sklearn.ensemble import RandomForestClassifier; X_sel = SelectKBest(chi2, k=1000).fit_transform(X, y); rf = RandomForestClassifier(n_estimators=50); rf.fit(X_sel, y)
```

## Evaluation signals

- Each Pseudomonas lifestyle mean AUC falls between 0.96 and 0.99; each Burkholderia/Paraburkholderia lifestyle AUC equals 1.00, matching Supplementary Fig. 2
- AUC is computed per lifestyle as the mean of the 5 cross-validated tests
- Fraction of confident assignments among unknown-lifestyle genomes is within tolerance of 137/145 (94.5%) for Burkholderia/Paraburkholderia and 740/976 (75.9%) for Pseudomonas
- Output lifestyle table contains genome, predicted-lifestyle, and confidence-value columns for all genomes
- Genomes with adjudication confidence <= 0.8 remain unassigned (e.g. intermediate scores such as 0.668 vs 0.368)

## Limitations

- Confidence threshold of >0.8 leaves genomes below it unassigned; 259/976 (24.1%) of Pseudomonas unknowns could not be assigned
- Model was trained and validated on Burkholderia/Paraburkholderia and Pseudomonas gene cluster matrices; transferability to other genera is not established here
- Feature space is restricted to the top 1000 chi2-selected gene cluster features, which may exclude informative low-prevalence gene clusters

## Evidence

- [methods] Top 1000 chi2 features feeding 5-fold CV: "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [methods] Confidence threshold for lifestyle assignment: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [methods] Classifiers trained on literature metadata: "binary classifiers for each lifestyle assigned based on metadata from literature (Supplementary Datasets 10)"
- [readme] README describes the lifestyle prediction module: "Employs a machine learning model to forecast bacterial lifestyle or other specified metadata"
