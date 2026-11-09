---
name: rhythmic-gene-periodicity-detection
description: Use when you have a timestamped bulk transcriptomic time series (e.g.,
  a GEO circadian dataset like GSE38622 or GSE157077 with samples ordered by adjusted
  circadian time) and need to identify a compact set of significantly rhythmic 24-hour-period
  genes as predictors.
when_to_use_negative:
- Your data are single-cell transcriptomes without pseudobulk aggregation — tauFisher
  cannot overcome the high sequencing dropout rate and cannot predict circadian time
  at a single-cell level
- You lack timestamped time-series training data (e.g., only a diseased single sample)
  — rhythmic gene detection requires a time course to fit periodicity
- You want a time signature from a full training series rather than a small rhythmic
  gene subset — comparator methods like ZeitZeiger identify features differently and
  may be intended instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: tauFisher
  role: Provides the run_metacycle wrapper and find_periodic_genes function used to
    run periodicity detection and select significant 24-hour-period rhythmic predictor
    genes
  repo: github.com/micnngo/tauFisher
- name: MetaCycle
  role: Underlying meta2d rhythmicity engine invoked via run_metacycle with method=c('JTK','LS')
    to compute JTK_Cycle and Lomb-Scargle period and p-value statistics per gene
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:681353325d67f71964d72236739f9971bae4c5490449846a8375e3f23656e0ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaCycle
  - TimeSignatR
  - ZeitZeiger
  - dplyr
  - fda
  - nnet
  - stats
  - tauFisher
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# rhythmic-gene-periodicity-detection

## Summary

Detect rhythmic genes in circadian time-course transcriptomic data by running MetaCycle's meta2d with both JTK_Cycle and Lomb-Scargle via tauFisher's run_metacycle wrapper, then selecting predictor genes with find_periodic_genes (top ten statistically significant 24-hour-period genes with JTK_pvalue < 0.05). This step supplies the rhythmic predictor gene set that downstream tauFisher training depends on.

## When to use

Use when you have a timestamped bulk transcriptomic time series (e.g., a GEO circadian dataset like GSE38622 or GSE157077 with samples ordered by adjusted circadian time) and need to identify a compact set of significantly rhythmic 24-hour-period genes as predictors.

## When NOT to use

- Your data are single-cell transcriptomes without pseudobulk aggregation — tauFisher cannot overcome the high sequencing dropout rate and cannot predict circadian time at a single-cell level
- You lack timestamped time-series training data (e.g., only a diseased single sample) — rhythmic gene detection requires a time course to fit periodicity
- You want a time signature from a full training series rather than a small rhythmic gene subset — comparator methods like ZeitZeiger identify features differently and may be intended instead

## Inputs

- Timestamped bulk transcriptomic time-course expression matrix (genes as rows, samples as columns) with circadian-time labels, preprocessed by uppercasing gene IDs and averaging duplicate genes
- Vector of adjusted circadian time points for each sample (wrapped to 24 hours)
- GEO accession datasets such as GSE38622, GSE54650, GSE54651, GSE56931, GSE113883, GSE157077, or GSE83855

## Outputs

- MetaCycle meta2d output file (e.g., GSE157077_full_adj.txt) with per-gene JTK_Cycle and Lomb-Scargle periodicity statistics
- List of top ten statistically significant 24-hour-period genes with JTK_pvalue < 0.05, returned by find_periodic_genes as JTK- and Lomb-Scargle-based predictor gene sets

## How to apply

Preprocess the expression matrix with tauFisher conventions: uppercase gene IDs, average duplicate genes, and order samples by adjusted circadian time (wrapping times to 24 hours). Run MetaCycle meta2d via tauFisher's run_metacycle wrapper with method=c('JTK','LS') so both JTK_Cycle and Lomb-Scargle periodograms are computed and written to a meta2d output file. Then call find_periodic_genes on the meta2d file, treating genes with JTK_pvalue < 0.05 as significantly rhythmic and selecting the top ten statistically significant genes with a 24-hour period length (e.g., this yields ~9 predictors for GSE38622 when combined with the 9 core clock genes). The resulting JTK-based and Lomb-Scargle-based gene lists become the predictor sets passed to train_tauFisher; in benchmarking, the JTK_Cycle gene set outperformed the Lomb-Scargle set.

## Related tools

- **tauFisher** (Provides the run_metacycle wrapper and find_periodic_genes function used to run periodicity detection and select significant 24-hour-period rhythmic predictor genes) — github.com/micnngo/tauFisher
- **MetaCycle** (Underlying meta2d rhythmicity engine invoked via run_metacycle with method=c('JTK','LS') to compute JTK_Cycle and Lomb-Scargle period and p-value statistics per gene)

## Examples

```
run_metacycle(df=bulk_adj, timepoints=time_adj, out_file="GSE157077_full_adj.txt"); genes = find_periodic_genes(input_file=meta2d_file)
```

## Evaluation signals

- The meta2d output file contains both JTK_Cycle and Lomb-Scargle statistics for every input gene (method=c('JTK','LS'))
- The selected predictor set contains the top ten genes with 24-hour period and JTK_pvalue < 0.05; for GSE38622 the combined predictor set is the expected ~18 genes (top ten plus 9 core clock genes)
- Downstream train_tauFisher runs successfully on the selected genes and the saved model artifact's Genes list matches the expected predictor gene set
- Benchmark expectations: tauFisher with JTK_Cycle genes outperforms the Lomb-Scargle gene set on the mouse benchmark datasets (2-hour accuracy within calc_error)

## Limitations

- Choosing the top ten diurnal genes does not guarantee the best performance, though it is a safe and reasonable choice across different datasets
- Significance is defined by JTK_pvalue < 0.05; genes significant under Lomb-Scargle but not JTK will be excluded from the primary predictor set
- Detection quality depends on preprocessing conventions (uppercase gene IDs, averaging duplicates, ordering by adjusted circadian time, filtering artifacts such as GSE56931's 38-hour wakefulness/recovery time points)

## Evidence

- [discussion] Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic.: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [discussion] It then selects the top ten statistically significant genes with a 24-hour period length.: "It then selects the top ten statistically significant genes with a 24-hour period length."
- [methods] run_metacycle(df=bulk_adj, timepoints=time_adj, out_file="../inst/extdata/GSE157077_full_adj.txt": "run_metacycle(df=bulk_adj, 
              timepoints=time_adj, 
              out_file="../inst/extdata/GSE157077_full_adj.txt","
- [readme] MetaCycle: "tauFisher requires R >=3.5.0 and has been tested on R 4.0.5. It also requires the following R packages: 

MetaCycle"
