---
name: response-class-labeling-microarray-samples
description: Use when running PIMENTo's SigGenesSAM on background-subtracted microarray
  data where samples fall into two biological conditions and a class vector must encode
  which sample belongs to which response class.
when_to_use_negative:
- Samples do not fall into discrete biological response classes (the response vector
  requires class labels)
- The expression matrix has not yet been background-subtracted/normalized by the PIMENTo
  Preprocess Data step
- The goal is only QC visualization (MA plots, dendrograms) rather than class-differential
  significance testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: PIMENTo
  role: R package whose SigGenesSAM command consumes the response class vector and
    performs SAM significance testing plus similarity/PCA visualization
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: samr
  role: R package providing the Significance Analysis of Microarrays code underlying
    SigGenesSAM
- name: R
  role: Language in which the response vector c(1,1,1,2,2,2) is defined and passed
    to PIMENTo commands
- name: RStudio
  role: IDE used to run the PIMENTo commands with autocomplete
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:edb3673ec252a0d6afb62de974730342f9adb5fa510fad0b24bf27a6566d8593
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  - samr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Response class labeling of microarray samples

## Summary

Defines a numeric response class vector encoding sample groups (e.g., c(1,1,1,2,2,2) for three WT and three villin-gp130Act small intestinal crypt samples) so that PIMENTo's Sig_GenesSAM significance testing and downstream similarity/PCA plots can compare the two classes.

## When to use

Use when running PIMENTo's SigGenesSAM on background-subtracted microarray data where samples fall into two biological conditions and a class vector must encode which sample belongs to which response class.

## When NOT to use

- Samples do not fall into discrete biological response classes (the response vector requires class labels)
- The expression matrix has not yet been background-subtracted/normalized by the PIMENTo Preprocess Data step
- The goal is only QC visualization (MA plots, dendrograms) rather than class-differential significance testing

## Inputs

- Background-subtracted expression matrix from PIMENTo preprocessing of the E-MTAB-2400 mouse microarray dataset
- Sample-to-class mapping: three wild-type (class 1) and three villin-gp130Act (class 2) small intestinal crypt samples
- FDR cutoff for Sig_GenesSAM (e.g., fdr.cutoff 0.4)

## Outputs

- Numeric response class vector (e.g., c(1, 1, 1, 2, 2, 2)) supplied to SigGenesSAM
- Tab-delimited significant genes output file at the chosen FDR cutoff
- Euclidean distance similarity plot of samples (Fig. 5)
- PCA plot of samples based on significant genes (Fig. 6)

## How to apply

After PIMENTo preprocessing/background subtraction of the E-MTAB-2400 mouse microarray data, define the response vector in the same sample order as the expression matrix columns, e.g. c(1, 1, 1, 2, 2, 2) with class 1 = wild-type and class 2 = villin-gp130Act small intestinal crypt samples. Pass this vector as the response argument to the PIMENTo SigGenesSAM command, which performs Significance Analysis of Microarrays via the R package 'samr'; set fdr.cutoff (e.g. 0.4) to identify genes whose expression differs significantly between the response classes. The class structure drives both the SAM delta table and the subsequent Euclidean distance similarity and PCA plots, so class labels must align exactly with sample column order. Validate by confirming the significant-genes tabular output is populated and that the distance matrix (Fig. 5) and PCA plot (Fig. 6) visually separate WT from villin-gp130Act samples while samples within the same class cluster together.

## Related tools

- **PIMENTo** (R package whose SigGenesSAM command consumes the response class vector and performs SAM significance testing plus similarity/PCA visualization) — https://www.github.com/MUSC-CGM/PIMENTo
- **samr** (R package providing the Significance Analysis of Microarrays code underlying SigGenesSAM)
- **R** (Language in which the response vector c(1,1,1,2,2,2) is defined and passed to PIMENTo commands)
- **RStudio** (IDE used to run the PIMENTo commands with autocomplete)

## Examples

```
response <- c(1, 1, 1, 2, 2, 2); SigGenesSAM(data = background_subtracted, response = response, fdr.cutoff = 0.4)
```

## Evaluation signals

- The response vector length matches the number of expression matrix samples (6 for c(1,1,1,2,2,2)) and class labels align with sample column order
- The significant-genes tabular output file is populated at the chosen fdr.cutoff (0.4)
- The Euclidean distance matrix plot shows large between-class distances and small within-class distances between WT and villin-gp130Act samples
- The PCA plot visually separates the two response classes
- Class clustering in downstream plots is consistent with the response vector structure (no batch-effect-driven misclustering)

## Limitations

- The chapter specifies a two-class, three-versus-three design; larger or multi-class designs require extending the response vector and were not demonstrated
- Downstream separation of classes depends on the upstream normalization choice (quantile normalization was judged better than LOESS for this data set)
- No version numbers are stated for any software dependency (limma, affy, samr, R, Bioconductor), so reproducibility of exact package versions is unverified
- An R Shiny UI for PIMENTo was not yet available; only the command-line R interface exists

## Evidence

- [other] Define the response class vector c(1,1,1,2,2,2) encoding three wild-type (class 1) and three villin-gp130Act (class 2) samples.: [evidence span withheld — anchor status: not_found]
- [other] Run the PIMENTo SigGenesSAM command, which performs Significance Analysis of Microarrays testing via the R package 'samr', with fdr.cutoff set to 0.4: [evidence span withheld — anchor status: not_found]
- [discussion] Significance testing is carried out using the code for Significance Analysis of Microarrays from the R package “samr”: [evidence span withheld — anchor status: not_found]
