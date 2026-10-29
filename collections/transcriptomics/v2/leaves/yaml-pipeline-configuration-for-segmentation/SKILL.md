---
name: yaml-pipeline-configuration-for-segmentation
description: Use when running BIDCell on subcellular spatial transcriptomics data
  (Xenium, CosMx, MERSCOPE, Stereo-seq) and file paths, cpus, model backbone, or single-cell
  reference settings must be specified in a .yaml config before segmentation.
when_to_use_negative:
- You only need the bundled demo run — example_small.py and params_small_example.yaml
  already ship with the small Xenium breast cancer subset, so no manual config editing
  is required.
- Nuclei images are separated into fields of view (e.g. CosMx) and only FOV stitching
  is needed — that is handled by model.stitch_nuclei(), not by config parameters.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_6398
tools:
- name: BIDCell
  role: Provides BIDCellModel.get_example_config() to copy platform .yaml templates
    and BIDCellModel(yaml) to consume the edited config during preprocessing, training,
    and prediction
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source of the Xenium Output Bundle In Situ Replicate 1 whose transcripts.csv.gz
    and morphology_mip.ome.tif paths must be entered in the config
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:8a204412b476579f80e572a4e5b533c7b17827257fb20341a49b6f83e23d1f01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - Cellpose
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_6398
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# yaml-pipeline-configuration-for-segmentation

## Summary

Configure the BIDCell cell segmentation pipeline by generating and editing a platform-specific YAML parameter file (e.g. xenium_example_config.yaml) that points to the transcripts file, DAPI image, and single-cell reference data, and tunes compute parameters before running preprocess/train/predict.

## When to use

Use when running BIDCell on subcellular spatial transcriptomics data (Xenium, CosMx, MERSCOPE, Stereo-seq) and file paths, cpus, model backbone, or single-cell reference settings must be specified in a .yaml config before segmentation.

## When NOT to use

- You only need the bundled demo run — example_small.py and params_small_example.yaml already ship with the small Xenium breast cancer subset, so no manual config editing is required.
- Nuclei images are separated into fields of view (e.g. CosMx) and only FOV stitching is needed — that is handled by model.stitch_nuclei(), not by config parameters.

## Inputs

- Vendor template config copied by BIDCellModel.get_example_config (e.g. xenium_example_config.yaml)
- Quality-filtered transcripts file (transcripts.csv.gz) from the Xenium Output Bundle In Situ Replicate 1
- DAPI/nucleus image (morphology_mip.ome.tif)
- Single-cell reference csv (sc_breast.csv) with positive and negative marker files (sc_breast_markers_pos.csv, sc_breast_markers_neg.csv)

## Outputs

- An edited .yaml parameter file with correct 'files' paths and adjusted cpus/GPU settings, used to instantiate BIDCellModel

## How to apply

Generate a vendor-specific template with BIDCellModel.get_example_config('xenium') (also available for cosmx, merscope, stereoseq), which copies a .yaml such as xenium_example_config.yaml into the working directory. Edit the parameters under 'files' to give correct paths for the quality-filtered transcripts (transcripts.csv.gz), the DAPI image (morphology_mip.ome.tif), and the provided breast cancer single-cell reference with positive/negative marker files (sc_breast.csv, sc_breast_markers_pos.csv, sc_breast_markers_neg.csv). Adjust the 'cpus' parameter to match the available system, since higher cpus speed up processing but may freeze the machine, and reduce cpus if a pickle.UnpicklingError (truncated pickle data) occurs. Optionally set model_params.name to a different encoder from segmentation-models-pytorch or modify the SegmentationModel class, and set CUDA_VISIBLE_DEVICES to select a specific GPU. Once paths and compute settings are valid, the same .yaml drives model.preprocess(), model.train(), and model.predict() (or model.run_pipeline()).

## Related tools

- **BIDCell** (Provides BIDCellModel.get_example_config() to copy platform .yaml templates and BIDCellModel(yaml) to consume the edited config during preprocessing, training, and prediction) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source of the Xenium Output Bundle In Situ Replicate 1 whose transcripts.csv.gz and morphology_mip.ome.tif paths must be entered in the config)

## Examples

```
from bidcell import BIDCellModel; BIDCellModel.get_example_config('xenium'); model = BIDCellModel('xenium_example_config.yaml')
```

## Evaluation signals

- The config file exists in the working directory after calling get_example_config('xenium') (xenium_example_config.yaml).
- Paths under 'files' resolve: transcripts.csv.gz, morphology_mip.ome.tif, and the sc_breast reference/marker csv files are readable.
- The 'cpus' value matches the system so preprocessing completes without 'pickle.UnpicklingError: pickle data was truncated' or system freezes.
- Instantiating BIDCellModel(xenium_example_config.yaml) and running model.preprocess()/train()/predict() completes the full pipeline end-to-end.

## Limitations

- Higher 'cpus' values allow faster runtimes but may freeze the system, and pickle truncation errors are resolved by reducing cpus.
- Segmentation at higher resolution requires a larger patch size and thus more GPU memory; a GPU with at least 12GB VRAM and 32GB RAM is strongly recommended.
- A GUI-related error ('[xcb] Unknown sequence number while processing queue') may occur and running without a GUI (e.g. via PuTTY) is suggested.

## Evidence

- [readme] Run the following to obtain examples:: [evidence span withheld — anchor status: not_found]
- [readme] Please ensure the correct paths are provided for the parameters under `files` in `xenium_example_config.yaml`: "Please ensure the correct paths are provided for the parameters under `files` in `xenium_example_config.yaml`, in particular, the paths for the transcripts (`transcripts.csv.gz`) and DAPI"
- [readme] cpus parameter must be tuned: "**Please modify `cpus` to suit your system. Higher `cpus` allow faster runtimes but may freeze your system.**"
- [methods] reference and marker files: "Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`)"
- [readme] GPU requirements: "A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing."
