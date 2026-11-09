---
name: benjamini-hochberg-multiple-testing-correction
description: Use when NeuronChat (or analogous permutation-based cell-cell communication
  inference) has produced raw per-interaction p-values across many ligand-receptor
  pairs and you need FDR-controlled significance calls before retaining or zeroing
  links.
when_to_use_negative:
- The task tests a single hypothesis or very few interactions, where family-wise or
  uncorrected testing suffices and BH gain is negligible
- P-values come from a method without a permutation-based null (e.g., spatial proximity
  enrichment scores already thresholded by spatial proximity, which is a separate
  NeuronChat filter)
- The communication strengths need to remain continuous for downstream net_aggregation
  without thresholding at p<0.05
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: NeuronChat
  role: Produces the raw permutation-test p-values (via run_NeuronChat) to which BH
    correction is applied; its workflow zeroes out links failing the corrected p<0.05
    cutoff
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Comparison framework sharing NeuronChat's computational workflow for significance
    testing of communication links
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:a0640eaf771cb8b388a79c05402f60eec04000bdd979fb77efdfb1d1a98116f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Apply Benjamini-Hochberg (BH) correction to the p-values from NeuronChat's permutation test so that inferred communication links are called significant at FDR-controlled p<0.05, with non-significant links zeroed out.

## When to use

Use when NeuronChat (or analogous permutation-based cell-cell communication inference) has produced raw per-interaction p-values across many ligand-receptor pairs and you need FDR-controlled significance calls before retaining or zeroing links.

## When NOT to use

- The task tests a single hypothesis or very few interactions, where family-wise or uncorrected testing suffices and BH gain is negligible
- P-values come from a method without a permutation-based null (e.g., spatial proximity enrichment scores already thresholded by spatial proximity, which is a separate NeuronChat filter)
- The communication strengths need to remain continuous for downstream net_aggregation without thresholding at p<0.05

## Inputs

- NeuronChat object with per-interaction permutation p-values from run_NeuronChat (e.g., M=100 or M=1000 permutations of cell-group labels)
- Significance threshold (p<0.05) and chosen number of permutations M

## Outputs

- BH-adjusted p-values for all tested communication links
- Binary significance calls (significant vs non-significant) at p<0.05
- Communication-strength matrix with non-significant links set to zero
- Optional CSV comparison of significant-link sets between permutation settings (e.g., M=100 vs M=1000)

## How to apply

After running run_NeuronChat (e.g., with M=100 or M=1000 permutations of cell-group labels), collect the raw permutation p-values for all interaction pairs and apply the Benjamini-Hochberg procedure to correct them for multiple testing across the full set of tested communications. Classify each communication as significant at the corrected p<0.05 threshold; for non-significant links set the communication strength values to zero. Validate stability by repeating the whole pipeline at two permutation counts (M=100 and M=1000) and comparing significant-link sets — high overlap between the two sets is expected, matching the stability reported in Supplementary Fig. 3. Note that permutation p-value computation is O(n) in the number of permutations, so a trade-off between accuracy and computation speed governs the choice of M.

## Related tools

- **NeuronChat** (Produces the raw permutation-test p-values (via run_NeuronChat) to which BH correction is applied; its workflow zeroes out links failing the corrected p<0.05 cutoff) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Comparison framework sharing NeuronChat's computational workflow for significance testing of communication links) — https://github.com/sqjin/CellChat

## Examples

```
x <- run_NeuronChat(x, M=1000); x@pval_adj <- p.adjust(x@pval, method='BH'); x@net[x@pval_adj >= 0.05] <- 0
```

## Evaluation signals

- All raw permutation p-values are BH-adjusted before thresholding, not compared directly to 0.05
- Every interaction pair flagged non-significant has its communication strength set exactly to zero in the final network
- Significant-link sets from M=100 and M=1000 permutation runs show high overlap (consistent with Supplementary Fig. 3 stability)
- Adjusted p-values are monotonic in raw p-values and bounded in [0,1]
- Number of retained links is consistent with an FDR of 5% given the tested interaction count

## Limitations

- There is a trade-off between accuracy and computation speed for permutation-based p-value calculations, since time complexity is O(n) in the number of permutations
- Genes expressed in less than 25% of cells are filtered out upstream, which restricts the set of interactions available for significance testing
- Ligand/target abundance estimated from transcriptomics may be inconsistent with protein or metabolite levels, so a 'significant' link may not reflect true signaling

## Evidence

- [discussion] BH correction applied to permutation p-values: "The p-value is corrected for multiple tests by using Benjamini–Hochberg procedure"
- [discussion] Permutation-test basis of the p-values: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] Significance threshold and zeroing of non-significant links: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [discussion] Runtime of the permutation step producing the p-values: "There is a trade-off between accuracy and computation speed for permutation-based p-value calculations, which are needed for all new applications of NeuronChat."
- [readme] Concrete run_NeuronChat invocation generating the p-values: "x <- run_NeuronChat(x,M=100)"
