---
name: marker-gene-expression-projection
description: Use when a PAGA-clustered single-cell reprogramming dataset needs visual
  validation that annotated clusters show expected marker enrichment, e.g., Apoa1
  in iEP clusters and Col1a2 in fibroblast clusters.
when_to_use_negative:
- Input is raw Cell Ranger output that has not been normalized, filtered, or embedded
  — projection requires an embedded, clustered graph first
- The goal is cluster-free marker quantification (e.g., differential expression testing)
  rather than visual projection onto a graph layout
- Dataset lacks fibroblast-to-iEP lineage structure where Apoa1/Col1a2 counter-expression
  is expected as a validation signal
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellOracle
  role: Python framework used to load, normalize, and embed the reprogramming dataset,
    run PAGA, and compute the force-directed layout onto which marker expression is
    projected
  repo: https://github.com/morris-lab/CellOracle
- name: Cell Ranger 6.0.1
  role: Upstream pipeline that processes the 10x Chromium data into the count matrix
    used for embedding and marker projection
- name: 10x Chromium
  role: Platform that generated the single-cell libraries underlying the count matrix
- name: Python
  role: Runtime environment (3.8, tested with 3.10) in which CellOracle-based projection
    is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:3fc3f0c66ec06a2c4e8a8e3072f806902ffb0e1f35557c2efe7a2cff5b3e88ae
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - 10x Chromium
  - Cell Ranger 6.0.1
  - CellOracle
  - Python
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# marker-gene-expression-projection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Projects marker gene expression (e.g., Apoa1 as an iEP marker and Col1a2 as a fibroblast marker) onto a force-directed single-cell graph layout derived from a PAGA-partitioned MEF-to-iEP reprogramming dataset, enabling visual validation that clusters recapitulate expected lineage trajectories as in Figure 1F.

## When to use

Use when a PAGA-clustered single-cell reprogramming dataset needs visual validation that annotated clusters show expected marker enrichment, e.g., Apoa1 in iEP clusters and Col1a2 in fibroblast clusters.

## When NOT to use

- Input is raw Cell Ranger output that has not been normalized, filtered, or embedded — projection requires an embedded, clustered graph first
- The goal is cluster-free marker quantification (e.g., differential expression testing) rather than visual projection onto a graph layout
- Dataset lacks fibroblast-to-iEP lineage structure where Apoa1/Col1a2 counter-expression is expected as a validation signal

## Inputs

- 27,663-cell, 8-time-point MEF-to-iEP scRNA-seq count matrix (10x Chromium, Cell Ranger v6.0.1 output)
- PAGA-partitioned neighbor graph with 15 annotated clusters
- Force-directed graph layout coordinates (e.g., CellOracle/scRNA embedding output)
- Marker gene list: Apoa1 (iEP marker), Col1a2 (fibroblast marker)

## Outputs

- Force-directed layout figure with cells colored by Apoa1 expression projection
- Force-directed layout figure with cells colored by Col1a2 expression projection
- Reproduction of Figure 1F structure: 15 clusters with marker-enriched regions corresponding to iEP and fibroblast identities

## How to apply

Start from the normalized, filtered 27,663-cell, 8-time-point MEF-to-iEP scRNA-seq count matrix processed with Cell Ranger v6.0.1 and embedded in the CellOracle (Python 3.8/3.10) environment. Run PAGA on the neighbor graph to partition cells into 15 manually annotated clusters grouped into five cell types (fibroblasts, early transition, transition, dead-end, and reprogrammed iEPs), then compute a force-directed graph layout on the PAGA-partitioned single-cell graph. Extract Apoa1 and Col1a2 expression vectors per cell and color cells by these marker signals on the force-directed layout, replicating the Figure 1F panel. Decision point: if a marker is absent or lowly detected in the count matrix, confirm gene symbol matching before projection. Validation is expert visual review against the published panel rather than a purely numeric threshold.

## Related tools

- **CellOracle** (Python framework used to load, normalize, and embed the reprogramming dataset, run PAGA, and compute the force-directed layout onto which marker expression is projected) — https://github.com/morris-lab/CellOracle
- **Cell Ranger 6.0.1** (Upstream pipeline that processes the 10x Chromium data into the count matrix used for embedding and marker projection)
- **10x Chromium** (Platform that generated the single-cell libraries underlying the count matrix)
- **Python** (Runtime environment (3.8, tested with 3.10) in which CellOracle-based projection is executed)

## Examples

```
import celloracle as co; oracle = co.load_h5ad("mef_iep.oracle.h5ad"); co.pl.embedding(adata=oracle.adata, basis="force_directed", color=["Apoa1", "Col1a2"])
```

## Evaluation signals

- The projected figure contains exactly 15 PAGA clusters matching the manual annotation
- Apoa1 signal is visibly enriched in reprogrammed iEP clusters and depleted in fibroblast clusters
- Col1a2 signal is visibly enriched in fibroblast clusters and depleted in iEP/dead-end clusters
- Expert review confirms the layout reproduces the structure of Figure 1F from the published panel

## Limitations

- Validation relies on expert visual comparison against the published Figure 1F rather than an automated numeric threshold
- Projection depends on upstream choices (normalization, filtering, neighbor graph, PAGA resolution) that affect cluster boundaries and layout geometry

## Evidence

- [other] Source card workflow description: [evidence span withheld — anchor status: not_found]
- [other] Source card marker projection step: [evidence span withheld — anchor status: not_found]
- [other] Source card validation criterion: [evidence span withheld — anchor status: not_found]
