---
name: scale-free-topology-assessment
description: Use when a gene co-expression network or module has been inferred (e.g.
  via SCANet/BioNERO WGCNA) and you need to validate that its graph topology is biologically
  plausible before selecting hubs or downstream modules.
when_to_use_negative:
- The input is a transcriptional regulatory network (GRN) rather than a co-expression
  module — SCANet derives GRNs from GCNs via RcisTarget TF-motif validation, and the
  scale-free check applies to the co-expression module graph.
- The network was inferred by a method whose node degree distribution is not expected
  to be scale-free (e.g. COTAN, scLink, or fcoex outputs), not by the modified-WGCNA/BioNERO
  route described here.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: SCANet
  role: Python package whose module-level graph topology check evaluates whether co-expression
    modules fit scale-free topology; also exposes hub detection
  repo: https://github.com/oubounyt/SCANet
- name: BioNERO
  role: Framework used to transform the gene-gene correlation matrix into a WGCNA
    signed adjacency matrix from which modules and their graphs are obtained for topology
    assessment
- name: WGCNA
  role: Underlying modified weighted gene co-expression network analysis method used
    by SCANet for network inference and module detection prior to the topology check
- name: Scanpy
  role: Loads and pre-processes the scRNA-seq matrix and generates pseudo-bulk cells
    (averaging gene expression across sub-clusters) feeding the GCN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:5d125d2ef82e27e808ca951f0c5aabb7637053b577fd9e09c494bc2456414d3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - BioNERO
  - Python
  - SCANet
  - Scanpy
  - WGCNA
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Scale-free Topology Assessment

## Summary

A validation step in SCANet's modified-WGCNA co-expression workflow that checks whether an inferred gene co-expression module's graph fits the scale-free topology characteristic of real-world biological networks, supporting hub-gene identification and module QC.

## When to use

Use when a gene co-expression network or module has been inferred (e.g. via SCANet/BioNERO WGCNA) and you need to validate that its graph topology is biologically plausible before selecting hubs or downstream modules.

## When NOT to use

- The input is a transcriptional regulatory network (GRN) rather than a co-expression module — SCANet derives GRNs from GCNs via RcisTarget TF-motif validation, and the scale-free check applies to the co-expression module graph.
- The network was inferred by a method whose node degree distribution is not expected to be scale-free (e.g. COTAN, scLink, or fcoex outputs), not by the modified-WGCNA/BioNERO route described here.

## Inputs

- Gene co-expression module graph (GCN) derived from a signed WGCNA adjacency matrix built by SCANet via BioNERO
- Preprocessed scRNA-seq pseudo-bulk expression matrix (rows = pseudo-bulk sub-cluster cells, columns = genes) from Scanpy

## Outputs

- Scale-free topology fit verdict per gene co-expression module
- QC-accepted set of modules eligible for hub detection (top 10% most connected genes) and trait association

## How to apply

After SCANet builds the pairwise gene-gene correlation matrix (Pearson's r, Spearman's, or biweight midcorrelation) and converts it into a signed WGCNA adjacency matrix via the BioNERO framework, run the scale-free topology check on each detected module's graph. The test asks whether the module graph fits the scale-free topology, which is 'a characteristic of real-world biological networks'; modules failing this check are candidates for exclusion, guided by the 'Modules selection guidelines' in Supplementary File 1. Complement the check with hub identification (top 10% most connected genes per module) and compare module sizes against reference scales, e.g. the 30-195 genes per module reported for classical monocytes in the Cillo et al. (2021) PBMC data. Accept the GCN only when modules are of comparable scale and pass the topology check.

## Related tools

- **SCANet** (Python package whose module-level graph topology check evaluates whether co-expression modules fit scale-free topology; also exposes hub detection) — https://github.com/oubounyt/SCANet
- **BioNERO** (Framework used to transform the gene-gene correlation matrix into a WGCNA signed adjacency matrix from which modules and their graphs are obtained for topology assessment)
- **WGCNA** (Underlying modified weighted gene co-expression network analysis method used by SCANet for network inference and module detection prior to the topology check)
- **Scanpy** (Loads and pre-processes the scRNA-seq matrix and generates pseudo-bulk cells (averaging gene expression across sub-clusters) feeding the GCN)

## Examples

```
import scanet as sn; sn.check_scalefree_topology(module_graph)
```

## Evaluation signals

- Each retained module's graph passes the scale-free topology check (module graph fits the power-law degree distribution characteristic of real-world biological networks).
- Module sizes are in a biologically plausible range comparable to the 30-195 genes per module reported for classical monocytes.
- Hub genes recovered as the intersection of the top 10% most connected genes per module are consistent with the retained modules (e.g. HESPB1, DNAJA4, CRYAB in M11).
- Module selection follows the recommended guidelines described in the 'Modules selection guidelines' section of Supplementary File 1.

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis.
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected.
- Passing the topology check validates graph plausibility only; downstream trait associations and module selection still require the supplementary guidelines.

## Evidence

- [methods] Scale-free check on module graphs: "They can also check whether the module graph fits the scale-free topology, a characteristic of real-world biological networks"
- [methods] Hub identification complements topology assessment: "users can identify network hubs by looking for the intersection of the top 10% of the most connected genes in the module"
- [intro] Underlying WGCNA basis: "SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection"
- [results] Expected module scale from classical monocytes: "We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module (M) (Fig. 2C)"
- [readme] README confirms SCANet GCN workflow context: "SCANet, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data and a complete analysis of the identified modules"
