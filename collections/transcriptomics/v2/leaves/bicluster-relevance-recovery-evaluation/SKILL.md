---
name: bicluster-relevance-recovery-evaluation
description: Use when benchmarking a biclustering algorithm (e.g. UniBic via runibic)
  on synthetic matrices with implanted ground-truth biclusters and you need quantitative
  relevance/recovery scores to compare tools.
when_to_use_negative:
- Input is real expression data with no planted ground-truth biclusters — relevance/recovery
  cannot be computed without genuine biclusters to match against
- You need biological validation of biclusters (e.g. GO enrichment) rather than synthetic-planted-bicluster
  recovery benchmarking
- Evaluating algorithms designed for narrow biclusters against UniBic's broad-bicluster
  benchmark, where the article itself notes BicSPAM is 'somewhat better at finding
  narrow biclusters'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: UniBic
  role: Biclustering algorithm under evaluation; discovers trend-preserving biclusters
    that are matched against planted ground truth
  repo: http://sourceforge.net/projects/unibic/files/?source=navbar
- name: runibic
  role: Parallel R implementation of UniBic used to run the algorithm on each synthetic
    matrix with default parameters (q=0.5, r=m, o=100, p<0.05)
  repo: https://github.com/athril/runibic
- name: biclust
  role: R package wrapper used to invoke biclustering methods and handle bicluster
    result objects
- name: R
  role: Runtime environment in which runibic, biclust, and the relevance/recovery
    score computation are executed
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:d88995b36c1b6ae0a74c9e19037944e4726881a7e8e892d1e313d7f911789bff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bicluster-relevance-recovery-evaluation

## Summary

Evaluates biclustering algorithm output by matching discovered biclusters against planted ground-truth biclusters using Jaccard-based relevance S(D,G) and recovery S(G,D) scores, as used to benchmark UniBic on synthetic trend-preserving matrices.

## When to use

Use when benchmarking a biclustering algorithm (e.g. UniBic via runibic) on synthetic matrices with implanted ground-truth biclusters and you need quantitative relevance/recovery scores to compare tools.

## When NOT to use

- Input is real expression data with no planted ground-truth biclusters — relevance/recovery cannot be computed without genuine biclusters to match against
- You need biological validation of biclusters (e.g. GO enrichment) rather than synthetic-planted-bicluster recovery benchmarking
- Evaluating algorithms designed for narrow biclusters against UniBic's broad-bicluster benchmark, where the article itself notes BicSPAM is 'somewhat better at finding narrow biclusters'

## Inputs

- Synthetic data matrices with implanted ground-truth trend-preserving biclusters (e.g. Type I matrices, or overlapping-bicluster matrices with overlap sizes 0x0, 3x3, 6x6, 9x9 at 200x150 dimensions) from the UniBic SourceForge deposit
- Ground-truth planted bicluster gene/condition sets per matrix
- Discovered biclusters output by the algorithm under test (e.g. runibic BCUnibic results)

## Outputs

- Per-matrix Jaccard-based relevance score S(D,G) between discovered and true biclusters
- Per-matrix Jaccard-based recovery score S(G,D) in the reverse direction
- Tabulated scores by condition (e.g. overlap_scores.csv with relevance and recovery by overlap size)

## How to apply

Run the biclustering algorithm (e.g. UniBic via the runibic R package with default settings: significance level 0.05, q=0.5 to skip data separation/granulation preprocessing, r=m, up to 100 trend-preserving biclusters with p-values < 0.05) on each synthetic matrix with planted ground-truth biclusters. Match each discovered bicluster set D against the genuine implanted bicluster set G. Compute Jaccard-based relevance S(D,G) — the fraction of genes in each discovered bicluster that belong to a true bicluster — and recovery S(G,D) in the reverse direction — the fraction of each true bicluster's genes that are recovered. Tabulate both scores per condition (e.g. overlap size 0x0 to 9x9) and compare against reported benchmark values (UniBic relevance 0.65 vs BicSPAM 0.33; recovery 0.69 vs OPSM 0.39 on Type I matrices) to confirm expected performance. For robustness studies, confirm scores remain stable as conditions vary, e.g. UniBic results changed little as implanted bicluster overlap increased.

## Related tools

- **UniBic** (Biclustering algorithm under evaluation; discovers trend-preserving biclusters that are matched against planted ground truth) — http://sourceforge.net/projects/unibic/files/?source=navbar
- **runibic** (Parallel R implementation of UniBic used to run the algorithm on each synthetic matrix with default parameters (q=0.5, r=m, o=100, p<0.05)) — https://github.com/athril/runibic
- **biclust** (R package wrapper used to invoke biclustering methods and handle bicluster result objects)
- **R** (Runtime environment in which runibic, biclust, and the relevance/recovery score computation are executed)

## Examples

```
library(runibic); library(biclust); res <- biclust(method=BCUnibic(), syntheticMatrix); # then compute Jaccard relevance S(D,G) and recovery S(G,D) against planted biclusters
```

## Evaluation signals

- Relevance and recovery scores fall in [0,1] for every matrix and every overlap condition
- UniBic scores on Type I matrices reproduce reported values: relevance ≈ 0.65 (vs 0.33 for BicSPAM) and recovery ≈ 0.69 (vs 0.39 for OPSM)
- Relevance and recovery remain stable as implanted bicluster overlap increases from 0x0 to 9x9 (UniBic found most implanted biclusters even when overlapped)
- Directionality is respected: S(D,G) measures genes of discovered biclusters found in true ones, S(G,D) measures the reverse — swapped computation invalidates the comparison
- All discovered biclusters have p-values < 0.05 and are ordered by decreasing significance before matching

## Limitations

- Scores only reflect recovery of planted trend-preserving biclusters; identification of general trend-preserving biclusters remains a challenging problem
- UniBic was outperformed by BicSPAM on narrow biclusters, so scores may not favor UniBic in that regime
- Evaluation depends on the specific synthetic implantation scheme (Type I, overlap grid) — conclusions may not transfer to other bicluster types

## Evidence

- [methods] Jaccard-based relevance S(D,G) between each discovered bicluster set and the genuine implanted biclusters: [evidence span withheld — anchor status: not_found]
- [results] Relevance and recovery benchmark values: [evidence span withheld — anchor status: not_found]
- [methods] Default significance and bicluster output settings: "We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance."
- [methods] q parameter for skipping preprocessing: "we chose a percentage parameter q with the default value set to 15/m (the value q = 0.5 is specially provided for data without separation preprocessing)"
- [results] Overlap robustness finding: [evidence span withheld — anchor status: not_found]
