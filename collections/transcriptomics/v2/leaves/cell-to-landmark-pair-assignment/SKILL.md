---
name: cell-to-landmark-pair-assignment
description: Use when a landmark-labeled low-dimensional expression matrix is available
  and you must quantify neighborhood-based cell state transitions between landmark
  pairs to build a weighted transition graph.
when_to_use_negative:
- Landmarks or landmark cell definitions have not yet been defined (run the upstream
  Mpath landmark-definition stage first)
- Cells are still in high-dimensional raw expression space without the low-dimensional
  embedding Mpath operates on
- The analysis starts from multiple independent progenitor cell types — Mpath does
  not yet support such development processes (planned future extension)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: Mpath
  role: 'R package implementing neighborhood-based cell state transition mapping:
    finds each cell''s two nearest landmarks and counts transitional cells to weight
    landmark-pair edges'
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: runtime environment for the Mpath R package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:07920072196fe0bbf2407bbf1326286651f62186c22cd71376ff14c105e7eb35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-to-landmark-pair-assignment

## Summary

Assigns each low-dimensional cell to the pair of Mpath landmark states that are its two nearest landmarks, then counts cells per landmark pair to weight candidate state-transition edges between landmarks.

## When to use

Use when a landmark-labeled low-dimensional expression matrix is available and you must quantify neighborhood-based cell state transitions between landmark pairs to build a weighted transition graph.

## When NOT to use

- Landmarks or landmark cell definitions have not yet been defined (run the upstream Mpath landmark-definition stage first)
- Cells are still in high-dimensional raw expression space without the low-dimensional embedding Mpath operates on
- The analysis starts from multiple independent progenitor cell types — Mpath does not yet support such development processes (planned future extension)

## Inputs

- landmark-labeled low-dimensional expression matrix (e.g., tSNE/embedded coordinates with landmark state labels)
- landmark cell definitions from the Mpath landmark-definition stage

## Outputs

- weighted landmark graph with edge weight w(e) = number of cells whose two nearest landmarks are that pair
- transition_network.csv edge list (landmark_a, landmark_b, transition-cell weight)

## How to apply

Load the landmark-labeled low-dimensional expression matrix and landmark cell definitions produced by the upstream Mpath landmark-definition stage (Mpath R package under R). For every cell in the pooled dataset, compute its two nearest landmarks and record the landmark pair, so each cell sits 'between' the two landmarks that define one candidate edge. Weight each landmark-pair edge w(e) by the count of cells whose two nearest landmarks are that pair, yielding a complete weighted graph over landmarks. This neighborhood-based counting is Mpath's mechanism to estimate potential state transitions; the resulting weights feed the downstream MST trimming step. Write the edge list (landmark_a, landmark_b, transition-cell weight) to transition_network.csv for the trimming stage.

## Related tools

- **Mpath** (R package implementing neighborhood-based cell state transition mapping: finds each cell's two nearest landmarks and counts transitional cells to weight landmark-pair edges) — https://github.com/JinmiaoChenLab/Mpath
- **R** (runtime environment for the Mpath R package)

## Examples

```
Rscript -e "library(Mpath); edges <- apply(embedded[, landmark_cols], 1, function(x) order(dist_to_landmarks(x))[1:2]); w <- table(paste(pmin(edges[1,],edges[2,]), pmax(edges[1,],edges[2,]), sep='_')); write.csv(w, 'transition_network.csv')"
```

## Evaluation signals

- transition_network.csv exists and contains an edge list of (landmark_a, landmark_b, transition-cell weight) triples
- Every retained edge has a weight supported by at least one transitional cell (w(e) >= 1)
- The landmark set over which edges are counted matches the full landmark set from the upstream landmark-definition stage
- Each pooled cell is assigned to exactly one landmark pair (its two nearest landmarks), so summed edge weights equal the total number of cells
- The weighted graph feeds a connected spanning structure after MST trimming (downstream validation)

## Limitations

- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, constraining landmark quality upstream
- Mpath does not yet support development processes starting from multiple independent progenitor cell types
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown
- Demonstrated utility limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts

## Evidence

- [abstract] Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions.: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [discussion] Mpath uses count of neighbouring cells to estimate potential state transition.: "Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping"
- [supplementary] Mpath reference manual provides detailed documentation of Mpath R package.: "Mpath reference manual provides detailed documentation of Mpath R package."
- [discussion] In future, we will extend the Mpath algorithm for development processes starting from multiple independent progenitor cell types: "In future, we will extend the Mpath algorithm for development processes starting from multiple independent progenitor cell types"
