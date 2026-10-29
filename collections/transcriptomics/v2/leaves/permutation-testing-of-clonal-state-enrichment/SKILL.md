---
name: permutation-testing-of-clonal-state-enrichment
description: Use when single-cell time-course data has CellTag lineage barcodes and
  a fate label (e.g. Capybara-assigned iEP conversion), and you must split cells at
  an intermediate time point into clone groups enriched or depleted for successful
  fate conversion.
when_to_use_negative:
- The dataset lacks clonal lineage barcodes (no CellTag integration), so fate can
  only be inferred from state, not clonally validated
- Only terminal (fully converted) cells are available with no intermediate time point
  whose cells need fate labeling
- The goal is generic differential expression between states rather than linking early
  state to outcome via clonal membership
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: newCloneCalling
  role: CellTag clone-calling pipeline that recovers the 1,158 clones whose membership
    is permuted in the enrichment test
  repo: https://github.com/morris-lab/newCloneCalling
- name: Capybara
  role: Assigns day-4 reprogramming states and iEP conversion outcome labels used
    as the fate classes in the permutation test
  repo: https://github.com/morris-lab/Capybara
- name: CellOracle
  role: Consumes the fate-partitioned day-4 groups to build sample-specific GRN/Links
    objects for downstream degree-centrality comparison
  repo: https://github.com/morris-lab/CellOracle
- name: Cell Ranger
  role: Processes the 10x Chromium scRNA-seq data (v6.0.1) into the digital gene expression
    matrix underlying state classification
- name: igraph
  role: Used downstream to compute TF degree centrality on the fate-group-specific
    GRNs produced after the permutation split
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:2eebc1701620029a4b18378122d4963b66f31b1d742bba153363f2c3af77d46e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/newCloneCalling
  tools:
  - Capybara
  - Cell Ranger
  - CellOracle
  - igraph
  - newCloneCalling
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# permutation-testing-of-clonal-state-enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical procedure that uses a randomized (permutation) test of CellTag clone membership among cells with a known fate outcome to partition intermediate-state cells into fate-enriched and fate-depleted groups, enabling clonal linking of early GRN state to reprogramming outcome.

## When to use

Use when single-cell time-course data has CellTag lineage barcodes and a fate label (e.g. Capybara-assigned iEP conversion), and you must split cells at an intermediate time point into clone groups enriched or depleted for successful fate conversion.

## When NOT to use

- The dataset lacks clonal lineage barcodes (no CellTag integration), so fate can only be inferred from state, not clonally validated
- Only terminal (fully converted) cells are available with no intermediate time point whose cells need fate labeling
- The goal is generic differential expression between states rather than linking early state to outcome via clonal membership

## Inputs

- Integrated CellTag barcode table mapping cells to 1,158 called clones (from morris-lab/newCloneCalling)
- Capybara cell-state and fate classifications along the reprogramming time course (day-4 cells, iEP conversion outcome)
- Cell-by-gene expression matrix (24,799 cells, 10x Chromium, processed with Cell Ranger v6.0.1)

## Outputs

- Fate-labeled day-4 cell groups: iEP-enriched (55 cells / 9 clones) and iEP-depleted (50 cells / 43 clones)
- Permutation test statistics of clone membership among successfully converted cells

## How to apply

After calling clones with the CellTag clone-calling pipeline (1,158 clones recovered from the 24,799-cell 10x Chromium reprogramming dataset) and classifying states/outcomes with Capybara, take the set of day-4 cells and test whether each clone is over-represented among successfully converted cells by permuting clone membership labels. Clones significantly enriched for successful iEP conversion define the iEP-enriched group (55 cells in 9 clones), while the remaining day-4 cells form the iEP-depleted group (50 cells in 43 clones). The rationale is that sister cells follow the same reprogramming trajectories, so clone-level fate enrichment identifies early network states predictive of outcome. The resulting fate-labeled day-4 groups then serve as inputs for building separate CellOracle GRN/Links objects and comparing TF degree centrality (Figure 2H) against native fibroblasts.

## Related tools

- **newCloneCalling** (CellTag clone-calling pipeline that recovers the 1,158 clones whose membership is permuted in the enrichment test) — https://github.com/morris-lab/newCloneCalling
- **Capybara** (Assigns day-4 reprogramming states and iEP conversion outcome labels used as the fate classes in the permutation test) — https://github.com/morris-lab/Capybara
- **CellOracle** (Consumes the fate-partitioned day-4 groups to build sample-specific GRN/Links objects for downstream degree-centrality comparison) — https://github.com/morris-lab/CellOracle
- **Cell Ranger** (Processes the 10x Chromium scRNA-seq data (v6.0.1) into the digital gene expression matrix underlying state classification)
- **igraph** (Used downstream to compute TF degree centrality on the fate-group-specific GRNs produced after the permutation split) — https://github.com/igraph/rigraph

## Examples

```
Rscript capybara_classify.R --input dge_matrix.mtx --reference iEP_timecourse; # then permute clone labels among Capybara-labeled successful iEP cells to split day-4 cells into iEP-enriched (9 clones) and iEP-depleted (43 clones) groups
```

## Evaluation signals

- Recovered clone count matches the reported 1,158 clones from the CellTag pipeline
- Fate-partitioned group sizes match the reported 55 cells / 9 clones (iEP-enriched) and 50 cells / 43 clones (iEP-depleted)
- Enrichment is assessed at clone level, not single-cell level, since sister cells follow the same reprogramming trajectories
- Downstream Figure 2H shows the reported degree-centrality separation: Klf6, Tbx5, Tfapb2, Foxs1 more connected in failing cells; Fos, Klf5, Junb more connected in iEP-destined cells

## Limitations

- Group sizes are small (9 iEP-enriched clones vs 43 iEP-depleted clones), so GRN comparisons between fate groups are necessarily subtle
- Conversion outcome is assumed established shortly after TF overexpression based on clonal sister-cell concordance; the test presumes clone-level fate determinism
- Result depends on prior clone-calling quality (CellTag barcode allowlisting and integration) and Capybara fate-class assignment thresholds

## Evidence

- [other] Permutation test of clone membership identifies fate-enriched day-4 clones: [evidence span withheld — anchor status: not_found]
