---
name: multi-tissue-expression-data-parsing
description: Use when you have absolute log10(x+1) expression values for gene deletion
  events across m conditions (e.g., 6 Drosophila tissues) and must build the EmpiricalData.data-style
  input required by CLOUDe's GenerateTrainingData, CLOUDeClassifyNN, and CLOUDePredictNN
  functions.
when_to_use_negative:
- Expression data are raw counts or non-log-transformed values — the format requires
  absolute log10(x+1) expression values, so transformation must precede this skill
- Data are single-species gene expression profiles without paired derived/survived/lost
  genes — the 3m column layout presumes two-species deletion events
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: CLOUDe
  role: R package whose GenerateTrainingData, CLOUDeClassifyNN, and CLOUDePredictNN
    functions consume the parsed multi-tissue expression file
  repo: https://github.com/anddssan/CLOUDe
- name: R
  role: Runtime environment in which CLOUDe.R is sourced and the input file is read
- name: dplyr
  role: One of the R libraries that must be installed to run the CLOUDe script that
    parses the input file
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:859af67212a9da4e366a1e4331335f41a1eda94040ff8a7f160c0fdc3c2bc470
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - dplyr
  - tensorflow
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# multi-tissue-expression-data-parsing

## Summary

Formats and validates multi-tissue expression data into CLOUDe's space-delimited input structure (N+1 rows by 3m columns with derived/survived/lost gene blocks) so deletion events can be classified and their evolutionary parameters predicted with the CLOUDe R package.

## When to use

Use when you have absolute log10(x+1) expression values for gene deletion events across m conditions (e.g., 6 Drosophila tissues) and must build the EmpiricalData.data-style input required by CLOUDe's GenerateTrainingData, CLOUDeClassifyNN, and CLOUDePredictNN functions.

## When NOT to use

- Expression data are raw counts or non-log-transformed values — the format requires absolute log10(x+1) expression values, so transformation must precede this skill
- Data are single-species gene expression profiles without paired derived/survived/lost genes — the 3m column layout presumes two-species deletion events

## Inputs

- Absolute log10(x+1) expression values for N deletion events across m conditions, split into derived, survived, and lost genes
- Number of conditions m (e.g., m=6 tissues in the Drosophila example)
- Number of deletion events N (e.g., N=100 for D. melanogaster/D. pseudoobscura)

## Outputs

- Space-delimited input file (e.g., EmpiricalData.data) with a one-row header and N event rows of 3m columns
- Feature file (training_prefix.features) with p=3m features consumed by CLOUDe classification and prediction functions

## How to apply

Construct a space-delimited file with N+1 rows and 3m columns, where each row (rows 2 to N+1) is one deletion event and the first row is the header eD1..eDm, eS1..eSm, eL1..eLm. Order columns so that columns j, m+j, and 2m+j hold the absolute log10(x+1) expression values at condition j in the derived, survived, and lost genes respectively — for the CLOUDe example this is N=100 D. melanogaster/D. pseudoobscura deletion events at m=6 tissues (18 columns). Place the file in the CLOUDe ExampleFiles directory (or pass its full name as empiricalFile to GenerateTrainingData) so the p=3m features can be extracted against OU-process training data. Verify parsing correctness by checking row/column counts and confirming that downstream functions emit one row per deletion event in the classifications, probabilities, and predictions files.

## Related tools

- **CLOUDe** (R package whose GenerateTrainingData, CLOUDeClassifyNN, and CLOUDePredictNN functions consume the parsed multi-tissue expression file) — https://github.com/anddssan/CLOUDe
- **R** (Runtime environment in which CLOUDe.R is sourced and the input file is read)
- **dplyr** (One of the R libraries that must be installed to run the CLOUDe script that parses the input file)

## Examples

```
source("CLOUDe.R"); CLOUDeClassifyNN("ExampleFiles/Training", "ExampleFiles/EmpiricalData", 2)
```

## Evaluation signals

- File has exactly N+1 rows and 3m columns (for the example: 101 rows and 18 columns for N=100 events, m=6 tissues)
- Header matches the pattern eD1..eDm, eS1..eSm, eL1..eLm for the m conditions
- Columns j, m+j, and 2m+j correspond to condition j in derived, survived, and lost genes, respectively
- Downstream CLOUDeClassifyNN and CLOUDePredictNN output files contain one row per deletion event (100 rows for the Drosophila empirical data)
- All expression entries are non-negative absolute log10(x+1) values

## Limitations

- The format assumes a fixed number of conditions m shared by all three genes of every deletion event; ragged or missing tissues require reformatting
- The example EmpiricalData.data covers only N=100 deletion events at m=6 tissues; other datasets must be reshaped to match the documented column semantics before CLOUDe functions will accept them
- Hyperparameter tuning (ClassifierCVnn/PredictorCVnn) is time-consuming, so pre-trained models (e.g., Training.nn.2.classifier.hdf5, Training.nn.3.predictor.hdf5) are typically reused rather than retrained per parsed dataset

## Evidence

- [readme] Format of input file: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions"
- [readme] Column semantics: "Specifically, columns j, m+j, and 2m+j represent the expression values at condition j (j = 1, 2, ..., m) in the derived, survived, and lost genes, respectively."
- [readme] Example dataset: "The ExampleFiles directory contains a file called EmpiricalData.data, which illustrates this format for N=100 deletion events at m=6 tissues."
- [full_text] Empirical application: [evidence span withheld — anchor status: not_found]
- [readme] Value transformation: "each column is an absolute log10(x+1) expression value at a particular condition in a particular gene corresponding to the deletion event"
