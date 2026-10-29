---
name: pathway-enrichment-dotplot-visualization
description: Use when a CCC decomposition factor's loadings (e.g., Factor 4 from the
  context x interaction x sender x receiver tensor built on GSE96583/EH2259 IFN-beta
  PBMCs) must be biologically interpreted via pathway enrichment dotplots such as
  JAK-STAT across STIM vs CTRL contexts.
when_to_use_negative:
- The per-sample ligand-receptor scoring or tensor assembly has not yet been done
  — there are no factor loadings to enrich, so build the tensor and decompose it first
  (rank 7 here).
- The goal is quantifying pathway activity in the original expression matrix rather
  than interpreting CCC factor loadings from a multi-context decomposition.
- The question concerns long-distance endocrine signalling or other signalling not
  represented by gene expression, which scRNA-seq-based CCC inference cannot capture.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LIANA
  role: Produces the per-sample SingleCellSignalR LRscore ligand-receptor results
    whose contexts form the tensor, providing the loadings that the dotplot interprets
  repo: https://github.com/saezlab/liana
- name: SingleCellSignalR
  role: Source of the regularised LRscore used for per-sample CCC scoring in the tensor
    tutorial
  repo: saezlab/SingleCellSignalR_v1
- name: tensor-cell2cell
  role: Generates the rank-7 context factors (e.g., Factor 4 separating STIM vs CTRL)
    whose loadings are projected onto pathways for the dotplot
  repo: saezlab/ccc_protocols
- name: PROGENy
  role: Supplies the pathway responsive gene sets (including JAK-STAT) used to enrich
    the factor loadings
  repo: saezlab/progeny
- name: decoupleR
  role: Infers pathway activities/enrichment statistics from the factor loadings prior
    to dotplot rendering
  repo: saezlab/decoupleR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:4b5def2f3e466cc3a7d6cbb465ab6a11900c729ee08ac372f30aa541c903fe8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - PROGENy
  - Scanpy
  - SingleCellSignalR
  - decoupleR
  - fgsea
  - h5py
  - msigdbr
  - scMDC
  - tensor-cell2cell
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pathway-enrichment-dotplot-visualization

## Summary

Visualise pathway enrichment of cell-cell communication (CCC) tensor factor loadings as dotplots — e.g., the JAK-STAT enrichment dotplot for Factor 4 of the rank-7 tensor-cell2cell decomposition that separates IFN-beta-stimulated (STIM) from control (CTRL) PBMC contexts in the LIANA x tensor-cell2cell workflow.

## When to use

Use when a CCC decomposition factor's loadings (e.g., Factor 4 from the context x interaction x sender x receiver tensor built on GSE96583/EH2259 IFN-beta PBMCs) must be biologically interpreted via pathway enrichment dotplots such as JAK-STAT across STIM vs CTRL contexts.

## When NOT to use

- The per-sample ligand-receptor scoring or tensor assembly has not yet been done — there are no factor loadings to enrich, so build the tensor and decompose it first (rank 7 here).
- The goal is quantifying pathway activity in the original expression matrix rather than interpreting CCC factor loadings from a multi-context decomposition.
- The question concerns long-distance endocrine signalling or other signalling not represented by gene expression, which scRNA-seq-based CCC inference cannot capture.

## Inputs

- Context-factor loadings from tensor_cell2cell decomposition (rank 7) of the context x interaction x sender x receiver CCC tensor
- Per-sample ligand-receptor interaction scores from liana_bysample using the SingleCellSignalR LRscore
- PROGENy pathway responsive gene sets (human)
- Donor-condition context metadata distinguishing STIM vs CTRL PBMC samples

## Outputs

- Pathway enrichment table assigning pathways (e.g., JAK-STAT) to factor loadings with enrichment/activity statistics
- Dotplot figure file factor4_stim_vs_ctrl_dotplot.png showing pathway enrichment across Factor 4 loadings separated by STIM vs CTRL

## How to apply

After running tensor_cell2cell at rank 7 on the 4D CCC tensor (contexts are the 8 donor-condition samples, scored per-sample by LIANA's liana_bysample with SingleCellSignalR's regularised LRscore), extract the loadings of the factor of interest — Factor 4, which reproduces the separation of STIM vs CTRL contexts. Score these loadings against a curated pathway gene-set resource such as PROGENy's pathway responsive genes using an activity-inference method like decoupleR, so that each pathway (e.g., JAK-STAT) receives an enrichment/activity statistic per factor loading. Render the results as a dotplot in which dots encode pathway enrichment magnitude (typically size and colour) across the factor loadings or contexts, making the interferon/JAK-STAT signal in stimulated donors directly visible. Save the rendered figure under the workflow's canonical name, factor4_stim_vs_ctrl_dotplot.png. The rationale is that factor loadings capture context-specific CCC programmes, and projecting them onto pathway responsive genes converts an abstract decomposition axis into an interpretable biological mechanism.

## Related tools

- **LIANA** (Produces the per-sample SingleCellSignalR LRscore ligand-receptor results whose contexts form the tensor, providing the loadings that the dotplot interprets) — https://github.com/saezlab/liana
- **SingleCellSignalR** (Source of the regularised LRscore used for per-sample CCC scoring in the tensor tutorial) — saezlab/SingleCellSignalR_v1
- **tensor-cell2cell** (Generates the rank-7 context factors (e.g., Factor 4 separating STIM vs CTRL) whose loadings are projected onto pathways for the dotplot) — saezlab/ccc_protocols
- **PROGENy** (Supplies the pathway responsive gene sets (including JAK-STAT) used to enrich the factor loadings) — saezlab/progeny
- **decoupleR** (Infers pathway activities/enrichment statistics from the factor loadings prior to dotplot rendering) — saezlab/decoupleR

## Evaluation signals

- The figure factor4_stim_vs_ctrl_dotplot.png is produced and contains a JAK-STAT (interferon) pathway term among the enriched pathways.
- Factor 4 loadings reproduce the separation of STIM vs CTRL contexts in the dotplot's factor/context axis.
- The underlying tensor decomposition is at rank 7, built from a context x interaction x sender x receiver tensor of per-sample LIANA LRscores.
- Scores entering the tensor are the regularised LRscore (squared transmitter/receiver expression), so they are comparable across the 8 donor-condition contexts.
- The enrichment uses a curated pathway resource (PROGENy) rather than ad hoc gene lists.

## Limitations

- No gold standard exists for CCC, so enrichment interpretations of factor loadings cannot be validated against definitive biological ground truth.
- scRNA-seq gene expression does not capture long-distance endocrine signalling, protein translation/processing, secretion, or diffusion, limiting what the enrichment can represent.
- Both the CCC method and resource choice considerably impact predicted interactions, so the dotplot's pathway signal is method- and resource-dependent.
- Gene expression is only a proxy of protein levels; cytokine-activity and receptor-protein coverage used to corroborate enrichments is restricted.

## Evidence

- [abstract] cytokine activities agreement supports pathway-level interpretation of CCC predictions: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance"
- [intro] SingleCellSignalR scoring underpinning the per-sample tensor: "LRscore—a regularised score calculated using the squared expression of the transmitter and receiver"
- [readme] CCC inference from single-cell transcriptomics enables study of coordinated biological processes across cell types, the basis for factor-level interpretation: [evidence span withheld — anchor status: not_found]
- [discussion] method and resource choice impacts the predictions the dotplot visualises: "Our results suggest that both the method and resource can considerably impact CCC inference predictions"
