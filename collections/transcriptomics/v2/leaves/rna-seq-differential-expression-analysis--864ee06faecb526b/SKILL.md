---
name: rna-seq-differential-expression-analysis
description: Use when you have a raw RNA-seq count matrix plus sample metadata with
  a defined experimental design and need robust DEG calls that are not biased by one
  method's assumptions.
when_to_use_negative:
- Input is already a DEG call set or consensus table rather than raw counts — no DE
  detection step is needed.
- The goal is downstream interpretation only (co-expression module detection or functional
  enrichment on existing DEGs) rather than DEG detection itself.
- The data are not RNA-seq counts (e.g. single-cell or proteomics data) for which
  DESeq2/edgeR/limma count-model assumptions do not apply.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package whose DEgenes Hunter module executes and integrates
    the multiple DE methods
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DEgenes Hunter
  role: pipeline within ExpHunter Suite that runs DESeq2, edgeR/limma and NOISeq and
    combines results by consensus/vote threshold
- name: DESeq2
  role: one of the individual DEG detection methods executed on the count matrix
- name: limma
  role: one of the individual DEG detection methods; showed higher precision in the
    article's benchmarks
- name: NOISeq
  role: DEG detection method included in DEgenes Hunter; showed high recall in the
    article's benchmarks
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:a56fceeaedef0acd62bd8bb0ea7e91975a64401d895128fc3b57cfe8ad289e4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rna-seq-differential-expression-analysis

## Summary

Detect differentially expressed genes from an RNA-seq count table by running multiple DE methods (DESeq2, edgeR/limma, NOISeq) through the DEgenes Hunter pipeline of ExpHunter Suite and combining their calls via a minimum vote threshold, because single-method results vary by dataset properties.

## When to use

Use when you have a raw RNA-seq count matrix plus sample metadata with a defined experimental design and need robust DEG calls that are not biased by one method's assumptions.

## When NOT to use

- Input is already a DEG call set or consensus table rather than raw counts — no DE detection step is needed.
- The goal is downstream interpretation only (co-expression module detection or functional enrichment on existing DEGs) rather than DEG detection itself.
- The data are not RNA-seq counts (e.g. single-cell or proteomics data) for which DESeq2/edgeR/limma count-model assumptions do not apply.

## Inputs

- RNA-seq table of raw counts (per-gene count matrix)
- sample metadata / experimental design (e.g. mutant vs wild-type phenotype)
- annotation information for functional or orthologous translation steps

## Outputs

- per-gene DEG consensus table recording each method's call, vote count, and estimated logFC (e.g. deg_results.csv)
- consensus DEG counts at chosen vote thresholds (e.g. 1081 genes by at least one method, 345 by all three in PMM2-CDG)

## How to apply

Load the count matrix and sample metadata, then run the DEgenes Hunter module of ExpHunter Suite, which executes DESeq2, edgeR/limma (and NOISeq where applicable) on the experimental design. Combine per-method DEG calls with a minimum vote threshold cut-off (optimum was 2 of 3 by F-measure on spike-in data, 3 for strictest consensus), recording each method's call, vote count and estimated logFC per gene. The article warns methods behave differently — e.g. DESeq2 was most conservative in the Lafora experiment but least conservative in PMM2-CDG, while limma showed higher precision — so consensus rather than a single method is the rationale. Also inspect logFC values, since incorrect fold-change estimation affects downstream analyses such as GSEA and qPCR target selection.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package whose DEgenes Hunter module executes and integrates the multiple DE methods) — https://github.com/seoanezonjic/ExpHunterSuite
- **DEgenes Hunter** (pipeline within ExpHunter Suite that runs DESeq2, edgeR/limma and NOISeq and combines results by consensus/vote threshold)
- **DESeq2** (one of the individual DEG detection methods executed on the count matrix)
- **limma** (one of the individual DEG detection methods; showed higher precision in the article's benchmarks)
- **NOISeq** (DEG detection method included in DEgenes Hunter; showed high recall in the article's benchmarks)

## Examples

```
Rscript -e "options(timeout=1000); devtools::install_github('seoanezonjic/ExpHunterSuite')" ; bash launch_DEgenesHunter.sh
```

## Evaluation signals

- Consensus table exists with per-method call, vote count and logFC columns for every gene.
- Vote-threshold filtering behaves monotonically: strictest threshold (all methods) yields a subset of any-method DEGs (e.g. 345 vs 1081 in the reproduced PMM2-CDG run).
- Outlier logFC values reproduce expected magnitudes (approximately -23.5, -8.8, -2.6) in the PMM2-CDG dataset.
- Per-method DEG counts are consistent with the article's qualitative benchmarks (DESeq2 most conservative in Lafora, least conservative in PMM2-CDG; limma higher precision).
- The pipeline's HTML results report is generated after execution (as produced by get_results.sh).

## Limitations

- The optimal vote threshold may not generalize: 'for the spike-in data the optimum threshold was 2 in terms of F-measure', but the authors are 'hesitant to extrapolate these findings to a more general rule' across organisms, tissues and designs.
- Method conservativeness is dataset-dependent — DESeq2 was the most conservative method in the Lafora experiment but by far the least conservative in PMM2-CDG.
- Incorrect fold-change estimation by individual methods can lead to poor qPCR target selection and affect downstream logFC analyses such as GSEA.
- More elaborate classifiers than Naïve Bayes for combining method results were left for future work.

## Evidence

- [intro] It incorporates the DEgenes Hunter pipeline13, in addition to co-expression analysis: "It incorporates the DEgenes Hunter pipeline13, in addition to co-expression analysis"
- [discussion] DEG detection via multiple methods combined by vote threshold: "we would suggest the use of a minimum vote threshold cut-off where possible"
- [discussion] optimum threshold on spike-in data: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [intro] prior work on combining DEG methods: "Previous studies have looked at combining results of multiple DEG methods to improve DEG detection, using consensus and ranking based strategies as well as p-value integration"
- [readme] install ExpHunterSuite: [evidence span withheld — anchor status: not_found]
- [intro] methods behave differently on different data: "some widely-used expression analysis methods can behave quite differently depending on the properties of the data, potentially over-predicting or estimating inaccurate values"
