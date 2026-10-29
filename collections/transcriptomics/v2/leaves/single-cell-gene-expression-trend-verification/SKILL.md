---
name: single-cell-gene-expression-trend-verification
description: Use when a filtered single-cell expression matrix (e.g. SiFT-filtered
  GSE145197 liver data) must be checked for preservation of a target temporal trend
  while an unwanted signal (spatial zonation) is eliminated.
when_to_use_negative:
- When the signal to be filtered lacks prior knowledge (gene sets or spatial mappings)
  to define the cell-cell similarity kernel, since SiFT strongly relies on such prior
  knowledge.
- When downstream analysis requires properly log-transformed expression, because SiFT
  modifies gene expression so the data is no longer properly log-transformed.
- When no reference temporal profile or published correlation targets exist, since
  evaluation otherwise lacks a ground truth reference.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SiFT
  role: Filters the zonation signal (via KnnKernel on zonated genes or MappingKernel
    on novoSpaRc mapping) whose trend-preservation is then verified
  repo: https://github.com/nitzanlab/sift-sc
- name: scanpy
  role: Loads the GSE145197 liver single-cell dataset (expression matrix, cell metadata,
    novoSpaRc spatial mapping) used for trend reconstruction
- name: novoSpaRc
  role: Provides the spatial cell-to-location mapping used to build the SiFT MappingKernel
    in the temporal-reconstruction comparison
- name: pip
  role: Installation route for the sift-sc package (pip install sift-sc)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:69319279d759dd4ac28e0e7e1855bd5ec65f4e39bd78592784ac7aed61ec27f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - novoSpaRc
  - pip
  - scanpy
  - sift.SiFT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-gene-expression-trend-verification

## Summary

Verifies that a biological signal of interest (e.g. the liver circadian temporal trend of genes like hlf and elovl3) is preserved after SiFT filtering, by reconstructing the temporal trajectory and comparing Pearson correlations against reported reference values.

## When to use

Use when a filtered single-cell expression matrix (e.g. SiFT-filtered GSE145197 liver data) must be checked for preservation of a target temporal trend while an unwanted signal (spatial zonation) is eliminated.

## When NOT to use

- When the signal to be filtered lacks prior knowledge (gene sets or spatial mappings) to define the cell-cell similarity kernel, since SiFT strongly relies on such prior knowledge.
- When downstream analysis requires properly log-transformed expression, because SiFT modifies gene expression so the data is no longer properly log-transformed.
- When no reference temporal profile or published correlation targets exist, since evaluation otherwise lacks a ground truth reference.

## Inputs

- SiFT-filtered and unfiltered single-cell liver expression matrices (GSE145197, loaded with scanpy)
- Reference temporal (circadian) profile for correlation targets
- Set of 0 or 10 reference temporal marker genes
- Temporal genes to trend-check (hlf, elovl3)

## Outputs

- Correlation table of Pearson correlations per filtering condition vs. reference temporal profile (fig4d_correlations.csv)
- Gene-trend check results confirming preservation or loss of hlf/elovl3 temporal trends
- Pass/fail assessment against Fig. 4d target values (0.81, 0.96, 0.71, 0.44)

## How to apply

After running SiFT with either a KnnKernel built from prior-knowledge zonated genes or a MappingKernel built from the novoSpaRc spatial cell-to-location mapping, reconstruct the temporal (circadian) signal from each filtered dataset plus the unfiltered original, optionally adding 0 or 10 reference temporal marker genes. Compute Pearson correlations of each reconstruction against the reference temporal profile for all four conditions and compare them to the Fig. 4d targets: SiFT novoSpaRc 0 markers = 0.81, SiFT novoSpaRc 10 markers = 0.96, SiFT zonated 10 markers = 0.71, original 10 markers = 0.44. Separately inspect the expression trends of specific temporal genes (hlf, elovl3) in the filtered data to confirm the circadian trend is qualitatively preserved while zonation-driven spatial variation is gone. Write the correlation table and gene-trend check results (e.g. to fig4d_correlations.csv) as the verification artifact.

## Related tools

- **SiFT** (Filters the zonation signal (via KnnKernel on zonated genes or MappingKernel on novoSpaRc mapping) whose trend-preservation is then verified) — https://github.com/nitzanlab/sift-sc
- **scanpy** (Loads the GSE145197 liver single-cell dataset (expression matrix, cell metadata, novoSpaRc spatial mapping) used for trend reconstruction)
- **novoSpaRc** (Provides the spatial cell-to-location mapping used to build the SiFT MappingKernel in the temporal-reconstruction comparison)
- **pip** (Installation route for the sift-sc package (pip install sift-sc))

## Examples

```
pip install sift-sc; import sift; sifter = sift.SiFT(kernel=sift.kernels.KnnKernel)
```

## Evaluation signals

- Reproduced Pearson correlations match Fig. 4d targets: SiFT novoSpaRc 0 markers = 0.81, SiFT novoSpaRc 10 markers = 0.96, SiFT zonated 10 markers = 0.71, original 10 markers = 0.44.
- hlf and elovl3 temporal trends are qualitatively preserved in the filtered data.
- Zonation-driven spatial variation is eliminated from the filtered expression matrix.
- Filtering with SiFT improves the temporal reconstruction relative to the unfiltered original (0.44), demonstrating signal enhancement rather than degradation.
- Correlation table and gene-trend checks are written to the output artifact (fig4d_correlations.csv).

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; inaccurate or coarse-grained mapping may lead to a loss of signal.
- SiFT lacks a ground truth reference for evaluating filtering results; integration metrics used for evaluation may be application-specific.
- SiFT modifies gene expression so the data is no longer properly log-transformed, which is expected in certain downstream analysis tasks; statistical implications of applying common tools to corrected expression are unexplored.

## Evidence

- [abstract] SiFT enhances the liver circadian signal by filtering spatial zonation: "SiFT enhances the liver circadian signal by filtering spatial zonation"
- [intro] we use prior knowledge regarding liver zonation to filter the spatial signal from single-cell liver data, thereby enhancing the temporal circadian signal encoded by the cells: "we use prior knowledge regarding liver zonation to filter the spatial signal from single-cell liver data, thereby enhancing the temporal circadian signal encoded by the cells"
- [discussion] it strongly relies on the prior knowledge used to define the cell-cell similarity kernel: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [discussion] SiFT modifies gene expression so that data is no longer properly log-transformed: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
- [intro] SiFT is available as an open-source software package: "SiFT is available as an open-source software package https://github.com/nitzanlab/sift-sc"
