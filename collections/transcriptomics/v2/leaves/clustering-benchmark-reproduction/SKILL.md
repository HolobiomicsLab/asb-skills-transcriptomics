---
name: clustering-benchmark-reproduction
description: Use when reproducing or validating the SSRE NMI/ARI clustering benchmark
  on Splatter-simulated scRNA-seq data, or when comparing a single-cell clustering
  method against published Tables S1-S2 scores.
when_to_use_negative:
- Input is a real (non-simulated) scRNA-seq dataset without ground-truth labels —
  the NMI/ARI validation depends on Splatter-simulated true labels; use the paper's
  separate ten-real-dataset evaluation instead
- Target datasets are large-scale — SSRE is noted as relatively time-consuming on
  large-scale datasets, making this benchmark protocol impractical there
- The goal is applying SSRE for downstream tasks (visualization, differential expression
  analysis) rather than benchmarking clustering accuracy against Tables S1-S2
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SSRE
  role: Clustering method under benchmark; its MATLAB/Python implementation is run
    with the default configuration (all three pairwise similarities for gene selection
    and similarity enhancement) on each simulated dataset
  repo: https://github.com/CSUBioGroup/SSRE
- name: Splatter
  role: R simulation tool generating the five scRNA-seq count matrices with the specified
    group proportions, per-dataset cell counts, and sparsity levels
- name: MPSSC
  role: Candidate competing spectral clustering framework (MATLAB); its README demonstrates
    the NMI/Purity/ARI metric computation pattern used for benchmark scoring
  repo: https://github.com/ishspsy/project
- name: SC3
  role: Candidate competing method for unsupervised clustering of cells from single-cell
    RNA-Seq experiments (R/BioConductor)
  repo: https://github.com/hemberg-lab/SC3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:de0272d7bc9b244e106c9418a2be857067048d101c5204163017a64f8b6f72ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  - SSRE
  - Splatter
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# clustering-benchmark-reproduction

## Summary

Benchmark-reproduction protocol for SSRE cell-type detection: simulate five Splatter scRNA-seq datasets spanning 500-1500 cells and sparsity 0.61-0.94, run SSRE (MATLAB/Python, default three-similarity configuration) plus competing clustering methods, and score NMI/ARI against simulated true labels to mirror supplementary Tables S1-S2.

## When to use

Use when reproducing or validating the SSRE NMI/ARI clustering benchmark on Splatter-simulated scRNA-seq data, or when comparing a single-cell clustering method against published Tables S1-S2 scores.

## When NOT to use

- Input is a real (non-simulated) scRNA-seq dataset without ground-truth labels — the NMI/ARI validation depends on Splatter-simulated true labels; use the paper's separate ten-real-dataset evaluation instead
- Target datasets are large-scale — SSRE is noted as relatively time-consuming on large-scale datasets, making this benchmark protocol impractical there
- The goal is applying SSRE for downstream tasks (visualization, differential expression analysis) rather than benchmarking clustering accuracy against Tables S1-S2

## Inputs

- Five Splatter-simulated scRNA-seq count matrices in R (group.prob=(0.65,0.25,0.1); nCells=1000/1000/1000/500/1500; sparsity 0.61/0.8/0.94/0.94/0.94) with retained true cell-type labels
- SSRE MATLAB/Python implementation from https://github.com/CSUBioGroup/SSRE
- Competing state-of-the-art single-cell clustering method implementations (e.g., MATLAB code such as MPSSC from ishspsy/project, SC3 from hemberg-lab/SC3)
- Published supplementary Tables S1 and S2 with target NMI/ARI scores

## Outputs

- Per-dataset, per-method NMI and ARI scores against the simulated true labels
- tables_s1_s2_reproduction.csv mirroring supplementary Tables S1 and S2

## How to apply

Simulate the five datasets in R with Splatter using group.prob=(0.65,0.25,0.1), per-dataset nCells of 1000/1000/1000/500/1500, and sparsity levels 0.61/0.8/0.94/0.94/0.94, retaining the simulated true cell-type labels. Run the SSRE implementation from CSUBioGroup/SSRE (MATLAB or Python) on each matrix in its default configuration, which applies all three pairwise similarities for gene selection and similarity enhancement — restricting to one or two similarities deviates from the published benchmark. Run each competing state-of-the-art single-cell clustering method on the same five simulated matrices. Compute NMI and ARI between each method's cluster assignments and the true labels per dataset, assemble the scores into comparison tables mirroring supplementary Tables S1 and S2, and save them as tables_s1_s2_reproduction.csv. Validate that the CSV is complete (SSRE plus every competitor, five datasets, both metrics), that values match the published scores within numerical tolerance, and that SSRE is superior in the majority of cases as reported.

## Related tools

- **SSRE** (Clustering method under benchmark; its MATLAB/Python implementation is run with the default configuration (all three pairwise similarities for gene selection and similarity enhancement) on each simulated dataset) — https://github.com/CSUBioGroup/SSRE
- **Splatter** (R simulation tool generating the five scRNA-seq count matrices with the specified group proportions, per-dataset cell counts, and sparsity levels)
- **MPSSC** (Candidate competing spectral clustering framework (MATLAB); its README demonstrates the NMI/Purity/ARI metric computation pattern used for benchmark scoring) — https://github.com/ishspsy/project
- **SC3** (Candidate competing method for unsupervised clustering of cells from single-cell RNA-Seq experiments (R/BioConductor)) — https://github.com/hemberg-lab/SC3

## Examples

```
[NMI, ~,clus_labs,~]=calc2_nmis(max(true_labs), double(P),true_labs); ARI=RandIndex(clus_labs,true_labs)
```

## Evaluation signals

- tables_s1_s2_reproduction.csv contains NMI and ARI entries for SSRE and every competing method across all five simulated datasets (5 datasets x all methods x 2 metrics)
- SSRE's NMI/ARI values on each simulated dataset match the scores in supplementary Tables S1 and S2 within numerical tolerance
- SSRE is superior to the competing methods in the majority of cases, consistent with the paper's reported finding
- Each simulated dataset matches the design points: three groups at proportions 0.65/0.25/0.1, cell counts 1000/1000/1000/500/1500, and sparsity 0.61/0.8/0.94/0.94/0.94
- Cluster-assignment vector lengths equal the number of simulated cells, so NMI/ARI are computed against the retained true labels without dimension mismatch

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; the paper defers acceleration to future parallel computing strategies
- Reproduction only holds within numerical tolerance if the exact Splatter configuration (group.prob, nCells, sparsity) and SSRE's default three-similarity setting are used; choosing one or two similarities changes results
- Validation relies on Splatter-simulated ground truth; performance on the paper's ten real scRNA-seq datasets is a separate evaluation not covered by this protocol

## Evidence

- [abstract] SSRE benchmarked on ten real and five simulated datasets, superior in most cases: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
- [abstract] SSRE MATLAB and Python implementations location: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] Default configuration uses all three pairwise similarities: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [discussion] Runtime limitation on large-scale datasets: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework [60]."
- [readme] Computational environment used for the simulations and scRNA-seq applications: [evidence span withheld — anchor status: external_doc]
