---
name: cyclic-smoothing-of-genome-methylation-along-cell-cycle
description: Use when you have matched single-cell methylome and scRNA-seq data (e.g.
  61 mESC-MT bisulfite cells with expression) and want methylation variation along
  a recovered cell cycle ordering.
when_to_use_negative:
- Input is bulk or population methylation data without matched single-cell transcriptomes,
  since per-cell mapping onto the reCAT pseudo time-series is impossible
- The goal is only to recover the cell cycle ordering itself from expression data
  (that is the get_ordIndex/get_hmm_order ordering task, not the methylation mapping
  task)
- The methylation and expression matrices do not share identifiable cells, so profiles
  cannot be assigned to positions in the ordIndex
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3308
tools:
- name: reCAT
  role: Recovers the cell cycle pseudo time-series (get_ordIndex consensus-TSP ordering,
    get_score bayes/mean scores, get_hmm_order phase segmentation) onto which the
    methylation profiles are mapped
  repo: github.com/tinglab/reCAT
- name: ggplot2
  role: Draws the smoothed whole-genome methylation curve along the cycle with phase
    annotations and saves the figure
- name: TSP
  role: Provides the traveling salesman problem solver underlying reCAT's consensus-TSP
    circular cell ordering used as the x-axis for the methylation curve
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:28f3d248c211ea7550d79f7750e47df2483fced9129203229c241afb599df1bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - ggplot2
  - reCAT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cyclic-smoothing-of-genome-methylation-along-cell-cycle

## Summary

Maps single-cell bisulfite whole-genome methylation profiles onto the reCAT-recovered cell cycle pseudo time-series and plots the smoothed methylation rate along the G1/S/G2-M cycle, revealing the methylation peak at G1/S reported in Fig. 4d of the reCAT paper.

## When to use

Use when you have matched single-cell methylome and scRNA-seq data (e.g. 61 mESC-MT bisulfite cells with expression) and want methylation variation along a recovered cell cycle ordering.

## When NOT to use

- Input is bulk or population methylation data without matched single-cell transcriptomes, since per-cell mapping onto the reCAT pseudo time-series is impossible
- The goal is only to recover the cell cycle ordering itself from expression data (that is the get_ordIndex/get_hmm_order ordering task, not the methylation mapping task)
- The methylation and expression matrices do not share identifiable cells, so profiles cannot be assigned to positions in the ordIndex

## Inputs

- 61-cell mESC-MT single-cell bisulfite whole-genome methylation matrix (cells x genomic methylation rates)
- Matched mESC-MT scRNA-seq expression matrix (log2 TPM+1), filtered to cycle base genes via get_test_exp
- ordIndex from reCAT get_ordIndex (consensus-TSP over GMM cluster paths)
- HMM phase labels (G1, S, G2/M) from reCAT get_hmm_order using bayes_score and mean_score from get_score

## Outputs

- Per-bin whole-genome methylation rates for cells in sliding windows along the recovered cycle
- Smoothed methylation curve along the circular ordering with G1/S/G2-M annotations, saved as methylation_along_cycle.png

## How to apply

Preprocess the paired mESC-MT scRNA-seq matrix as log2(TPM+1) and restrict it to cycle base genes with reCAT's get_test_exp, then recover the circular ordering with get_ordIndex (threadnum 10, step_size 2, base_cycle_range c(6:9)) and segment phases with get_hmm_order using cls_num = 3 (G1, S, G2/M) on the bayes_score and mean_score from get_score. Using the ordIndex, assign each of the 61 bisulfite methylation profiles to its position in the pseudo time-series and its HMM phase label. Bin cells in sliding windows along the cycle, compute the whole-genome methylation rate per bin, and smooth the binned rates across the circular ordering so the curve is continuous around the cycle. Draw the smoothed curve with ggplot2, annotating the G1, S, and G2/M regions, and save it (e.g. methylation_along_cycle.png); the decision point is whether a visible methylation peak appears at the G1/S boundary, matching Fig. 4d.

## Related tools

- **reCAT** (Recovers the cell cycle pseudo time-series (get_ordIndex consensus-TSP ordering, get_score bayes/mean scores, get_hmm_order phase segmentation) onto which the methylation profiles are mapped) — github.com/tinglab/reCAT
- **ggplot2** (Draws the smoothed whole-genome methylation curve along the cycle with phase annotations and saves the figure)
- **TSP** (Provides the traveling salesman problem solver underlying reCAT's consensus-TSP circular cell ordering used as the x-axis for the methylation curve)

## Examples

```
source("get_ordIndex.R"); ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- The saved smoothed methylation curve shows a visible peak of the whole-genome methylation rate at the G1/S boundary, matching the qualitative pattern of Fig. 4d
- HMM segmentation with cls_num = 3 yields coherent G1, S, and G2/M regions consistent with the bayes_score and mean_score plots along the ordIndex
- Every one of the 61 mESC-MT methylation profiles is assigned to exactly one position in the recovered circular ordering (no unmapped or duplicated cells)
- Smoothed binned rates are continuous around the circular ordering (no discontinuity at the cycle start/end junction)
- The output figure methylation_along_cycle.png exists and annotates all three phase regions G1, S, and G2/M

## Limitations

- The finding is qualitative: the curve must exhibit a visible G1/S methylation peak rather than a statistically tested difference, and 61 cells limit bin resolution
- reCAT relies on a preselected gene set (378 genes) rather than data-driven gene selection, which the authors note could bias the recovered ordering that positions the methylation profiles
- The bayes-scores and mean-scores used for phase segmentation are noisy; the authors state they 'need improvements to be less noisy and more informative'
- The mapping depends on HMM phase labels (rdata regions and myord start) judged from score plots, so misjudged class regions would shift the annotated G1/S boundary

## Evidence

- [abstract] Methylation variation was exhibited along the recovered cell cycle: "Moreover, we leverage reCAT to exhibit methylation variation along the recovered cell cycle."
- [methods] log2(TPM+1) preprocessing with get_test_exp: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [methods] get_ordIndex recovers the cell time series: "you can get its order(cell's time series) easily with ***get_ordIndex*** function"
- [methods] get_hmm_order sorts cells into cycle phases: "you can use ***get_hmm_order*** function
to sort the cells."
- [readme] cls_num = 3 corresponds to G1, S, G2/M: "cls_num is class number you want, generally is 3(G1, S, G2/M) or 4(G0, G1, S, G2/M)."
- [readme] get_ordIndex parameters threadnum, step_size, base_cycle_range: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)."
