---
name: minimum-dominating-set-identification
description: Use when a directed cell-lineage-specific GRN has been constructed and
  driver-node candidates must be extracted via minimum dominating set / minimum feedback
  vertex set ILPs and ranked by attention-derived influence scores.
when_to_use_negative:
- No lineage GRN exists yet (raw scRNA-seq expression matrix or prior network only)
  — CEFCON's contrastive graph-attention GRN construction must run first
- The goal is regulon-like gene module (RGM) detection or RGM activity scoring (cefcon_results.RGM_activity())
  rather than ranking driver-regulator candidates
- The analysis must jointly process multiple fate trajectories with bifurcations —
  CEFCON can only handle one cell lineage at a time
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: Framework whose cefcon_results.driver_regulators() merges the MFVS/MDS control-based
    node sets with the attention-derived influence score to output the ranked driver
    regulators
  repo: https://github.com/WPZgithub/CEFCON
- name: gurobipy
  role: Solves the minimum feedback vertex set and minimum dominating set integer
    linear programs formulated on the directed lineage GRN (>=10.0.0)
  repo: https://pypi.org/project/gurobipy/
- name: networkx
  role: Represents the cell-lineage-specific GRN as a directed graph on which the
    MFVS/MDS problems are formulated (>=3.0)
- name: cvxpy
  role: Optimization dependency (>=1.2.0) accompanying CEFCON's control-based driver-regulator
    computations
- name: pandas
  role: Loads cell_lineage_GRN.csv and gene_embs.csv for graph construction and score
    merging
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:27de2f2e4309de82803c32671b2b309c1f4fc00e8872dd55ef4017eadd375b93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - Python
  - cvxpy
  - gurobipy
  - networkx
  - pandas
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# minimum-dominating-set-identification

## Summary

Solves the minimum dominating set (MDS) and minimum feedback vertex set (MFVS) optimization problems on a cell-lineage-specific GRN to obtain control-theoretic driver-node candidates, which are then ranked by a log-transformed influence score derived from the GRN constructor's attention coefficients (lambda=0.8) to yield the driver regulators of cell fate decisions.

## When to use

Use when a directed cell-lineage-specific GRN has been constructed and driver-node candidates must be extracted via minimum dominating set / minimum feedback vertex set ILPs and ranked by attention-derived influence scores.

## When NOT to use

- No lineage GRN exists yet (raw scRNA-seq expression matrix or prior network only) — CEFCON's contrastive graph-attention GRN construction must run first
- The goal is regulon-like gene module (RGM) detection or RGM activity scoring (cefcon_results.RGM_activity()) rather than ranking driver-regulator candidates
- The analysis must jointly process multiple fate trajectories with bifurcations — CEFCON can only handle one cell lineage at a time

## Inputs

- Constructed cell-lineage-specific GRN as a directed graph (cell_lineage_GRN.csv)
- Gene embeddings from the trained CEFCON graph attention network (gene_embs.csv)
- Learned attention coefficients from the GRN constructor used to derive the gene influence score (scaled with lambda=0.8)

## Outputs

- Minimum dominating set (MDS) and minimum feedback vertex set (MFVS) node sets from the optimal ILP solutions
- Ranked driver regulator list with log-transformed influence scores written to driver_regulators.csv

## How to apply

Represent the constructed cell-lineage-specific GRN (cell_lineage_GRN.csv, loaded with pandas/networkx as a directed graph) and formulate the minimum dominating set as a binary integer linear program in gurobipy — one selection variable per gene, with constraints ensuring every non-selected gene is adjacent to a selected one — solving it to optimality alongside the minimum feedback vertex set (MFVS) ILP; the union of these control-based node sets forms the driver-gene candidate pool. Compute the gene influence score for each gene from the learned attention coefficients of the graph attention GRN constructor, scaled with lambda=0.8 and log-transformed, via cefcon_results.gene_influence_score(). Obtain the thresholded GRN results (edge_threshold_avgDegree=8 in the paper's example) and call cefcon_results.driver_regulators() to merge the MFVS/MDS solution nodes with the influence scores and rank candidates by score. Write the ranked driver regulators with their influence scores to driver_regulators.csv in the output directory. Rationale: CEFCON models the Waddington landscape with network control theory, so node sets that dominate (MDS) or break cycles in (MFVS) the GRN are the candidate genes able to steer cell fate dynamics, with attention-derived influence scores providing the ranking.

## Related tools

- **CEFCON** (Framework whose cefcon_results.driver_regulators() merges the MFVS/MDS control-based node sets with the attention-derived influence score to output the ranked driver regulators) — https://github.com/WPZgithub/CEFCON
- **gurobipy** (Solves the minimum feedback vertex set and minimum dominating set integer linear programs formulated on the directed lineage GRN (>=10.0.0)) — https://pypi.org/project/gurobipy/
- **networkx** (Represents the cell-lineage-specific GRN as a directed graph on which the MFVS/MDS problems are formulated (>=3.0))
- **cvxpy** (Optimization dependency (>=1.2.0) accompanying CEFCON's control-based driver-regulator computations)
- **pandas** (Loads cell_lineage_GRN.csv and gene_embs.csv for graph construction and score merging)

## Examples

```
cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8); cefcon_results.gene_influence_score(); cefcon_results.driver_regulators()
```

## Evaluation signals

- driver_regulators.csv exists in the output directory and pairs each gene with a finite influence score value
- The gurobipy solver reports optimal status for both the MFVS and MDS ILP formulations on the directed GRN
- Every returned driver regulator is a node of the input GRN, and the list is sorted by descending log-transformed influence score
- Influence scores are consistent with cefcon_results.gene_influence_score() output derived from attention coefficients scaled with lambda=0.8
- The candidate set equals the union of the MFVS and MDS solutions computed on the same GRN object used downstream

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are modeled; cell–cell communication and spatial location are not considered
- Candidate quality is bounded by upstream GRN accuracy — GRN construction from scRNA-seq data alone is still not accurate enough, and ground-truth label datasets for supervised alternatives are often lacking

## Evidence

- [intro] CEFCON characterizes the cell fate dynamics based on the network control theory and identifies the driver regulators through combining the control-based methods with our proposed influence score: "CEFCON characterizes the cell fate dynamics based on the network control theory and identifies the driver regulators through combining the control-based methods with our proposed influence score"
- [abstract] then models cell fate dynamics through network control theory to identify driver regulators and the associated gene modules: "then models cell fate dynamics through network control theory to identify driver regulators and the associated gene modules"
- [methods] cefcon_results.driver_regulators(): "cefcon_results.driver_regulators()"
- [readme] Subsequently, CEFCON characterizes gene regulatory dynamics from the perspective of network control theory and identifies the driver regulators that steer cell fate decisions (Fig. c).: "Subsequently, CEFCON characterizes gene regulatory dynamics from the perspective of network control theory and identifies the driver regulators that steer cell fate decisions (Fig. c)."
- [readme] - networkx(>=3.0)
- cvxpy(>=1.2.0)
- [gurobipy(>=10.0.0)](https://pypi.org/project/gurobipy/): "networkx(>=3.0)
- cvxpy(>=1.2.0)
- [gurobipy(>=10.0.0)](https://pypi.org/project/gurobipy/)"
