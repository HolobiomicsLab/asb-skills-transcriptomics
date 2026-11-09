---
name: rmarkdown-reproducible-analysis-execution
description: Use when running the SkewC R Markdowns in the folder's listed order,
  especially 3-SkewC_TrimClustering.Rmd on Coverage_means_DF_Clust to partition cells
  into typical and skewed groups.
when_to_use_negative:
- Input is not gene body coverage data processed through the earlier SkewC steps (0_split10XbyBarcode.sh,
  1_geneBodyCoverage.sh, 2_SkewC.sh), since the R Markdowns must run in listed order
- The computing environment lacks git and at least one of docker, udocker, or singularity,
  which SkewC requires
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SkewC
  role: Pipeline whose R Markdowns (including 3-SkewC_TrimClustering.Rmd) implement
    the reproducible analysis execution
  repo: https://github.com/LSBDT/SkewC.git
- name: tclust
  role: R function invoked by SkewC_TrimClustering.Rmd to perform the trim clustering
    on Coverage_means_DF_Clust
- name: R
  role: Language in which the SkewC R Markdowns and the tclust trim clustering are
    executed
- name: docker
  role: Container runtime used to reproduce the SkewC workflow environment
- name: udocker
  role: User-space alternative to docker for running the SkewC container without root
    privileges
  repo: https://github.com/indigo-dc/udocker
- name: singularity
  role: Recommended container runtime; the SkewC docker image is converted to a Singularity
    Image File (SIF)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:2e3e864f2209a25bd6632900c0181a12238b4dc2a97d0b8bcbff416f40337cc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  - docker
  - singularity
  - tclust
  - udocker
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# rmarkdown-reproducible-analysis-execution

## Summary

Execute the ordered R Markdown scripts of the SkewC pipeline (e.g. 3-SkewC_TrimClustering.Rmd) so that trim clustering of gene body coverage runs reproducibly with a user-supplied alpha and produces deterministic typical/skewed cell lists.

## When to use

Use when running the SkewC R Markdowns in the folder's listed order, especially 3-SkewC_TrimClustering.Rmd on Coverage_means_DF_Clust to partition cells into typical and skewed groups.

## When NOT to use

- Input is not gene body coverage data processed through the earlier SkewC steps (0_split10XbyBarcode.sh, 1_geneBodyCoverage.sh, 2_SkewC.sh), since the R Markdowns must run in listed order
- The computing environment lacks git and at least one of docker, udocker, or singularity, which SkewC requires

## Inputs

- Coverage_means_DF_Clust data frame derived from gene body coverage results (coverage.r)
- alpha trimming level, numeric in [0..1]
- TestData/coverage.r sample coverage data

## Outputs

- TypicalCellsID.tsv (list of typical cells)
- SkewedCellsID.tsv (list of skewed cells)
- CLUSTResult clustering plot as PDF

## How to apply

Clone the LSBDT/SkewC repository and run its R Markdowns in the same order as listed in the folder (1-SkewC_Create_Coverage_Matrixes.Rmd, 2-SkewC_Plot_Gene_Body_Coverage.Rmd, 3-SkewC_TrimClustering.Rmd, 4-SkewC_Plot_Typical_Skewed_Coverage.Rmd), since each step's output feeds the next. For the trim clustering step, load the input data frame Coverage_means_DF_Clust derived from the gene body coverage results stored in coverage.r, and supply an alpha value constrained to [0..1] (for large 10X Genomics datasets such as pbmc_8k, start with alpha = 0.099 and evaluate the result). The R Markdown calls the R tclust function to perform the trim clustering. Outputs are TypicalCellsID.tsv and SkewedCellsID.tsv plus a CLUSTResult clustering plot (PDF). Validate that the two .tsv files together partition all cells in Coverage_means_DF_Clust with no cell assigned to both lists; errors such as 'No alpha value computation found' indicate a missing user-supplied alpha.

## Related tools

- **SkewC** (Pipeline whose R Markdowns (including 3-SkewC_TrimClustering.Rmd) implement the reproducible analysis execution) — https://github.com/LSBDT/SkewC.git
- **tclust** (R function invoked by SkewC_TrimClustering.Rmd to perform the trim clustering on Coverage_means_DF_Clust)
- **R** (Language in which the SkewC R Markdowns and the tclust trim clustering are executed)
- **docker** (Container runtime used to reproduce the SkewC workflow environment)
- **udocker** (User-space alternative to docker for running the SkewC container without root privileges) — https://github.com/indigo-dc/udocker
- **singularity** (Recommended container runtime; the SkewC docker image is converted to a Singularity Image File (SIF))

## Examples

```
Rscript -e "rmarkdown::render('RMarkdown/3-SkewC_TrimClustering.Rmd', params=list(alpha=0.099))"
```

## Evaluation signals

- TypicalCellsID.tsv and SkewedCellsID.tsv together partition all cells present in Coverage_means_DF_Clust
- No cell appears in both TypicalCellsID.tsv and SkewedCellsID.tsv
- A CLUSTResult clustering plot PDF is generated
- No 'No alpha value computation found' error appears, confirming a valid alpha in [0..1] was supplied
- The R Markdowns ran in the same order as listed in the folder, so each step's expected input exists

## Limitations

- A missing or uncomputable alpha value causes the clustering to fail with 'No alpha value computation found'
- An R package error (stringi/libicui18n.so.57) terminates SkewC and is not fixable from the user's side
- git and at least one of docker, udocker, or singularity must be installed, otherwise cloning/running fails
- For large 10X Genomics datasets the alpha must be tuned empirically (e.g. start with alpha = 0.099 and evaluate)

## Evidence

- [intro] Run the R Markdowns in the same order as listed in the folder: "This folder contains several R Markdowns and should be run in the same order as listed below:"
- [abstract] tclust trim clustering performed by SkewC_TrimClustering.Rmd: "The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function."
- [methods] alpha supplied to 2_SkewC.sh: "bash 2_SkewC.sh $prjname $indir $outdir $alpha"
- [discussion] missing alpha failure mode: [evidence span withheld — anchor status: not_found]
- [readme] SkewC container dependencies: "git clone https://github.com/LSBDT/SkewC.git"
