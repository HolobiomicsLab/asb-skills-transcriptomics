---
name: newick-tree-parsing
description: Use when simulating multiple discrete or continuous cell populations
  in SymSim, since SimulateTrueCounts requires a phyla tree object (Newick file read
  with ape::read.tree, or built-in Phyla5()/Phyla3()) to define population structure.
when_to_use_negative:
- Simulating a single population — SymSim only requires a tree 'when there are multiple
  populations'; a one-population SimulateTrueCounts call (evf_type='one.population')
  needs no phyla object.
- The analysis needs observed counts rather than true counts — the tree workflow covers
  SimulateTrueCounts; observed-count generation additionally requires True2ObservedCounts
  and DivideBatches, which do not consume the tree.
- The input is not a Newick-format tree (e.g., a distance matrix or a nested clade
  list) — read.tree expects Newick text and will not parse other formats.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Provides the built-in Newick tree generators Phyla5()/Phyla3(), ships the
    extdata/Newick_ABCDE.txt five-leaf tree, and consumes the parsed phyla object
    in SimulateTrueCounts to structure multi-population simulation
  repo: https://github.com/YosefLab/SymSim
- name: PCA
  role: Dimensionality reduction applied downstream (n_pc=20) before t-SNE visualization
    of the tree-structured simulated counts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:e4e7bb613a7f59890d54f87583a0d94ad992f290b27299634724064f9b106962
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - PCA
  - SymSim
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# newick-tree-parsing

## Summary

Load or generate a Newick-format cell lineage tree (e.g., the five-leaf tree from SymSim's extdata/Newick_ABCDE.txt or via Phyla5()) so that it can drive multi-population simulations and t-SNE interpretation in SymSim. The tree defines the relationships between discrete populations or the branching structure of continuous trajectories.

## When to use

Use when simulating multiple discrete or continuous cell populations in SymSim, since SimulateTrueCounts requires a phyla tree object (Newick file read with ape::read.tree, or built-in Phyla5()/Phyla3()) to define population structure.

## When NOT to use

- Simulating a single population — SymSim only requires a tree 'when there are multiple populations'; a one-population SimulateTrueCounts call (evf_type='one.population') needs no phyla object.
- The analysis needs observed counts rather than true counts — the tree workflow covers SimulateTrueCounts; observed-count generation additionally requires True2ObservedCounts and DivideBatches, which do not consume the tree.
- The input is not a Newick-format tree (e.g., a distance matrix or a nested clade list) — read.tree expects Newick text and will not parse other formats.

## Inputs

- Newick-format tree file (e.g., extdata/Newick_ABCDE.txt from the SymSim package)
- Built-in tree generators Phyla5() (five leaves) or Phyla3() (three leaves)
- Simulation parameters: ncells_total, min_popsize, i_minpop, ngenes, nevf, n_de_evf, evf_type, vary, Sigma, randseed

## Outputs

- ape phylo tree object (phyla) suitable for passing to SimulateTrueCounts
- Simulated true counts matrix with cell metadata and EVFs reflecting the tree structure (persistable as RDS/CSV)
- t-SNE plots (via PlotTsne on log2(true_counts+1), n_pc=20) colored by population or tree edge membership

## How to apply

First install SymSim from GitHub with devtools::install_github('YosefLab/SymSim') and load the ape package. Either read a Newick file shipped with the package — e.g., ape::read.tree(system.file('extdata', 'Newick_ABCDE.txt', package='SymSim')) — or generate a five-leaf tree programmatically with Phyla5() (Phyla3() for three leaves). Pass the resulting phyla object to SimulateTrueCounts as the phyla argument alongside nevf, n_de_evf, and evf_type ('discrete' for populations, 'continuous' for trajectories); for the published five-leaf example use ncells_total=300, min_popsize=50, i_minpop=2, ngenes=500, nevf=10, n_de_evf=9, vary='s' for discrete and ncells_total=500, nevf=20, n_de_evf=12 for continuous. The tree structure determines how Extracellular Variation Factors (EVFs) evolve along branches (e.g., Brownian motion with a given Sigma), so downstream PlotTsne output should mirror the tree's leaf structure when coloring by population or by tree edge membership.

## Related tools

- **SymSim** (Provides the built-in Newick tree generators Phyla5()/Phyla3(), ships the extdata/Newick_ABCDE.txt five-leaf tree, and consumes the parsed phyla object in SimulateTrueCounts to structure multi-population simulation) — https://github.com/YosefLab/SymSim
- **PCA** (Dimensionality reduction applied downstream (n_pc=20) before t-SNE visualization of the tree-structured simulated counts)

## Examples

```
library(SymSim); phyla <- ape::read.tree(system.file("extdata", "Newick_ABCDE.txt", package = "SymSim")); true_counts_res <- SimulateTrueCounts(ncells_total=300, min_popsize=50, i_minpop=2, ngenes=500, nevf=10, evf_type="discrete", n_de_evf=9, vary="s", Sigma=0.4, phyla=Phyla5(), randseed=0)
```

## Evaluation signals

- The parsed object is a valid ape phylo tree with the expected number of leaves (five for Phyla5()/Newick_ABCDE.txt).
- The saved t-SNE figure shows five cell groups for the discrete simulation whose separation mirrors the five leaves of the input tree.
- For evf_type='continuous', the t-SNE shows a connected trajectory structure following the tree, with increasing Sigma scattering cells around the main paths.
- SimulateTrueCounts runs reproducibly for a fixed randseed (e.g., randseed=0 discrete, randseed=1 continuous), and results persist as RDS/CSV.

## Limitations

- Newick_ABCDE/Phyla5 provide a fixed five-leaf topology; arbitrary population hierarchies require supplying a custom Newick file.
- SymSim's pipeline omits several steps, including reverse transcription and library cleaning up, so the tree-conditioned simulation does not model every experimental process.
- SymSim has been generalized by scMultiSim, which additionally supports multi-omics and spatial data guided by gene regulatory networks; the Newick-based workflow described here is the SymSim scope.

## Evidence

- [methods] When there are multiple populations, users need to provide a tree.: "When there are multiple populations, users need to provide a tree."
- [methods] A tree with five leaves (five populations) can be generated as follows: "A tree with five leaves (five populations) can be generated as follows"
- [methods] devtools::install_github('YosefLab/SymSim'): "devtools::install_github('YosefLab/SymSim')"
- [readme] Refer to the SymSim vignette for examples of using SymSim to simulate datasets.: "Refer to the [SymSim vignette](https://github.com/YosefLab/SymSim/blob/master/vignettes/SymSimTutorial.Rmd) for examples of using SymSim to simulate datasets."
