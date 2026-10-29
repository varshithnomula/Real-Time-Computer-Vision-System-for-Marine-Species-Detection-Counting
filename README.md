# Real-Time Computer Vision System for Marine Species Detection and Counting

This repository contains a two-stage computer-vision pipeline for detecting,
classifying, and counting marine zooplankton species in microscopy images.

## Repository contents

```text
.
├── classification_model/
│   ├── classification_report.csv
│   ├── metrics.json
│   ├── mobilenet_v3_small_best.pth
│   └── mobilenet_v3_small_finetuned_best.pth
└── pi_depoly/
    ├── pipeline_zooplankton.py
    ├── requirements.txt
    ├── yolo_best.pt
    ├── mobilenet_v3_small_finetuned_best.pth
    ├── pelgas-zooplankton (1).ipynb
    ├── yolo-zooplankton.ipynb
    ├── comparison_report.csv
    ├── comparison_chart.png
    └── comparison_temp/ and gradio_results/
```

### `classification_model`

Contains the MobileNetV3-small classifier weights and evaluation results:

- `mobilenet_v3_small_best.pth`: classifier checkpoint.
- `mobilenet_v3_small_finetuned_best.pth`: fine-tuned classifier checkpoint
  used by the deployment pipeline.
- `classification_report.csv`: per-class precision, recall, F1-score, and
  support.
- `metrics.json`: saved evaluation metrics.

### `pi_depoly`

This is the deployment application. The folder name is retained for
compatibility with the existing project layout.

- `pipeline_zooplankton.py`: Gradio application and inference pipeline.
- `yolo_best.pt`: YOLO object-detection weights.
- `mobilenet_v3_small_finetuned_best.pth`: classification weights used by the
  application.
- `requirements.txt`: Python dependencies.
- `pelgas-zooplankton (1).ipynb`: training/evaluation notebook.
- `yolo-zooplankton.ipynb`: YOLO-related notebook.
- `comparison_temp/` and `gradio_results/`: example generated outputs.

## How the inference pipeline works

1. YOLO detects objects in the uploaded image.
2. Each detected object is cropped from the original image.
3. MobileNetV3-small classifies each crop into one of the supported species.
4. The application draws bounding boxes and labels with confidence scores.
5. It saves an annotated image and a CSV report containing species,
   confidence, and bounding-box coordinates.

The application runs on CPU by default, which is suitable for Raspberry Pi
deployment but may be slower than GPU inference.

## Installation

From the repository root, create and activate a virtual environment:

### Windows PowerShell

```powershell
cd pi_depoly
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Linux or Raspberry Pi

```bash
cd pi_depoly
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If a Raspberry Pi has limited memory or a platform-specific PyTorch wheel is
required, install the compatible PyTorch and torchvision wheels for that
device before installing the remaining packages.

## Run the Gradio application

Run the following commands from inside `pi_depoly`, so the relative model
paths in the script resolve correctly:

```bash
python pipeline_zooplankton.py
```

Then open [http://127.0.0.1:7860](http://127.0.0.1:7860) in a browser.

### Single image analysis

1. Open the **Single Image** tab.
2. Upload a microscopy image.
3. Select **Run Analysis**.
4. Review the annotated image and species-count table.
5. Download the generated CSV report.

Generated single-image files are written to `gradio_results/`.

### Compare two sample folders

1. Open the **Sample Comparison** tab.
2. Enter the paths to two folders containing images.
3. Select **Compare Samples**.
4. Review the comparison chart and table.
5. Download the comparison CSV report.

The script accepts common image formats such as `.jpg`, `.jpeg`, `.png`, and
`.bmp`. The comparison folders must be accessible from the machine running the
application.

To stop the server, press `Ctrl+C` in the terminal.

## Run the notebooks

Start Jupyter from the repository root or from `pi_depoly`:

```bash
pip install jupyter
jupyter notebook
```

Open either notebook in the `pi_depoly` folder and run the cells in order.
The notebooks contain the training/evaluation workflow and YOLO experiments.
Update dataset paths in the notebook before running it on a new dataset.

## Model and file path notes

`pipeline_zooplankton.py` expects these files in the same directory as the
script:

```text
pi_depoly/yolo_best.pt
pi_depoly/mobilenet_v3_small_finetuned_best.pth
```

Do not move the model files without also updating `YOLO_WEIGHTS` and
`CLASSIFIER_WEIGHTS` near the top of the script.

## Troubleshooting

- **Model not found:** run the script from `pi_depoly`, or update the model
  paths in `pipeline_zooplankton.py`.
- **Port already in use:** stop the existing process using port `7860`, or
  change `server_port` near the end of the script.
- **Slow inference:** the application is configured with `DEVICE = 'cpu'`.
  CPU inference is expected on Raspberry Pi.
- **Dependency errors:** use a fresh virtual environment and install the
  versions listed in `pi_depoly/requirements.txt`.

The local virtual environment and Gradio runtime certificate are intentionally
excluded from version control.
