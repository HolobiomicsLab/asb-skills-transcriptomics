---
name: multi-tf-regulome-gradient-comparison
description: Use when direct TF expression in spatial transcriptomics spots is too
  sparse to visualize, but an inferred fibroblast GRN edge list with target genes
  is available and TF regulome spatial patterns must be compared across tissue regions.
when_to_use_negative:
- Direct TF expression in the spatial spots is sufficiently abundant to plot and interpret
  without target-gene aggregation.
- No GRN edge list linking TFs to target genes is available — the regulome scores
  depend entirely on the inferred target-gene sets.
- The tissue of interest lacks defined spatial regions or fibrotic markers (COL1A1/POSTN/FN1)
  to anchor the gradient comparison.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMEGA
  role: Provides GRNSpatialPlot to overlay TF regulome scores onto spatial tissue
    coordinates
  repo: github.com/CostaLab/scMEGA
- name: Seurat
  role: Loads and normalizes the myocardial infarction spatial transcriptomics object
    and hosts the spatial assay used by GRNSpatialPlot
  repo: github.com/satijalab/seurat (branch seurat5)
- name: R
  role: Implementation environment for scMEGA and the spatial regulome workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:3058bf8cd22206102bd5fa6b78f8b6034f234549fe416ab02943a024eecd3007
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - scMEGA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# multi-tf-regulome-gradient-comparison

## Summary

Spatially maps and compares transcription factor regulomes by aggregating mean expression of each TF's GRN target genes per spatial transcriptomics spot, enabling delineation of TF activity gradients (e.g. NR3C2 vs RUNX1 in fibrotic myocardial infarction regions) when direct TF expression is too sparse to detect.

## When to use

Use when direct TF expression in spatial transcriptomics spots is too sparse to visualize, but an inferred fibroblast GRN edge list with target genes is available and TF regulome spatial patterns must be compared across tissue regions.

## When NOT to use

- Direct TF expression in the spatial spots is sufficiently abundant to plot and interpret without target-gene aggregation.
- No GRN edge list linking TFs to target genes is available — the regulome scores depend entirely on the inferred target-gene sets.
- The tissue of interest lacks defined spatial regions or fibrotic markers (COL1A1/POSTN/FN1) to anchor the gradient comparison.

## Inputs

- Inferred fibroblast GRN edge list (df.grn2, filtered at correlation > 0.4, saved as final_grn.Rds)
- scMEGA trajectory object with the target assay
- Kuppe et al. 2022 human myocardial infarction spatial transcriptomics data as a Seurat spatial object
- TF names of interest (e.g. NR3C2, RUNX1) present in the GRN edge list

## Outputs

- Per-spot regulome scores for each TF (mean expression of GRN target genes aggregated per spatial spot)
- Spatial overlay plots of regulome scores rendered via GRNSpatialPlot
- Qualitative comparison of regulome gradients across fibrotic regions (e.g. mutually exclusive NR3C2 vs RUNX1 patterns)

## How to apply

Load the inferred fibroblast GRN edge list (df.grn2, filtered at correlation > 0.4 and saved as final_grn.Rds) together with the scMEGA trajectory object containing the target assay. Load the Kuppe et al. 2022 human myocardial infarction spatial transcriptomics data into a Seurat spatial object and normalize its expression assay. For each TF of interest (e.g. NR3C2 and RUNX1), aggregate the mean expression of its GRN target genes per spatial spot to compute a regulome score, bypassing the sparsity of direct TF expression in spatial spots. Overlay each regulome score onto tissue spatial coordinates with the scMEGA function GRNSpatialPlot(object, assay, df.grn, tf.use, min.cutoff, max.cutoff, vis.option). Compare the resulting regulome plots within fibrotic regions (marker-positive for COL1A1/POSTN/FN1) to identify mutually exclusive spatial gradients — the expected finding is spatially disjoint regulome gradients across the fibrotic zone.

## Related tools

- **scMEGA** (Provides GRNSpatialPlot to overlay TF regulome scores onto spatial tissue coordinates) — github.com/CostaLab/scMEGA
- **Seurat** (Loads and normalizes the myocardial infarction spatial transcriptomics object and hosts the spatial assay used by GRNSpatialPlot) — github.com/satijalab/seurat (branch seurat5)
- **R** (Implementation environment for scMEGA and the spatial regulome workflow)

## Examples

```
GRNSpatialPlot(object = spatial.obj, assay = "Spatial", df.grn = final_grn, tf.use = "NR3C2", min.cutoff = "q5", max.cutoff = "q95", vis.option = "plot1")
```

## Evaluation signals

- Visual inspection confirms the rendered spatial plots display the two regulomes (e.g. NR3C2 and RUNX1) as spatially disjoint gradients across the fibrotic zone.
- Regulome plots localize to fibrotic regions, i.e. spots positive for the myofibroblast markers COL1A1, POSTN and FN1.
- The GRN edge list used as input is the filtered df.grn2 (correlation > 0.4, final_grn.Rds), not the unfiltered network.
- Each regulome score is computed as the mean expression of the TF's GRN target genes aggregated per spatial spot, not from direct TF expression.
- GRNSpatialPlot is called with explicit assay, tf.use, min.cutoff and max.cutoff arguments so score scaling is reproducible across compared TFs.

## Limitations

- Direct TF expression is too sparse in spatial spots to detect, which is why target-gene aggregation is required; the skill cannot be validated on TF expression itself.
- Regulome quality depends on the upstream GRN inference (edge filtering at correlation > 0.4); false target-gene assignments propagate into the spatial scores.
- Software/package versions are not specified in the tutorials (only sessionInfo() is referenced), so exact reproducibility may vary.

## Evidence

- [methods] task workflow: aggregate regulome scores: [evidence span withheld — anchor status: not_found]
- [methods] task workflow: GRNSpatialPlot call: [evidence span withheld — anchor status: not_found]
- [results] task finding: mutually exclusive gradients: [evidence span withheld — anchor status: not_found]
- [methods] article: fibrotic marker cells: "some cells highly express COL1A1, POSTN, and FN1 and they could be terminally differentiated myofibroblasts"
- [readme] readme: scMEGA install: "Install scMEGA
devtools::install_github("CostaLab/scMEGA")"
