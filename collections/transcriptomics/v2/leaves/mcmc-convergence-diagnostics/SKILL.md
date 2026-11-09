---
name: mcmc-convergence-diagnostics
description: Use when running get_trees() MCMC sampling in Canopy2 across a Klist
  of subclone counts and you must verify chain mixing and convergence before get__tree()
  selects the minimum-BIC configuration.
when_to_use_negative:
- No MCMC sampling has been run yet — diagnostics require trees and posteriors from
  get_trees() and cannot be applied to raw Rs, Xs, Rb, Xb read count matrices
- The underlying inference method is not Metropolis-within-Gibbs MCMC (e.g., comparator
  tools Canopy or Cardelino), where these ACF/density/trace criteria and pburn/thin
  parameters do not apply
tools:
- name: Canopy2
  role: R package supplying get_trees() to run the MCMC sampler and get_diagnostics()
    to produce convergence diagnostic plots; used to diagnose chains over subclone
    counts Klist before BIC selection
  repo: https://github.com/annweideman/canopy2
- name: R
  role: Language in which the Canopy2 package (get_trees/get_diagnostics/get_best_tree)
    is implemented and executed, ideally on a computing cluster for 50k-100k iterations
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:98d81d462f327537c13fa45a34d2271733160b7ff3b269060061dd78f5869fc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - Canopy
  - Canopy2
  - Cardelino
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# mcmc-convergence-diagnostics

## Summary

Assess Markov chain Monte Carlo convergence for Canopy2 tumor phylogeny inference using get_diagnostics() on trees and posteriors from get_trees(), checking lag ACF, posterior density alignment, and trace plots before selecting a tree by BIC.

## When to use

Use when running get_trees() MCMC sampling in Canopy2 across a Klist of subclone counts and you must verify chain mixing and convergence before get_best_tree() selects the minimum-BIC configuration.

## When NOT to use

- No MCMC sampling has been run yet — diagnostics require trees and posteriors from get_trees() and cannot be applied to raw Rs, Xs, Rb, Xb read count matrices
- The underlying inference method is not Metropolis-within-Gibbs MCMC (e.g., comparator tools Canopy or Cardelino), where these ACF/density/trace criteria and pburn/thin parameters do not apply

## Inputs

- trees and posteriors output by get_trees() (MCMC samples over Z, Ps, Pb configurations)
- Metropolis-Hastings acceptance rates from get_trees()
- MCMC settings: Klist (e.g., 3:10), thinning (thin=20), burn-in fraction (pburn=0.2), number of chains (>=10)

## Outputs

- Diagnostic plots (lag ACF, posterior density overlays, trace plots) per chain and subclone count
- Per-chain convergence verdict used to accept or reject a chain before BIC-based tree selection
- Accepted minimum-BIC tree configuration (Z, Ps, Pb) for the patient

## How to apply

Run get_trees() with multiple chains (>=10 recommended), thinning (e.g., thin=20), burn-in (e.g., pburn=0.2), and 50k-100k iterations for each subclone count k in Klist (e.g., 3:10). Then feed the resulting trees and posteriors to get_diagnostics() and check three criteria per chain and subclone count: lag autocorrelation ACF values below 0.3, posterior density traces across chains aligning to the same distribution, and trace plots resembling random noise rather than trends or drift. Only chains passing these diagnostics should be carried into get_best_tree(), which picks the tree with the highest mean posterior density per k and then minimizes the modified BIC. Convergence is fragile because Canopy2 samples via Metropolis-within-Gibbs, where successive values depend on previous iterations, so poor proposal distributions can delay mixing or trap the chain at a local optimum.

## Related tools

- **Canopy2** (R package supplying get_trees() to run the MCMC sampler and get_diagnostics() to produce convergence diagnostic plots; used to diagnose chains over subclone counts Klist before BIC selection) — https://github.com/annweideman/canopy2
- **R** (Language in which the Canopy2 package (get_trees/get_diagnostics/get_best_tree) is implemented and executed, ideally on a computing cluster for 50k-100k iterations) — https://github.com/annweideman/canopy2

## Examples

```
trees.out <- get_trees(Rs=Rs, Xs=Xs, Rb=Rb, Xb=Xb, alpha=alpha, beta=beta, kappa=1, tau=999, Klist=3:10, thin=20, pburn=0.2); get_diagnostics(trees.out)
```

## Evaluation signals

- Lag ACF values for each chain and subclone count fall below 0.3
- Posterior density curves across all chains and post-burn-in iterations align to a common distribution
- Trace plots resemble random noise with no visible trend or drift, and successive iterations are not state-dependent
- get_best_tree() converges on a single minimum-BIC configuration across the Klist range (e.g., K=3:10)
- Metropolis-Hastings acceptance rates output by get_trees() are consistent across chains for the accepted configuration

## Limitations

- Reliance on Metropolis-within-Gibbs sampling creates a convergence bottleneck influenced by proposal distributions
- Proposals that do not allow for sufficient mixing can lead to dependencies on the initial state, delaying convergence or causing it to settle on a local rather than global optimum
- Canopy2 is slow relative to comparator methods, so large iteration counts (50k-100k) across Klist and >=10 chains require cluster computing

## Evidence

- [methods] get_diagnostics: "The trees and posteriors obtained from `get_trees()` can be used to produce diagnostic plots"
- [other] MCMC convergence workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Metropolis-within-Gibbs bottleneck: "A limitation of our methodology is its reliance on sampling via Metropolis-within-Gibbs, where successive values in the chain depend on previous iterations"
- [discussion] Proposal distribution failure mode: "Proposals that do not allow for sufficient mixing can lead to dependencies on the initial state, delaying convergence or causing it to settle on a local rather than global optimum."
- [readme] Klist BIC selection: [evidence span withheld — anchor status: external_doc]
