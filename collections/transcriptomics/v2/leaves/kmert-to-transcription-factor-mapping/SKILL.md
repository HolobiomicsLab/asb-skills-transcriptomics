---
name: kmert-to-transcription-factor-mapping
description: Use when a scATAC-seq trajectory has been reconstructed with STREAM using
  k-mer features and you need to identify which transcription factors drive each detected
  branch.
when_to_use_negative:
- Mapping newly profiled cells whose fates are totally different from those in the
  reference structure — the article notes the mapping procedure may not be optimal
  in this case
- Trajectory topology is cyclic or disconnected — STREAM/ElPiGraph support for these
  more general structures is not yet adapted
- Single-cell methylation, nucleosome positioning, or protein-level data — informative
  features and distance metrics for these assays are not yet defined
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: STREAM
  role: Runs the branching trajectory reconstruction on k-mer features and ranks k-mers
    along each branch; the k-mer workflow uses the stream_atac add-on in the env_stream
    conda environment
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core algorithm behind STREAM that learns the branching principal graph used
    to define branches and pseudotime for k-mer ranking
  repo: https://github.com/pinellolab/STREAM
- name: chromVARmotifs
  role: Provides curated human/mouse motif collections (PWMatrixList from TFBSTools)
    to map k-mer motifs to transcription factors
  repo: https://github.com/buenrostrolab/chromVARmotifs
- name: anndata
  role: Data class used by STREAM to hold the k-mer count matrix and trajectory results
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:ab27a327786ab613ffa2265186f17fe977ab7450cc8f8bc42b8223c41024056a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  - chromVARmotifs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kmert-to-transcription-factor-mapping

## Summary

Ranks k-mer accessibility features along each STREAM-detected branching trajectory from scATAC-seq data and maps the top k-mers to their associated transcription factors, identifying lineage master regulators (e.g. GATA1 for erythroid, CEBPA for myeloid).

## When to use

Use when a scATAC-seq trajectory has been reconstructed with STREAM using k-mer features and you need to identify which transcription factors drive each detected branch.

## When NOT to use

- Mapping newly profiled cells whose fates are totally different from those in the reference structure — the article notes the mapping procedure may not be optimal in this case
- Trajectory topology is cyclic or disconnected — STREAM/ElPiGraph support for these more general structures is not yet adapted
- Single-cell methylation, nucleosome positioning, or protein-level data — informative features and distance metrics for these assays are not yet defined

## Inputs

- Raw scATAC-seq k-mer count matrix from QC-passed cells (e.g. 2034 human bone-marrow cells) loaded into an anndata object
- STREAM trajectory output with cells assigned to branches and pseudotime
- Optionally: TF motif collections as PWMatrixList objects (chromVARmotifs human_pwms_v1/v2, mouse_pwms_v1/v2, homer_pwms, encode_pwms)

## Outputs

- Ranked k-mer list per detected branch (ranked by dynamics along pseudotime)
- k-mer-to-transcription-factor mapping table
- Trajectory figure annotated with the reconstructed lineage hierarchy (e.g. HSC→MPP→erythroid/lymphoid/myeloid)

## How to apply

After STREAM (built on ElPiGraph) assigns cells to branches with pseudotime, rank the k-mer features along each detected branch by their accessibility deviation dynamics. Take the top-ranked k-mers per branch and map them to their associated transcription factors, optionally using motif collections such as chromVARmotifs (human_pwms_v1/v2 from cisBP, HOMER, ENCODE PWMatrixList objects) to link k-mer motifs to TF names. In the human bone-marrow case this recovers GATA1 as the master regulator of the erythroid branch and CEBPA for the myeloid branch. Save the ranked k-mer-to-TF table alongside the trajectory figure so TF activity can be read directly against the reconstructed HSC→MPP→erythroid/lymphoid/myeloid hierarchy.

## Related tools

- **STREAM** (Runs the branching trajectory reconstruction on k-mer features and ranks k-mers along each branch; the k-mer workflow uses the stream_atac add-on in the env_stream conda environment) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core algorithm behind STREAM that learns the branching principal graph used to define branches and pseudotime for k-mer ranking) — https://github.com/pinellolab/STREAM
- **chromVARmotifs** (Provides curated human/mouse motif collections (PWMatrixList from TFBSTools) to map k-mer motifs to transcription factors) — https://github.com/buenrostrolab/chromVARmotifs
- **anndata** (Data class used by STREAM to hold the k-mer count matrix and trajectory results)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 stream_atac jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- Top k-mers on the erythroid branch map to GATA1 and on the myeloid branch to CEBPA, matching known hematopoietic master regulators
- Trajectory figure recovers the expected hematopoietic hierarchy: HSC through MPP into erythroid, lymphoid, and myeloid branches
- All 2034 QC-passed bone-marrow cells are assigned to a branch with pseudotime values
- Ranked k-mer-to-TF table is produced per branch and saved as an output file alongside the trajectory figure

## Limitations

- Further improvement is necessary to adapt STREAM to cyclic or disconnected graph topologies
- ElPiGraph supports learning complex graphs but more work is necessary to provide intuitive visualizations for sophisticated topologies
- The mapping procedure may not be optimal if new cells have totally different fates compared to the ones present in the reference structure

## Evidence

- [other] Rank k-mers along each detected branch and map the top k-mers to their associated transcription factors.: [evidence span withheld — anchor status: not_found]
- [intro] TF binding dynamics along trajectory: "STREAM also provides a mapping feature and a set of interactive tools to explore and visualize both cell type composition and relevant genes (or transcription factor binding dynamics for scATAC-seq"
- [discussion] Few TI methods applied to scATAC: "so far very few trajectory inference methods (Monocle2 and STREAM) have been applied to analyze single-cell chromatin accessibility"
- [readme] chromVARmotifs purpose: [evidence span withheld — anchor status: external_doc]
- [readme] STREAM supports epigenomic trajectories: "capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data."
