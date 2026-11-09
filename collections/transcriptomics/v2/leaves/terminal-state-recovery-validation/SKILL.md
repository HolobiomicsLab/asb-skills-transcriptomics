---
name: terminal-state-recovery-validation
description: Use when validating Cytopath trajectory inference outputs on a dataset
  with known terminal cell identities (e.g. pancreatic endocrinogenesis alpha/delta
  states) and expected co-occurring processes such as cell cycling.
when_to_use_negative:
- Root and terminal cell states are not clearly identifiable — the authors explicitly
  do not recommend Cytopath trajectory inference in this situation
- The underlying RNA velocity estimation is faulty or unrepresentative (e.g. time-invariant
  rate assumptions violated), as Cytopath inference becomes meaningless
- The dataset has no independent marker-based or annotated terminal identities against
  which endpoint recovery can be checked
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: Cytopath
  role: Performs Markov chain simulation-based trajectory inference, undirected simulations
    from random cells, and supplies pseudotime/step assignments used for terminal-state
    and cell-cycle correlation checks
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Estimates spliced/unspliced RNA velocity that defines the Cytopath transition
    matrix input
- name: cytopath-notebooks
  role: Example notebooks demonstrating Cytopath analysis on publicly available single
    cell RNA sequencing datasets, including validation workflows
  repo: https://github.com/aron0093/cytopath-notebooks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:f5d4dc2aa32d47a8f23e99f3eebb0aa4fe95d31f5c96d4cc69eeb06328535166
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - cytopath-notebooks
  - scvelo
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# terminal-state-recovery-validation

## Summary

Validation procedure for Cytopath trajectory inference that checks whether consensus trajectories terminate in biologically annotated endocrine terminal states (alpha, beta, gamma, delta) and whether interlaced processes such as cell cycling at the onset of differentiation are recapitulated by correlating cell cycle phase with individual simulation steps.

## When to use

Use when validating Cytopath trajectory inference outputs on a dataset with known terminal cell identities (e.g. pancreatic endocrinogenesis alpha/delta states) and expected co-occurring processes such as cell cycling.

## When NOT to use

- Root and terminal cell states are not clearly identifiable — the authors explicitly do not recommend Cytopath trajectory inference in this situation
- The underlying RNA velocity estimation is faulty or unrepresentative (e.g. time-invariant rate assumptions violated), as Cytopath inference becomes meaningless
- The dataset has no independent marker-based or annotated terminal identities against which endpoint recovery can be checked

## Inputs

- spliced/unspliced scRNA-seq counts with RNA velocity estimated by scvelo (e.g. GSE132188 pancreatic endocrinogenesis)
- annotated terminal cell states (alpha, beta, gamma, delta endocrine clusters)
- annotated cell cycle phase labels (G1/S/G2M) per cell
- root/terminal probability threshold setting within [0.85, 1]

## Outputs

- trajectory step assignments per cell from the static+velocity alignment score
- terminal state calls for consensus trajectories (recovery of alpha and delta endpoints)
- undirected simulation results identifying regions of transcriptional state switching
- Spearman correlation statistics between cell cycle phase and individual simulation steps
- results table such as endocrinogenesis_cytopath_results.csv

## How to apply

After running Cytopath Markov sampling with terminal state supervision (alpha, beta, gamma, delta) and root/terminal probability thresholds within the reported robust range [0.85, 1], check that the inferred consensus trajectories terminate in the expected endocrine states — for the pancreatic endocrinogenesis dataset (GEO:GSE132188, processed with scvelo), alpha and delta must be recovered among the trajectory endpoints. Align simulations terminating at a common terminal state with dynamic time warping and assign cells to trajectory steps via the static+velocity alignment score. Then run undirected simulations initialized at random cells to capture regions of transcriptional state switching, such as the cell cycle-to-differentiation transition at the onset of endocrinogenesis. Finally, compute the Spearman correlation of annotated cell cycle phase (G1/S/G2M) with the transition steps of each individual simulation; a correlation indicates faithful recapitulation of cycling stages at the single-simulation level.

## Related tools

- **Cytopath** (Performs Markov chain simulation-based trajectory inference, undirected simulations from random cells, and supplies pseudotime/step assignments used for terminal-state and cell-cycle correlation checks) — https://github.com/aron0093/cytopath
- **scvelo** (Estimates spliced/unspliced RNA velocity that defines the Cytopath transition matrix input)
- **cytopath-notebooks** (Example notebooks demonstrating Cytopath analysis on publicly available single cell RNA sequencing datasets, including validation workflows) — https://github.com/aron0093/cytopath-notebooks

## Examples

```
import cytopath; cytopath.trajectory_inference(adata, n_jobs=8, n_nodes_par=100, watermark=True); cytopath.trajectory_visualization(adata)
```

## Evaluation signals

- Alpha and delta terminal states are recovered among the inferred trajectory endpoints for the pancreatic endocrinogenesis dataset
- Spearman correlation between annotated cell cycle phase (G1/S/G2M) and transition steps of individual simulations is positive/nonzero, mirroring dominant cell cycling at the onset of differentiation
- Root and terminal probability thresholds fall within the reported robust range [0.85, 1]
- Simulations terminating at a common terminal state align coherently under dynamic time warping before consensus construction
- Undirected simulations initialized at random cells localize the cell cycle-to-islet differentiation switch reported for this dataset

## Limitations

- Robustness of Cytopath pseudotime to root/terminal probability thresholds was reported only as 'data not shown'
- Incorrect RNA velocity estimation leads to spurious root/terminal state inference (e.g. spurious root points toward the terminus in the erythroid gastrulation dataset)
- Correctness of RNA velocity estimation cannot be verified in every scenario, even when independent marker-based information is available

## Evidence

- [readme] cytopath can model complex behaviours like cycling and convergence as well as cooccurring combinations of multiple: "Cytopath can model complex behaviours like cycling and convergence as well as cooccurring combinations of multiple"
- [results] terminal states are derived from the transition matrix: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself"
- [results] assignment of cells uses static and velocity profiles: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [results] undirected simulations find state switching: "we initialized simulations at random cells in the dataset (Figure 4C)."
- [discussion] threshold robustness caveat: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [discussion] not recommended without clear terminal states: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
