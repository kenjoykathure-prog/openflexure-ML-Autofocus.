# ML-Predictive-Autofocus-OpenFlexure

**Machine Learning-Based Predictive Autofocus for High-Resolution OpenFlexure Microscopy Using Representative Defocus Ranges**

[![Platform](https://img.shields.io/badge/platform-OpenFlexure%20Microscope-blue)]()
[![Hardware](https://img.shields.io/badge/hardware-Raspberry%20Pi%204-red)]()
[![License](https://img.shields.io/badge/license-MIT-green)]()

> Code, trained models, and evaluation scripts accompanying the manuscript submitted to *SLAS Technology*.

## Overview

Conventional autofocus on low-cost, embedded microscopy platforms relies on iterative axial search, which is slow and computationally expensive on resource-constrained hardware. This repository implements a **predictive,  autofocus framework** for the [OpenFlexure Microscope](https://openflexure.org/) that estimates the required Z-axis correction directly from image-derived sharpness features — eliminating exhaustive z-stack search.

Two regression models are trained and compared:

- **Random Forest Regression (RF)** — test R² = 0.981, MAE = 137.44 motor steps (6.87 µm)
- **Support Vector Regression (SVR, RBF kernel)** — test R² = 0.96, MAE = 224.87 motor steps (11.24 µm)

Both models run inference on a **Raspberry Pi 4** in under 5 seconds per field of view, independent of the initial defocus magnitude — a 10–30× speed-up over the microscope's native and custom Variance-of-Laplacian search algorithms. Both models are also benchmarked live, on-device, against the OpenFlexure's **inbuilt autofocus** and a **custom Variance-of-Laplacian search**, across three specimen types.

## Key Features

- Lightweight, six-feature representation (no CNN, no GPU required) derived from a **pair of images** taken 200 motor steps apart
- Trained and validated on **1,040 labelled images** across 8 fields of view, with ground truth established via hierarchical coarse-to-fine search
- Live, on-device comparison of **four autofocus methods**: SVR, Random Forest, a custom Variance-of-Laplacian search, and the OpenFlexure's inbuilt autofocus
- Evaluated for generalization across **three specimen types**: Giemsa-stained blood smears (100× oil immersion), unstained blood smears, and formalin-fixed paraffin-embedded gastric cancer tissue (40× dry)
- Full evaluation suite: MAE, RMSE, R², relative error, coefficient of variation (repeatability), CPU/RAM utilization

## Repository Structure

```
├── training_data/                     # Labelled image dataset (1,040 images, 8 fields of view) used for model training/testing
├── images/                            # Raw/representative microscopy images (stained, unstained, cancer tissue)
├── training/
│   ├── train_svr.py                    # SVR training, grid search (C, gamma, epsilon), 5-fold CV
│   └── train_random_forest.py          # Random Forest training, grid search (n_estimators, max_depth, min_samples_split), 5-fold CV
├── live_testing/
│   ├── test_svr.py                     # Live autofocus trial runner — SVR
│   ├── test_random_forest.py           # Live autofocus trial runner — Random Forest
│   ├── test_custom_af.py               # Live autofocus trial runner — custom Variance-of-Laplacian search
│   └── test_inbuilt_af.py              # Live autofocus trial runner — OpenFlexure inbuilt autofocus
├── plotting/
│   ├── plot_stained.py                 # Figures for the Giemsa-stained blood smear results (Panel A in Figs. 1-6)
│   ├── plot_unstained.py               # Figures for the unstained blood smear results (Panel B)
│   └── plot_cancer_tissue.py           # Figures for the FFPE cancer tissue results (Panel C)
├── models/                             # Serialized trained models (.pkl)
├── results/                            # Output figures/tables (VoL, execution time, CV, relative error, CPU/RAM)
├── requirements.txt
└── README.md
```

> File names above are placeholders matching the four training/testing/plotting groups you described — rename them to match your actual scripts if they differ.

## Hardware Requirements

- OpenFlexure Microscope (motorised XYZ stage)
- Raspberry Pi 4 Model B
- Raspberry Pi Camera Module V2 (Sony IMX219)
- Objective lenses used in this study: 100× oil immersion (NA 1.25), 40× dry (NA 0.65)

## Installation

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
```

## Usage

### 1. Train the models

```bash
python training/train_svr.py --data training_data/ --output models/svr_model.pkl
python training/train_random_forest.py --data training_data/ --output models/rf_model.pkl
```

Both scripts run the grid search and 5-fold cross-validation described in the manuscript (§2.8) and report held-out test MAE, RMSE, and R².

### 2. Run live autofocus trials on the OpenFlexure Microscope

Each script runs the full trial protocol (initial defocus offsets of 500/1000/1500 motor steps, n = 7–10 trials per condition) for one autofocus method and logs execution time, final VoL, final Z-position, CPU, and RAM usage.

```bash
python live_testing/test_svr.py            --model models/svr_model.pkl --sample stained
python live_testing/test_random_forest.py  --model models/rf_model.pkl  --sample stained
python live_testing/test_custom_af.py      --sample stained
python live_testing/test_inbuilt_af.py     --sample stained
```

Repeat with `--sample unstained` and `--sample cancer_tissue` for the other two specimen types.

### 3. Generate figures

Each plotting script reads the logged trial results for one sample type and reproduces the corresponding panel of Figs. 1–6 (final VoL, execution time, CV, relative error, CPU, RAM):

```bash
python plotting/plot_stained.py        --results results/stained/
python plotting/plot_unstained.py      --results results/unstained/
python plotting/plot_cancer_tissue.py  --results results/cancer_tissue/
```

## Dataset

The training dataset (1,040 labelled images, 8 fields of view) is in `training_data/`, or available upon reasonable request if not included directly in the repo — see [Data Availability](#data-availability). Ground-truth focal planes were established using a four-stage hierarchical search (ultra-coarse → coarse → fine → verification) with the Variance of Laplacian as the sharpness metric. Representative raw images (stained, unstained, cancer tissue) are in `images/`.

## Results Summary

| Model | Test MAE | Test RMSE | Test R² |
|---|---|---|---|
| Random Forest | 137.44 steps (6.87 µm) | 228.16 steps (11.41 µm) | 0.981 |
| SVR (RBF) | 224.87 steps (11.24 µm) | 291.78 steps (14.58 µm) | 0.970 |

Both models completed autofocus in **under 5 seconds** on stained, unstained, and cancer-tissue samples across 500–1500 motor-step defocus offsets, versus 29–118 seconds for the custom exhaustive search. Full per-condition breakdowns (VoL, execution time, CV, relative error, CPU/RAM) for all four methods across all three sample types are tabulated in `SUPPLEMENTARY.pdf` / `SUPPLEMENTARY.md` and generated by the `plotting/` scripts into `results/`.

## Citation

If you use this code or dataset, please cite:

```
K. Kathure, A. Kiroe, C. Ominde, D. M. Memeu.
"Machine Learning-Based Predictive Autofocus for High-Resolution OpenFlexure
Microscopy Using Representative Defocus Ranges." Submitted to SLAS Technology, 2026.
```

## Data Availability

The datasets generated during this study are available from the corresponding author upon reasonable request.

## Authors

- **Kenjoy Kathure Kinyua** — Department of Physics, JKUAT (corresponding author) — kenjoykathure@gmail.com
- **Dr. Anthony J. Kiroe** — Department of Physics, JKUAT
- **Dr. Calvine Ominde** — Department of Physics, JKUAT
- **Dr. Daniel M. Memeu** — Department of Physical Sciences, MUST

## Acknowledgements

Built on the open-source [OpenFlexure Microscope](https://openflexure.org/) platform.

## License

This project is released under the MIT License — see `LICENSE` for details. (Update if a different license applies.)
