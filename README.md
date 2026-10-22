[README.md](https://github.com/user-attachments/files/33798453/README.md)
# STARMorph-MMWHS-Preprocessed

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23536475.svg)](https://doi.org/10.5281/zenodo.23536475)

Preprocessed MMWHS 2017 dataset (128 x 128 x 128) for cross-modality (CT-MR) cardiac image registration, as used in the STARMorph paper.

## Overview

This repository provides the **preprocessing pipeline** and **data split definitions** for the Multi-Modality Whole Heart Segmentation (MMWHS) 2017 dataset. The preprocessed volumes (in `.pkl` format) are hosted on [Zenodo](https://doi.org/10.5281/zenodo.23536475).

> **Note:** The raw MMWHS 2017 dataset must be obtained separately from the [MMWHS Challenge](http://www.sdspeople.fudan.edu.cn/zhuangxiahai/0/mmwhs/) organizers. This repository only redistributes the **preprocessed** derivatives that we produced for our experiments.

---

## Causal Necessity of Preprocessing

The CT and MR volumes in MMWHS were acquired **independently** from the same subjects at different times and on different scanners. Consequently, they reside in different physical coordinate spaces with large positional misalignments. Directly feeding these raw, unaligned volumes into a deep learning registration network is practically infeasible.

To address this, we employ a **two-stage paradigm**:
1. **Classical affine pre-alignment** (ANTs, label-guided) to coarsely align MR into CT space.
2. **Learning-based deformable refinement** (STARMorph) to learn the residual dense deformation field.

This repository documents the first stage and provides the resulting preprocessed data.

---

## Data Split

The MMWHS 2017 dataset contains **20 paired CT-MR cardiac volumes** (subject IDs: 1001-1020). We strictly partition them into non-overlapping subsets:

| Split | Subject IDs | # Pairs | Purpose |
|-------|-------------|---------|---------|
| Train | 1001-1015 | 15 | Model training |
| Val   | 1016-1017 | 2  | Model selection & convergence monitoring |
| Test  | 1018-1020 | 3  | Final evaluation  |

> **Important:** The validation set is used solely for selecting the best model checkpoint during training. The test set remains **completely unseen** until the final quantitative evaluation.

---

## Preprocessing Pipeline

### Step 1: Label-Guided Affine Pre-Alignment (ANTs)
- **Fixed image:** CT label map
- **Moving image:** MR label map
- **Method:** ANTs affine registration (`ants.registration`)
- **Transformation application:**
  - MR intensity image: linear interpolation
  - MR label map: nearest-neighbor interpolation (to preserve discrete labels)
- **Purpose:** Coarsely align MR into the CT coordinate space, ensuring cardiac regions overlap.

### Step 2: Canonical Reorientation
- Reorient all volumes to the standard **RAS** coordinate system.
- Eliminates scanner-specific orientation differences.

### Step 3: ROI Cropping
- Compute a 3D bounding box from the **union** of CT and MR label masks.
- Extract a cubic region centered on the cardiac region with a **1.2x padding margin**.
- Out-of-bound regions are padded with the background value.

### Step 4: Isotropic Resizing
- Resize cropped cubes to **128 x 128 x 128** voxels.
- Intensity images: bicubic interpolation (`order=3`)
- Label maps: nearest-neighbor interpolation (`order=0`)

### Step 5: Intensity Normalization
- Robust percentile-based normalization:
  - Clip intensity values at the **1st and 99th percentiles**
  - Linearly scale to **[0, 1]**

### Step 6: Label Remapping
- Original MMWHS label values: `[0, 205, 420, 500, 550, 600, 820, 850]`
- Remapped to contiguous integers: `[0, 1, 2, 3, 4, 5, 6, 7]`

| Remapped ID | Original Value | Structure |
|-------------|----------------|-----------|
| 0 | 0 | Background |
| 1 | 205 | Myocardium |
| 2 | 420 | Left Atrium |
| 3 | 500 | Left Ventricle |
| 4 | 550 | Right Atrium |
| 5 | 600 | Right Ventricle |
| 6 | 820 | Ascending Aorta |
| 7 | 850 | Pulmonary Artery |

---

## Final Image Dimensions

- **Input size to network:** 128 x 128 x 128 voxels
- **Format:** Python pickle (`.pkl`), each file contains a tuple `(image, label)`
- **Image dtype:** `float32` (normalized to [0, 1])
- **Label dtype:** `int32` (remapped to 0-7)

---

## Download

The preprocessed `.pkl` volumes are available on Zenodo:

**DOI:** [10.5281/zenodo.23536475](https://doi.org/10.5281/zenodo.23536475)


> If this preprocessed dataset is helpful to your research, we would be delighted. We hope this open resource facilitates verification by the research community.

---


## License & Disclaimer

- The **preprocessing code** in this repository is released under the MIT License.
- The **preprocessed data** on Zenodo is a derivative of the MMWHS 2017 Challenge dataset. Researchers must comply with the original MMWHS data usage agreement.
- The **raw MMWHS 2017 dataset** is not included in this repository. Please obtain it directly from the [MMWHS Challenge organizers].

---

## Contact

For questions regarding the preprocessing pipeline or the STARMorph framework, please open an issue in this repository.
