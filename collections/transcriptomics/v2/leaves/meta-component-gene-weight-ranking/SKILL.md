---
name: meta-component-gene-weight-ranking
description: Use when a task needs the gene members of a specific MetaTiME MeC ranked
  by z-weight — e.g., to build a signature score from the top 20 genes of MeC-42 or
  a top-100 gene set for Lisa ChIP-seq TF analysis.
when_to_use_negative:
- Input is already a curated marker gene list or feature table that does not come
  from the MetaTiME pretrained MeC space
- The goal is annotating new single cells rather than extracting gene signatures —
  use the MetaTiME annotator projection workflow instead
- Single-cell data must be re-integrated with dataset-specific batch correction (Harmony/scVI/scArches)
  rather than mapped onto the pretrained MeC directions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Provides the pretrained MeC gene-weight table via the metatime.mecs module
    from which top z-weight genes are extracted for each of the 86 meta-components
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:2499e6f65b395967fd9be4ad51e80bac850efe47122ffdafb4fecce63e772632
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# meta-component-gene-weight-ranking

## Summary

Rank genes within MetaTiME meta-components (MeCs) by their pretrained z-weights to derive interpretable per-MeC gene signatures (e.g., top 20 or top 100 genes of the 86 MeCs) used for downstream scoring, survival association, and TF regulator prioritization.

## When to use

Use when a task needs the gene members of a specific MetaTiME MeC ranked by z-weight — e.g., to build a signature score from the top 20 genes of MeC-42 or a top-100 gene set for Lisa ChIP-seq TF analysis.

## When NOT to use

- Input is already a curated marker gene list or feature table that does not come from the MetaTiME pretrained MeC space
- The goal is annotating new single cells rather than extracting gene signatures — use the MetaTiME annotator projection workflow instead
- Single-cell data must be re-integrated with dataset-specific batch correction (Harmony/scVI/scArches) rather than mapped onto the pretrained MeC directions

## Inputs

- Pretrained MetaTiME MeC gene-weight table (loaded via metatime.mecs), covering the 86 meta-components
- Target MeC identifier or functional name (e.g., MeC-42, M_Macrophage-IL1-NFKB)
- Number of top genes to extract (20 for signature scoring, 100 for TF regulator analysis)

## Outputs

- Per-MeC ranked gene list of top z-weight genes (e.g., top 20 or top 100 per MeC)
- Gene signature sets written to file (e.g., CSV) for downstream scoring or Lisa analysis

## How to apply

Load the pretrained MeC gene-weight table through the metatime.mecs module of the MetaTiME package (pip install metatime), which stores the 86 data-driven meta-components learnt from millions of TME single cells. Select the target MeC by its identifier or functional name (e.g., MeC-42 / M_Macrophage-IL1-NFKB, MeC-0 interferon response, MeC-37, MeC-20 CXCL13+ Tfh) and sort its gene weights by z-weight descending. Extract the required number of top genes depending on the downstream use: top 20 for averaged-expression signature scoring of bulk RNA-seq (e.g., TCGA tumor survival analysis), top 100 for TF regulatory potential comparison with Lisa ChIP-seq scores. Rationale: the z-weights encode each gene's contribution to the independent transcriptional direction, so the top-ranked genes form the biologically interpretable signature the MeC was named for. Record the extracted gene list (e.g., to CSV) so downstream scoring or regulator-ranking steps are reproducible and traceable to the pretrained model.

## Related tools

- **MetaTiME** (Provides the pretrained MeC gene-weight table via the metatime.mecs module from which top z-weight genes are extracted for each of the 86 meta-components) — https://github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime; from metatime import mecs; mec_weights = mecs.load_mec(); top20 = mec_weights.nlargest(20, 'MeC-42')
```

## Evaluation signals

- The extracted gene set contains the expected signature members, e.g., the top 20 MeC-42 genes reflect the IL1B/NFKB macrophage program
- The number of ranked genes matches the requested cutoff (20 or 100) and the table covers all 86 MeCs when the full ranking is written
- Ranking is by z-weight in descending order and is reproducible from the pretrained table across runs
- Downstream validation: the top-20 MeC-42 score reproduces the reported survival association (lower survival in LGG and KIRC), and top-100 MeC-0 genes reproduce STAT1 as top Lisa-prioritized regulator
- MeC identities match the published annotations (MeC-0 interferon response, MeC-20 CXCL13+ Tfh, MeC-37 PPARG)

## Limitations

- The MeC gene-weight table is a pretrained model; rankings reflect the training cohorts of millions of TME single cells and may not capture dataset-private programs
- There are only 86 meta-components, so signatures for states outside the learnt TME landscape cannot be extracted
- For TF regulator prioritization, TF binding evidence depends on external ChIP-seq contexts (e.g., GR ChIP-seq is not available in relevant T cell states and is inferred from other cell lines)

## Evidence

- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities.: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment.: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
- [abstract] By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data.: "By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data."
- [discussion] Though GR ChIP-seq is not available in the contexts of the relevant T cell states, GR ChIP-seq in other cell lines demonstrate robust binding nearby the top gene CXCL13.: "Though GR ChIP-seq is not available in the contexts of the relevant T cell states, GR ChIP-seq in other cell lines demonstrate robust binding nearby the top gene CXCL13."
