---
name: count-concordance-correlation-analysis
description: Use when two or more barcode pre-processing pipelines have been run on
  the same FASTQ samples and you need to quantify agreement — overlap of detected
  barcodes and correlation of counts against a published dataset.
when_to_use_negative:
- No gold-standard dataset with known barcode abundance exists — the article notes
  benchmarking against similar tools was difficult for this reason, so pure accuracy
  claims cannot be made, only inter-tool agreement
- Input is raw FASTQ that has not been processed by at least two comparable pre-processing
  pipelines to generate count tables
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bartools
  role: R package used to compute barcode set overlaps across tools and correlation
    of counts against the published dataset
  repo: https://github.com/DaneVass/bartools_manuscript_code/tree/main/tools-comparison
- name: BARtab
  role: Nextflow pipeline generating the barcode counts table under evaluation in
    single-bulk reference-free mode
  repo: https://github.com/DaneVass/BARtab
- name: pycashier
  role: Comparison pre-processing tool run on the same dataset with STAR Methods parameterizations
  repo: https://github.com/brocklab/pycashier
- name: timemachine
  role: Comparison pre-processing tool (Rewind/TimeMachine) run on the same dataset
  repo: https://github.com/arjunrajlaboratory/timemachine
- name: starcode
  role: Message-passing Levenshtein clustering (distance 3, ratio 3) producing the
    consensus barcode repertoire used for count comparison
  repo: https://github.com/gui11aume/starcode
- name: R
  role: Environment in which bartools overlap and correlation analyses are executed
  repo: https://github.com/DaneVass/bartools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:30380921d5471685a96aa742bcd47effa56b77e75f0e04ee649fdf0d41ac3bd3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BARtab
  - R
  - bartools
  - pycashier
  - starcode
  - timemachine
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# count-concordance-correlation-analysis

## Summary

Benchmark the concordance of barcode pre-processing tools (BARtab, pycashier, TimeMachine) by computing barcode set overlaps with bartools in R and Pearson/Spearman correlations of per-sample barcode counts against a published reference dataset.

## When to use

Use when two or more barcode pre-processing pipelines have been run on the same FASTQ samples and you need to quantify agreement — overlap of detected barcodes and correlation of counts against a published dataset.

## When NOT to use

- No gold-standard dataset with known barcode abundance exists — the article notes benchmarking against similar tools was difficult for this reason, so pure accuracy claims cannot be made, only inter-tool agreement
- Input is raw FASTQ that has not been processed by at least two comparable pre-processing pipelines to generate count tables

## Inputs

- Per-sample barcode count tables per tool (e.g. starcode/<sample_id>_starcode.tsv-derived counts from BARtab)
- Published reference barcode counts dataset (Goyal et al., 22-sample population-level barcoding dataset)
- tools_comparison_results.tsv workspace for overlap and correlation summaries

## Outputs

- tools_comparison_results.tsv with three-tool barcode overlap counts and percentages
- Pairwise overlap sets (e.g. BARtab+TimeMachine-only barcodes)
- Pearson and Spearman correlation coefficients of barcode counts vs published dataset
- Per-tool wall-clock runtime and peak memory figures

## How to apply

Combine per-sample barcode counts from each tool into single tables (e.g. counts/all_counts_combined.tsv via COMBINE_BARCODE_COUNTS outer join), then use bartools in R to compute set overlaps across tools. For the Goyal 22-sample population-level dataset, this yielded 144,745 barcodes (97.3% of barcodes with at least 0.001% frequency within a sample) detected by all three tools and 1,810 barcodes uniquely shared by BARtab and TimeMachine. Next compute Pearson and Spearman correlation of BARtab-derived barcode counts against the published Goyal et al. dataset, expecting concordance above 0.96. Also record wall-clock runtime and peak memory per tool (BARtab 35 min / 6.83 GB; pycashier 36 min / 1.81 GB; TimeMachine 4h59min / 3.52 GB) to contextualize agreement with cost.

## Related tools

- **bartools** (R package used to compute barcode set overlaps across tools and correlation of counts against the published dataset) — https://github.com/DaneVass/bartools_manuscript_code/tree/main/tools-comparison
- **BARtab** (Nextflow pipeline generating the barcode counts table under evaluation in single-bulk reference-free mode) — https://github.com/DaneVass/BARtab
- **pycashier** (Comparison pre-processing tool run on the same dataset with STAR Methods parameterizations) — https://github.com/brocklab/pycashier
- **timemachine** (Comparison pre-processing tool (Rewind/TimeMachine) run on the same dataset) — https://github.com/arjunrajlaboratory/timemachine
- **starcode** (Message-passing Levenshtein clustering (distance 3, ratio 3) producing the consensus barcode repertoire used for count comparison) — https://github.com/gui11aume/starcode
- **R** (Environment in which bartools overlap and correlation analyses are executed) — https://github.com/DaneVass/bartools

## Examples

```
nextflow run DaneVass/bartab --input samples.csv -profile test_SE,docker && Rscript tools-comparison/compute_overlaps.R --counts counts/all_counts_combined.tsv --reference goyal_counts.tsv --out tools_comparison_results.tsv
```

## Evaluation signals

- tools_comparison_results.tsv reports a three-tool overlap of 144,745 barcodes (97.3% of barcodes with at least 0.001% frequency within a sample)
- BARtab+TimeMachine-only overlap is 1,810 barcodes
- Pearson and Spearman correlations of BARtab counts vs the published Goyal et al. dataset exceed 0.96
- Per-tool runtime/memory match reported figures: BARtab 35 min / 6.83 GB, pycashier 36 min / 1.81 GB, TimeMachine 4h59min / 3.52 GB

## Limitations

- Lack of a gold-standard dataset with known barcode abundance makes absolute accuracy benchmarking impossible; results measure inter-tool agreement only
- Differing parameter flexibility between tools complicates fair benchmarking
- Benchmarking on simulated data was not performed, so accuracy and optimal parameter selection remain open questions

## Evidence

- [results] 144,745 barcodes (97.3% of barcodes with at least 0.001% frequency within a sample) were detected by all three tools: [evidence span withheld — anchor status: not_found]
- [discussion] benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools: "benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools."
- [methods] Code and results are available here: https://github.com/DaneVass/bartools_manuscript_code/tree/main/tools-comparison: "Code and results are available here: [https://github.com/DaneVass/bartools_manuscript_code/tree/main/tools-comparison]"
- [readme] Comparison of BARtab with pycashier and timemachine in tools-comparison/: [evidence span withheld — anchor status: external_doc]
