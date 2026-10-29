---
name: r-bioconductor-dependency-installation
description: Use when setting up an R environment to run CellRouter or reproduce its
  analyses and the required packages (reshape, reshape2, pheatmap, tsne, igraph, ggplot2,
  mclust, grid, Rtsne, cccd) are not yet installed.
when_to_use_negative:
- The working environment is Windows, since the README states CellRouter was not tested
  in Windows
- A CellRouter R package is already installed (the README notes an R package was in
  development, which would supersede manual package-list installation)
- The target analysis uses a different pipeline whose dependency list differs from
  CellRouter's CRAN/Bioconductor package set
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: Target platform whose class must be sourced after dependencies are installed;
    requires libdir set to its Java libraries
  repo: https://github.com/edroaldo/cellrouter
- name: R
  role: Runtime environment; packages are installed via install.packages with CRAN
    and Bioconductor (biocLite) repositories
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:e395d4273e0c8ecce1efc74fd9668f60ba0d03ad829f1e60425c363179e22279
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Monocle 1
  - Monocle 2
  - R
  - Splatter
  - Waterfall
  - Wishbone
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# r-bioconductor-dependency-installation

## Summary

Installs the R/CRAN/Bioconductor package stack required to run CellRouter, then sources the CellRouter class with the correct Java library directory. This environment setup is the prerequisite for reproducing any CellRouter analysis, including the Splatter synthetic-data benchmark against Monocle, DPT, Wishbone, and Waterfall.

## When to use

Use when setting up an R environment to run CellRouter or reproduce its analyses and the required packages (reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd) are not yet installed.

## When NOT to use

- The working environment is Windows, since the README states CellRouter was not tested in Windows
- A CellRouter R package is already installed (the README notes an R package was in development, which would supersede manual package-list installation)
- The target analysis uses a different pipeline whose dependency list differs from CellRouter's CRAN/Bioconductor package set

## Inputs

- Cloned/downloaded edroaldo/cellrouter repository (including CellRouter_Class.R and the Java libraries folder)
- Local R installation on a Unix-based system or Mac

## Outputs

- R environment with reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd, and Vennerable installed
- Sourced CellRouter class with libdir correctly pointing to the Java libraries, ready for trajectory analysis

## How to apply

Clone or download the edroaldo/cellrouter repository, then install the package list ('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd') by checking installed.packages() and calling install.packages on the missing ones with the CRAN mirror (http://cran.rstudio.com/) and the Bioconductor biocLite repository. If any package is still missing after this pass, install it explicitly via source('http://bioconductor.org/biocLite.R') followed by biocLite('package_name'). Also install Vennerable from R-Forge (install.packages('Vennerable', repos='http://R-Forge.R-project.org')). Finally, source the CellRouter class file (CellRouter_Class.R) and set the libdir variable to the path of the Java libraries inside the CellRouter folder; an incorrect libdir will break the class. Note CellRouter was tested only on Unix-based systems and Mac, not Windows.

## Related tools

- **CellRouter** (Target platform whose class must be sourced after dependencies are installed; requires libdir set to its Java libraries) — https://github.com/edroaldo/cellrouter
- **R** (Runtime environment; packages are installed via install.packages with CRAN and Bioconductor (biocLite) repositories)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- All packages in the list ('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd') appear in installed.packages()[,'Package'] after the install pass
- Vennerable installs successfully from http://R-Forge.R-project.org
- source('path/to/CellRouter_Class.R') completes without error and libdir points to a directory containing the Java libraries inside the CellRouter folder
- Downstream CellRouter calls (e.g., subpopulation detection with community detection on simulated Splatter data) execute without 'package not found' or Java library errors

## Limitations

- CellRouter was tested only in Unix-based systems and Mac; it was not tested in Windows
- If a package remains missing after the combined CRAN/Bioconductor install pass, it must be installed individually via biocLite
- The libdir variable must be manually corrected to the Java libraries path inside the CellRouter folder, or the sourced class will fail

## Evidence

- [readme] CRAN package list: "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')"
- [readme] biocLite fallback: [evidence span withheld — anchor status: not_found]
- [readme] Vennerable R-Forge install: [evidence span withheld — anchor status: not_found]
- [methods] libdir requirement: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [readme] platform restriction: "CellRouter was tested in Unix-based systems. Therefore, we strongly recommend to use CellRouter in Unix-based systems or Mac. CellRouter was not tested in Windows."
