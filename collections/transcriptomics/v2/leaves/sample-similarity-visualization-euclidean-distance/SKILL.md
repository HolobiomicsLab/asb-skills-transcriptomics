---
name: sample-similarity-visualization-euclidean-distance
description: Use when you have run SAM significance testing (e.g., Sig_GenesSAM with
  fdr.cutoff 0.4) on background-subtracted microarray data and need to visually check
  whether samples cluster by response class using the significant genes.
when_to_use_negative:
- Input is the raw unnormalized file (e.g., example_unnorm.txt) rather than the background-subtracted,
  normalized matrix output of the Preprocess Data step
- No significance testing (Sig_GenesSAM) has been performed yet, so there is no significant-genes
  set to restrict the distance computation to
- Samples lack a defined response class structure, so there is no class labeling against
  which to interpret the similarity plot
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: R package that runs the SAM-based significance testing (Sig_GenesSAM) and
    generates the Euclidean distance similarity plot of samples based on the significant
    genes
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: samr
  role: R package supplying the Significance Analysis of Microarrays code used by
    Sig_GenesSAM to select the significant genes used in the similarity plot
- name: R
  role: Programming language in which PIMENTo is built and the plotting commands are
    run
- name: RStudio
  role: IDE used to run the PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:3993282e4bb16192ad7480be21e3648766fe148b58d0f44d46a4cca975457db9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  - samr
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sample-similarity-visualization-euclidean-distance

## Summary

Generates a Euclidean distance similarity plot of microarray samples based on significant genes identified by PIMENTo's Sig_GenesSAM command, so that class structure (e.g., WT vs. villin-gp130Act) can be visually validated after SAM significance testing.

## When to use

Use when you have run SAM significance testing (e.g., Sig_GenesSAM with fdr.cutoff 0.4) on background-subtracted microarray data and need to visually check whether samples cluster by response class using the significant genes.

## When NOT to use

- Input is the raw unnormalized file (e.g., example_unnorm.txt) rather than the background-subtracted, normalized matrix output of the Preprocess Data step
- No significance testing (Sig_GenesSAM) has been performed yet, so there is no significant-genes set to restrict the distance computation to
- Samples lack a defined response class structure, so there is no class labeling against which to interpret the similarity plot

## Inputs

- Background-subtracted, normalized expression matrix from upstream PIMENTo preprocessing (e.g., E-MTAB-2400 mouse microarray data from wild-type vs. villin-gp130Act small intestinal crypts)
- Response class vector encoding sample classes (e.g., c(1,1,1,2,2,2) for three WT and three villin-gp130Act samples)
- Significant-genes table exported by Sig_GenesSAM at the chosen FDR cutoff (e.g., 0.4)

## Outputs

- Euclidean distance similarity plot of samples based on the significant genes (Fig. 5)
- Companion PCA plot of samples based on the significant genes (Fig. 6)

## How to apply

After running PIMENTo's Sig_GenesSAM command (which wraps the R 'samr' Significance Analysis of Microarrays code) on the background-subtracted expression matrix with a response vector such as c(1,1,1,2,2,2) and fdr.cutoff set to 0.4, use the exported significant-genes table as the feature set for the similarity plot. Compute Euclidean distances between samples restricted to the significant genes and render the distance matrix as a similarity plot (Fig. 5 in the chapter). Correct application is judged by whether samples within the same response class appear similar to each other while samples of different response classes show a clear difference, consistent with the class structure of the response vector. This step is typically paired with a PCA plot (Fig. 6) computed on the same significant genes.

## Related tools

- **PIMENTo** (R package that runs the SAM-based significance testing (Sig_GenesSAM) and generates the Euclidean distance similarity plot of samples based on the significant genes) — https://www.github.com/MUSC-CGM/PIMENTo
- **samr** (R package supplying the Significance Analysis of Microarrays code used by Sig_GenesSAM to select the significant genes used in the similarity plot)
- **R** (Programming language in which PIMENTo is built and the plotting commands are run)
- **RStudio** (IDE used to run the PIMENTo commands with autocomplete capability)

## Examples

```
devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo); Sig_GenesSAM(data, response = c(1,1,1,2,2,2), fdr.cutoff = 0.4)
```

## Evaluation signals

- The significant-genes output file is populated at the chosen fdr.cutoff (0.4 in the chapter)
- The distance matrix (Fig. 5) shows a clear difference between samples of different response classes
- Samples within the same response class appear similar to each other in the similarity plot
- The companion PCA plot (Fig. 6) visually separates the wild-type and villin-gp130Act response classes, consistent with the class structure of the response vector
- The clustering pattern aligns with the sample response classes (no batch effect present, per Fig. 3 inspection in the chapter)

## Limitations

- The interpretation depends on a correctly specified response vector; mislabeled classes (e.g., wrong order of the 1s and 2s in c(1,1,1,2,2,2)) will make the plot appear inconsistent
- Similarity is computed only on the significant genes from the SAM run at the chosen FDR cutoff; different cutoffs will change the plot
- The chapter does not report version numbers for software dependencies (limma, affy, samr, R, Bioconductor), so reproducibility of the exact plot may vary

## Evidence

- [discussion] Significance testing is carried out using the code for Significance Analysis of Microarrays from the R package: [evidence span withheld — anchor status: not_found]
- [full_text] distance matrix shows class separation: [evidence span withheld — anchor status: not_found]
- [full_text] response vector and fdr.cutoff: [evidence span withheld — anchor status: not_found]
