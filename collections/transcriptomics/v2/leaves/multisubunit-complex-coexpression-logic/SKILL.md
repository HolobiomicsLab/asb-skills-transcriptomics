---
name: multisubunit-complex-coexpression-logic
description: Use when computing pairwise ligand-receptor communication scores between
  cell populations from scRNAseq, bulk RNAseq, microarray, or spatial transcriptomic
  profiles in which ligands or receptors (e.g. checkpoint, chemokine, cytokine complexes)
  have multiple subunits requiring co-expression logic.
when_to_use_negative:
- The input already contains pre-computed interaction scores or a feature table that
  bypasses expression-level scoring
- The study requires non-human ligand-receptor curation — the ICELLNET database is
  curated exclusively from human studies
- Downstream intracellular signaling effectors must be modeled; the paper states downstream
  effectors were deliberately not integrated in ICELLNET
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ICELLNET
  role: Provides the curated 1669-interaction ligand-receptor database and the scoring
    function that applies multi-subunit co-expression logic to compute per-interaction
    and global communication scores
  repo: github.com/soumelis-lab/ICELLNET
- name: devtools
  role: Used to install the ICELLNET package from GitHub via install_github with ref='master'
    and subdir='icellnet'
  repo: github.com/soumelis-lab/ICELLNET
- name: data.table
  role: Loading the gene expression matrix and interaction database files into R for
    score computation
- name: readxl
  role: Reading the manually curated ICELLNET ligand-receptor interactions database
    supplied as Excel input
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:322265ecffc6a3ee453cc4082e4f356554246c41f9f09b07d76138fbae4d9e4a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ICELLNET
  - R
  - data.table
  - devtools
  - readxl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# multisubunit-complex-coexpression-logic

## Summary

ICEllNET's rule set for scoring ligand-receptor communication from transcriptomic profiles while enforcing logical co-expression of the multiple protein subunits that make up multimeric ligand and receptor complexes, so that non-functional partial expression of a complex contributes nothing to the communication score.

## When to use

Use when computing pairwise ligand-receptor communication scores between cell populations from scRNAseq, bulk RNAseq, microarray, or spatial transcriptomic profiles in which ligands or receptors (e.g. checkpoint, chemokine, cytokine complexes) have multiple subunits requiring co-expression logic.

## When NOT to use

- The input already contains pre-computed interaction scores or a feature table that bypasses expression-level scoring
- The study requires non-human ligand-receptor curation — the ICELLNET database is curated exclusively from human studies
- Downstream intracellular signaling effectors must be modeled; the paper states downstream effectors were deliberately not integrated in ICELLNET

## Inputs

- Gene x cell-cluster average-expression profile matrix (scaled expression, e.g. per-cluster means from a scRNA-seq matrix with cluster annotations)
- ICELLNET manually curated human ligand-receptor interactions database (1669 interactions, with multi-subunit complex definitions)
- Cell-to-cluster annotation assigning source (central) and target (partner) populations

## Outputs

- Global ICELLNET communication score for each central-partner cell pair (e.g. CM3->CT3b = 1527, CM3->CT0a = 1123)
- Per-interaction score matrix with individual ligand-receptor scores (e.g. CD86/CD28, CD86/CTLA4, PDCD1LG2/PDCD1, CCL22/CCR4, VEGFA/FLT1), writable as communication_score_matrix.csv or icellnet_scores.csv

## How to apply

Compute each individual ligand-receptor interaction score as the product of the scaled ligand expression in the source (central) cell and the receptor expression in the target (partner) cell, using ICELLNET's manually curated human ligand-receptor database (extended to 1669 interactions in v2) which encodes systematic combinatorial rules for assembling protein subunits into multimeric ligands and receptors. For multi-subunit complexes, apply geometric averaging together with the package's logical co-expression rules so that a complex is only scored as functional when its required subunits are co-expressed in the relevant cell; otherwise the interaction contributes zero (e.g. VEGFA/FLT1 scores 21 from CM3 to CT3b but 0 to CT0a). Sum the individual interaction scores into a global communication score for each central-partner pair, with an optional weight coefficient, and normalize global scores to a reference condition for cross-condition comparison. Validate the logic by reproducing known score separations, e.g. the lupus nephritis cDC cluster CM3 scores higher to CT3b (1527) than to CT0a (1123), with per-interaction scores such as CD86/CD28 92 vs 40 and CD86/CTLA4 92 vs 14.

## Related tools

- **ICELLNET** (Provides the curated 1669-interaction ligand-receptor database and the scoring function that applies multi-subunit co-expression logic to compute per-interaction and global communication scores) — github.com/soumelis-lab/ICELLNET
- **devtools** (Used to install the ICELLNET package from GitHub via install_github with ref='master' and subdir='icellnet') — github.com/soumelis-lab/ICELLNET
- **data.table** (Loading the gene expression matrix and interaction database files into R for score computation)
- **readxl** (Reading the manually curated ICELLNET ligand-receptor interactions database supplied as Excel input)

## Examples

```
devtools::install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet")
```

## Evaluation signals

- Reproduced global scores match published values (CM3->CT3b = 1527, CM3->CT0a = 1123 in the lupus nephritis scRNA-seq dataset)
- Reproduced per-interaction scores match published values: CD86/CD28 92 vs 40, CD86/CTLA4 92 vs 14, PDCD1LG2/PDCD1 72 vs 19, CCL22/CCR4 100 vs 39, VEGFA/FLT1 21 vs 0
- All cell-pair scores in the output matrix are finite and non-negative, and normalized relative to the reference condition
- Interactions whose multi-subunit ligand or receptor lacks a required co-expressed subunit contribute zero rather than a partial score
- Version-dependency check: ICELLNET v2.0.0 runs only on R < 4.1; use v2.2.x for R > 4.1.x

## Limitations

- ICELLNET includes a relatively small number of interactions compared to other databases, and early versions lacked chemokine and checkpoint interactions (since extended to 1669 interactions in v2)
- The database is curated exclusively from human studies, so cross-species application is out of scope
- Downstream effectors are not integrated; approaches that add them can lead to unpredictable false positive and false negative predictions
- Some receptors still have no known ligands (DR6/TNFRSF21, RELT, TROY, NGFR) and some ligands no known receptors (IL17D)

## Evidence

- [discussion] coexpression_rules_discussion: "Similar to CellPhoneDB10, ICELLNET takes into account the multiple subunits of ligands and receptors, by introducing logical rules for co-expression of protein subunits."
- [readme] readme_multisubunit_db: "ICELLNET database takes into account the **multiple** subunits of ligand and receptor complexes (more than 1600 interactions)"
- [intro] intro_gap_motivation: "without necessarily providing systematic combinatorial rules for the association of protein subunits into multimeric ligands or receptors."
- [readme] readme_db_extension_v2: "New structure and extension of ICELLNET database up to 1669 interactions."
