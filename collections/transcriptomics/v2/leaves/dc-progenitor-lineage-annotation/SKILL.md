---
name: dc-progenitor-lineage-annotation
description: Use when scRNA-seq of FACS-sorted dendritic cell progenitors (MDP, CDP,
  preDC subsets) needs trajectory reconstruction plus annotation of which preDC branches
  are exclusively committed to cDC1 versus cDC2 fates.
when_to_use_negative:
- Developmental processes starting from multiple independent progenitor cell types
  — Mpath does not yet support these (planned future extension)
- Highly heterogeneous single-cell datasets in which specific subset information is
  not known — Mpath depends on signature genes of differentiated cell types and FACS-sorted
  terminal subsets for fate prediction
- Datasets lacking end-stage contrast data from which to derive mature subset signature
  genes
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Mpath
  role: R package that derives multi-branching developmental trajectories via neighborhood-based
    cell state transitions, builds the aggregation tree, and predicts cell fate by
    travelling back in time from differentiated subsets
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Runtime environment for the Mpath R package and for the ANOVA/Tukey/Benjamini-Hochberg
    DEG derivation of signature genes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:4fd4b96a023aed3c2c0dbb5797b442b72a5bb6011647f43a80ef356bc17a59f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# dc-progenitor-lineage-annotation

## Summary

Reconstructs the multi-branching mouse cDC developmental trajectory (MDP → CDP → preDC) from bone marrow progenitor scRNA-seq using Mpath's neighborhood-based cell state transitions, and annotates lineage commitment of preDC branches using mature cDC1/cDC2 signature genes.

## When to use

Use when scRNA-seq of FACS-sorted dendritic cell progenitors (MDP, CDP, preDC subsets) needs trajectory reconstruction plus annotation of which preDC branches are exclusively committed to cDC1 versus cDC2 fates.

## When NOT to use

- Developmental processes starting from multiple independent progenitor cell types — Mpath does not yet support these (planned future extension)
- Highly heterogeneous single-cell datasets in which specific subset information is not known — Mpath depends on signature genes of differentiated cell types and FACS-sorted terminal subsets for fate prediction
- Datasets lacking end-stage contrast data from which to derive mature subset signature genes

## Inputs

- Mouse bone marrow DC progenitor scRNA-seq expression matrix from GEO accession GSE60781 (MDP, CDP, preDC FACS-sorted populations)
- Mature cDC1- and cDC2-specific signature gene lists derived from end-stage contrasts of GSE60782
- FACS-sorted population labels for landmark cluster designation (MDP, CDP, preDC stages)

## Outputs

- Mpath aggregation tree mapping the multi-branching MDP→CDP→preDC developmental trajectory
- 11-cluster partition with 8 FACS-landmark clusters assigned
- Per-cell fate predictions assigning preDC branches as exclusively cDC1- or cDC2-committed
- Branching event localization at the early preDC stage (preDC_a)

## How to apply

Load the mouse bone marrow DC progenitor scRNA-seq expression matrix (GEO GSE60781) into R and derive mature cDC1- and cDC2-specific signature genes from GSE60782 by differential expression on end-stage contrasts, identifying DEGs with ANOVA followed by post-hoc Tukey's test and Benjamini-Hochberg P-value adjustment. Run Mpath in R to aggregate cells via neighborhood-based cell state transitions, using the count of neighbouring cells to estimate potential state transitions and deliberately imposing no a priori temporal ordering of the developmental subsets. Partition cells into 11 clusters and designate the 8 landmark clusters corresponding to the FACS-sorted populations (MDP, CDP, and the preDC stages preDC_a, preDC_b, preDC_c, preDC_d), then build the aggregation tree spanning MDP through CDP to preDC. Predict individual cell fate by travelling back in time from the differentiated cDC1 and cDC2 subsets using the derived signature genes. The decision point is the branching event at the early preDC stage: the branch spanning preDC_a, preDC_b and preDC_c should be exclusively cDC2-committed, while the branch spanning preDC_a and preDC_d should be exclusively cDC1-committed.

## Related tools

- **Mpath** (R package that derives multi-branching developmental trajectories via neighborhood-based cell state transitions, builds the aggregation tree, and predicts cell fate by travelling back in time from differentiated subsets) — JinmiaoChenLab/Mpath
- **R** (Runtime environment for the Mpath R package and for the ANOVA/Tukey/Benjamini-Hochberg DEG derivation of signature genes)

## Examples

```
Rscript -e "library(Mpath)"  # build aggregation tree over 11 clusters (8 FACS landmarks) from the GSE60781 expression matrix
```

## Evaluation signals

- The 11-cluster / 8-landmark-cluster structure is recovered, with the 8 landmark clusters matching the FACS-sorted MDP, CDP, and preDC populations
- The trajectory recapitulates the expected MDP-to-CDP-to-preDC progression with a branching event at the early preDC stage
- Fate prediction assigns the preDC_a–preDC_b–preDC_c branch exclusively to cDC2 and the preDC_a–preDC_d branch exclusively to cDC1
- Signature genes were derived only from end-stage contrasts (ANOVA + post-hoc Tukey's test, Benjamini-Hochberg adjustment) without using a priori temporal ordering of developmental subsets
- Early subset-commitment markers emerge from the late CDP stage onwards, consistent with the reported biology

## Limitations

- Mpath does not yet support developmental processes starting from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell datasets in which specific subset information is not known (planned future extension)
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction

## Evidence

- [abstract] Mpath derives trajectories via neighborhood transitions: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [abstract] Branching event reveals cDC1/cDC2-committed preDC subsets: "The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages."
- [discussion] Fate prediction by travelling back in time: "The algorithm travels back in time from differentiated cell subsets and maps the developmental path from known terminal differentiated cell subsets back to its progenitors."
- [supplementary] DEG identification statistics: "The DEGs were identified using ANOVA followed by post-hoc Tukey's test."
- [supplementary] Multiple-testing correction method: "P values were adjusted for multi-test correction using Benjamini-Hochberg."
- [readme] Mpath README description: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
