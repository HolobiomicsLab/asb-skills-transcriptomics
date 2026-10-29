---
name: deg-ground-truth-label-generation
description: Use when benchmarking or training DEG-combination methods (e.g. Naïve
  Bayes vote classifiers) and you need simulated count tables paired with known ground-truth
  DEG labels.
when_to_use_negative:
- You need ground truth for the ERCC spike-in dataset, where truth is defined by spike-in
  concentrations rather than TCC simulation.
- You are detecting DEGs on real experimental RNA-seq data (no known labels exist),
  rather than benchmarking methods on synthetic data.
- Your input is already a validated count table from a real experiment that should
  go straight into DEgenes Hunter DEG detection.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ExpHunterSuite
  role: R/Bioconductor package whose auxiliary scripts implement the TCC-based RNA-seq
    simulation that produces the simulated count table and ground-truth DEG labels
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: TCC
  role: R package supplying the A. thaliana reference count dataset used as the basis
    for simulated expression data
- name: R
  role: Execution environment for the simulation helper and output of simulated_counts.tsv
    / deg_labels.tsv
- name: Bioconductor
  role: Package ecosystem through which the TCC reference dataset and ExpHunterSuite
    are installed (verify with BiocManager::version())
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:2ec4678f46ecad78f989540642158b586a4ab079849886c83f09f3a979770dff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - Bioconductor
  - ExpHunter Suite
  - ExpHunterSuite
  - R
  - TCC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# deg-ground-truth-label-generation

## Summary

Generates simulated RNA-seq count matrices together with ground-truth DEG labels using the ExpHunterSuite TCC-based simulation methodology, so that differential expression methods can be benchmarked against known positives. This underpins the suite's Naïve Bayes classifier, which is trained on simulated data.

## When to use

Use when benchmarking or training DEG-combination methods (e.g. Naïve Bayes vote classifiers) and you need simulated count tables paired with known ground-truth DEG labels.

## When NOT to use

- You need ground truth for the ERCC spike-in dataset, where truth is defined by spike-in concentrations rather than TCC simulation.
- You are detecting DEGs on real experimental RNA-seq data (no known labels exist), rather than benchmarking methods on synthetic data.
- Your input is already a validated count table from a real experiment that should go straight into DEgenes Hunter DEG detection.

## Inputs

- A. thaliana reference count dataset from the R package TCC (Bioconductor)
- Lafora disease and PMM2-CDG rare disease experiment count datasets
- Simulation parameter grid (108 parameter combinations, 100 datasets per experiment)

## Outputs

- simulated_counts.tsv — gene-by-sample count matrix with genes as rows (e.g. 10,000 genes) and control/treatment conditions
- deg_labels.tsv — ground-truth DEG labels for the simulated genes

## How to apply

Install the ExpHunterSuite R/Bioconductor package (devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)) and confirm the environment via BiocManager::version(). Run the suite's RNA-seq simulation helper, which implements the TCC simulation methodology, generating simulated datasets from the two rare disease experiments (Lafora and PMM2-CDG) and an A. thaliana dataset from the R package TCC, spanning 108 parameter combinations with 100 datasets simulated per experiment (24,300 total). Write the simulated gene-by-sample count matrix to simulated_counts.tsv with genes as rows (e.g. 10,000 genes) and control vs treatment conditions as columns. Write the ground-truth DEG labels for the simulated genes to deg_labels.tsv, one label per simulated gene. Validate that both output files exist and that the row count of simulated_counts.tsv equals the simulated gene count.

## Related tools

- **ExpHunterSuite** (R/Bioconductor package whose auxiliary scripts implement the TCC-based RNA-seq simulation that produces the simulated count table and ground-truth DEG labels) — https://github.com/seoanezonjic/ExpHunterSuite
- **TCC** (R package supplying the A. thaliana reference count dataset used as the basis for simulated expression data)
- **R** (Execution environment for the simulation helper and output of simulated_counts.tsv / deg_labels.tsv)
- **Bioconductor** (Package ecosystem through which the TCC reference dataset and ExpHunterSuite are installed (verify with BiocManager::version()))

## Examples

```
Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'
```

## Evaluation signals

- Both simulated_counts.tsv and deg_labels.tsv exist after the run.
- Row count of simulated_counts.tsv equals the simulated gene count (e.g. 10,000 genes).
- deg_labels.tsv contains exactly one ground-truth label per simulated gene, keyed to the same gene identifiers as the count matrix.
- The environment reports the expected Bioconductor version via BiocManager::version().
- Simulated samples carry the two conditions (control vs treatment) required for downstream DEG benchmarking.

## Limitations

- Simulated data derive from a fixed set of references — two rare disease experiments and an A. thaliana TCC dataset — so realism is bounded by those sources.
- The benchmark scale is large (108 parameter combinations, 100 datasets per experiment, 24,300 total), which is computationally demanding.
- The ExpHunterSuiteExamples replication scripts require Autoflow to be installed, and the suite depends on reactome.db, which can exceed R's default 60-second download timeout unless installed with options(timeout=1000).

## Evidence

- [other] Run the ExpHunterSuite (R/Bioconductor package) RNA-seq simulation helper, which implements the TCC simulation methodology, to generate a simulated count table with control and treatment conditions.: [evidence span withheld — anchor status: not_found]
- [other] spanning 108 parameter combinations with 100 datasets simulated per experiment (24,300 total): [evidence span withheld — anchor status: not_found]
- [intro] the ExpHunter Suite, implemented as an R/Bioconductor package including auxiliary scripts for assessing performance and simulating RNA-seq data: "the ExpHunter Suite, implemented as an R/Bioconductor package including auxiliary scripts for assessing performance and simulating RNA-seq data"
- [readme] Install and configure your R environment as is explained in https://github.com/seoanezonjic/ExpHunterSuite: [evidence span withheld — anchor status: external_doc]
