---
name: pearson-correlation-deconvolution-assessment
description: Use when simulated bulk HiChIP mixtures have known subpopulation-specific
  loop counts and you must quantify deconvolution accuracy or benchmark a model-based
  inference against a simple plug-in baseline.
when_to_use_negative:
- Real experimental data without known subpopulation-specific HiChIP ground truth
  — PCC against truth cannot be computed; use biological validation (e.g., FACS-isolated
  marker-positive cells) instead
- Evaluating clustering quality of single cells rather than deconvolution accuracy
  of bulk profiles
- Fast approximate benchmarking where the 24 h Poisson MAP runtime is prohibitive
  and only the 3-min plug-in baseline needs scoring
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DC3
  role: Source of the deconvolution methods under evaluation — the Poisson model-based
    MAP inference and the simple plug-in baseline whose subpopulation HiChIP outputs
    are scored by mean PCC
  repo: github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:4f5118c338f43f215b8a132485d55c1c074d73c37adb782ddec3d22bc3f96ebc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pearson-correlation-deconvolution-assessment

## Summary

Evaluation skill for subpopulation deconvolution of bulk HiChIP data: score inferred subpopulation-specific loop-count vectors against ground-truth vectors using the mean Pearson Correlation Coefficient (PCC) across replicate runs, and compare model-based versus plug-in deconvolution methods on accuracy and runtime.

## When to use

Use when simulated bulk HiChIP mixtures have known subpopulation-specific loop counts and you must quantify deconvolution accuracy or benchmark a model-based inference against a simple plug-in baseline.

## When NOT to use

- Real experimental data without known subpopulation-specific HiChIP ground truth — PCC against truth cannot be computed; use biological validation (e.g., FACS-isolated marker-positive cells) instead
- Evaluating clustering quality of single cells rather than deconvolution accuracy of bulk profiles
- Fast approximate benchmarking where the 24 h Poisson MAP runtime is prohibitive and only the 3-min plug-in baseline needs scoring

## Inputs

- True subpopulation-specific HiChIP loop-count vectors from the in-silico mixture simulation (ground truth)
- Predicted subpopulation-specific HiChIP vectors reconstructed from estimated latent loop counts C_ijk (Poisson method) or plug-in proportions
- scRNA-seq expression matrix E (146 cells) and scATAC-seq openness matrix O (746 cells) with cluster labels for computing cluster-mean profiles P1, P2
- Bulk HiChIP enhancer-promoter loop counts down-sampled from GM12878/K562
- Runtime measurements per method

## Outputs

- Mean PCC between true and predicted subpopulation-specific HiChIP vectors over 50 runs, per method
- Per-subpopulation PCC values for each deconvolution run
- Method comparison table of PCC and runtime (e.g., 24 h Poisson vs 3 min plug-in) analogous to Supplementary Table 2

## How to apply

Generate in-silico mixtures (scRNA-seq E with 146 cells, scATAC-seq O with 746 cells, bulk HiChIP down-sampled from GM12878/K562) with known subpopulation-specific HiChIP profiles. Run the deconvolution method under test — e.g., the Poisson MAP inference treating single-cell profiles as known, initialized by multinomial allocation of C_ij with proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk) — plus the simple plug-in baseline p_ijk^3 = alpha * d_ij * lambda_k * p_ik^1 * p_jk^2 on identical inputs. For each of 50 runs, compute the Pearson Correlation Coefficient between each predicted and true subpopulation-specific HiChIP vector, then average PCC across runs and subpopulations. Record runtime alongside PCC, since the Poisson model trades far higher cost (about 24 h versus 3 min) for better interpretability and slightly better simulated performance. Accept the method if its mean PCC is comparable to or slightly better than the plug-in baseline, consistent with the Supplementary Fig. 3 / Supplementary Table 2 comparison.

## Related tools

- **DC3** (Source of the deconvolution methods under evaluation — the Poisson model-based MAP inference and the simple plug-in baseline whose subpopulation HiChIP outputs are scored by mean PCC) — github.com/SUwonglab/DC3

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Mean PCC over 50 runs for the Poisson method is comparable to or slightly higher than the plug-in baseline (matching Supplementary Fig. 3 / Supplementary Table 2)
- PCC values fall in a high-correlation range (near 1) between predicted and true subpopulation-specific HiChIP vectors
- Runtime asymmetry reproduced: Poisson inference ~24 h versus ~3 min for the plug-in method
- Per-run PCC is stable across the 50 simulation replicates (low variance across runs)
- Cluster-mean profiles P1 and P2 recomputed from cluster labels match those used to initialize the multinomial proportions rho_ijk

## Limitations

- Evaluation relies on simulated in-silico mixtures with known truth; performance on real bulk HiChIP deconvolution is not directly measured by this PCC benchmark
- The Poisson model-based inference has a very high time cost (24 h vs 3 min for the simple method), which may be prohibitive for routine benchmarking
- The improvement of the model-based method over the plug-in baseline is only slight in simulations, so small differences in mean PCC may not be robust

## Evidence

- [supplementary] The Poisson model-based inference treats single-cell data type profiles as known and, compared to the simple method, offers better interpretation and slightly better performance in simulations: [evidence span withheld — anchor status: not_found]
- [supplementary] evaluate deconvolution as the mean Pearson Correlation Coefficient (PCC) between true and predicted subpopulation-specific HiChIP vectors over 50 runs: [evidence span withheld — anchor status: not_found]
- [abstract] DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [readme] Based on simulation experiments and biological experiments, it is shown that this method can decompose bulk HiChIP loop counts into subpopulation-specific loop counts.: "Based on simulation experiments and biological experiments, it is shown that this method can decompose bulk HiChIP loop counts into subpopulation-specific loop counts."
