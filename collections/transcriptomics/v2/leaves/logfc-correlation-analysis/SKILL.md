---
name: logfc-correlation-analysis
description: Use when benchmarking DEG methods on data with known fold changes (e.g.
  synthetic spike-in transcripts) and you need to verify that estimated logFC values
  are accurate, not just that DEG calls are correct.
when_to_use_negative:
- The data have no known ground-truth fold changes (e.g. real Lafora or PMM2-CDG disease
  datasets) — there is no truth to correlate against, so only DEG-call metrics apply
- You only need to evaluate DEG detection calls (F-measure of gene selection) rather
  than the accuracy of fold-change magnitudes
- Input is raw FASTQ — counts must first be produced via the clean&map step described
  in the spike-ins README section
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor framework whose DEgenes Hunter pipeline runs the individual
    DEG methods that supply the logFC estimates being benchmarked
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: limma
  role: DEG method whose logFC estimates achieved the best reported correlation with
    spike-in truth (mean 0.92, Table 2)
- name: DESeq2
  role: DEG method whose logFC estimates are correlated against spike-in truth as
    part of the per-method comparison
- name: NOISeq
  role: DEG method whose logFC estimates are correlated against spike-in truth as
    part of the per-method comparison
- name: ExpHunterSuiteExamples
  role: Repository providing the spike-in example data and scripts needed to regenerate
    the logFC evaluation
  repo: https://github.com/fmjabato/ExpHunterSuiteExamples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:8df2aa9f92cb5ac167e068e9642c80fd95a1cc8037d07ff1b196fc928fe641f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - DESeq2
  - ExpHunter Suite
  - ExpHunterSuiteExamples
  - NOISeq
  - limma
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# logfc-correlation-analysis

## Summary

Evaluates the accuracy of log2 fold-change estimates from individual DEG methods by correlating estimated logFC against known spike-in ground truth, reproducing Table 2 of the ExpHunter Suite spike-in benchmark (e.g. limma mean correlation 0.92).

## When to use

Use when benchmarking DEG methods on data with known fold changes (e.g. synthetic spike-in transcripts) and you need to verify that estimated logFC values are accurate, not just that DEG calls are correct.

## When NOT to use

- The data have no known ground-truth fold changes (e.g. real Lafora or PMM2-CDG disease datasets) — there is no truth to correlate against, so only DEG-call metrics apply
- You only need to evaluate DEG detection calls (F-measure of gene selection) rather than the accuracy of fold-change magnitudes
- Input is raw FASTQ — counts must first be produced via the clean&map step described in the spike-ins README section

## Inputs

- Count table for the synthetic mouse spike-in RNA-seq experiment (FASTQ files from SRA converted to counts as described in the ExpHunterSuiteExamples README)
- Known spike-in transcript ground-truth fold changes
- Per-method differential expression results (logFC columns) from DEgenes Hunter for DESeq2, limma and NOISeq

## Outputs

- Table of logFC correlations per DEG method (e.g. limma mean 0.92, Table 2 reference values)
- CSV file of per-method logFC correlation metrics for verification against published values

## How to apply

Run the DEgenes Hunter pipeline (ExpHunter Suite, R/Bioconductor) on the spike-in count table so each individual method (DESeq2, limma, NOISeq) produces its own logFC estimates. For each method, correlate the estimated logFC values against the known spike-in fold changes to obtain a per-method correlation (the paper reports limma achieving a mean of 0.92 in Table 2). Rationale: DEG call accuracy alone (F-measure) does not capture magnitude errors, and the paper stresses that incorrect fold-change estimation can lead to poor qPCR target selection and distort downstream logFC-based analyses such as GSEA. Write the per-method logFC correlation values to a CSV table alongside the F-measure table and compare against published reference values within tolerance.

## Related tools

- **ExpHunter Suite** (R/Bioconductor framework whose DEgenes Hunter pipeline runs the individual DEG methods that supply the logFC estimates being benchmarked) — https://github.com/seoanezonjic/ExpHunterSuite
- **limma** (DEG method whose logFC estimates achieved the best reported correlation with spike-in truth (mean 0.92, Table 2))
- **DESeq2** (DEG method whose logFC estimates are correlated against spike-in truth as part of the per-method comparison)
- **NOISeq** (DEG method whose logFC estimates are correlated against spike-in truth as part of the per-method comparison)
- **ExpHunterSuiteExamples** (Repository providing the spike-in example data and scripts needed to regenerate the logFC evaluation) — https://github.com/fmjabato/ExpHunterSuiteExamples

## Examples

```
bash launch_DEgenesHunter.sh
```

## Evaluation signals

- Per-method logFC correlation for limma falls within tolerance of the published Table 2 value (mean 0.92)
- Correlation values for all methods (DESeq2, limma, NOISeq) are recorded in a CSV alongside the F-measure table for the same methods
- The spike-in DEG run reproduces the Table 1 F-measure context (DESeq2 0.88, combined-FDR 0.88, limma 0.76, NOISeq 0.74, threshold-2 0.88), confirming the logFC estimates come from the same correctly configured pipeline
- Correlations are computed per method against the known spike-in ground truth, not against another method's estimates

## Limitations

- Findings are specific to the spike-in dataset; the paper notes the optimal settings may differ for other organisms, tissues and experimental designs and should not be extrapolated as a general rule
- Incorrect fold-change estimation propagates to downstream logFC-based analyses such as GSEA, so a low-correlation method should not be trusted for magnitude-sensitive analyses
- The paper emphasizes the need for stable implementations of the proposed methodologies and workflows

## Evidence

- [discussion] its incorrect estimation could lead to poor target selection. This also has important implications for downstream analyses that use logFC, such as the popular GSEA method: "its incorrect estimation could lead to poor target selection. This also has important implications for downstream analyses that use logFC, such as the popular GSEA method"
- [discussion] the Naïve Bayes classifier presented here performed as well as the best performing single and combination methods on the spike-in dataset: [evidence span withheld — anchor status: not_found]
- [readme] Use count tables obtained to execute DEgenes Hunter expression analysis as done in Lafora or PMM2 but changing input file.: [evidence span withheld — anchor status: external_doc]
- [intro] some widely-used expression analysis methods can behave quite differently depending on the properties of the data, potentially over-predicting or estimating inaccurate values: "some widely-used expression analysis methods can behave quite differently depending on the properties of the data, potentially over-predicting or estimating inaccurate values"
