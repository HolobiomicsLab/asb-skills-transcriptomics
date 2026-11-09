---
name: replicate-aware-da-statistics
description: Use when performing differential abundance testing on ELVAR-enriched
  cell communities where biological inter-subject variability must be modeled via
  replicates, e.g. regressing naive/mature cell counts on age in Tabula Muris Senis
  lung Cd4+ T-cells.
when_to_use_negative:
- When each condition lacks genuine biological replicates (e.g. only one mouse per
  age-group) — the negative binomial regression replicate structure in DoDA requires
  replicate-level variance estimation
- When communities have not been tested for attribute enrichment via ProcessEVA first
  — running DoDA on arbitrary Louvain clusters loses the ELVAR sensitivity advantage
  over the Louvain benchmark
- When the analysis goal is differential expression rather than differential abundance
  of cell-type/community counts
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ELVAR
  role: Provides the DoDA function that fits negative binomial regressions of the
    DA variable (varDA='naive') on age with replicate structure (varREP='mouse') over
    ProcessEVA-significant communities
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Holds the lung Cd4+ T-cell count matrix and metadata (age, naive status, mouse
    replicate) passed to DoDA
- name: MASS
  role: Supplies the negative binomial regression machinery underlying the DA test
    statistics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:34306a94cca0bfac9c601e15bafdd641f3275887c619bd3f9604759721a3e777
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - ggplot2
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# replicate-aware-da-statistics

## Summary

Fits negative binomial regressions (MASS::glm.nb) of cell-type counts (e.g. naive vs mature Cd4+ T-cells) on a biological variable such as age, using biological replicates (e.g. mouse) as the replicate structure, to produce Wald and likelihood-ratio test statistics for differential abundance in ELVAR.

## When to use

Use when performing differential abundance testing on ELVAR-enriched cell communities where biological inter-subject variability must be modeled via replicates, e.g. regressing naive/mature cell counts on age in Tabula Muris Senis lung Cd4+ T-cells.

## When NOT to use

- When each condition lacks genuine biological replicates (e.g. only one mouse per age-group) — the negative binomial regression replicate structure in DoDA requires replicate-level variance estimation
- When communities have not been tested for attribute enrichment via ProcessEVA first — running DoDA on arbitrary Louvain clusters loses the ELVAR sensitivity advantage over the Louvain benchmark
- When the analysis goal is differential expression rather than differential abundance of cell-type/community counts

## Inputs

- Seurat object of ELVAR-enriched cells (e.g. 537-cell lung Cd4+ T-cell count matrix from lungCD4Tinfo) with metadata columns for the DA variable (naive/mature status from Lef1), the condition (age coded 0/1/2/3/5 for 1m/3m/18m/21m/30m), and the replicate variable (mouse)
- Significant-cluster object (sigcl.o) returned by ProcessEVA, containing cells from Bonferroni-significant age-enriched communities
- ELVAR R-package (aet21/ELVAR) and MASS for negative binomial regression

## Outputs

- Wald and likelihood-ratio (LRT) test statistics from negative binomial regressions of naive/mature counts on age (saved e.g. as elvar_da_statistics.csv)
- Differential abundance P-values demonstrating significant age-related naive-to-mature Cd4+ T-cell shift, comparable to Fig. 3g

## How to apply

After Eva_partitions/ProcessEVA yield Bonferroni-significant age-enriched communities (with two mouse replicates confirmed per age-group), run the ELVAR DoDA function on the Seurat object and significant-cluster object with varDA set to the cell-status variable (e.g. 'naive') and varREP set to the replicate variable (e.g. 'mouse'). DoDA fits negative binomial regressions (MASS) of the DA variable on age using mouse replicates as the replicate structure, which guards against confounding by biological inter-subject variability. Save the Wald and LRT statistics (e.g. to elvar_da_statistics.csv) for comparison against Louvain-based benchmarks (LVnonseq). Correct application reproduces Fig. 3g: a highly significant decrease of naive (CS1) and increase of mature/activated (CS0) Cd4+ T-cell fractions with age, stronger than the Louvain benchmark.

## Related tools

- **ELVAR** (Provides the DoDA function that fits negative binomial regressions of the DA variable (varDA='naive') on age with replicate structure (varREP='mouse') over ProcessEVA-significant communities) — https://github.com/aet21/ELVAR
- **Seurat** (Holds the lung Cd4+ T-cell count matrix and metadata (age, naive status, mouse replicate) passed to DoDA)
- **MASS** (Supplies the negative binomial regression machinery underlying the DA test statistics)

## Examples

```
da.o <- DoDA(seu.o, sigcl.o, varDA="naive", varREP="mouse")
```

## Evaluation signals

- Wald and LRT statistics show a significant decrease of the naive (CS1) fraction with age and a significant increase of the mature/activated (CS0) fraction, reproducing Fig. 3g
- P-values are highly significant and robust across the 100 Eva_partitions runs at alpha=0.8
- Each age-group in sigcl.o is supported by two mouse replicates before DoDA is run
- Significance is stronger than the equivalent regression on Louvain-analog (LVnonseq) communities, confirming the ELVAR advantage
- Output CSV (elvar_da_statistics.csv) contains both Wald and LRT statistics per community without missing replicate information

## Limitations

- Biological inter-subject variability and technical batch effects can confound naive DA-analyses; the replicate (mouse) structure mitigates this but cannot remove it entirely
- ELVAR's improvement over its Louvain-analog is most pronounced for weaker clustering structure; in strongly clustered data the replicate-aware DA statistics may show little advantage over Louvain-based DA
- Batch correction prior to the ELVAR pipeline rarely improved sensitivity, so batch correction is generally unnecessary but the DA results still depend on the upstream Eva_partitions/ProcessEVA quality

## Evidence

- [methods] DoDA fits negative binomial regressions with replicate structure: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [intro] NBR step is part of the ELVAR DA-testing pipeline in R: "We implement the whole DA-testing pipeline (including the cell-attribute-aware clustering and NBR steps) in the programming language R"
- [discussion] ELVAR detects the age-related naive-to-mature Cd4+ T-cell shift with higher sensitivity: "ELVAR displayed higher sensitivity than Milo and DA-seq to detect biologically important and plausible DA-shifts, such as the age-related shift from naïve to mature Cd4+ T-cells"
- [abstract] DoDA regression test statistics feed the Louvain-benchmark comparison: "demonstrating that ELVAR improves the sensitivity to detect cell-type composition shifts in relation to aging, precancerous states and Covid-19 phenotypes"
- [readme] ELVAR is an open-source R-package installable from GitHub: "devtools::install_github("aet21/ELVAR")"
