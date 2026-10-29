---
name: python-object-serialization-of-network-models
description: Use when a CellOracle workflow has produced one Links object per cell
  cluster and each must be saved with its edge-weight table, then validated as a loadable
  pickled object with a non-empty pruned edge list.
when_to_use_negative:
- The base GRN (pre-pruning TF-target edge list from scATAC-seq data) still needs
  filtering by cluster expression — serialize only the final pruned Links objects
- The data are raw count matrices (e.g. the Biddy et al. 2018 scRNA-seq expression
  matrix) rather than fitted network models
- A language-agnostic, schema-versioned export is required; pickle files are Python-version-sensitive
  and not portable outside the CellOracle/Python 3.8–3.10 environment
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellOracle
  role: Source of the Links objects being serialized; provides the pickled GRN model
    format
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Runtime for pickle-based serialization of Links objects and edge-weight tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:5f8908ef8b5051800413bac78bab59f11eb886e9bb49141f4e3fb6f1d7da5eca
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
  - CellOracle
  - Python
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# python-object-serialization-of-network-models

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Persist per-cluster CellOracle Links objects (pruned, weighted GRN configurations) and their edge-weight tables to disk as Python pickle files (e.g. cluster_grn_links.pkl) so inferred network models can be reloaded for downstream perturbation simulation and graph analysis.

## When to use

Use when a CellOracle workflow has produced one Links object per cell cluster and each must be saved with its edge-weight table, then validated as a loadable pickled object with a non-empty pruned edge list.

## When NOT to use

- The base GRN (pre-pruning TF-target edge list from scATAC-seq data) still needs filtering by cluster expression — serialize only the final pruned Links objects
- The data are raw count matrices (e.g. the Biddy et al. 2018 scRNA-seq expression matrix) rather than fitted network models
- A language-agnostic, schema-versioned export is required; pickle files are Python-version-sensitive and not portable outside the CellOracle/Python 3.8–3.10 environment

## Inputs

- Per-cluster CellOracle Links objects containing pruned, weighted cluster-specific GRNs
- Exported edge-weight tables for each cluster's retained TF-target edges

## Outputs

- cluster_grn_links.pkl — pickled CellOracle Links object(s) with non-empty pruned edge lists
- Edge-weight tables loadable for every cell cluster

## How to apply

After assembling one CellOracle Links object per Louvain cluster in the MEF-to-iEP reprogramming dataset, serialize each object together with its exported edge-weight tables using Python's pickle module into a single file such as cluster_grn_links.pkl. Serialization should happen only after the Links object contains the pruned, regression-weighted edge list for that cluster, so the stored model reflects the cluster-specific GRN configuration rather than the raw base GRN from the Mouse sci-ATAC-seq Atlas. CellOracle runs under Python 3.8 (tested through 3.10), so pickle compatibility should be checked against the environment used for the regularized regression network-inference step. On reload, deserialize with pickle.load and confirm the object type is a loadable CellOracle Links instance before feeding it into downstream graph-theory or in silico TF perturbation simulation steps.

## Related tools

- **CellOracle** (Source of the Links objects being serialized; provides the pickled GRN model format) — https://github.com/morris-lab/CellOracle
- **Python** (Runtime for pickle-based serialization of Links objects and edge-weight tables)

## Examples

```
import pickle; pickle.dump(links, open('cluster_grn_links.pkl', 'wb')); links2 = pickle.load(open('cluster_grn_links.pkl', 'rb'))
```

## Evaluation signals

- One Links object and one edge-weight table exist for every Louvain cell cluster in the trajectory
- Each stored file deserializes (pickle.load) into a loadable CellOracle Links object
- Every loaded Links object has a non-empty pruned edge list
- No edge in a cluster's Links object targets genes absent from that cluster's expression matrix
- Reloaded edge-weight tables match the values exported before serialization (round-trip integrity check)

## Limitations

- Pickle files are Python- and library-version-sensitive; a Links object pickled under one CellOracle release may fail to load under another
- Serialization preserves the model but not the fit; the pruned edge list is only as valid as the cluster-specific filtering and regularized regression steps that produced it

## Evidence

- [methods] save_after_links_fit: [evidence span withheld — anchor status: not_found]
- [methods] loadable_links_validation: [evidence span withheld — anchor status: not_found]
- [intro] pruned_cluster_grns: [evidence span withheld — anchor status: not_found]
