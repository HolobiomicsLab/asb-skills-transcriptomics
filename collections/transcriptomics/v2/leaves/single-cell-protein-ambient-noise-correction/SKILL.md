---
name: single-cell-protein-ambient-noise-correction
description: Use when raw ADT/protein count matrices from CITE-seq, ASAP-seq, or TEA-seq
  experiments show antibody-specific background from unbound antibodies, and both
  a cell protein matrix and an empty-droplet protein matrix are available.
when_to_use_negative:
- No empty droplets are available in the experiment (use the dsb 'no_empty_drops'
  variant instead)
- Input is already normalized protein data (e.g. CLR-normalized or previously dsb-rescaled
  values), where re-standardization against empty droplets would be invalid
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: dsb
  role: R package implementing step-I ambient background estimation and rescaling
    via DSBNormalizeProtein()
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Downstream EM-based Gaussian mixture modeling of step-I corrected values per
    cell (step II part I)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:04bf3e8352526986ec5dfc590c30208ea39a50fd167743c35495b97b1f2a9659
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - mclust
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-protein-ambient-noise-correction

## Summary

Denoises droplet-based single-cell protein (ADT) data by removing ambient, unbound-antibody background noise: protein counts in empty droplets estimate the expected ambient background per antibody, and each cell's log-transformed counts are standardized by that background's mean and standard deviation (dsb step I).

## When to use

Use when raw ADT/protein count matrices from CITE-seq, ASAP-seq, or TEA-seq experiments show antibody-specific background from unbound antibodies, and both a cell protein matrix and an empty-droplet protein matrix are available.

## When NOT to use

- No empty droplets are available in the experiment (use the dsb 'no_empty_drops' variant instead)
- Input is already normalized protein data (e.g. CLR-normalized or previously dsb-rescaled values), where re-standardization against empty droplets would be invalid

## Inputs

- Cell-containing ADT/protein count matrix (cells x proteins), e.g. barcodes from filtered Cell Ranger output
- Empty-droplet/background ADT/protein count matrix, e.g. remaining raw barcodes with protein reads (dsb::empty_drop_citeseq_mtx)

## Outputs

- Ambient-noise-corrected, background-standardized protein expression matrix (rescaled values 'Y') per cell per protein
- Optionally best_fit_k_proportions.csv when extending to Gaussian mixture BIC model-fit diagnostics per cell

## How to apply

Load the cell-containing protein matrix (e.g. dsb::cells_citeseq_mtx) and the empty-droplet matrix (e.g. dsb::empty_drop_citeseq_mtx). Log-transform cell counts with a pseudocount of 10 (log(cell + 10)), then for each protein subtract the background mean μ_N and divide by the background standard deviation σ_N computed from the empty droplets. In R this is done via DSBNormalizeProtein() with cell_protein_matrix and empty_drop_citeseq_mtx as inputs. The result is ambient-noise-corrected values 'Y' which serve as input to dsb step II (Gaussian mixture modeling with mclust and technical component regression). This step is critical because empty-droplet ADT reads closely match the background noise levels observed in cells.

## Related tools

- **dsb** (R package implementing step-I ambient background estimation and rescaling via DSBNormalizeProtein()) — https://cran.r-project.org/package=dsb
- **mclust** (Downstream EM-based Gaussian mixture modeling of step-I corrected values per cell (step II part I))

## Examples

```
library(dsb); dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_citeseq_mtx = empty_drop_citeseq_mtx, denoise.counts = FALSE)
```

## Evaluation signals

- Rescaled values should be centered per protein: empty-droplet background mean μ_N subtracted and divided by σ_N yields approximately zero-mean, unit-variance background levels
- Extending to step-II diagnostics: proportion of cells where k=2 Gaussian mixture has the highest BIC should exceed 85% (paper reports 81% in batch-1 cells and 89% average across four external CITE-seq datasets)
- Sanity-check the background definition: dsb-normalized values are sensitive to how background droplets are defined, so verify the empty-droplet subset is reasonable
- Low-signal proteins (maximum raw UMI count < 5 across all cells, e.g. CD34 in two datasets) should be removed prior to normalization

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'
- Batch normalization behavior depends on the extent of uniformity among batches
- Caveats for datasets without isotype control antibody measurements (relevant to step II technical component) are deferred to the Supplementary Note; software version numbers are not stated

## Evidence

- [discussion] Step I: estimate ambient background noise per antibody from empty droplets and rescale cell counts: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [methods] log transformation with pseudocount 10: "# log transformation
dlog = log(cell + 10)"
- [methods] standardization by background mean and standard deviation: "Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$"
- [methods] DSBNormalizeProtein call: "dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx"
- [discussion] low-signal protein filter: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein from two datasets."
- [discussion] background droplet definition sensitivity: "defining a reasonable subset of barcodes as background droplets still requires care"
