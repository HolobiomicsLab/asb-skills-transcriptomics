---
name: minimum-spanning-tree-network-trimming
description: Use when a complete weighted graph over cell states (landmarks) has been
  built from transitional-cell counts and must be reduced to a sparse, connected multi-branching
  trajectory network retaining only maximum-support edges.
when_to_use_negative:
- Development processes starting from multiple independent progenitor cell types —
  the article states Mpath does not yet support this and lists it as a planned future
  extension
- Highly heterogeneous single-cell data sets in which specific subset information
  is not known — the article notes Mpath is not yet extended to this setting
- Graphs whose nodes are not biologically meaningful landmark states with transitional-cell
  support, since the −w(e) MST heuristic presumes edge weights are cell counts
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Mpath
  role: Implements neighborhood-based cell state transition counting and MST-based
    network trimming to derive multi-branching developmental trajectories
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Runtime environment for the Mpath R package used to execute the trimming workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:874122911c8ec2b9ce211642bc74ade3543afaab734c58378645526ba588c8cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# minimum-spanning-tree-network-trimming

## Summary

Trims a complete landmark-to-landmark graph by computing a minimum spanning tree over edge distances defined as the negative count of transitional cells (−w(e)), so that only the highest-support edges survive. This is the graph-pruning step of Mpath that converts neighborhood-based cell state transition counts into a connected, multi-branching developmental skeleton.

## When to use

Use when a complete weighted graph over cell states (landmarks) has been built from transitional-cell counts and must be reduced to a sparse, connected multi-branching trajectory network retaining only maximum-support edges.

## When NOT to use

- Development processes starting from multiple independent progenitor cell types — the article states Mpath does not yet support this and lists it as a planned future extension
- Highly heterogeneous single-cell data sets in which specific subset information is not known — the article notes Mpath is not yet extended to this setting
- Graphs whose nodes are not biologically meaningful landmark states with transitional-cell support, since the −w(e) MST heuristic presumes edge weights are cell counts

## Inputs

- Landmark-labeled low-dimensional expression matrix from the upstream Mpath landmark-definition stage
- Landmark cell definitions (landmark identities for all cells, e.g. Mpath landmarks such as preDC_a, preDC_b, preDC_c, CDP)
- Per-cell landmark assignments (two nearest landmarks per cell) or the data to compute them

## Outputs

- transition_network.csv: trimmed state transition network as an edge list with columns (landmark_a, landmark_b, transition-cell weight)
- A connected spanning graph over the full landmark set forming a multi-branching skeleton

## How to apply

Starting from the landmark-labeled low-dimensional expression matrix, assign each cell to the pair of its two nearest landmarks so each cell votes for exactly one candidate edge. Compute the edge weight w(e) for every landmark pair as the count of cells whose two nearest landmarks are that pair, yielding a complete weighted graph over landmarks. Because higher cell counts indicate stronger state transitions, convert weights to distances as −w(e) and compute the minimum spanning tree: this heuristic removes low-support edges while keeping all landmarks connected and retaining the maximum number of cells on retained edges. Write the trimmed network as an edge list (landmark_a, landmark_b, transition-cell weight). The rationale, per Mpath, is that count of neighbouring cells estimates potential state transitions, and MST trimming reconstructs the multi-branching state transition network without requiring a priori temporal ordering of subsets.

## Related tools

- **Mpath** (Implements neighborhood-based cell state transition counting and MST-based network trimming to derive multi-branching developmental trajectories) — JinmiaoChenLab/Mpath
- **R** (Runtime environment for the Mpath R package used to execute the trimming workflow)

## Examples

```
Rscript -e 'library(Mpath); mpath_obj <- Mpath(landmark_data, landmarks); net <- constructMST(mpath_obj); write.csv(net$edges, "transition_network.csv", row.names=FALSE)'
```

## Evaluation signals

- transition_network.csv exists and parses as an edge list with landmark_a, landmark_b, and weight columns
- Retained edges form a connected spanning structure over the full landmark set (no landmark is disconnected)
- Every retained edge has a transition-cell weight of at least 1 (each edge supported by at least one transitional cell)
- Edge count equals |landmarks| − 1, the expected size of a minimum spanning tree over the landmark graph
- No low-support edge present in the complete graph but excluded by the MST appears in the output, confirming trimming removed low-count edges

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension)
- Demonstrated utility limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts

## Evidence

- [discussion] Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping: "Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping"
- [abstract] Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions: "Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions"
- [discussion] without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts.: "without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts."
- [discussion] The algorithm travels back in time from differentiated cell subsets and maps the developmental path from known terminal differentiated cell subsets back to its progenitors.: "The algorithm travels back in time from differentiated cell subsets and maps the developmental path from known terminal differentiated cell subsets back to its progenitors."
- [readme] Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
