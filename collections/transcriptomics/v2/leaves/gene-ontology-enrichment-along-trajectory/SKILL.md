---
name: gene-ontology-enrichment-along-trajectory
description: Use when you have LatentVelo cell trajectories (z_traj, times from ltv.cell_trajectories)
  for a lineage such as beta cell differentiation and need biological validation of
  the trajectory via enriched GO terms.
when_to_use_negative:
- No latent trajectory has been inferred yet — this skill presupposes z_traj/times
  from ltv.cell_trajectories on a trained LatentVelo model
- The gene list is not trajectory-specific (e.g., a whole-dataset highly variable
  gene list), in which case GO terms will not validate the trajectory
- The lineage of interest lacks known marker biology or GO terms to compare against
  (e.g., uncharacterized terminal states)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GSEApy
  role: Runs GO enrichment (gsea, prerank, ssgsea, enrichr modules) on genes along
    the inferred beta-cell trajectory to validate it
  repo: https://github.com/zqfang/GSEApy
- name: LatentVelo
  role: Infers latent developmental trajectories (z_traj, times via ltv.cell_trajectories)
    whose member genes are tested for GO enrichment
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes the velocity graph (scv.tl.velocity_graph with vkey='spliced_velocity')
    used to plot and sanity-check the trajectories being validated
- name: AnnData
  role: Holds the spliced/unspliced layers and celltype annotations consumed by the
    LatentVelo pipeline that produces the trajectory gene lists
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:00081f40b3e9868e46dc36710798017209d3693a32b8ca01cc8a8c4bfb949186
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  repo_url: https://github.com/zqfang/GSEApy
  tools:
  - AnnData
  - GSEApy
  - LatentVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-ontology-enrichment-along-trajectory

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate latent developmental trajectories inferred by LatentVelo (e.g., the Pax4-high beta versus Arx-low alpha trajectories in pancreatic endocrinogenesis) by running GO enrichment with GSEApy on genes along the trajectory, expecting biologically coherent terms such as regulation of insulin secretion and beta cell development.

## When to use

Use when you have LatentVelo cell trajectories (z_traj, times from ltv.cell_trajectories) for a lineage such as beta cell differentiation and need biological validation of the trajectory via enriched GO terms.

## When NOT to use

- No latent trajectory has been inferred yet — this skill presupposes z_traj/times from ltv.cell_trajectories on a trained LatentVelo model
- The gene list is not trajectory-specific (e.g., a whole-dataset highly variable gene list), in which case GO terms will not validate the trajectory
- The lineage of interest lacks known marker biology or GO terms to compare against (e.g., uncharacterized terminal states)

## Inputs

- LatentVelo trajectory outputs z_traj and times from ltv.cell_trajectories on the trained AnnotVAE model
- Latent AnnData with spliced_velocity velocities and latent time per cell (from ltv.output_results)
- Gene list or ranked gene list along the beta-cell trajectory (e.g., .rnk format for gseapy prerank)
- GO gene set library in GMT format or an Enrichr library name (e.g., GO_Biological_Process)

## Outputs

- GO enrichment results (enriched terms, p-values, enrichment scores) for genes along the beta-cell trajectory
- Validation evidence: enrichment of 'regulation of insulin secretion' and 'beta cell development' terms
- Optional publication-quality enrichment figures from GSEApy

## How to apply

After training the AnnotVAE model and exporting results with ltv.output_results (gene_velocity=True, decoded=True, embedding='umap'), call ltv.cell_trajectories(model, adata) to obtain z_traj and times describing the inferred developmental path. Select the genes associated with the trajectory of interest (e.g., the Pax4-high beta cell trajectory), rank or subset them, and run GO enrichment with GSEApy (e.g., the gseapy enrichr or prerank modules with a GO Biological Process gene set library). Judge success by recovery of expected terms: for the beta cell trajectory, 'regulation of insulin secretion' and 'beta cell development' should be enriched, and the beta versus alpha trajectories should reconcile with marker biology (Pax4-high beta, Arx-low alpha). GSEApy is used because it performs GO enrichment inside the Python console without switching to R, producing publication-quality figures.

## Related tools

- **GSEApy** (Runs GO enrichment (gsea, prerank, ssgsea, enrichr modules) on genes along the inferred beta-cell trajectory to validate it) — https://github.com/zqfang/GSEApy
- **LatentVelo** (Infers latent developmental trajectories (z_traj, times via ltv.cell_trajectories) whose member genes are tested for GO enrichment) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes the velocity graph (scv.tl.velocity_graph with vkey='spliced_velocity') used to plot and sanity-check the trajectories being validated)
- **AnnData** (Holds the spliced/unspliced layers and celltype annotations consumed by the LatentVelo pipeline that produces the trajectory gene lists)

## Examples

```
import gseapy; enr = gseapy.enrichr(gene_list=beta_traj_genes, gene_sets='GO_Biological_Process_2023', outdir='go_beta_traj')
```

## Evaluation signals

- Enrichment of 'regulation of insulin secretion' among genes along the beta-cell trajectory
- Enrichment of 'beta cell development' terms along the same trajectory
- Consistency of trajectories with marker biology: Pax4-high beta cell trajectory and Arx-low alpha cell trajectory
- Velocity stream plot on the latent UMAP (scv.pl.velocity_embedding_stream colored by latent_time) reproduces the expected progenitor-to-terminal endocrine topology (alpha, beta, delta, epsilon states)

## Limitations

- Trajectory validity depends on LatentVelo's assumption of a single initial latent state from which all cell trajectories start and reachability by continuous trajectories
- Sparse cell clusters with few observed transient cells can cause latent trajectories to start from unexpected initial states (e.g., the disconnected microglia cluster in dentate gyrus), undermining downstream GO validation
- LatentVelo may require hyperparameter adjustment (larger latent dimension, cell-type annotated AnnotVAE, or gene subsetting) before trajectories are reliable enough for enrichment testing

## Evidence

- [readme] GSEApy enables GO validation of LatentVelo trajectories in Python: [evidence span withheld — anchor status: external_doc]
- [readme] GSEApy supports convenient GO enrichment: [evidence span withheld — anchor status: external_doc]
