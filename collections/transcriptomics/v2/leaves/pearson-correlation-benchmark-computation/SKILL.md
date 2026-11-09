---
name: pearson-correlation-benchmark-computation
description: Use when validating deconvolution output of bulk profiles into subpopulation-specific
  HiChIP/RNA-seq/ATAC-seq profiles by correlating predicted vs. true per-subpopulation
  profile vectors (e.g., in silico GM12878/K562 mixtures).
when_to_use_negative:
- When there are no ground-truth subpopulation-specific profiles to correlate against
  (e.g., purely biological bulk-only samples without simulated mixtures) — PCC benchmarking
  requires known true profiles
- When evaluating clustering quality of single cells rather than deconvolution accuracy
  of bulk profiles — other metrics (e.g., cluster assignment concordance) are needed
- When the data types differ from RNA-seq, ATAC-seq, or HiChIP profiles, since the
  Table 1 ranges only cover these three modalities
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3169
tools:
- name: DC3
  role: Produces the deconvolved subpopulation-specific profiles whose accuracy the
    PCC benchmark measures
  repo: github.com/SUwonglab/DC3
- name: scipy
  role: Computes Pearson correlation coefficients between true and predicted profile
    vectors
- name: scikit-learn
  role: Supports the underlying DC3 clustering/deconvolution runs whose outputs are
    scored
- name: pandas
  role: Assembles per-run PCC values into the 4x3 mean-PCC table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:0d83b9afa3fd209c75831781e4516253ff59e6acf76be4455f1ea0c94f6573eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pearson-correlation-benchmark-computation

## Summary

Computes per-run and mean Pearson correlation coefficients (PCC) between true and predicted subpopulation-specific profile vectors (HiChIP loop counts, RNA-seq expression, ATAC-seq openness) to benchmark DC3 deconvolution accuracy against the ranges reported in Table 1 (HiChIP 0.78–0.95, RNA-seq 0.85–0.99, ATAC-seq 0.88–0.99).

## When to use

Use when validating deconvolution output of bulk profiles into subpopulation-specific HiChIP/RNA-seq/ATAC-seq profiles by correlating predicted vs. true per-subpopulation profile vectors (e.g., in silico GM12878/K562 mixtures).

## When NOT to use

- When there are no ground-truth subpopulation-specific profiles to correlate against (e.g., purely biological bulk-only samples without simulated mixtures) — PCC benchmarking requires known true profiles
- When evaluating clustering quality of single cells rather than deconvolution accuracy of bulk profiles — other metrics (e.g., cluster assignment concordance) are needed
- When the data types differ from RNA-seq, ATAC-seq, or HiChIP profiles, since the Table 1 ranges only cover these three modalities

## Inputs

- True subpopulation-specific HiChIP enhancer-promoter interaction count vectors (from exampledata/hichip.txt in the in silico GM12878/K562 setup)
- True subpopulation-specific scRNA-seq expression vectors (exampledata/E.txt, genes listed in gene.txt)
- True subpopulation-specific scATAC-seq openness vectors (exampledata/O.txt, enhancers listed in enhancer.txt)
- DC3-predicted subpopulation-specific profile vectors from each of 50 runs per input setting
- Random-deconvolution null baseline for significance comparison

## Outputs

- Per-run PCC values for HiChIP, RNA-seq, and ATAC-seq deconvolution
- Mean PCC per input setting per data type (averaged over 50 runs)
- A 4x3 table (four input combinations x three data types) of mean PCC scores comparable to Table 1

## How to apply

Run DC3 (dc3.py, k=2 clusters) under each of the four input-combination settings (scRNA-seq/scATAC-seq/scHi-C; scRNA-seq/scATAC-seq/bulk HiChIP; scRNA-seq/bulk ATAC-seq/bulk HiChIP; bulk RNA-seq/scATAC-seq/bulk HiChIP), executing 50 independent runs with different initializations and stopping each optimization iteration when the relative error falls below 10^-4. For each run, compute the Pearson correlation coefficient between the true subpopulation-specific profile vector and the predicted one, separately for HiChIP, RNA-seq, and ATAC-seq deconvolution, using scipy for correlation calculation. Average the PCCs over the 50 runs to obtain mean PCC scores per setting and data type, then assemble a 4x3 (input combinations x data types) table for comparison with the published Table 1 ranges. Validation passes when each reproduced cell falls within the corresponding reported range (e.g., setting-2 HiChIP PCC of 0.86–0.95) and all values exceed the random-deconvolution null baseline.

## Related tools

- **DC3** (Produces the deconvolved subpopulation-specific profiles whose accuracy the PCC benchmark measures) — github.com/SUwonglab/DC3
- **scipy** (Computes Pearson correlation coefficients between true and predicted profile vectors)
- **scikit-learn** (Supports the underlying DC3 clustering/deconvolution runs whose outputs are scored)
- **pandas** (Assembles per-run PCC values into the 4x3 mean-PCC table)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Each cell of the reproduced 4x3 PCC table falls within the corresponding reported Table 1 range: HiChIP 0.78–0.95, RNA-seq 0.85–0.99, ATAC-seq 0.88–0.99
- Setting-2 HiChIP PCC lands in the 0.86–0.95 band, matching the specific reported example
- All mean PCC values exceed the random-deconvolution null baseline
- Per-setting reproducibility across 50 independent runs (different initializations) shows stable mean PCC with no setting falling below its Table 1 floor
- Optimization convergence verified: each iteration stopped only when relative error < 10^-4

## Limitations

- PCC ranges were established on an in silico two-subpopulation (GM12878/K562) mixture with fixed cell counts (n1=146 scRNA-seq, n2=746 scATAC-seq, n3=200 scHi-C cells); performance on other mixtures or more than k=2 subpopulations is not covered by these benchmarks
- Results are initialization-dependent, hence the need for 50 independent runs; single-run PCCs may not fall in the reported ranges
- Benchmark does not cover jointly-profiled single-cell data types (e.g., sci-CAR), which DC3 does not yet incorporate

## Evidence

- [results] DC3 achieved mean PCCs of 0.78–0.95 for HiChIP deconvolution, 0.85–0.99 for RNA-seq deconvolution, and 0.88–0.99 for ATAC-seq deconvolution across the four input settings: [evidence span withheld — anchor status: not_found]
- [intro] comprehensive simulation experiments: "Based on comprehensive simulation experiments, we show that this method can deconvolve bulk profiles into subpopulation-specific profiles"
- [readme] README example invocation parameters: "python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt  -lambda1 10000 -lambda2 0.0005"
- [readme] DC3 outputs include cluster-specific profiles used for validation: "cluster-hichip.txt                                      the cluster-specific hichip"
