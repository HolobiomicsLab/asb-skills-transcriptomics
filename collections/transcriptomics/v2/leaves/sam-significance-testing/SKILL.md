---
name: sam-significance-testing
description: Use when you have a background-subtracted microarray expression matrix
  with a class-encoded response vector (e.g. WT vs. mutant samples) and need to call
  differentially expressed genes at a specified FDR cutoff.
when_to_use_negative:
- Input is raw, non-normalized microarray data that has not yet passed through PIMENTo's
  Preprocess Data and background-subtraction steps
- The analysis goal is normalization or batch-effect QC (quantile/LOESS normalization,
  dendrogram generation) rather than class-differential significance testing
- Samples lack a defined two-class response structure, so the SAM response vector
  cannot be specified
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: PIMENTo
  role: Provides the Sig_GenesSAM command that runs SAM testing and generates the
    distance-matrix and PCA plots from significant genes
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: samr
  role: R package supplying the Significance Analysis of Microarrays code used under
    the hood for significance testing
- name: R
  role: Programming language in which PIMENTo and the samr-based significance testing
    are executed
- name: RStudio
  role: IDE for running the PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:017cefdf1b6e46ea322860c52cfe58eb9998610f58207d3c85a130935c9ad00e
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

# sam-significance-testing

## Summary

Run Significance Analysis of Microarrays (SAM) inside PIMENTo via the R 'samr' package to identify genes whose expression differs between response classes in background-subtracted microarray data, then visualize separation with a Euclidean distance matrix and PCA of the significant genes.

## When to use

Use when you have a background-subtracted microarray expression matrix with a class-encoded response vector (e.g. WT vs. mutant samples) and need to call differentially expressed genes at a specified FDR cutoff.

## When NOT to use

- Input is raw, non-normalized microarray data that has not yet passed through PIMENTo's Preprocess Data and background-subtraction steps
- The analysis goal is normalization or batch-effect QC (quantile/LOESS normalization, dendrogram generation) rather than class-differential significance testing
- Samples lack a defined two-class response structure, so the SAM response vector cannot be specified

## Inputs

- Background-subtracted microarray expression matrix from PIMENTo preprocessing (e.g. from the E-MTAB-2400 mouse intestinal crypt dataset)
- Response class vector encoding group membership, e.g. c(1, 1, 1, 2, 2, 2) for three WT (class 1) and three villin-gp130Act (class 2) samples
- FDR cutoff value (e.g. 0.4)

## Outputs

- Tabular significant-genes output file at the chosen FDR cutoff
- Euclidean distance similarity matrix/plot of samples based on the significant genes
- PCA plot of samples based on the significant genes

## How to apply

Load the background-subtracted expression matrix produced by the upstream PIMENTo preprocessing step (for the E-MTAB-2400 mouse intestinal crypt dataset, three WT and three villin-gp130Act samples). Define the response class vector encoding the two groups, e.g. c(1,1,1,2,2,2). Run PIMENTo's Sig_GenesSAM command, which wraps SAM testing from the R 'samr' package, setting fdr.cutoff (e.g. 0.4 in the paper's example) to select genes differing significantly between response classes, and export the significant genes to a tabular output file. Then generate a Euclidean distance similarity plot and a principal component analysis (PCA) plot of samples based on the significant genes to confirm class separation. Validate that the significant-genes file is populated at the chosen FDR and that the similarity and PCA plots visually separate the two response classes while samples within a class cluster together.

## Related tools

- **PIMENTo** (Provides the Sig_GenesSAM command that runs SAM testing and generates the distance-matrix and PCA plots from significant genes) — https://www.github.com/MUSC-CGM/PIMENTo
- **samr** (R package supplying the Significance Analysis of Microarrays code used under the hood for significance testing)
- **R** (Programming language in which PIMENTo and the samr-based significance testing are executed)
- **RStudio** (IDE for running the PIMENTo commands with autocomplete capability)

## Examples

```
library(PIMENTo); SigGenesSAM(data = bgsub_matrix, response = c(1, 1, 1, 2, 2, 2), fdr.cutoff = 0.4)
```

## Evaluation signals

- The significant-genes output file is populated (non-empty) at the chosen FDR cutoff (0.4 in the paper's example)
- The Euclidean distance matrix (Fig. 5) shows clear differences between samples of different response classes and similarity within the same class
- The PCA plot (Fig. 6) visually separates the wild-type and villin-gp130Act response classes
- Sample clustering in the distance/PCA outputs is consistent with the class structure of the response vector c(1,1,1,2,2,2)
- The tabular output contains genes whose expression differs significantly between the two response classes

## Limitations

- FDR cutoff of 0.4 used in the paper's example is lenient; the significant-gene set depends on this threshold
- No version numbers are stated for any software dependency (limma, affy, samr, R, Bioconductor)
- The citation placeholder '[ref]' for the samr package is incomplete in the source article
- The analysis dataset (wild-type and villin-gp130Act small intestinal crypt expression profiles) is listed under accession E-MEXP-E-MTAB-2400 in Array Express, but the article notes no hosting/accession details in the discussion

## Evidence

- [discussion] SAM significance testing wrapped by PIMENTo: [evidence span withheld — anchor status: not_found]
