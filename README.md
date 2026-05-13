# ImageProcessing_Group1
Image Enhancement Pipeline for Road Damage Detection

# 🛣️ Road Damage Detection System

An Image processing pipeline for detecting and segmenting road surface damage from image sequences using OpenCV. Supports three damage types: **potholes**, **alligator cracking**, and **transverse cracking**.

---

## Features

- **Adaptive image enhancement** — automatically detects and corrects blur, noise, and low contrast before segmentation
- **Three specialized segmentation algorithms** — each tuned for a specific damage type's visual characteristics
- **Frame-by-frame playback** — browse through extracted video frames with pause/resume controls
- **Visual overlay output** — green fill + cyan contour drawn over detected damage regions
- **Binary mask output** — clean black-and-white mask for downstream use

---

## How It Works

### 1. Enhancement Pipeline (`dynamic_preprocess`)

Each frame is analyzed before segmentation and corrected only where needed:

| Issue Detected | Correction Applied |
|---|---|
| Blurry (Laplacian variance < 200) | Sharpening kernel (3×3) |
| Noisy (mean abs diff > 10) | Gaussian blur (3×3 / 5×5 / 7×7 based on severity) |
| Low contrast (std dev < 40 or histogram spread < 150) | CLAHE (if noisy/blurry) or Global Histogram Equalization |

### 2. Segmentation Algorithms

#### Pothole (`segment_pothole_fused`)
Uses adaptive thresholding on a heavily blurred image, filters contours by position (centre region of frame), draws thick contours to fuse nearby edges, then erodes the filled blob to recover the true pothole boundary.

#### Alligator Cracking (`segment_alligator`)
Uses morphological **black-hat** transform to isolate dark crack networks against lighter pavement. Filters resulting regions by area (≥ 8000 px²), aspect ratio (0.3–4.0), and solidity (> 0.4).

#### Transverse Cracking (`segment_precise_transverse`)
Applies Canny edge detection followed by a wide horizontal dilation kernel to bridge gaps in transverse cracks. Keeps only wide, shallow regions (aspect ratio > 5.0, area ≥ 10000 px²).

---

## Requirements

```
Python 3.x
opencv-python
numpy
```

## Usage

### 1. Prepare your data

Extract frames from a road damage video into a folder as sequentially numbered JPEG files (e.g. `50.jpg`, `55.jpg`, `60.jpg`, ...).

### 2. Configure the script

Edit the parameters at the bottom of the notebook:

```python
base_path   = "path/to/your/extracted/frames/"
damage_type = "pothole"   # "pothole" | "alligator" | "transverse"
start_index = 50          # First frame number to load
step        = 5           # Increment between frames
delay_ms    = 500         # Milliseconds between frames during playback
max_frames  = 100         # Maximum frames to process
```

### 3. Run the notebook

Open `final_code.ipynb` in Jupyter and run all cells.

Two windows will appear:
- **Segmentation Overlay** — enhanced frame with damage highlighted in green (cyan border)
- **Binary Mask** — the raw segmentation mask

### Keyboard Controls

| Key | Action |
|-----|--------|
| `p` | Pause / Resume playback |
| `q` | Quit |

---

## Project Structure

```
road-damage-detection/
│
├── final_code.ipynb
|__ pipeline.ipynb  # Main notebook
└── README.md
```

Your frame images should be stored separately and pointed to via `base_path`.

---

## Example Output

| Window | Description |
|--------|-------------|
| Segmentation Overlay | Grayscale frame with green damage fill and cyan contour border |
| Binary Mask | White region = detected damage, black = background |

---

## Damage Type Reference

| Type | Description | Typical Appearance |
|------|-------------|-------------------|
| Pothole | Bowl-shaped depression in pavement | Dark, roughly circular region |
| Alligator cracking | Network of interconnected cracks | Irregular mesh of thin dark lines |
| Transverse cracking | Single crack running across the road width | Wide, horizontal dark line |

---

## Limitations & Future Work

- Currently processes grayscale images only
- Segmentation thresholds are tuned for specific lighting conditions — may need adjustment for different datasets
- No deep learning component; purely classical image processing
- Potential improvements:
  - Multi-class detection in a single pass
  - Integration with a neural segmentation model (e.g. U-Net) for higher accuracy
  - Real-time video stream support
  - Severity scoring per detected region

---

## License

This project is for educational and research purposes.
