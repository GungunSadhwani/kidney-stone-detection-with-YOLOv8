# Kidney Stone Detection with YOLOv8

Object-detection project that locates kidney stones in medical images using **YOLOv8n**
(Ultralytics). The model is fine-tuned from COCO-pretrained weights on a custom single-class
dataset (`kidney_stone`) and predicts a bounding box around each stone.

> **Disclaimer:** Educational project only. It is not a medical device and must not be used
> for diagnosis or treatment decisions.

## Highlights
- YOLOv8 Nano fine-tuned with transfer learning (fast enough to train on a free Colab GPU)
- Single-class detector: `kidney_stone`
- Training, validation, test-set evaluation and bounding-box visualisation in one notebook/script

## Tech stack
Python, Ultralytics YOLOv8, OpenCV, Matplotlib, Google Colab

## Dataset
- Source: _add dataset name and link here (e.g. Roboflow / Kaggle)_
- Imaging type: _CT / ultrasound / X-ray_
- Size: _N train / N val / N test images_
- Format: YOLO (one `.txt` label per image: `class x_center y_center width height`, normalised)

Expected layout (the dataset is not included in this repo):

```
Kidney_dataset/
├── train/  images/  labels/
├── valid/  images/  labels/
└── test/   images/  labels/
```

## Setup
```bash
git clone https://github.com/<your-username>/kidney-stone-detection.git
cd kidney-stone-detection
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Usage
The project was developed in Google Colab (`Kidney_Stone_Detection.ipynb`): mount Google Drive, put the
dataset at `/content/drive/MyDrive/Kidney_dataset`, and run the cells in order.

Equivalent commands with the YOLO CLI:

```bash
# Train
yolo detect train data=data.yaml model=yolov8n.pt epochs=50 imgsz=512 batch=8 \
     project=runs/train name=kidney_stone_detect

# Evaluate on the test split
yolo detect val data=data.yaml model=runs/train/kidney_stone_detect/weights/best.pt split=test imgsz=512

# Predict on an image
yolo detect predict model=runs/train/kidney_stone_detect/weights/best.pt source=path/to/image.jpg save=True
```

`data.yaml`:
```yaml
path: /path/to/Kidney_dataset
train: train/images
val: valid/images
test: test/images
nc: 1
names: ['kidney_stone']
```

## Training configuration
| Setting | Value |
|---|---|
| Model | YOLOv8n (pretrained `yolov8n.pt`) |
| Epochs | 50 |
| Image size | 512 |
| Batch size | 8 |

## Results
Evaluated with Ultralytics 8.3.235 on a Tesla T4 GPU (Google Colab), using the best checkpoint
(`best.pt`: 72 layers, ~3.0M parameters, 8.1 GFLOPs) on the **validation split**
(123 images, 325 stone instances):

| Split | Images | Instances | Precision | Recall | mAP@0.5 | mAP@0.5:0.95 |
|---|---|---|---|---|---|---|
| Validation | 123 | 325 | 0.786 | 0.680 | 0.731 | 0.307 |

**Inference speed (T4):** ~1.8-2.6 ms preprocess, ~3.9-5.1 ms inference, ~2.5-4.2 ms postprocess per image
(roughly 10-12 ms end to end, i.e. real-time capable).

**Interpretation**
- Precision of 0.786 means about 79% of predicted boxes are real stones; recall of 0.68 means about 68% of
  labelled stones are found.
- mAP@0.5 of 0.731 shows the model localises stones reasonably well. The lower mAP@0.5:0.95 (0.307)
  indicates the boxes are not always tight, which is expected for very small objects like stones.
- These numbers are on the validation split, which was also used to select the best checkpoint during
  training, so they may be slightly optimistic. A final score on the untouched test split
  (`split=test`) is still to be added.

Example predictions and curves are saved by Ultralytics in `runs/detect/val/` (e.g. `val_batch0_pred.jpg`,
`PR_curve.png`, `confusion_matrix.png`). Add a few of them here:

```markdown
![Predictions](assets/val_batch0_pred.jpg)
```

## Limitations
- Moderate recall (0.68): about one in three labelled stones is missed, which matters for a screening use case.
- Small, single-source dataset: results may not generalise to other scanners or hospitals.
- Single class only; no stone size/type classification.
- Not clinically validated.

## Future work
- Larger/more diverse data and stronger augmentation
- Compare YOLOv8s/m, tune hyperparameters
- Deploy a Streamlit/Gradio demo

## License
MIT (add a `LICENSE` file). Check the dataset's own license before redistributing.
