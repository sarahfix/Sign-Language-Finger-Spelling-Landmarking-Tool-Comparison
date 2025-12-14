# Sign Language Finger Spelling Landmarking Tool Comparison

This project compares pose estimation tools for sign language finger spelling recognition. Tools are evaluated against a manually annotated baseline with 8 key landmarks: nose, left eye, right eye, thumb, pointer finger, middle finger, ring finger, and pinky finger.

## Quick Start

1. **Configure settings** in `models.ipynb`:
   - Set `FRAME_SET` (e.g., "frames2") to specify which frames to process
   - Set `OUTPUT_SUBDIR` (e.g., "2") to organize outputs

2. **Run pose estimation**:
   - Open `models.ipynb`
   - Execute cells for each tool (AlphaPose, OpenPose, YOLOv8, BlazePose)
   - Results save to `outputs/{tool_name}/{OUTPUT_SUBDIR}/`

3. **Compare with baseline**:
   - Open comparison notebooks (`alphapose.ipynb`, `openpose.ipynb`, etc.)
   - Run all cells to generate comparison visualizations and metrics

## Which Tools Were Evaluated

| Tool | Keypoints | Strengths | Weaknesses |
|------|-----------|-----------|------------|
| **AlphaPose** | 17 COCO | High accuracy, multi-person | Finger landmarks approximated |
| **OpenPose** | 25 body + face + hands | Excellent face detection, best overall | Finger landmarks approximated |
| **YOLOv8-Pose** | 17 COCO | Fast inference, balanced performance | Limited keypoints, finger landmarks approximated |
| **BlazePose MediaPipe** | 33 MediaPipe | Fast, optimized for mobile, better finger approximations | Finger landmarks still approximated (188-551px error) |
| **MediaPipe Hands** | 21 per hand | **Direct finger tip detection** | Hand-only (no body/face) |

## Installation

### Prerequisites
- Python 3.10+
- CUDA-capable GPU (recommended for AlphaPose, OpenPose, YOLO)

### Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install opencv-python numpy pandas matplotlib scipy ultralytics mediapipe

# Install tool-specific dependencies
# - YOLOv8: Installed via ultralytics package
# - BlazePose: Installed via mediapipe package
```

### Downloading AlphaPose and OpenPose

**These directories are not included in the repository and must be downloaded separately:**

#### AlphaPose Setup

1. Clone the AlphaPose repository:
   ```bash
   git clone https://github.com/MVIG-SJTU/AlphaPose.git
   ```

2. Follow the installation instructions in the AlphaPose repository:
   - Install dependencies (PyTorch, etc.)
   - Download pretrained models to `AlphaPose/pretrained_models/`
   - See: https://github.com/MVIG-SJTU/AlphaPose/blob/master/docs/INSTALL.md

3. Ensure the `AlphaPose/` directory is in the project root (same level as `models.ipynb`)

#### OpenPose Setup

**Option 1: Pre-built Binaries (Recommended for Windows)**

1. Download pre-built binaries from:
   https://github.com/CMU-Perceptual-Computing-Lab/openpose/releases

2. Extract to `openpose/` directory in the project root

3. Download models using the included script:
   ```bash
   cd openpose/models
   getBaseModels.bat  # Windows
   # or getBaseModels.sh  # Linux/Mac
   ```

**Option 2: Build from Source**

1. Follow the build instructions:
   https://github.com/CMU-Perceptual-Computing-Lab/openpose#installation

2. Place the built `openpose/` directory in the project root

3. Ensure the binary is at: `openpose/bin/OpenPoseDemo.exe` (Windows) or `openpose/bin/OpenPoseDemo` (Linux/Mac)

## Project Structure

```
landmarking/
├── models.ipynb              # Main notebook - run all tools
├── alphapose.ipynb           # AlphaPose comparison
├── openpose.ipynb            # OpenPose comparison
├── yolo.ipynb                # YOLOv8 comparison
├── blazepose_mediapipe.ipynb # BlazePose MediaPipe comparison
├── mediapipe_hands.ipynb     # MediaPipe Hands detection
├── AlphaPose/                # NOT INCLUDED - must download separately (see Setup)
├── openpose/                 # NOT INCLUDED - must download separately (see Setup)
├── frames/                   # Input frames (not tracked in git)
│   ├── frames1/
│   ├── frames2/              # Primary test set
│   └── ...
├── outputs/                  # Tool outputs (not tracked in git)
│   ├── alphapose/{OUTPUT_SUBDIR}/
│   ├── openpose/{OUTPUT_SUBDIR}/
│   ├── yolov8/{OUTPUT_SUBDIR}/
│   ├── blazepose_mediapipe/{OUTPUT_SUBDIR}/
│   ├── mediapipe_hands/{OUTPUT_SUBDIR}/
│   ├── simple_landmarks/{OUTPUT_SUBDIR}/  # Baseline annotations
│   └── comparisons/
└── videos/                   # Source videos (not tracked in git)
```

**Note:** The `AlphaPose/` and `openpose/` directories are not included in this repository due to their large size. You must download them separately (see Setup section below).

## Configuration

### Frame Set Selection

In `models.ipynb`, configure which frames to process:

```python
FRAME_SET = "frames2"  # Options: "frames1", "frames2", "frames3", etc.
OUTPUT_SUBDIR = "2"    # Output subdirectory number
```

### Output Directory Structure

All tools save outputs to: `outputs/{tool_name}/{OUTPUT_SUBDIR}/`

- **AlphaPose**: `outputs/alphapose/{OUTPUT_SUBDIR}/overlays/` and `json/`
- **OpenPose**: `outputs/openpose/{OUTPUT_SUBDIR}/overlays/` and `json/`
- **YOLOv8**: `outputs/yolov8/{OUTPUT_SUBDIR}/` (images and JSON)
- **BlazePose**: `outputs/blazepose_mediapipe/{OUTPUT_SUBDIR}/`

## Results Summary

### Overall Performance

| Tool | Average Error | Face Landmarks | Finger Landmarks | Notes |
|------|---------------|----------------|------------------|-------|
| **OpenPose** | **564.78px** | 5-15px | 500-1600px | Best overall, but finger landmarks approximated |
| **AlphaPose** | 595.90px | 5-25px | 500-1700px | Good body pose |
| **YOLOv8** | ~580px | 5-25px | 500-1700px | Fast, balanced |
| **BlazePose MediaPipe** | **232.43px** | 4-24px | 188-551px | Fast, better than general tools, but finger landmarks approximated |
| **MediaPipe Hands** | **20-100px** | N/A | **20-100px** | **Direct finger detection** |

### Key Findings

1. **Face Detection**: All tools excel (errors < 25px)
2. **Finger Detection**: 
   - **Hand-specific models (MediaPipe Hands)**: Excellent (20-100px errors) - direct detection of all 5 finger tips
   - **BlazePose**: Partial detection (188-551px average):
     - Directly detects: thumb, index, pinky (200-350px errors) - much better than traditional tools
     - Approximates: middle, ring fingers (500+px errors) - uses wrist/pinky positions
   - **General pose tools (OpenPose, AlphaPose, YOLOv8)**: Poor (500-1700px errors) - approximate all fingers from wrist/elbow positions
3. **Critical Limitation**: General pose tools don't detect individual finger tips directly - they approximate from wrist/hand positions

## Conclusions

### For Sign Language Finger Spelling

**Best Approach: Hybrid Method**

1. **MediaPipe Hands** for finger detection
   - Direct finger tip detection (5 finger tips)
   - Expected error: 20-100px (vs. 500-1700px with general tools)
   - Fast, real-time capable

2. **OpenPose** for face/body context
   - Face landmarks (nose, eyes)
   - Expected error: 5-15px for face
   - Full body pose context

## Baseline Annotations

Baseline annotations were created using this repository: https://github.com/krasch/simple_landmarks.git.
The annotations use this JSON structure:

```json
{
    "nose": {"coordinates": {"x": 528, "y": 928}, "status": "ok"},
    "left_eye": {"coordinates": {"x": 473, "y": 870}, "status": "ok"},
    "right_eye": {"coordinates": {"x": 604, "y": 876}, "status": "ok"},
    "thumb": {"coordinates": {"x": 361, "y": 896}, "status": "ok"},
    "pointer_finger": {"coordinates": {"x": 307, "y": 585}, "status": "ok"},
    "middle_finger": {"coordinates": {"x": 272, "y": 505}, "status": "ok"},
    "ring_finger": {"coordinates": {"x": 195, "y": 480}, "status": "ok"},
    "pinky_finger": {"coordinates": {"x": 112, "y": 563}, "status": "ok"}
}
```

## Methodology

### Evaluation Metrics

- **Euclidean Distance**: Pixel distance between predicted and ground truth
- **Mean Error**: Average error across all landmarks per frame
- **Per-Landmark Error**: Individual error for each of 8 landmarks

### Baseline Annotation

- Manually annotated landmarks for test frames
- 8 landmarks: nose, left_eye, right_eye, thumb, pointer_finger, middle_finger, ring_finger, pinky_finger
- Stored in `outputs/simple_landmarks/{OUTPUT_SUBDIR}/json/`

### Comparison Process

1. Run each tool on test frames
2. Extract relevant keypoints matching baseline landmarks
3. Map tool-specific keypoint indices to baseline landmarks
4. Calculate Euclidean distances
5. Generate visualizations and summary statistics

## Limitations

- Small test set (6 frames evaluated so far)
- Approximation mapping for finger landmarks in general tools
- Single person evaluation
- Static images


## Acknowledgments

- [AlphaPose](https://github.com/MVIG-SJTU/AlphaPose)
- [OpenPose](https://github.com/CMU-Perceptual-Computing-Lab/openpose)
- [YOLOv8](https://github.com/ultralytics/ultralytics)
- [MediaPipe](https://github.com/google/mediapipe)

