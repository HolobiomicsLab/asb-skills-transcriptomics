---
name: ligand-receptor-downstream-gene-selection
description: Use when a signaling pathway's spatial range must be estimated from scRNA-seq
  data and the canonical ligand, receptor, and downstream target genes for that pathway
  are known and can be listed explicitly.
when_to_use_negative:
- The ligand/receptor/downstream genes for the pathway of interest are unknown — the
  article notes that 'cell–cell communication inference relies on known genes involved
  in specific signaling'
- You only have a cell–cell communication score requirement without spatial range
  estimation; spatial_signaling_ot alone suffices and gene-list-based range inference
  is unnecessary
- No spatial cell–cell distance matrix exists yet (run the transport_plan and cell_cell_distance
  steps first)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpaOTsc
  role: Consumes the curated ligand/receptor/downstream gene lists via spatial_signaling_ot
    and infer_signal_range_ml to infer signaling spatial range
  repo: github.com/zcang/SpaOTsc
- name: pandas
  role: Loads the normalized scRNA-seq expression matrix (dge_normalized.txt) from
    which ligand, receptor, and downstream gene expression vectors are extracted
- name: scikit-learn
  role: Provides the random forest regression used downstream, with receptor expression
    as sample weights and ligand features L̃(η)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:3621e43a3e22dcdc0d91dfec88a7cd58b5ef1cedde07432f99994eeed5b8258e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ligand-receptor-downstream-gene-selection

## Summary

Selects the known ligand, receptor, and downstream target genes that define a signaling system of interest so SpaOTsc can infer its spatial signaling range. This curated gene triple (e.g. Wnt5/fz with CycD and dpp as downstream genes) supplies both the random-forest features and the sample weights for range estimation.

## When to use

Use when a signaling pathway's spatial range must be estimated from scRNA-seq data and the canonical ligand, receptor, and downstream target genes for that pathway are known and can be listed explicitly.

## When NOT to use

- The ligand/receptor/downstream genes for the pathway of interest are unknown — the article notes that 'cell–cell communication inference relies on known genes involved in specific signaling'
- You only have a cell–cell communication score requirement without spatial range estimation; spatial_signaling_ot alone suffices and gene-list-based range inference is unnecessary
- No spatial cell–cell distance matrix exists yet (run the transport_plan and cell_cell_distance steps first)

## Inputs

- Normalized scRNA-seq expression matrix (e.g. dge_normalized.txt from Synapse syn15665609) loaded as a pandas DataFrame
- Curated ligand gene list (e.g. ['Wnt5'])
- Curated receptor gene list (e.g. ['fz'])
- Curated downstream target gene lists for upstream/downstream roles (e.g. DSgenes_up=['CycD'], DSgenes_down=['dpp'])
- Spatial cell–cell distance matrix from the upstream SpaOTsc cell_cell_distance step

## Outputs

- Signaling gene triple used as features/weights/targets for infer_signal_range_ml
- Inferred spatial signaling range per ligand (e.g. ~100 µm Wg, ~125 µm Dpp) written to signaling_ranges.csv

## How to apply

Curate explicit gene lists for the signaling system: the ligand(s) expressed in sender cells, the receptor(s) expressed in receiver cells, and downstream target genes whose expression the ligand is expected to regulate, as specified for the Drosophila Wg and Dpp systems in Supplementary Methods section 3.1 of the SpaOTsc paper. Feed these lists into SpaOTsc's communication and range-inference routines, where ligand expression aggregated over neighborhoods of varying radius η forms the feature L̃(η), receptor expression levels act as sample weights in the random-forest fit, and the downstream genes are the prediction targets augmented with genes highly correlated to them. The η at which the ligand feature attains highest feature importance is reported as the signaling spatial distance (≈100 µm for Wg, ≈125 µm for Dpp). Rationale: the approach depends on known pathway members, so gene-list curation is the fidelity-limiting step.

## Related tools

- **SpaOTsc** (Consumes the curated ligand/receptor/downstream gene lists via spatial_signaling_ot and infer_signal_range_ml to infer signaling spatial range) — github.com/zcang/SpaOTsc
- **pandas** (Loads the normalized scRNA-seq expression matrix (dge_normalized.txt) from which ligand, receptor, and downstream gene expression vectors are extracted)
- **scikit-learn** (Provides the random forest regression used downstream, with receptor expression as sample weights and ligand features L̃(η))

## Examples

```
spsc.spatial_signaling_ot(['Wnt5'],['fz'],DSgenes_up=['CycD'],DSgenes_down=['dpp']); signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])
```

## Evaluation signals

- Recovered signaling ranges reproduce the reported values of approximately 100 µm for Wg and 125 µm for Dpp (Fig 4c,d)
- For each ligand, the η with maximum random-forest feature importance of L̃(η) is unique and well-defined across the tested effect_ranges grid (e.g. [10, 50, 100])
- All genes in the ligand, receptor, and downstream lists are present as columns in the loaded dge_normalized.txt expression matrix
- signaling_ranges.csv contains one inferred range per ligand with values in a biologically plausible µm range

## Limitations

- Inference depends on known signaling genes; novel or unannotated pathway members are missed
- The approach does not consider time delay in cell–cell communication (ligand diffusion time, intracellular cascade reacting time)
- Downstream target prediction relies on genes highly correlated to the target gene as additional features, which may confound indirect regulators

## Evidence

- [readme] spsc.spatial_signaling_ot(['Wnt5'],['fz'],DSgenes_up=['CycD'],DSgenes_down=['dpp']): "spsc.spatial_signaling_ot(['Wnt5'],['fz'],DSgenes_up=['CycD'],DSgenes_down=['dpp'])"
- [methods] signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100]): "signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])"
- [discussion] Cell–cell communication inference relies on known genes involved in specific signaling: [evidence span withheld — anchor status: not_found]
- [other] aggregate the ligand expression of cells within neighborhoods of varying radius η defined on the spatial cell–cell distance matrix to form L̃(η), and combine with genes highly correlated to the downstream target gene as features: [evidence span withheld — anchor status: not_found]
