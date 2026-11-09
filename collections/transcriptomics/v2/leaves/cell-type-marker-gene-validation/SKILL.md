---
name: cell-type-marker-gene-validation
description: Use when scBridge (or a baseline such as scJoint, Seurat or GLUE) has
  produced predicted cell-type labels on target scATAC/ADT data and you need marker-based
  evidence that the predicted populations (e.g. Naive vs Effector CD4+ T cells) are
  biologically correct.
when_to_use_negative:
- When no marker genes or protein (ADT) measurements are available for the predicted
  cell types — the check has no biological anchor
- When the goal is to compare method performance quantitatively (use label transfer
  accuracy and silhouette score instead)
- When source annotations themselves are under test (corrupted labels cannot be validated
  by markers derived from those same annotations)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scBridge
  role: Produces the transferred labels (obs["Prediction"]) on target cells that are
    subsequently validated with marker expression
  repo: https://github.com/XLearning-SCU/scBridge
- name: Python
  role: Implementation environment (PyTorch-based) for subsetting predicted cell groups
    and generating marker plots
- name: Signac
  role: Tool used to transform scATAC-seq peak data into the gene activity matrix
    from which marker activity is read
  repo: https://github.com/stuart-lab/signac
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:c9c76b3bf90142ba215f92fb370d5fbc571ddcd7e12b452523987e7f4f1c40d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - GLUE
  - PyTorch
  - Python
  - Signac
  - scBridge
  - scJoint
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# Cell-type marker gene validation

## Summary

Biological sanity check of transferred cell-type labels by comparing known marker abundance (e.g. protein/ADT markers such as CD45RA/CCR7 and CD45RO/PLEKHG3 on PBMC CITE-seq/ASAP-seq data) across predicted cell populations. It confirms that label-transfer predictions reflect genuine marker biology rather than integration artifacts.

## When to use

Use when scBridge (or a baseline such as scJoint, Seurat or GLUE) has produced predicted cell-type labels on target scATAC/ADT data and you need marker-based evidence that the predicted populations (e.g. Naive vs Effector CD4+ T cells) are biologically correct.

## When NOT to use

- When no marker genes or protein (ADT) measurements are available for the predicted cell types — the check has no biological anchor
- When the goal is to compare method performance quantitatively (use label transfer accuracy and silhouette score instead)
- When source annotations themselves are under test (corrupted labels cannot be validated by markers derived from those same annotations)

## Inputs

- Target-cell h5ad (ASAP-seq) with transferred labels in obs["Prediction"] from scBridge
- PBMC CITE-seq/ASAP-seq data (GSE156478) with protein (ADT) abundance or gene activity matrices for marker genes
- Marker panel list, e.g. CD45RA, CCR7 (Naive CD4+ T cells) and CD45RO, PLEKHG3 (Effector/Memory CD4+ T cells)

## Outputs

- Marker expression comparison plot (e.g. marker_validation.png) contrasting marker abundance across predicted cell groups
- Qualitative pass/fail assessment of whether predicted labels align with expected marker biology

## How to apply

After running scBridge to obtain transferred annotations in obs["Prediction"], subset the target cells predicted as Naive CD4+ T cells and Effector CD4+ T cells, then compare marker abundance between these groups. For the PBMC CITE-seq/ASAP-seq example, Naive cells are validated by CD45RA/CCR7 expression and Effector/Memory cells by CD45RO/PLEKHG3 expression, plotted (e.g. marker_validation.png) from the protein/activity data carried in the h5ad objects. Correctly validated labels should show enriched Naive markers in the Naive-predicted group and enriched Effector/Memory markers in the Effector-predicted group; disagreement indicates over- or under-integration of the transferred labels. This step complements quantitative metrics (label transfer accuracy, silhouette score) with a biological check.

## Related tools

- **scBridge** (Produces the transferred labels (obs["Prediction"]) on target cells that are subsequently validated with marker expression) — https://github.com/XLearning-SCU/scBridge
- **Python** (Implementation environment (PyTorch-based) for subsetting predicted cell groups and generating marker plots)
- **Signac** (Tool used to transform scATAC-seq peak data into the gene activity matrix from which marker activity is read) — https://github.com/stuart-lab/signac

## Examples

```
python main.py --data_path="PBMC/" --source_data="CITE-seq.h5ad" --target_data="ASAP-seq.h5ad"  # then plot CD45RA/CCR7 vs CD45RO/PLEKHG3 for Naive vs Effector CD4+ predicted cells -> marker_validation.png
```

## Evaluation signals

- Naive-predicted CD4+ T cells show higher CD45RA and CCR7 than Effector-predicted cells
- Effector/Memory-predicted CD4+ T cells show higher CD45RO and PLEKHG3 than Naive-predicted cells
- Marker-validated concordance should be consistent across all label-corruption rates (0/5/10/20%) tested
- Marker-based grouping agrees with the majority-vote label in obs["Prediction"] for the two CD4+ T cell populations

## Limitations

- Validation relies on marker availability and cell-type specificity; markers like PLEKHG3 may not discriminate all Effector subtypes
- Marker evidence is qualitative and cannot substitute for the label transfer accuracy and silhouette score benchmarks
- Investigators were not blinded to allocation during library preparation, experiments, or analysis, so marker-based validation is not an unbiased readout

## Evidence

- [discussion] Protein-data integration: "the protein data from the human peripheral blood mononuclear dataset"
- [readme] scBridge predictions stored per cell: "The source data should include the cell type annotation in `obs.["CellType"]`."
- [discussion] PBMC CITE-seq/ASAP-seq provenance: "The original CITE-seq data, ASAP-seq data, and fragments file are provided in GSE15647838"
