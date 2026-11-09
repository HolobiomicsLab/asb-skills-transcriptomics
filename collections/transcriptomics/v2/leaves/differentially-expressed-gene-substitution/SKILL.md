---
name: differentially-expressed-gene-substitution
description: Use when benchmarking a rare-population detector (e.g., FiRE) on synthetic
  single-cell data and you need to control the 'difficulty' of detection by varying
  the number of DE genes distinguishing a ~5% minor subpopulation.
when_to_use_negative:
- Real scRNA-seq datasets where true rare-cell identities are already known — substitution
  simulation is only needed to create controlled benchmarks, not to score real data
- When the goal is to detect technical outliers or doublets rather than a biologically
  distinct minor subpopulation — the article notes FiRE 'does not discriminate between
  an outlier and cells representing minor cell types' and that doublet rates (1%-30%)
  do not necessarily comply with the notion of cell rareness
- If the input is already a feature-reduced table (e.g., after ranger_preprocess selecting
  1000 variable genes) — substitution must occur on the full simulated gene space
  (5000 genes) before any variable-gene filtering
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: splatter
  role: R package used to simulate two-group single-cell expression data with a 5%
    rare population and controlled DE-gene effects
- name: FiRE
  role: Assigns a rareness score to every simulated cell profile; scores are evaluated
    by AUC-ROC with the rare group as positives
  repo: github.com/princethewinner/FiRE
- name: Python
  role: Runtime for the FiRE module installed via './INSTALL --py' (python setup.py
    install --record fireInstall.log) used to score simulations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:d4bd0e762b65cc111c8a306de4ecb86d4765bf5b8a72133c130151edb332d903
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# differentially-expressed-gene-substitution

## Summary

Simulation recipe for measuring how a rare-cell detector's accuracy depends on the number of differentially expressed (DE) genes marking the minor population: DE genes with de.facLoc=0.4 and de.facScale=0.8 are substituted into non-DE loci in splatter-simulated two-group scRNA-seq data (95%/5% split), and detector performance is scored as AUC-ROC.

## When to use

Use when benchmarking a rare-population detector (e.g., FiRE) on synthetic single-cell data and you need to control the 'difficulty' of detection by varying the number of DE genes distinguishing a ~5% minor subpopulation.

## When NOT to use

- Real scRNA-seq datasets where true rare-cell identities are already known — substitution simulation is only needed to create controlled benchmarks, not to score real data
- When the goal is to detect technical outliers or doublets rather than a biologically distinct minor subpopulation — the article notes FiRE 'does not discriminate between an outlier and cells representing minor cell types' and that doublet rates (1%-30%) do not necessarily comply with the notion of cell rareness
- If the input is already a feature-reduced table (e.g., after ranger_preprocess selecting 1000 variable genes) — substitution must occur on the full simulated gene space (5000 genes) before any variable-gene filtering

## Inputs

- Splatter simulation parameters: batchCells=500, nGenes=5000, group.prob=c(0.95, 0.05)
- DE effect parameters: de.facLoc=0.4, de.facScale=0.8
- Sweep range of DE-gene counts (1 to 150)
- FiRE model instance (e.g., L=100, M=50, H=1017881, seed=5489)
- Binary group labels per simulated cell (rare group = positives)

## Outputs

- auc_roc_curve.csv: mean AUC-ROC (over 1000 repetitions) versus number of DE genes (1-150)
- Per-repetition FiRE rareness scores for every simulated cell
- AUC-ROC values with the 5% rare group treated as positives

## How to apply

Generate two-group single-cell expression data with the splatter R package using batchCells=500, nGenes=5000 and group.prob=c(0.95, 0.05) so the rare group is 5% of cells, sampling DE effects from de.facLoc=0.4 and de.facScale=0.8. For each DE-gene count from 1 to 150, replace that many non-DE genes with pre-identified differentially expressed genes, creating a graded signal for the minor population. Score every cell with the FiRE Python module (built via './INSTALL --py', which runs 'python setup.py install --record fireInstall.log'), assigning rareness scores where higher values indicate rarer cells. Repeat the simulate-and-score procedure 1000 times per DE-gene count and compute AUC-ROC with the rare group as positives, averaging over repetitions. Write the mean AUC-ROC versus DE-gene-count curve to auc_roc_curve.csv; a correct run reproduces the paper's Fig. 4 behavior where FiRE performs poorly with few DE genes but improves sharply once 20 or more DE genes are present.

## Related tools

- **splatter** (R package used to simulate two-group single-cell expression data with a 5% rare population and controlled DE-gene effects)
- **FiRE** (Assigns a rareness score to every simulated cell profile; scores are evaluated by AUC-ROC with the rare group as positives) — github.com/princethewinner/FiRE
- **Python** (Runtime for the FiRE module installed via './INSTALL --py' (python setup.py install --record fireInstall.log) used to score simulations)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- AUC-ROC curve shows a sharp improvement once the number of substituted DE genes reaches ~20 or more, matching the paper's Fig. 4 finding
- Mean AUC-ROC is low when only a few DE genes distinguish the 5% population (FiRE 'struggled to detect the minor cell population with only a few DE genes')
- The sweep covers the full 1-150 DE-gene range with 1000 simulation repetitions per count before averaging
- Rare group makes up exactly ~5% of simulated cells (group.prob=c(0.95, 0.05), batchCells=500 -> 25 rare cells per simulation)
- Higher FiRE scores correctly correspond to rare-group cells, consistent with the README: 'Higher values of FiRE score represent rare cells.'

## Limitations

- Detection is unreliable with few DE genes: FiRE struggled to detect the minor population until 20 or more DE genes were introduced
- FiRE does not discriminate between technical outliers and cells representing minor cell types, which can inflate apparent performance if outliers are scored as rare positives
- Results depend on preprocessing choices (the demo pipeline keeps the top 1000 variable genes via ranger_preprocess); substituted DE genes may be dropped if they are not selected as variable
- Simulated doublet/rareness assumptions: doublet rates between 1% and 30% 'do not necessarily comply with the notion of cell rareness', so this benchmark does not validate doublet detection

## Evidence

- [other] Fig. 4 sweep definition: [evidence span withheld — anchor status: not_found]
- [other] Fig. 4 finding: [evidence span withheld — anchor status: not_found]
- [readme] FiRE scoring semantics: [evidence span withheld — anchor status: external_doc]
- [intro] Evaluation scope: "We evaluated FiRE on a number of real and simulated datasets."
- [readme] FiRE model constructor used for scoring: [evidence span withheld — anchor status: external_doc]
