---
name: rna-seq-count-data-simulation
description: Use when you need simulated gene-by-sample RNA-seq count matrices with
  known differential-expression ground truth to benchmark DEG detection methods such
  as DESeq2, edgeR, limma, or NOISeq.
when_to_use_negative:
- Input is already a real RNA-seq count table destined for DEG detection rather than
  synthetic benchmark data generation
- The analysis goal is method combination (combined-FDR, Naïve Bayes classifier) on
  existing DEG results rather than generating simulated data itself
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3519
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: Implements the TCC-based RNA-seq simulation helper that generates simulated
    count tables and ground-truth DEG labels
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: R
  role: Runtime environment in which the simulation module runs modularly
- name: Bioconductor
  role: Ecosystem through which ExpHunterSuite and the TCC reference dataset are distributed
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:cabfe25e44a793fe141e95da376f10b119465906a42bbfa8a74bff25ce59f24e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - devtools
  - edgeR
  - fRNC
  - limma
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3519
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# rna-seq-count-data-simulation

## Summary

Generates synthetic RNA-seq count datasets using the TCC simulation methodology implemented in the ExpHunter Suite (R/Bioconductor), spanning 108 parameter combinations with 100 simulated datasets per experiment (24,300 total) and paired ground-truth DEG labels for benchmarking DEG detection methods.

## When to use

Use when you need simulated gene-by-sample RNA-seq count matrices with known differential-expression ground truth to benchmark DEG detection methods such as DESeq2, edgeR, limma, or NOISeq.

## When NOT to use

- Input is already a real RNA-seq count table destined for DEG detection rather than synthetic benchmark data generation
- The analysis goal is method combination (combined-FDR, Naïve Bayes classifier) on existing DEG results rather than generating simulated data itself

## Inputs

- A. thaliana TCC reference count dataset (Bioconductor TCC package)
- Two rare disease reference experiments (Lafora disease dataset, PMM2-CDG dataset)
- Simulation parameter combinations (108 combinations, 100 datasets per experiment)

## Outputs

- simulated_counts.tsv — gene-by-sample count matrix with genes as rows (e.g. 10,000 genes)
- deg_labels.tsv — ground-truth differential-expression labels for the simulated genes

## How to apply

Load an A. thaliana TCC reference count dataset obtained through the Bioconductor ecosystem (confirm the environment via BiocManager::version()), then run the ExpHunterSuite RNA-seq simulation helper, which implements the TCC simulation methodology, over control and treatment conditions. The paper's synthetic experiments span 108 parameter combinations with 100 datasets simulated per experiment, drawing on the two rare disease (Lafora disease and PMM2-CDG) experiments plus the A. thaliana TCC dataset. Write the simulated gene-by-sample count matrix to simulated_counts.tsv with genes as rows, and the ground-truth DEG labels to deg_labels.tsv. Validate that both files exist and that simulated_counts.tsv has the expected number of rows (e.g. 10,000 genes). Installation of the ExpHunterSuite environment is done per the seoanezonjic/ExpHunterSuite README using devtools, with a raised timeout for the large reactome.db package.

## Related tools

- **ExpHunter Suite** (Implements the TCC-based RNA-seq simulation helper that generates simulated count tables and ground-truth DEG labels) — https://bioconductor.org/packages/ExpHunterSuite
- **R** (Runtime environment in which the simulation module runs modularly)
- **Bioconductor** (Ecosystem through which ExpHunterSuite and the TCC reference dataset are distributed) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e "options(timeout=1000); devtools::install_github('seoanezonjic/ExpHunterSuite', dependencies=TRUE)"
```

## Evaluation signals

- Both output files (simulated_counts.tsv, deg_labels.tsv) exist after the run
- Row count of simulated_counts.tsv equals the expected simulated gene count (e.g. 10,000 genes)
- Simulated matrix contains both control and treatment condition columns
- Environment reports the expected Bioconductor version via BiocManager::version() before simulation
- Total datasets generated matches the design: 108 parameter combinations × 100 datasets per experiment (24,300 total)

## Limitations

- Simulated dataset properties depend on the chosen reference experiment; method behavior 'can behave quite differently depending on the properties of the data'
- The paper emphasizes 'the need for the stable implementations of proposed methodologies and workflows', so simulation tooling may require version pinning
- README workflow requires Autoflow to be installed and launch.sh configured with the user's environment and results folder, adding setup overhead

## Evidence

- [methods] The simulation methodology generates simulated expression datasets from the two rare disease experiments and an A. thaliana dataset from the R package TCC, spanning 108 parameter combinations with 100 datasets simulated per experiment (24,300 total).: [evidence span withheld — anchor status: not_found]
- [abstract] ExpHunter Suite is implemented as an R/Bioconductor package available from https://bioconductor.org/packages/ExpHunterSuite: "ExpHunter Suite is implemented as an R/Bioconductor package available from https://bioconductor.org/packages/ExpHunterSuite"
- [intro] the ExpHunter Suite, implemented as an R/Bioconductor package including auxiliary scripts for assessing performance and simulating RNA-seq data: "including auxiliary scripts for assessing performance and simulating RNA-seq data"
- [readme] Code and reports for the studies are available from https://github.com/fmjabato/ExpHunterSuiteExamples: [evidence span withheld — anchor status: external_doc]
- [readme] reactome.db download fails because this package is too large (>400 Mb) and R has a download timeout of 60 seconds by default: [evidence span withheld — anchor status: external_doc]
