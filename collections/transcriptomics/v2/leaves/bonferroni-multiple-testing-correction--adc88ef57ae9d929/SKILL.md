---
name: bonferroni-multiple-testing-correction
description: Use when testing many source–sink causal paths per subnetwork (SEMace
  effect='source2sink') or declaring perturbed nodes/edges in SEMtree modules, and
  family-wise error control at alpha=0.05 is needed.
when_to_use_negative:
- When false discovery rate rather than family-wise error control is the target —
  the paper's own SEMace example uses method = 'BH' (Benjamini–Hochberg), which is
  the less conservative alternative for exploratory scans.
- When the family of hypotheses is undefined or tests are not part of a single source–sink
  enumeration per module, since Bonferroni depends on knowing the exact number of
  tested paths.
- When testing a single pre-specified path without a multiplicity burden — correction
  would only reduce power.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SEMace
  role: Performs the source2sink ACE tests whose P-values are Bonferroni-adjusted
    (method parameter, alpha = 0.05) to yield combined PVAL(E) per subnetwork.
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: R package hosting SEMace, SEMbap (with method='bonferroni' deconfounding),
    and SEMtree used to generate the subnetworks under test.
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Supplies the GSE172114 COVID-19 expression data, group labels, and reference
    networks on which the Bonferroni-corrected ACE family is computed.
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:fb369c60d62eb87b1b430fcb95e71e48f2a7c86af223fcd03780a869555e4638
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMacE
  - SEMace
  - SEMdata
  - SEMgraph
  - SEMgsa
  - SEMpath
  - SEMrun
  - SEMtree()
  - org.Hs.eg.db
  - transformData
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# bonferroni-multiple-testing-correction

## Summary

Family-wise multiple-testing correction applied to Average Causal Effect (ACE) P-values over directed source–sink paths (SEMace) and to deconfounding steps (SEMbap) when comparing perturbation across subnetworks. In the SEMtree COVID-19 workflow it controls the 5% error rate when testing many X->Y paths per CAT module, yielding Bonferroni-combined P-values (PVAL(E)) used to rank the seven extracted subnetworks.

## When to use

Use when testing many source–sink causal paths per subnetwork (SEMace effect='source2sink') or declaring perturbed nodes/edges in SEMtree modules, and family-wise error control at alpha=0.05 is needed.

## When NOT to use

- When false discovery rate rather than family-wise error control is the target — the paper's own SEMace example uses method = 'BH' (Benjamini–Hochberg), which is the less conservative alternative for exploratory scans.
- When the family of hypotheses is undefined or tests are not part of a single source–sink enumeration per module, since Bonferroni depends on knowing the exact number of tested paths.
- When testing a single pre-specified path without a multiplicity burden — correction would only reduce power.

## Inputs

- Nonparanormal-transformed COVID-19 RNA-seq expression matrix (GSE172114, data.npn from transformData())
- Group vector {0: control, 1: case} from SEMdata
- SEMtree-extracted subnetwork graph (e.g., ST, Kou's Steiner tree) as an igraph object
- Full set of directed source–sink paths (paths with > 2 nodes) enumerated per subnetwork

## Outputs

- Bonferroni-adjusted ACE P-values per source–sink pair (combined PVAL(E))
- Set of significant ACE paths at alpha = 0.05 (e.g., ATG16L1 -> CCR5, 14 nodes / 13 edges)
- Percentage of significant paths per subnetwork (ACEs%) feeding the Table 2 module ranking

## How to apply

First apply the nonparanormal transform with transformData() to the RNA-seq expression matrix (e.g., GSE172114) to relax normality, then enumerate the family of tests: all directed X->Y source2sink paths (only paths with > 2 nodes are counted) in each SEMtree-derived subnetwork, such as Kou's Steiner tree (ST) module. Call SEMace() with type='optimal' (adjustment set blocking all backdoor paths), effect='source2sink', the correction method set to 'bonferroni' (instead of the default 'BH' shown in the paper's example), and alpha=0.05, so the per-path P-values are scaled by the number of tested paths and combined into a Bonferroni-adjusted ACE P-value (PVAL(E)) over all source–sink pairs. Declare an ACE (e.g., the ATG16L1 (55054) -> CCR5 (1234) path in the ST module) significant only if its Bonferroni-corrected P-value is < 0.05, and express significant paths as a percentage (ACEs%) of the tested paths. Use the same Bonferroni correction style in SEMbap (method='bonferroni') when deconfounding, and rank the seven CAT subnetworks by combined perturbation metrics (ACEs% up, PVAL(E) down) to reproduce the Table 2 comparison identifying ST as most perturbed. Rationale: Bonferroni gives strict family-wise error control appropriate when the same family of source–sink tests is re-run across seven modules and a single most-perturbed module must be singled out.

## Related tools

- **SEMace** (Performs the source2sink ACE tests whose P-values are Bonferroni-adjusted (method parameter, alpha = 0.05) to yield combined PVAL(E) per subnetwork.) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (R package hosting SEMace, SEMbap (with method='bonferroni' deconfounding), and SEMtree used to generate the subnetworks under test.) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Supplies the GSE172114 COVID-19 expression data, group labels, and reference networks on which the Bonferroni-corrected ACE family is computed.) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
ace <- SEMace(graph = st.graph, data = data.npn, type = "optimal", effect = "source2sink", method = "bonferroni", alpha = 0.05)
```

## Evaluation signals

- The ACE table contains only source-to-sink pairs significant under Bonferroni correction (corrected P < 0.05) — explicitly required by the ATG16L1→CCR5 path validation.
- Number of tests equals the count of directed X->Y paths with > 2 nodes enumerated per subnetwork; verify alpha/|tests| scaling is applied to raw P-values.
- The assembled Table 2 reproduces the paper's pattern: ST (Kou's Steiner tree) ranked most perturbed with highest ACEs% and lowest PVAL(E) among the seven CAT modules.
- Along a recovered significant path, perturbed vs unperturbed gene counts match expectations (ATG16L1→CCR5: 14 nodes, 13 edges, exactly 2 genes with non-significant group coefficient, p >= 0.05).
- Using method='BH' instead of Bonferroni should yield more significant ACEs (anti-correlated check on correction stringency).

## Limitations

- Bonferroni is conservative and power drops as the number of enumerated source–sink paths grows; the paper's default SEMace example uses 'BH' at alpha = 0.05, so switching to Bonferroni can eliminate borderline ACEs.
- P-values must come from valid per-path tests; data violating normality (raw RNA-seq counts) can distort the family-wise control unless the nonparanormal transform (transformData) is applied first.
- The Discussion section of the source document is absent, so caveats on the correction choice across the seven-module comparison are not fully documented.

## Evidence

- [methods] ace <- SEMace(graph = alsData$graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05): "ace <- SEMace(graph = alsData$graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05)"
- [abstract] Condition-specific changes from differential expression and gene–gene co-expression are recovered with statistical testing of node, directed edge, and directed path difference between groups.: "Condition-specific changes from differential expression and gene–gene co-expression are recovered with statistical testing of node, directed edge, and directed path difference between groups."
- [methods] we apply a nonparanormal transform to relax the normality assumption.: "we apply a nonparanormal transform to relax the normality assumption."
