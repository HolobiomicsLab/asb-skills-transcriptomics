---
name: tabular-feature-index-export
description: Use when a feature-selection step has produced a reduced feature set
  (e.g. mode-wise top-ranked features) and the retained features' identities (descriptor
  mode + original column index) must be written to a CSV artifact for later model
  training or validation.
when_to_use_negative:
- Input is already a reduced feature table with resolved feature identities — re-exporting
  indices would duplicate an existing artifact.
- The feature selection has not yet been fit on the training partitions of each CV
  fold; exporting indices before fitting would leak or freeze an unstable selection.
- The working dataset is Dataset.csv (raw sequences) rather than the generated All_Features_Dataset.csv
  — column indices do not yet exist for the proposed features.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Used to construct and write the selected feature-index mapping to selected_feature_indices.csv
    and to load All_Features_Dataset.csv for resolving original column indices.
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: scikit-learn
  role: Supplies the RandomForestClassifier whose Gini (mean-decrease-in-impurity)
    importances determine the top-100 features per mode that populate the exported
    index set.
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: numpy
  role: Supports array manipulation of feature importances and column indices before
    the export step.
  repo: https://github.com/zahid6454/iRNA5hmC-PS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:20c5af9b92ae5635189ea16dd3df5deca262c68a0d3b94917dfa21d0d526397d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zahid6454/iRNA5hmC-PS
  tools:
  - iRNA5hmC-PS
  - numpy
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tabular-feature-index-export

## Summary

Persist the reduced feature-index set (descriptor mode plus original column index for each retained feature) to a CSV file with pandas, so that the group-wise Random Forest feature-selection outcome (4 x 100 top-Gini features plus GC-content = 401 features) is reproducible and auditable downstream.

## When to use

Use when a feature-selection step has produced a reduced feature set (e.g. mode-wise top-ranked features) and the retained features' identities (descriptor mode + original column index) must be written to a CSV artifact for later model training or validation.

## When NOT to use

- Input is already a reduced feature table with resolved feature identities — re-exporting indices would duplicate an existing artifact.
- The feature selection has not yet been fit on the training partitions of each CV fold; exporting indices before fitting would leak or freeze an unstable selection.
- The working dataset is Dataset.csv (raw sequences) rather than the generated All_Features_Dataset.csv — column indices do not yet exist for the proposed features.

## Inputs

- Selected feature set: top-100 highest-Gini-importance features from each of four descriptor modes plus the GC-content feature (401 total)
- Original feature table All_Features_Dataset.csv with columns for the four position-specific descriptor modes (A-D), GC-content, and class labels
- Feature-to-index mapping derived from the All_Features_Dataset.csv column layout

## Outputs

- selected_feature_indices.csv — a pandas-written table listing each retained feature as (descriptor mode, original column index)

## How to apply

After fitting the independent scikit-learn RandomForestClassifier per descriptor mode on the training partition of each 5-fold CV split and ranking features by Gini importance, collect the top-100 features from each of the four position-specific modes (A-D) to form a 400-feature set, then append the GC-content feature to reach 401. Build a tabular record that maps each retained feature to its descriptor mode and its original column index within All_Features_Dataset.csv. Write this mapping with pandas to selected_feature_indices.csv so that subsequent model training reads only the selected columns. Validate the export by confirming the file contains exactly 401 entries (4 x 100 plus GC-content) and that the selectors were fit on training partitions only.

## Related tools

- **pandas** (Used to construct and write the selected feature-index mapping to selected_feature_indices.csv and to load All_Features_Dataset.csv for resolving original column indices.) — https://github.com/zahid6454/iRNA5hmC-PS
- **scikit-learn** (Supplies the RandomForestClassifier whose Gini (mean-decrease-in-impurity) importances determine the top-100 features per mode that populate the exported index set.) — https://github.com/zahid6454/iRNA5hmC-PS
- **numpy** (Supports array manipulation of feature importances and column indices before the export step.) — https://github.com/zahid6454/iRNA5hmC-PS

## Examples

```
import pandas as pd; pd.DataFrame({'mode': selected_modes, 'column_index': selected_indices}).to_csv('selected_feature_indices.csv', index=False)
```

## Evaluation signals

- selected_feature_indices.csv contains exactly 401 feature entries (4 x 100 mode-wise selections plus the GC-content feature).
- Every exported entry carries both a descriptor mode label and an original column index resolvable against All_Features_Dataset.csv.
- All exported indices refer to features whose selectors were fit on training partitions only — no held-out fold data was used.
- Re-running the notebook pipeline (Feature Generation.ipynb then Model.ipynb) reproduces the same All_Features_Dataset.csv column layout from which the indices were derived.
- The downstream Model.ipynb can load All_Features_Dataset.csv and subset to the 401 exported columns without key errors.

## Limitations

- The export is only valid if All_Features_Dataset.csv is regenerated with the same column ordering (via Feature Generation.ipynb) before indices are applied; changing feature generation invalidates the indices.
- Index stability depends on the Random Forest feature selectors being refit per CV split; a single global export assumes the documented group-wise selection strategy was followed.
- The pipeline requires Dataset.csv, Feature Generation.ipynb, and Model.ipynb to be co-located in the same Jupyter folder, and pip packages (python, pandas, numpy, sklearn, matplotlib, imblearn, tabulate) to be installed.

## Evidence

- [other] Write the reduced feature-index set (descriptor mode plus original column index for each retained feature) to selected_feature_indices.csv using pandas.: [evidence span withheld — anchor status: not_found]
- [abstract] We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information: "We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information"
- [readme] After the execution of the "Feature Generation.ipynb", a new file titled "All_Features_Dataset.csv" will be avaiable in the folder.: "After the execution of the "Feature Generation.ipynb", a new file titled "All_Features_Dataset.csv" will be avaiable in the folder."
- [methods] When "All_Features_Dataset.csv" file is available execute the "Model.ipynb" file just like "Feature Generation.ipynb" file was executed earlier.: "When "All_Features_Dataset.csv" file is available execute the "Model.ipynb" file just like "Feature Generation.ipynb" file was executed earlier."
