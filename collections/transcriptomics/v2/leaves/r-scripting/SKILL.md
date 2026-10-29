---
name: r-scripting
description: Use when a transcriptomic cell-cell communication analysis (e.g., ICELLNET
  scoring, gene-expression scaling, subsampling robustness) must be implemented or
  reproduced as R code in an R environment.
when_to_use_negative:
- The analysis is not implementable in R or requires tooling outside the R/ICELLNET
  ecosystem (ICELLNET and all listed dependencies are R packages)
- Only a single precomputed communication score is needed with no scripting, resampling,
  or customization required
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ICELLNET
  role: R package providing the curated ligand-receptor database and icellnet score
    computation invoked from scripts
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Scripting language and runtime for the whole pipeline; version determines
    ICELLNET version choice (v2.0.0 for R < 4.1, v2.2.x for newer R)
- name: devtools
  role: Used to install the ICELLNET package from GitHub via install_github
- name: ggplot2
  role: Used to plot results such as SD of communication scores versus subsampling
    percentage
- name: readxl
  role: CRAN dependency installed for reading input data into R
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:6b306782aeb18b3d4581a908c2f2cc0b26a5ede3e63b3bd948b9bf98f1bfe79e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ICELLNET
  - R
  - devtools
  - ggplot2
  - readxl
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# r-scripting

## Summary

Implement cell-cell communication analyses such as ICELLNET scaling, scoring, and subsampling robustness tests as R scripts, using the ICELLNET package installed from GitHub and its CRAN/Bioconductor dependencies.

## When to use

Use when a transcriptomic cell-cell communication analysis (e.g., ICELLNET scoring, gene-expression scaling, subsampling robustness) must be implemented or reproduced as R code in an R environment.

## When NOT to use

- The analysis is not implementable in R or requires tooling outside the R/ICELLNET ecosystem (ICELLNET and all listed dependencies are R packages)
- Only a single precomputed communication score is needed with no scripting, resampling, or customization required

## Inputs

- Gene expression matrix in R (rows = genes, columns = cell populations) from scRNAseq or bulk transcriptomic data
- Cluster mean expression profiles (e.g., CM3, CT0a, CT3b)
- ICELLNET curated ligand-receptor interactions database (1669 interactions)

## Outputs

- Scaled expression matrices written to CSV (e.g., scaled_expression.csv)
- Communication score tables (e.g., subsampling_sd_results.csv)
- ggplot2 plots of score variability or communication scores

## How to apply

Set up the R environment by installing CRAN dependencies (devtools, jetset, readxl, psych, GGally, gplots, ggplot2, RColorBrewer, data.table, grid, gridExtra, ggthemes, scales, rlist) and Bioconductor packages (BiocGenerics, org.Hs.eg.db, hgu133plus2.db, annotate), then install ICELLNET via install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet"). Respect version constraints: ICELLNET v2.0.0 requires R < 4.1; for R >= 4.1.x use ICELLNET v2.2.x. Write scripts that load gene-expression matrices (rows = genes, columns = cell populations), apply paper-specific operations such as dividing each gene's expression by its maximum (mean of the 5% highest values) times 10 with a cap at 10, compute communication scores with the icellnet score implementation, and use ggplot2 for visualization. Persist intermediate and final results as CSV files (e.g., scaled_expression.csv, subsampling_sd_results.csv) with fixed random seeds for reproducibility.

## Related tools

- **ICELLNET** (R package providing the curated ligand-receptor database and icellnet score computation invoked from scripts) — github.com/soumelis-lab/ICELLNET
- **R** (Scripting language and runtime for the whole pipeline; version determines ICELLNET version choice (v2.0.0 for R < 4.1, v2.2.x for newer R))
- **devtools** (Used to install the ICELLNET package from GitHub via install_github)
- **ggplot2** (Used to plot results such as SD of communication scores versus subsampling percentage)
- **readxl** (CRAN dependency installed for reading input data into R)

## Examples

```
install.packages(c("devtools","ggplot2","data.table")); library(devtools); install_github("soumelis-lab/ICELLNET",ref="master",subdir="icellnet")
```

## Evaluation signals

- All CRAN dependencies (devtools, jetset, readxl, psych, GGally, gplots, ggplot2, RColorBrewer, data.table, gridExtra, ggthemes, scales, rlist) and Bioconductor packages install and load without error
- install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet") succeeds and library(icellnet) loads
- R/ICELLNET version pairing is consistent: ICELLNET v2.0.0 with R < 4.1, or v2.2.x with R >= 4.1.x
- Output CSV files exist and validation checks pass (e.g., scaled values non-negative and capped at 10; SD at 10% subsampling ~146 for CT0a and ~215 for CT3b versus ~37 and ~59 at 90%)
- Random subsampling uses a fixed random seed so results are reproducible across runs

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; newer R versions need ICELLNET v2.2.x
- The ICELLNET database contains a relatively small number of interactions compared to other databases, and chemokine and checkpoint interactions are not yet included
- Receptors without known ligands (DR6/TNFRSF21, RELT, TROY, NGFR) and ligands without known receptors (IL17D) remain unpaired

## Evidence

- [readme] install.packages(c("devtools", "jetset", "readxl", "psych", "GGally", "gplots", "ggplot2", "RColorBrewer", "data.table", "grid", "gridExtra", "ggthemes", "scales","rlist")): "install.packages(c("devtools", "jetset", "readxl", "psych", "GGally", "gplots", "ggplot2", "RColorBrewer", "data.table", "grid", "gridExtra", "ggthemes", "scales","rlist"))"
- [methods] install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet"): "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [methods] ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x.: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
- [other] Divide each gene's expression values by this computed maximum and multiply by 10 so the reference gene reaches a scaled value of 10.: [evidence span withheld — anchor status: not_found]
- [other] Plot the SD of the score as a function of subsampling percentage for CM3/CT0a/CT3b using ggplot2.: [evidence span withheld — anchor status: not_found]
