---
name: cross-language-python-r-pipeline-execution
description: Use when an analysis requires tools from both ecosystems — e.g. simulating
  scRNA-seq data with splatter in R and scoring cells with the FiRE Python module
  — and the intermediate expression matrix must be exchanged between languages.
when_to_use_negative:
- The entire pipeline can run in one language — e.g. FiRE's R package (install.packages('FiRE')
  or './INSTALL --R') suffices if R scoring is acceptable, avoiding a file handoff
- The input is a single static dataset rather than a simulated-data loop, where language
  choice can follow the primary toolchain without R/Python interop overhead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3170
tools:
- name: FiRE
  role: Python module (built with './INSTALL --py') that computes a rareness score
    for every cell on the data simulated in R; also installable in R via './INSTALL
    --R' for a single-language alternative
  repo: github.com/princethewinner/FiRE
- name: splatter
  role: R package used to simulate two-group scRNA-seq data (5% rare population) with
    configurable numbers of DE genes feeding the Python FiRE scoring stage
- name: Python
  role: Hosts the FiRE module installation ('python setup.py install --record fireInstall.log'),
    data preprocessing, FiRE scoring and AUC-ROC aggregation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:379f4872b0769fcdb2dc4989b55a499c6def3060d6f6c5c5a3a2301ce7dda62b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - splatter
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-language-python-r-pipeline-execution

## Summary

Orchestrates a single analysis pipeline that spans R and Python: R (e.g. the splatter package) generates simulated scRNA-seq data while the Python FiRE module (built via './INSTALL --py') assigns rareness scores, joining the two halves through flat data files such as CSV.

## When to use

Use when an analysis requires tools from both ecosystems — e.g. simulating scRNA-seq data with splatter in R and scoring cells with the FiRE Python module — and the intermediate expression matrix must be exchanged between languages.

## When NOT to use

- The entire pipeline can run in one language — e.g. FiRE's R package (install.packages('FiRE') or './INSTALL --R') suffices if R scoring is acceptable, avoiding a file handoff
- The input is a single static dataset rather than a simulated-data loop, where language choice can follow the primary toolchain without R/Python interop overhead

## Inputs

- R splatter simulation parameters (batchCells=500, nGenes=5000, group.prob=c(0.95, 0.05), de.facLoc=0.4, de.facScale=0.8, DE-gene counts 1–150)
- Simulated scRNA-seq expression matrix exchanged as a flat file (cells x genes) from R to Python
- FiRE repository source with INSTALL script and boost library >= 1.54.0
- Binary cell labels marking the 5% rare population as positives for AUC-ROC

## Outputs

- Per-repetition FiRE rareness scores (np.array[nCells]) for every DE-gene count
- auc_roc_curve.csv containing mean AUC-ROC (over 1000 repetitions) versus DE-gene count
- fireInstall.log recording the Python module installation for reproducibility/uninstallation

## How to apply

Split the pipeline by language strength: in R, generate the two-group splatter simulation with batchCells=500, nGenes=5000 and group.prob=c(0.95, 0.05), substituting 1–150 DE genes using de.facLoc=0.4 and de.facScale=0.8, then export the expression matrix. Build FiRE from the repository with './INSTALL --py' (which runs 'python setup.py install --record fireInstall.log'); C++ prerequisites are g++ >= 4.8.4 and boost >= 1.54.0, and Python prerequisites are cython >= 0.23.4, numpy, pandas and statsmodels. In Python, load the exported matrix as cells-by-genes, run ranger_preprocess to keep 1000 variable genes, fit model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489), and call model.score to obtain a rareness score per cell (higher = rarer). Repeat the full simulate-and-score loop 1000 times per DE-gene count, computing AUC-ROC with the rare 5% group as positives in Python. Record decisions such as which genes are replaced and how repetitions are averaged so the R and Python halves remain reproducible.

## Related tools

- **FiRE** (Python module (built with './INSTALL --py') that computes a rareness score for every cell on the data simulated in R; also installable in R via './INSTALL --R' for a single-language alternative) — github.com/princethewinner/FiRE
- **splatter** (R package used to simulate two-group scRNA-seq data (5% rare population) with configurable numbers of DE genes feeding the Python FiRE scoring stage)
- **Python** (Hosts the FiRE module installation ('python setup.py install --record fireInstall.log'), data preprocessing, FiRE scoring and AUC-ROC aggregation)

## Examples

```
Rscript -e "library(splatter); sim <- splatSimulate(groupCells=500, nGenes=5000, group.prob=c(0.95,0.05), de.facLoc=0.4, de.facScale=0.8); write.csv(t(as.matrix(assay(sim,'counts'))), 'sim.csv', row.names=FALSE)" && python -c "import FiRE; model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489); scores = model.score(model.fit(X))"
```

## Evaluation signals

- fiRE installation log (fireInstall.log) exists and 'import FiRE' succeeds in Python after './INSTALL --py'
- Simulated matrix has expected shape: 500 cells x 5000 genes with the rare group comprising 5% (25 cells) of cells
- Mean AUC-ROC at 0 or few DE genes is low but rises sharply once 20 or more DE genes are introduced, reproducing the Fig. 4 sensitivity curve
- 1000 repetitions completed per DE-gene count and the averaged curve is written to auc_roc_curve.csv
- FiRE scores in Python follow the README convention that higher scores represent rare cells (rare group as positives yields AUC > 0.5 with sufficient DE genes)

## Limitations

- Uninstall with '--inplace' builds may fail in Python 3; the workaround is manually removing the generated .so file
- boost is required for the C++ build (only <boost/random.hpp> is needed); if not at the default location, '--boost-path' must be supplied
- The README's Python prerequisites specify python 2.7; python 3 works with standard installation but the inplace/uninstall path is fragile

## Evidence

- [methods] Install FiRE in python environment: "--py                       : python        : Install FiRE in python environment."
- [other] run python setup.py install and record log: [evidence span withheld — anchor status: not_found]
- [readme] README install options for both languages: "--R                        : R             : Install FiRE in R environment."
- [readme] README reports all manuscript results generated with the python module: [evidence span withheld — anchor status: external_doc]
- [readme] README FiRE score semantics: [evidence span withheld — anchor status: external_doc]
