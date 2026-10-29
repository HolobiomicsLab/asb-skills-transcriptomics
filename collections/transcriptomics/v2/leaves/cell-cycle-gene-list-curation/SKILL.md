---
name: cell-cycle-gene-list-curation
description: 'Use when preparing DeepCycle inputs: you need an organism-specific cell-cycle
  gene list, a base cycling gene (Nusap1 for mESCs, MELK for human fibroblasts), and
  expression/filter thresholds for gene inclusion.'
when_to_use_negative:
- No cycling genes are detected after filtering — the article warns results will not
  be reliable and deeper sequencing may be needed rather than further gene-list tuning
- The dataset lacks spliced/unspliced layers, since expression thresholds are defined
  on unspliced/spliced reads, not on total counts or feature tables
- The organism of interest has no available GO cell_cycle (GO:0007049) annotation,
  so the candidate gene list cannot be assembled
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: DeepCycle
  role: Consumes the curated gene list via --gene_list, applies --expression_threshold
    and --hotelling filtering, and uses --base_gene to seed the transcriptional phase
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the imputed unspliced (Mu) and spliced (Ms) moments layers used to
    evaluate the expression threshold on candidate genes
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: Preprocessing of the anndata input that feeds the gene-expression filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:623d10ce4fde9eebc5259c78b40e050a6f193565a72febc6f2439657153a2b18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - scVelo
  - velocyto
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cell-cycle-gene-list-curation

## Summary

Curates the candidate cycling gene list (e.g. organism-specific GOterm cell_cycle GO:0007049 annotation) and the base gene used to seed the transcriptional phase, then filters the list by average unspliced/spliced expression and the optional Hotelling filter before DeepCycle inference.

## When to use

Use when preparing DeepCycle inputs: you need an organism-specific cell-cycle gene list, a base cycling gene (Nusap1 for mESCs, MELK for human fibroblasts), and expression/filter thresholds for gene inclusion.

## When NOT to use

- No cycling genes are detected after filtering — the article warns results will not be reliable and deeper sequencing may be needed rather than further gene-list tuning
- The dataset lacks spliced/unspliced layers, since expression thresholds are defined on unspliced/spliced reads, not on total counts or feature tables
- The organism of interest has no available GO cell_cycle (GO:0007049) annotation, so the candidate gene list cannot be assembled

## Inputs

- Organism-specific GO cell-cycle annotation file (e.g. go_annotation/GO_cell_cycle_annotation_mouse.txt)
- Anndata preprocessed with velocyto and scvelo.pp.moments (Mu/Ms layers) for checking expression levels
- Candidate base gene name (Nusap1 for mESCs, MELK for human fibroblasts)

## Outputs

- Filtered list of candidate cycling genes passing the expression threshold and optional Hotelling filter
- Chosen base gene for initial phase estimation
- ./hotelling/ directory listing genes showing more than one state of expression (when --hotelling is used)

## How to apply

Extract the organism-matched cell-cycle gene list from a GO annotation file (e.g. GO_cell_cycle_annotation_mouse.txt for mESCs) covering GOterm cell_cycle GO:0007049. Choose a base gene to seed the initial transcriptional phase — Nusap1 for the mESC dataset and MELK for the human fibroblast dataset (leiden cluster 0 subpopulation). If unsure which gene cycles, run DeepCycle once with a random gene from the gene_list together with the --hotelling flag and inspect ./hotelling/ for genes showing at least two expression states. Apply the minimum average expression threshold (--expression_threshold, suggested ~1.0, adjusted to sequencing depth; the published mESC rerun used 0.5) to drop genes with insufficient unspliced/spliced coverage, and optionally enable the Hotelling filter on the gene list.

## Related tools

- **DeepCycle** (Consumes the curated gene list via --gene_list, applies --expression_threshold and --hotelling filtering, and uses --base_gene to seed the transcriptional phase) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the imputed unspliced (Mu) and spliced (Ms) moments layers used to evaluate the expression threshold on candidate genes) — https://scvelo.readthedocs.io/
- **velocyto** (Preprocessing of the anndata input that feeds the gene-expression filtering)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- The filtered gene set retains detectable cycling genes; if no cycling genes remain, the README states the results will not be reliable
- With --hotelling, the ./hotelling/ directory contains genes selected to show at least two different states of expression
- The base gene shows clear unspliced-spliced cycling structure in ./fits/ plots when used to seed the phase
- Threshold value is consistent with sequencing depth (suggested ~1.0 average expression; 0.5 used in the published mESC rerun)
- Gene list file matches the organism of the input anndata (mouse GO annotation for mESCs, human cell cycle GO annotation for fibroblasts)

## Limitations

- If you do not find any cycling genes the results will not be reliable; sequencing deeper to increase RNAs per cell may be required
- The suggested expression threshold (~1.0) depends on sequencing depth per cell and must be adjusted per dataset
- Defining cell cycle phases based solely on gene markers lacks generalizability across cellular models; future methods should rely on dynamical features of gene expression

## Evidence

- [methods] Gene list selection and base gene rationale: "list of possible cycling genes for the organism of interest (e.g. GOterm cell_cycle)"
- [readme] Hotelling filter strategy for detecting cycling genes: "A good strategy to detect a cycling gene is to first run DeepCycle with a random gene from the gene_list file together with the"
- [methods] Expression threshold guidance: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [methods] Unreliable results when no cycling genes found: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper"
- [discussion] Generalizability caveat for marker-based definitions: "Given the variability in the cell cycle signatures among cellular models, defining the cell cycle phases in RNA data based solely on gene markers lacks generalizability."
