---
name: hierarchical-clustering-sample-class-visualization
description: Use when a ranked significant-gene list (e.g. top 50/100/150 from samr
  SAM output) is available alongside a quantile-normalized expression matrix and you
  need to visualize whether samples cluster by phenotype class.
when_to_use_negative:
- The expression matrix has not yet been normalized — run PIMENTo's Preprocess Data
  command (quantile or LOESS) first, since quantile normalization was judged better
  for this data set.
- No ranked significant-gene list exists yet — significance testing must first be
  carried out with the samr package code before heatmap generation.
- Batch effects are suspected in the samples — the dendrogram alignment with response
  classes was validated only after confirming no batch effect was present.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: Provides the Create Heatmap command that generates the sample-clustering heatmap
    and fold-change tables from SAM output and the normalized expression matrix
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Language in which PIMENTo and the Create Heatmap command are run; built with
    the R programming language and open-source packages from CRAN and Bioconductor
- name: RStudio
  role: IDE used to run the PIMENTo heatmap commands with autocomplete capability
- name: Bioconductor
  role: Source of the open-source packages (limma, affy, samr) that PIMENTo incorporates
    for normalization and significance testing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:d721c76d4f151f75b7cb3c4eb22ba8c6d199df576e9685db2c4b22d6654cde93
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bioconductor
  - PIMENTo
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hierarchical-clustering-sample-class-visualization

## Summary

Generate sample-clustering heatmaps of SAM-ranked significant genes using PIMENTo's Create Heatmap command, so that the column dendrogram reveals whether wild-type and villin-gp130Act microarray samples separate by response class.

## When to use

Use when a ranked significant-gene list (e.g. top 50/100/150 from samr SAM output) is available alongside a quantile-normalized expression matrix and you need to visualize whether samples cluster by phenotype class.

## When NOT to use

- The expression matrix has not yet been normalized — run PIMENTo's Preprocess Data command (quantile or LOESS) first, since quantile normalization was judged better for this data set.
- No ranked significant-gene list exists yet — significance testing must first be carried out with the samr package code before heatmap generation.
- Batch effects are suspected in the samples — the dendrogram alignment with response classes was validated only after confirming no batch effect was present.

## Inputs

- SAM significance output from the samr-based Significance Analysis of Microarrays testing step, ranked by significance score
- Gene-list text files (e.g. 'heatmap_input/top 50.txt', 'heatmap_input/top 100.txt', 'heatmap_input/top 150.txt')
- Quantile-normalized expression matrix forwarded from the PIMENTo Preprocess Data step

## Outputs

- Heatmap of top-ranked genes (e.g. top-50 heatmap, Fig. 8) with samples clustering by response class, exported as TIFF, PDF, Postscript, and FIG into heatmap_output
- Per-gene fold-change tables written as CSV files into heatmap_output

## How to apply

Load the SAM significance output from the upstream samr-based Significance Analysis of Microarrays step and rank genes by significance score. Write the top-ranked gene lists to text files (e.g. 'heatmap_input/top 50.txt', 'heatmap_input/top 100.txt', 'heatmap_input/top 150.txt') and run PIMENTo's Create Heatmap command in R (via RStudio) on the SAM output together with the quantile-normalized expression matrix forwarded from the Preprocess Data step — quantile normalization was the better method for this data set. Export the heatmap (Fig. 8 shows the top-50-gene heatmap) in TIFF, PDF, Postscript, and FIG formats into the heatmap_output subdirectory, and write per-gene fold-change tables as CSVs to the same directory. Judge success by inspecting the column dendrogram: in the validated run, samples aligned with response classes (WT vs. villin-gp130Act) rather than batch.

## Related tools

- **PIMENTo** (Provides the Create Heatmap command that generates the sample-clustering heatmap and fold-change tables from SAM output and the normalized expression matrix) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Language in which PIMENTo and the Create Heatmap command are run; built with the R programming language and open-source packages from CRAN and Bioconductor)
- **RStudio** (IDE used to run the PIMENTo heatmap commands with autocomplete capability)
- **Bioconductor** (Source of the open-source packages (limma, affy, samr) that PIMENTo incorporates for normalization and significance testing)

## Examples

```
devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo); # run Create Heatmap on SAM output + quantile-normalized matrix via 'heatmap_input/top 50.txt'
```

## Evaluation signals

- heatmap_output contains the top-50 heatmap in all four graphics formats (TIFF, PDF, Postscript, FIG) plus the fold-change CSV files
- The column dendrogram in the top-50 heatmap separates samples according to response class (WT vs. villin-gp130Act)
- Clusters align with sample response classes, confirming no batch effect is present (as in the article's Fig. 3 validation)
- The heatmap covers exactly the genes in the supplied gene-list input file (top 50/100/150 ranked by SAM significance score)

## Limitations

- No version numbers are stated for any software dependency (limma, affy, samr, R, Bioconductor), so reproducibility of exact package versions is not guaranteed.
- The samr package citation is an incomplete placeholder ('[ref]') in the article.
- The wild-type and villin-gp130Act small intestinal crypt expression profiles analyzed are described in Taniguichi et al. [16] without an accession or hosting location given in the discussion.
- No R Shiny user interface is available yet; PIMENTo is run as R commands via RStudio.

## Evidence

- [methods] one can see that quantile normalization is the better of the two normalization methods for this data set, as the data appears linear on the MA scale.: [evidence span withheld — anchor status: not_found]
- [discussion] Significance testing is carried out using the code for Significance Analysis of Microarrays from the R package “samr”: [evidence span withheld — anchor status: not_found]
- [methods] PIMENTo generates a dendrogram plot of all chosen samples.: [evidence span withheld — anchor status: not_found]
