---
name: neighborhood-based-transition-edge-weighting
description: Use when reconstructing cell-state transition graphs from landmark- or
  cluster-labeled single-cell data, where edge support should be quantified by counting
  cells whose two nearest landmark states define each candidate edge.
when_to_use_negative:
- The developmental process starts from multiple independent progenitor cell types
  — Mpath does not yet support this and the authors list it as a planned future extension.
- Specific subset information is unknown in a highly heterogeneous dataset — the article
  notes Mpath is not yet extended to such settings.
- Terminal differentiated subsets or end-stage contrast signature genes are unavailable,
  since Mpath relies on them to predict individual cell fate via travelling back in
  time.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: 'R package that performs neighborhood-based cell state transition mapping:
    nearest-landmark assignment, edge weighting by transitional cell counts, and MST-based
    network trimming'
  repo: JinmiaoChenLab/Mpath
- name: R
  role: runtime environment for executing the Mpath package and writing the trimmed
    transition_network.csv edge list
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:68830147cc0a2ef95897b0b3c6acfba7e199fdd61f4b1cd78e81cf704056e68f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# neighborhood-based-transition-edge-weighting

## Summary

Mpath's method for building a landmark state-transition network: each pooled cell is assigned to the pair of its two nearest landmarks, and the number of cells supporting each landmark pair becomes that edge's weight; a minimum spanning tree over -w(e) then trims the complete weighted graph to a multi-branching skeleton.

## When to use

Use when reconstructing cell-state transition graphs from landmark- or cluster-labeled single-cell data, where edge support should be quantified by counting cells whose two nearest landmark states define each candidate edge.

## When NOT to use

- The developmental process starts from multiple independent progenitor cell types — Mpath does not yet support this and the authors list it as a planned future extension.
- Specific subset information is unknown in a highly heterogeneous dataset — the article notes Mpath is not yet extended to such settings.
- Terminal differentiated subsets or end-stage contrast signature genes are unavailable, since Mpath relies on them to predict individual cell fate via travelling back in time.

## Inputs

- landmark-labeled low-dimensional expression matrix (cells with assigned landmark states) from the upstream Mpath landmark-definition stage
- landmark cell definitions (the set of landmark states)
- pooled single-cell dataset of transitional and landmark cells

## Outputs

- trimmed multi-branching state transition network as an edge list (landmark_a, landmark_b, transition-cell weight), e.g. transition_network.csv

## How to apply

Load the landmark-labeled low-dimensional expression matrix and landmark cell definitions from the upstream Mpath landmark-definition stage (Mpath R package under R). For every cell in the pooled dataset, compute its two nearest landmarks and record that landmark pair, placing the cell 'between' the two landmarks that define one candidate edge. Define each edge weight w(e) as the count of cells whose two nearest landmarks are that pair, yielding a complete weighted graph over landmarks. Trim this graph by computing the minimum spanning tree over distances -w(e), which retains only the highest-support edges while keeping all landmarks connected in a multi-branching skeleton. Write the resulting trimmed network as an edge list of (landmark_a, landmark_b, transition-cell weight) triples.

## Related tools

- **Mpath** (R package that performs neighborhood-based cell state transition mapping: nearest-landmark assignment, edge weighting by transitional cell counts, and MST-based network trimming) — JinmiaoChenLab/Mpath
- **R** (runtime environment for executing the Mpath package and writing the trimmed transition_network.csv edge list)

## Evaluation signals

- transition_network.csv exists and its edges form a connected spanning structure over the full landmark set
- every retained edge in the trimmed network is supported by at least one transitional cell (weight >= 1)
- the trimmed graph spans all landmarks while retaining the maximum number of cells on edges, consistent with an MST over distances -w(e)
- the resulting network is multi-branching rather than a linear chain, capable of representing multiple cell fates from a single progenitor

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension).
- Mpath is not yet extended to highly heterogeneous single-cell datasets where specific subset information is unknown (planned future extension).
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, constraining the upstream landmark setup feeding this step.

## Evidence

- [discussion] neighborhood-based edge weighting: "Mpath uses count of neighbouring cells to estimate potential state transition."
- [abstract] neighborhood-based derivation: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [readme] multi-branching reconstruction: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
- [discussion] multi-branching capability: "The Mpath algorithm is able to resolve multi-branching single-cell developmental trajectories, allowing for multiple cell fates stemming from a single progenitor cell type."
