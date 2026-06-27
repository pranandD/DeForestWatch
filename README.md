# 🌳 DeforestWatch

> Satellite-based deforestation detection using transfer learning. Fine-tunes ResNet50 on EuroSAT to classify Sentinel-2 imagery into 10 land-cover types (98% accuracy), then applies patch-based change detection to quantify forest loss over time. Demonstrated on the Maya Forest, Petén, Guatemala (2018 → 2026).

---

## 📋 Table of Contents
- [Overview](#overview)
- [Results](#results)
- [Dataset](#dataset)
- [Model Architecture](#model-architecture)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)
- [Requirements](#requirements)
- [Limitations](#limitations)
- [Paper](#paper)
- [Author](#author)

---

## Overview

DeforestWatch is a reproducible machine learning pipeline for automated land-cover classification and deforestation change detection using freely available satellite imagery.

**The pipeline has three stages:**

1. **Train** — Fine-tune a ResNet50 CNN on the EuroSAT benchmark (27,000 Sentinel-2 patches, 10 land-cover classes)
2. **Classify** — Apply the trained model patch-by-patch to two Sentinel-2 true-colour images of the same region from different years
3. **Detect** — Compare the two land-cover maps to quantify where and how much forest was lost

Everything runs on free infrastructure: [Kaggle Notebooks](https://www.kaggle.com) (GPU) + [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu) (satellite imagery).

---

## Results

### EuroSAT Classification Accuracy

| Metric | Score |
|--------|-------|
| Overall Accuracy | **98%** |
| Macro Precision | 0.98 |
| Macro Recall | 0.98 |
| Macro F1-Score | 0.98 |
| Test Set Size | 2,688 patches |

### Maya Forest Change Detection (Petén, Guatemala)

| Land Cover Class | 2018 | 2026 | Change |
|-----------------|------|------|--------|
| Forest | 61.2% | 15.4% | ▼ −45.8 pp |
| HerbaceousVegetation | 20.9% | 52.6% | ▲ +31.7 pp |
| Pasture | 11.7% | — | — |
| AnnualCrop | — | 10.9% | ▲ +10.9 pp |

**51.9%** of the study area experienced forest loss over the 8-year period.  
**4.6%** showed regrowth.  
**15.9%** remained stable forest.

![Land Cover Change Detection](figures/maya_forest_land_cover_map.png)

---

## Dataset

### Training: EuroSAT
- **Source:** [EuroSAT RGB on Kaggle](https://www.kaggle.com/datasets/apollo2506/eurosat-dataset)
- **Size:** 27,000 Sentinel-2 image patches
- **Patch size:** 64 × 64 pixels at 10 m/pixel
- **Classes:** AnnualCrop, Forest, HerbaceousVegetation, Highway, Industrial, Pasture, PermanentCrop, Residential, River, SeaLake
- **Split:** 80% train / 10% validation / 10% test (seed=837)

### Inference: Sentinel-2 TCI
- **Source:** [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu)
- **Tile:** T15QYV (Petén, Guatemala)
- **Resolution:** 10 m/pixel (TCI RGB product)
- **Before image:** 8 January 2018
- **After image:** 23 April 2026
- **Study area:** ~114 km²

---

## Model Architecture

```
Input (64×64×3)
    ↓
Resizing (224×224)
    ↓
RandomFlip [training only]
    ↓
ResNet50 preprocess_input
    ↓
ResNet50 backbone (ImageNet pretrained)
    ↓
GlobalAveragePooling → 2048-dim embedding
    ↓
Dense(10, activation='linear')
    ↓
Output: 10 land-cover class logits
```

**Training — Phase 1 (Head Only)**
- Frozen backbone, train Dense layer only
- Adam, lr = 1×10⁻³, 10 epochs

**Training — Phase 2 (Fine-Tuning)**
- Unfreeze top ResNet50 layers (BatchNorm stays frozen)
- Adam, lr = 1×10⁻⁵, up to 15 epochs
- EarlyStopping: patience=3, restore_best_weights=True


---

## How to Run

### Step 1 — Clone the repo
```bash
git clone https://github.com/prananddesai/DeforestWatch.git
cd DeforestWatch
```

### Step 2 — Install dependencies
```bash
pip install -r requirements.txt
```

### Step 3 — Download the EuroSAT dataset
Download from [Kaggle](https://www.kaggle.com/datasets/apollo2506/eurosat-dataset) and place in:
```
/kaggle/input/datasets/prananddesai/eurosat/2750/
```

### Step 4 — Train the model
Run `DeForestWatch.ipynb` on a GPU runtime (Kaggle or Google Colab).  
The trained model is saved to `/kaggle/working/eurosat_model.keras`.

### Step 5 — Download Sentinel-2 imagery
1. Register at [dataspace.copernicus.eu](https://dataspace.copernicus.eu)
2. Search for tile **T15QYV**, Sentinel-2 L2A, TCI product
3. Download one image from **June–August 2018** and one from the same season in your target year
4. Place the `.jp2` files in your dataset folder

### Step 6 — Run change detection
Run `notebooks/02_deforestation_detection.ipynb`.  
This generates:
- `maya_forest_land_cover_map.png` — side-by-side land cover maps
- `maya_forest_change_map.png` — forest loss visualisation

---

## Requirements

```
tensorflow>=2.10
numpy
Pillow
scikit-learn
matplotlib
```

Install with:
```bash
pip install -r requirements.txt
```

> **JP2 support** (for Sentinel-2 files): run `apt-get install -y libopenjp2-7` before loading `.jp2` files in Pillow.

---

## Limitations

- **Domain shift** — Model trained on European EuroSAT imagery; tropical forest in Guatemala has different spectral characteristics
- **Patch resolution** — Each patch covers 640 × 640 m; small-scale clearings are not detectable
- **RGB only** — 10 additional Sentinel-2 spectral bands (including near-infrared) are not used
- **No field validation** — Results are model predictions, not ground-truth verified
- **Seasonal variation** — Two images from slightly different calendar months; phenological changes may influence results
---

## Acknowledgements

- [EuroSAT](https://github.com/phelber/EuroSAT) — Helber et al., IEEE JSTARS 2019
- [Copernicus Programme](https://www.copernicus.eu) — European Space Agency, Sentinel-2 imagery
- [Global Forest Watch](https://www.globalforestwatch.org) — World Resources Institute
- [Hansen Global Forest Change](https://glad.earthengine.app/view/global-forest-change) — University of Maryland GLAD Laboratory

---

*Built as part of an undergraduate research project at SPIT, Mumbai.*
