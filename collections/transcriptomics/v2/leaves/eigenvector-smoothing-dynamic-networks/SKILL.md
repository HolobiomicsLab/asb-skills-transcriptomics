---
name: eigenvector-smoothing-dynamic-networks
description: Use when you have networks from multiple subjects sampled over multiple
  time points (e.g., personalized gene co-expression networks from population-scale
  scRNA-seq) and need community detection that shares signal across subjects and time.
when_to_use_negative:
- You have a single-subject or single-time-point network — no other setting covers
  networks from multiple subjects and multiple time points simultaneously, but MuDCoD's
  smoothing penalties α/β have no signal to share in that case
- Networks have not been pre-filtered/constructed from scRNA-seq data — for raw scRNA-seq,
  genes should first be filtered and robust networks built with Dozer before applying
  MuDCoD
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MuDCoD
  role: 'Core algorithm: iterative smoothed spectral community detection across subjects
    and time via fit_predict; installable with pip install mudcod'
  repo: https://github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model used to generate multi-subject dynamic degree-corrected block
    model networks (simulate_mus_dynamic_dcbm(setting=1)) for validating the skill
  repo: https://github.com/bo1929/MuDCoD
- name: Dozer
  role: Recommended upstream R package to filter genes and construct robust personalized
    co-expression networks from population-scale scRNA-seq before running MuDCoD
  repo: https://github.com/shanlu01/Dozer
- name: numpy
  role: Provides the multi-dimensional array format MuDCoD takes as its input network
    argument
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:efc3842d2d6a871475d78b9bd47acb1db65ef27ac812879b941768585bc8690d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  - pip
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# eigenvector-smoothing-dynamic-networks

## Summary

MuDCoD's eigenvector-smoothing approach infers communities in multi-subject dynamic networks by smoothing spectral node representations across adjacent time points (penalty α) and across subjects relative to a mean projection matrix (penalty β), enabling information sharing along both dimensions.

## When to use

Use when you have networks from multiple subjects sampled over multiple time points (e.g., personalized gene co-expression networks from population-scale scRNA-seq) and need community detection that shares signal across subjects and time.

## When NOT to use

- You have a single-subject or single-time-point network — no other setting covers networks from multiple subjects and multiple time points simultaneously, but MuDCoD's smoothing penalties α/β have no signal to share in that case
- Networks have not been pre-filtered/constructed from scRNA-seq data — for raw scRNA-seq, genes should first be filtered and robust networks built with Dozer before applying MuDCoD

## Inputs

- Multi-dimensional numpy adjacency tensor with first dimension indexing subjects and second indexing time points (S x T x n x n)
- Hyper-parameters alpha (per time point) and beta (per subject), max_K, n_iter, opt_K, monitor_convergence
- Optionally, Dozer-filtered gene co-expression networks derived from population-scale scRNA-seq data (.rda output from Dozer vignette up to Section 4)

## Outputs

- Predicted community assignment array of shape S x T x n (e.g., pred_MuDCoD.npy, 16 x 8 x 500)
- Smoothed spectral representations of nodes across subjects and time

## How to apply

Assemble per-subject, per-time adjacency matrices into a multi-dimensional numpy array with the first dimension indexing subjects (S) and the second indexing time points (T). Grid-search the smoothing hyper-parameters alpha (temporal smoothness of projection matrices over adjacent time points) and beta (subject-specific deviation from the mean projection matrix) using the re-sampling network cross-validation procedure of Li et al. 2020 (examples/cross_validation.py), selecting the pair that maximizes the DCBM likelihood. Then call MuDCoD(verbose=False).fit_predict on the adjacency tensor with alpha=0.05*np.ones((T, 2)), beta=0.05*np.ones(S), max_K=10, n_iter=30, opt_K='null', monitor_convergence=True; the algorithm iteratively computes smoothed spectral representations of nodes, selects the number of communities K via the eigengap of the degree-normalized Laplacian (up to max_K), and clusters nodes into communities. The output is a predicted assignment array of shape S x T x n (e.g., 16 x 8 x 500 for n=500, k=10, T=8, S=16 in the MuSDynamicDCBM setting=1 simulation), saved as pred_MuDCoD.npy.

## Related tools

- **MuDCoD** (Core algorithm: iterative smoothed spectral community detection across subjects and time via fit_predict; installable with pip install mudcod) — https://github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model used to generate multi-subject dynamic degree-corrected block model networks (simulate_mus_dynamic_dcbm(setting=1)) for validating the skill) — https://github.com/bo1929/MuDCoD
- **Dozer** (Recommended upstream R package to filter genes and construct robust personalized co-expression networks from population-scale scRNA-seq before running MuDCoD) — https://github.com/shanlu01/Dozer
- **numpy** (Provides the multi-dimensional array format MuDCoD takes as its input network argument)

## Examples

```
from mudcod.community_detection import MuDCoD; model = MuDCoD(verbose=False); pred = model.fit_predict(adj_tensor, alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30, opt_K='null', monitor_convergence=True)
```

## Evaluation signals

- Output assignment array exists with the expected shape S x T x n (e.g., 16 x 8 x 500 for the MuSDynamicDCBM setting=1 simulation)
- Convergence monitoring (monitor_convergence=True) terminated without error within n_iter=30 iterations
- Accuracy of predicted communities can be compared against ground-truth labels z_mus_dynamic_true (ARI comparison as in the paper's evaluation)
- Cross-validation selected the alpha/beta pair that maximizes the DCBM likelihood under the Li et al. 2020 re-sampling network cross-validation procedure
- Selected number of communities K respects the eigengap criterion on the degree-normalized Laplacian and the max_K=10 bound

## Limitations

- The article/README report no concrete numeric ARI values in text (code prints computed values, e.g. mean(ARI) of MuDCoD, but no output values appear), so expected accuracy ranges must be established empirically
- Performance relies on hyper-parameters alpha and beta being tuned by network cross-validation; inappropriate values weaken the intended smoothness constraints
- K selection via eigengap is bounded by max_K and depends on the degree-normalized Laplacian spectrum being informative

## Evidence

- [methods] penalty term with parameter α enforces smoothness: [evidence span withheld — anchor status: not_found]
- [intro] eigenvector smoothing rationale: "It allow signal sharing between time-steps and subjects by applying eigenvector smoothing."
- [methods] cross-validation for alpha and beta: "You can use cross-validation (see `examples/cross_validation.py`) for `alpha` and `beta`."
- [methods] input array format: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [methods] fit_predict run and evaluation: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [abstract] leverages shared signals robustly: "MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks"
