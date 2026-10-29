---
name: held-out-donor-validation-design
description: Use when validating that a reference-mapping pipeline (e.g., Symphony)
  can transfer annotations (such as surface proteins) to query cells lacking the modality,
  and you need a donor-level split so query donors are never seen during reference
  construction.
when_to_use_negative:
- When the query cells are annotated from modality-matched data (e.g., mRNA-to-mRNA
  cell-type label transfer), since protein-inference correlation scoring does not
  apply
- When the held-out donors' cells are also included in the reference atlas — this
  leaks donor identity and invalidates the validation
- When the reference lacks a second modality (proteins) entirely; there is no ground
  truth to correlate predictions against
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Symphony
  role: Builds the compressed CCA-integrated multimodal reference (buildReference)
    and maps held-out mRNA-only donor queries into the reference embedding (mapQuery)
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs donor-aware integration of the joint mRNA/protein embedding on which
    the Symphony reference is built
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:d56850627a9a96c40450f3ab45b86a5841dc30fad14fa6c4ee05bb92c6e3277d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# held-out-donor-validation-design

## Summary

Design a donor-level held-out validation of reference-mapping-based annotation transfer: split a multimodal CITE-seq memory T-cell atlas into 217 training donors and 54 held-out donors whose mRNA-only data serve as queries, then score surface-protein inference by per-protein Pearson correlation between predicted and measured values.

## When to use

Use when validating that a reference-mapping pipeline (e.g., Symphony) can transfer annotations (such as surface proteins) to query cells lacking the modality, and you need a donor-level split so query donors are never seen during reference construction.

## When NOT to use

- When the query cells are annotated from modality-matched data (e.g., mRNA-to-mRNA cell-type label transfer), since protein-inference correlation scoring does not apply
- When the held-out donors' cells are also included in the reference atlas — this leaks donor identity and invalidates the validation
- When the reference lacks a second modality (proteins) entirely; there is no ground truth to correlate predictions against

## Inputs

- CITE-seq memory T-cell reference expression matrix and metadata for 217 training donors (395,373 joint mRNA/protein cells)
- mRNA-only expression matrices for 54 held-out donors (protein measurements masked from the pipeline but retained as ground truth)
- Ground-truth surface protein measurements (e.g., ADT counts) for the 54 held-out donors

## Outputs

- cite_seq_protein_correlations.csv containing per-protein Pearson r values for both the CCA-based multimodal reference and the mRNA-only PCA baseline across all 54 query samples
- Per-cell predicted surface protein abundances from knnPredict for each held-out query donor

## How to apply

Split the CITE-seq memory T-cell dataset (395,373 cells) by donor: retain 217 donors in the training reference and hold out 54 donors as queries, masking their protein measurements so they are mRNA-only. Build the Symphony reference with symphony::buildReference on the CCA-integrated multimodal embedding (integrating over donor), then map each held-out donor with mapQuery using the same normalization as the reference. Infer per-cell surface protein abundance with knnPredict using k=50 nearest neighbors in the harmonized embedding, aggregate predicted versus measured protein levels per held-out donor, and compute per-protein Pearson correlations. Repeat the identical procedure against an mRNA-only PCA reference (no CCA/protein information) as a baseline, and write both correlation sets for all 54 query samples to cite_seq_protein_correlations.csv. The rationale is that donor-level splitting guarantees the mapping must generalize to unseen individuals rather than memorize reference cells.

## Related tools

- **Symphony** (Builds the compressed CCA-integrated multimodal reference (buildReference) and maps held-out mRNA-only donor queries into the reference embedding (mapQuery)) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs donor-aware integration of the joint mRNA/protein embedding on which the Symphony reference is built)

## Examples

```
reference = symphony::buildReference(...); query = mapQuery(query_exp, ...); query = knnPredict(query, k = 50); cor(predicted, measured)
```

## Evaluation signals

- Per-protein Pearson correlations from the 50-NN CCA-reference prediction fall within the reported 0.88–0.99 range
- Per-protein Pearson correlations from the mRNA-only PCA baseline fall within the reported 0.65–0.97 range
- cite_seq_protein_correlations.csv contains correlation values for both the CCA reference and the PCA-only baseline for all 54 query samples
- Held-out donors contribute zero cells to the reference build (verify donor split: 217 train / 54 test)
- Predicted vs. measured protein levels are aggregated per held-out donor before computing correlations, not pooled globally

## Limitations

- The CCA strategy used for CITE-seq integration may not be optimal for all multimodal applications, since multimodal single-cell integration is an active research area
- Symphony mapping metrics may be less informative when a novel query population closely resembles an existing reference population
- Good protein-inference correlation depends on comprehensive cell-type coverage in the reference (Condition I), which limits equivalence with de novo integration

## Evidence

- [abstract] Symphony was demonstrated on inferring surface protein expression with a multimodal CITE-seq atlas of memory T cells: [evidence span withheld — anchor status: not_found]
- [methods] Map query and predict via knnPredict: "query = knnPredict(query,       # query object"
- [discussion] Symphony compresses reference to enable fast mapping: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [readme] Symphony available on CRAN: "Symphony is available on CRAN:

 r
install.packages("symphony")"
