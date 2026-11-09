---
name: microarray-differential-expression-analysis
description: Use when you have a background-subtracted microarray expression matrix
  (e.g. Illumina Bead array data like E-MTAB-2400 WT vs villin-gp130Act crypt samples)
  and a class vector, and need significant genes plus sample-separation plots.
when_to_use_negative:
- Input is raw, unnormalized array data — run PIMENTo's Preprocess Data command (quantile
  or LOESS normalization, background subtraction) first
- Input is RNA-seq count data rather than microarray intensities — the README shows
  PIMENTo's dependency stack (affy, limma, DESeq2) but this skill's SAM workflow targets
  microarray expression matrices
- Goal is batch-effect assessment rather than between-class significance testing —
  use the preprocessing/dendrogram QC step instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: Provides the SigGenesSAM command that runs SAM testing, exports significant
    genes, and generates the distance-matrix and PCA sample-separation plots
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: samr
  role: R package supplying the underlying Significance Analysis of Microarrays significance-testing
    code invoked by SigGenesSAM
- name: R
  role: Language in which PIMENTo is built and run for this skill
- name: RStudio
  role: Recommended IDE for running the PIMENTo commands with autocomplete
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:918e185327d1363670972bd5900c894e71ba842744d3fcd280643aa37c095d75
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  - samr
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# microarray-differential-expression-analysis

## Summary

Identify genes whose expression differs significantly between response classes in background-subtracted microarray data using PIMENTo's Significance Analysis of Microarrays (SAM) command, which wraps the R 'samr' package and exports significant genes plus Euclidean distance and PCA sample-separation plots.

## When to use

Use when you have a background-subtracted microarray expression matrix (e.g. Illumina Bead array data like E-MTAB-2400 WT vs villin-gp130Act crypt samples) and a class vector, and need significant genes plus sample-separation plots.

## When NOT to use

- Input is raw, unnormalized array data — run PIMENTo's Preprocess Data command (quantile or LOESS normalization, background subtraction) first
- Input is RNA-seq count data rather than microarray intensities — the README shows PIMENTo's dependency stack (affy, limma, DESeq2) but this skill's SAM workflow targets microarray expression matrices
- Goal is batch-effect assessment rather than between-class significance testing — use the preprocessing/dendrogram QC step instead

## Inputs

- Background-subtracted, normalized microarray expression matrix (tab-separated text, e.g. the output of PIMENTo Preprocess Data on example_unnorm.txt-style input)
- Numeric response class vector encoding sample groups (e.g. c(1,1,1,2,2,2) for three WT and three villin-gp130Act samples)
- FDR cutoff for significance testing (e.g. 0.4)

## Outputs

- Tabular file of significant genes at the chosen FDR cutoff
- Euclidean distance similarity plot of samples based on the significant genes (e.g. Fig. 5)
- Principal component analysis (PCA) plot of samples based on the significant genes (e.g. Fig. 6)

## How to apply

Load the background-subtracted expression matrix produced by the upstream PIMENTo Preprocess Data step and define a numeric response class vector encoding the groups (e.g. c(1,1,1,2,2,2) for three WT and three villin-gp130Act samples). Run the PIMENTo SigGenesSAM command, which performs Significance Analysis of Microarrays testing via the R package 'samr', setting an FDR cutoff (the worked example uses fdr.cutoff 0.4) to select genes differing significantly between response classes. Export the significant genes to a tabular output file, then generate the Euclidean distance similarity plot and the principal component analysis (PCA) plot computed on the significant genes. Judge success by checking that the significant-genes file is populated at the chosen FDR cutoff and that the distance matrix and PCA plots visually separate the response classes while samples within a class cluster together, matching the response-vector class structure.

## Related tools

- **PIMENTo** (Provides the SigGenesSAM command that runs SAM testing, exports significant genes, and generates the distance-matrix and PCA sample-separation plots) — https://www.github.com/MUSC-CGM/PIMENTo
- **samr** (R package supplying the underlying Significance Analysis of Microarrays significance-testing code invoked by SigGenesSAM)
- **R** (Language in which PIMENTo is built and run for this skill)
- **RStudio** (Recommended IDE for running the PIMENTo commands with autocomplete)

## Examples

```
library(PIMENTo); Sig_GenesSAM(background_subtracted, response = c(1,1,1,2,2,2), fdr.cutoff = 0.4)
```

## Evaluation signals

- The significant-genes output file is populated at the chosen FDR cutoff (e.g. 0.4)
- The Euclidean distance matrix (Fig. 5) shows a clear difference between samples of different response classes and similarity within the same class
- The PCA plot (Fig. 6) visually separates the wild-type and villin-gp130Act classes consistent with the response vector c(1,1,1,2,2,2)
- Clustering/PCA clusters align with the sample response classes, i.e. no batch effect drives the separation

## Limitations

- The chapter uses an unusually permissive FDR cutoff (0.4) in the worked example; significance stringency should be re-evaluated for other datasets
- No version numbers are stated for the software dependencies (limma, affy, samr, R, Bioconductor), so exact reproducibility of the testing behavior is not guaranteed
- The samr citation in the article is an incomplete placeholder '[ref]', so provenance of the significance-testing implementation must be traced through PIMENTo itself

## Evidence

- [discussion] Significance testing is carried out using the code for Significance Analysis of Microarrays from the R package “samr”: [evidence span withheld — anchor status: not_found]
