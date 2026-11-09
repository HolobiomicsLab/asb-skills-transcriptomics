---
name: manuscript-result-reproduction-scripting
description: Use when a task asks to verify or reproduce specific reported numerical
  results (e.g. cluster-purity scores) from a paper whose package and reproduction
  scripts are publicly available on GitHub.
when_to_use_negative:
- The task targets a different algorithm's cluster quality (e.g. silhouette, DendroSplit,
  distance ratio), which the paper states are 'not comparable among datasets and have
  poor interpretability of cluster purity'
- The input is a continuous-differentiation or low-quality dataset where the recommended
  0.9 purity threshold is not appropriate and the threshold must be derived from global
  ROGUE values
- The goal is to assess purity of integrated cell populations from different protocols
  and platforms, which the paper lists as not yet supported (future work)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: ROGUE
  role: Entropy-based statistic (R package) whose rogue()/CalculateRogue() functions
    compute the cluster-purity values being reproduced
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency R package that must be installed before installing ROGUE
- name: devtools
  role: Used to install the ROGUE package from GitHub via devtools::install_github()
- name: R
  role: Runtime environment for executing the reproduction workflow and writing rogue_values.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:586a8310beb443a8d314af031f7cd2ffe2a9cd81eacd474de7d7a232340eb28d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# manuscript-result-reproduction-scripting

## Summary

Reproduces the quantitative values reported in a published manuscript by installing the authors' package, running their released scripts on the referenced datasets (here ROGUE purity scores on ERCC spike-in, FACS-purified PBMC, and colorectal-cancer T-cell data), and comparing computed values against reported ones in a result table.

## When to use

Use when a task asks to verify or reproduce specific reported numerical results (e.g. cluster-purity scores) from a paper whose package and reproduction scripts are publicly available on GitHub.

## When NOT to use

- The task targets a different algorithm's cluster quality (e.g. silhouette, DendroSplit, distance ratio), which the paper states are 'not comparable among datasets and have poor interpretability of cluster purity'
- The input is a continuous-differentiation or low-quality dataset where the recommended 0.9 purity threshold is not appropriate and the threshold must be derived from global ROGUE values
- The goal is to assess purity of integrated cell populations from different protocols and platforms, which the paper lists as not yet supported (future work)

## Inputs

- ERCC spike-in droplet expression matrix (GSE118127)
- FACS-purified PBMC population expression matrices (CD4 naive, CD8 naive, CD14 monocytes, CD34+ cells)
- Colorectal-cancer T-cell expression matrix with cluster labels for six well-defined T-cell subtypes plus the complete unsplit T-cell dataset
- Authors' reproduction scripts from the PaulingLiu/ROGUE GitHub repository ./scripts directory

## Outputs

- Table of computed ROGUE values per dataset, population/subtype (rogue_values.csv)
- Side-by-side comparison of computed versus manuscript-reported ROGUE values

## How to apply

Install the dependency 'tidyverse', then install the authors' package from GitHub with devtools::install_github("PaulingLiu/ROGUE"). Follow the paper's reproduction path: run rogue() with platform = 'droplet' on the ERCC spike-in droplet matrix (GSE118127), expecting ROGUE ~1 for the ERCC-only control; compute per-population ROGUE on FACS-purified PBMC matrices, expecting CD4/CD8 naive T cells at 0.94-1 and CD14 monocytes / CD34+ cells at ~0.8; and use CalculateRogue() with provided cluster labels on the colorectal-cancer T-cell data, expecting each of six well-defined T-cell subtypes at ROGUE >0.9 versus 0.78 for the complete unsplit dataset. Use the package's helper steps (matr.filter, entropy_fit, DetermineK, CalculateRogue) as exposed in the methods, and assemble dataset, population/subtype, and ROGUE columns into a comparison table (rogue_values.csv) against manuscript-reported values. The rationale is that the authors released the exact scripts used for all quantitative results, so running them on the same input matrices provides a direct reproduction check.

## Related tools

- **ROGUE** (Entropy-based statistic (R package) whose rogue()/CalculateRogue() functions compute the cluster-purity values being reproduced) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency R package that must be installed before installing ROGUE)
- **devtools** (Used to install the ROGUE package from GitHub via devtools::install_github())
- **R** (Runtime environment for executing the reproduction workflow and writing rogue_values.csv)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- ERCC-only droplet control yields a ROGUE of approximately 1 when run with platform = 'droplet'
- FACS-purified CD4/CD8 naive T-cell populations yield ROGUE values in the 0.94-1 range; CD14 monocytes and CD34+ cells fall at ~0.8
- All six colorectal-cancer T-cell subtypes achieve ROGUE >0.9, and the complete unsplit T-cell dataset yields ~0.78
- Computed ROGUE values in rogue_values.csv match the manuscript-reported values within tolerance
- Reproduction uses the authors' released scripts from the repository's ./scripts directory rather than re-implemented logic

## Limitations

- The 0.9 purity threshold 'may not be suitable for low-quality or continuous data, where it must be determined from global ROGUE values'
- Current ROGUE 'does not yet assess purity of integrated cell populations from different protocols and platforms' (proposed as future work)
- ROGUE must be run with platform-appropriate settings (droplet vs. UMI); the workflow specifies UMI-appropriate settings for the FACS-sorted PBMC populations

## Evidence

- [methods] Reproduce reported ROGUE purity scores: [evidence span withheld — anchor status: not_found]
- [methods] installation steps: "Before installing ROGUE, the “tidyverse” package should be installed first"
- [methods] reproduction scripts location: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [intro] core statistic definition: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
- [readme] package usage entry point: "For more details and basic usage see following tutorials:
1.	[Guided Tutorial]"
