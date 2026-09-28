# IM-1 dataset 3

Real and synthetic IM-1 landing-site images for before/after change detection.

## Contents

| Folder(s) | Contents |
| --- | --- |
| [real_no_lander](real_no_lander/metadata.json), [real_lander](real_lander/metadata.json) | Main aligned observations: 28 before landing, 31 after. |
| [real_float32](real_float32/metadata.json) | Calibrated float32 GeoTIFF inputs for the same 59 main observations; NaN marks invalid pixels. |
| [real_no_lander_extra](real_no_lander_extra/metadata.json), [real_lander_extra](real_lander_extra/metadata.json) | Additional aligned observations: 7 before, 17 after. |
| [real_float32_extra](real_float32_extra/metadata.json) | Calibrated float32 GeoTIFFs for the 24 additional observations; excluded from the benchmark. |
| [sim_no_lander](sim_no_lander/metadata.json), [sim_lander](sim_lander/metadata.json) | 83 sunlight-matched predictions per state. |
| [real_original](real_original/metadata.json) | 83 unrectified original-camera PNG crops. |
| [real_original_data](real_original_data/metadata.json) | 83 calibrated source `.IMG` files with row-range/checksum JSON sidecars. |
| [sun_sweep_sim_no_lander](sun_sweep_sim_no_lander/metadata.json), [sun_sweep_sim_lander](sun_sweep_sim_lander/metadata.json) | 100 paired Sun directions; 200 predictions total. |

Folder metadata uses folder-local filenames. Product IDs (or sweep sample IDs)
pair files across folders.
Selection rules are in [im1_dataset_filter_criteria.txt](im1_dataset_filter_criteria.txt).
The exact four benchmark group assignments are in
[benchmark_assignments.json](benchmark_assignments.json); seeds alone do not
reconstruct these custom assignments.

Evaluation results: [outputs](outputs/README.md).

## Metadata labels

- `metadata_version` is the file-format version.
- `capture_period` means before/after landing, not visual confirmation. `quality_group`
  is main/extra, stated once in separated folders.
- `model.simulated_scene` means with/without lander. `matched_real_observation`
  describes the matched real image, not the synthetic scene.
- `max_nonlinear_correction_px` measures applied control-point warp, not remaining
  error; `affine_stretch_ratio` is 1 for equal stretching. False
  `sun_angle_within_training_range` means lighting extrapolation.

## Generation

Real images are public calibrated NASA LRO Narrow Angle Camera (LROC NAC) imagery
from the Planetary Data System. Camera geometry and lunar terrain produce a common
top-down view (orthorectification); matched terrain features then align the images.

Synthetic images are data-driven lighting predictions: a weighted least-squares
linear model at each pixel predicts brightness from Sun direction. Models use only the main
28 no-lander and 31 lander images; extras are not used.
Both states are predicted at each observation's Sun angle. The sweep covers the
observed lighting envelope with a 3% expansion.

## Important notes

- All PNGs are 2048x2048, 8-bit grayscale. Aligned real and synthetic images share
  a north-up, 0.7 m/pixel grid centered on the landing site. Original-camera PNGs
  are not aligned. Their `geometry_point_pixel_xy` is not an exact lander annotation.
- Files in `real_float32/` are 2048x2048, single-band float32 GeoTIFFs in I/F on
  the same aligned grid. Finite values are valid observations and NaN
  values are unsupported pixels. These 59 files, rather than the display-scaled
  PNGs, are the scientific inputs used by the benchmark.
- `real_float32_extra/` uses the same format for the 24 additional observations,
  which remain outside detector fitting, calibration and testing.
- Subsolar coordinates give Moon-fixed Sun direction. Camera angles are evaluated
  at `camera_geometry_location`, separately from the landing-site anchor.
- PNGs are display-scaled and may clip highlights; use the calibrated `.IMG` data
  for measured intensity. These preserve signed 16-bit little-endian samples and
  labels and 12,000 full-width rows per image. Omitted rows read
  as zero but are not observations. Consult the sidecars and export sparse-aware
  (e.g. `tar --sparse`). `first_line_one_based` starts at 1; I/F = count * `if_scale`.
- Main observations helped fit the predictions, so their real/synthetic comparisons
  are not independent validation.
