---
name: xgmml-network-export-and-cytoscape-visualization
description: Use when a runmodule result (global or local RNC) needs to be persisted
  as a network file and visually inspected; trigger on any detected module object
  that must be shared, visualized, or validated in Cytoscape.
when_to_use_negative:
- The differential expression or module search has not been run yet — there is no
  module object to export
- The analysis requires a format other than XGMML and no saveNetwork type is available
  for it
- The goal is quantitative module re-scoring rather than visualization/export; rerun
  runmodule instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0602
tools:
- name: fRNC
  role: Provides saveNetwork and savelocalM functions that export detected global
    and local RNC modules as XGMML files
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: Cytoscape
  role: Environment in which the exported XGMML module file is loaded and visually
    inspected
- name: R
  role: Runtime for executing the fRNC export functions on runmodule results
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:8c1f03a67e26169136e24e1c33dae28f016442d9d3ef635cf70842ecb49b8201
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/BioinformaticsSTU/fRNC
  tools:
  - Cytoscape
  - R
  - edgeR
  - fRNC
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# xgmml-network-export-and-cytoscape-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports fRNC-detected RBP-ncRNA circuit (RNC) modules to XGMML files via saveNetwork/savelocalM so the scored sub-networks can be loaded and inspected in the Cytoscape environment.

## When to use

Use when a runmodule result (global or local RNC) needs to be persisted as a network file and visually inspected; trigger on any detected module object that must be shared, visualized, or validated in Cytoscape.

## When NOT to use

- The differential expression or module search has not been run yet — there is no module object to export
- The analysis requires a format other than XGMML and no saveNetwork type is available for it
- The goal is quantitative module re-scoring rather than visualization/export; rerun runmodule instead

## Inputs

- Global module object returned by runmodule(method="global") (res.list_global$module)
- Local module list returned by runmodule(method="local", maxsize, seletN)
- Scored RBP-ncRNA interaction network from interStringency with node_gene_ID/type/target_gene_ID columns
- Node weight table produced by combinp(dataNo[,c('type','logFC','PValue')], islog=TRUE)

## Outputs

- XGMML network file (e.g. ceRNA_module_transcription.XGMML) loadable in Cytoscape
- Local module files written by savelocalM for each user-defined seed node

## How to apply

After runmodule returns the maximum-scoring RNC, write the global module with saveNetwork(res.list_global$module, file=..., type="XGMML"), which emits an XGMML file loadable into Cytoscape for visual inspection of the scored RBP and ncRNA nodes. For local greedy-search results (from runmodule(method="local", maxsize=..., seletN=c(...))), use savelocalM(res.list_local), which saves each seed-anchored module. The XGMML export preserves the node_gene_ID/type/target_gene_ID interaction network and the combinp-derived node weights, so the visualized module reflects both topology and differential-expression scoring. Validation is by checking the emitted XGMML file exists, is Cytoscape-loadable, and contains the nodes returned by the search.

## Related tools

- **fRNC** (Provides saveNetwork and savelocalM functions that export detected global and local RNC modules as XGMML files) — https://github.com/BioinformaticsSTU/fRNC
- **Cytoscape** (Environment in which the exported XGMML module file is loaded and visually inspected)
- **R** (Runtime for executing the fRNC export functions on runmodule results)

## Examples

```
saveNetwork(res.list_global$module, file="ceRNA_module_transcription", type="XGMML"); savelocalM(res.list_local)
```

## Evaluation signals

- The XGMML file exists on disk after saveNetwork returns
- The XGMML file loads without error in Cytoscape
- The exported module contains the scored RBP-ncRNA nodes returned by the global and/or local runmodule searches
- savelocalM emits files for each entry in the local module list (seed-anchored modules)
- Node attributes in the XGMML reflect combinp-derived weights consistent with the input gene2weight table

## Limitations

- Export only covers the module object produced by runmodule; if the search returns an empty module there is nothing meaningful to visualize
- Validation of Cytoscape-loadability depends on an external Cytoscape installation; the R side only guarantees the XGMML file is written

## Evidence

- [other] task validation of XGMML loadability: [evidence span withheld — anchor status: not_found]
