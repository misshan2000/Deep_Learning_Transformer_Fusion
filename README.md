# Deep Learning and Transformer Fusion Pipeline for Medical Image Classification

## Overview

This project provides a medical image classification pipeline based on the Onekey framework. The complete workflow includes data splitting, deep learning model training, Grad-CAM visualization, deep learning feature extraction, and Transformer-based feature fusion.

The pipeline is suitable for medical imaging data such as CT, MRI, DWI, endoscopy, and X-ray images. The input images are generally expected to be converted into `.jpg` or `.png` format before model training and feature extraction.

## Workflow

The project consists of five major steps:

```text
Step 0. Generate Data Splits
Step 1. Deep Learning Model Training
Step 2. Model Visualization with Grad-CAM
Step 3. Deep Learning Feature Extraction
Step 4. Transformer Fusion
```

## Project Structure

```text
.
├── Step0.Generate Data Splits.ipynb
├── Step1. Deep Learning Model Training.ipynb
├── Step2. Model Visualization with Grad-CAM.ipynb
├── Step3. Deep Learning Feature Extraction.ipynb
├── Step4. Transformer Fusion.ipynb
├── split_info/
├── dl_models/
├── features/
└── README.md
```

## Step 0. Generate Data Splits

The notebook `Step0.Generate Data Splits.ipynb` is used to generate training, validation, and testing splits.

Main functions include:

1. Reading a CSV file containing image IDs and labels.
2. Randomly splitting the dataset according to label distribution.
3. Supporting single-center random data splitting.
4. Supporting multi-center data splitting with a fixed testing set and re-sampled training/validation sets.
5. Generating `train-RND-*.txt` and `val-RND-*.txt` files for deep learning model training.

Example input label file:

```csv
ID,label,group
case001.png,0,train
case002.png,1,train
case003.png,0,test
```

Example output files:

```text
split_info/label-RND-0.csv
split_info/train-RND-0.txt
split_info/val-RND-0.txt
```

The generated `train-RND-0.txt` and `val-RND-0.txt` files usually contain two columns:

```text
case001.png    0
case002.png    1
```

The first column represents the image file name or image path, and the second column represents the class label.

## Step 1. Deep Learning Model Training

The notebook `Step1. Deep Learning Model Training.ipynb` is used to train medical image classification models.

The training process is based on the Onekey classification module and mainly uses the List mode. In this mode, the user needs to provide a training list, a validation list, a label file, and the image data directory.

Key parameters include:

```python
train_f = r''
val_f = r''
labels_f = r''
data_pattern = r''
model_name = 'ViT'
batch_size = 32
epochs = 3000
init_lr = 0.001
optimizer = 'sgd'
```

The default model is Vision Transformer, with the following settings:

```python
vit_settings = {
    'patch_size': 64,
    'dim': 1024,
    'depth': 6,
    'heads': 16,
    'mlp_dim': 768
}
```

The notebook also provides a Grid Search example for batch training across multiple random splits, model architectures, and learning-rate settings.

Supported model families include but are not limited to:

```text
AlexNet
VGG
ResNet
DenseNet
Inception
SqueezeNet
ShuffleNetV2
MobileNet
MNASNet
EfficientNet
Vision Transformer
```

Example output directory:

```text
dl_models/
```

After training, model weights, logs, and visualization-related files are usually saved under the model output directory.

## Step 2. Model Visualization with Grad-CAM

The notebook `Step2. Model Visualization with Grad-CAM.ipynb` is used to generate Grad-CAM visualizations for trained deep learning models.

Grad-CAM highlights the image regions that contribute most to the model prediction, which can help improve the interpretability of the classification model.

Main parameters:

```python
sample_dir = get_param_in_cwd('data_pattern')
model_root = r''
target_layer = ''
```

Parameter descriptions:

| Parameter | Description |
| --------- | ----------- |
| `sample_dir` | Directory containing images for visualization |
| `model_root` | Directory of the trained model |
| `target_layer` | Target network layer for Grad-CAM visualization |

If the target layer name is unknown, the model structure can be printed first. The user can then select a proper layer from the printed `Feature name` list, such as:

```text
layer4.1.conv2
features.denseblock4.denselayer32.conv2
avgpool
```

The Grad-CAM results are saved to:

```text
model_root/Grad-CAM/
```

Each Grad-CAM figure usually contains:

1. The original image.
2. The heatmap.
3. The overlay of the heatmap and the original image.

## Step 3. Deep Learning Feature Extraction

The notebook `Step3. Deep Learning Feature Extraction.ipynb` is used to extract deep learning features from trained models.

The extracted features can be used for downstream machine learning, statistical analysis, or Transformer-based feature fusion.

Main procedures include:

1. Loading image files for feature extraction.
2. Loading a trained deep learning model.
3. Printing the network layer names.
4. Selecting a specific layer for feature extraction.
5. Saving the extracted deep learning features.
6. Optionally compressing high-dimensional features.

By default, the notebook reads `.jpg` and `.png` files:

```python
samples = [
    os.path.join(mydir, f) 
    for f in os.listdir(mydir) 
    if f.endswith('.jpg') or f.endswith('.png')
]
```

The default feature extraction layer is:

```python
feature_name = 'avgpool'
```

Output files include:

```text
features/{model_name}_features.csv
features/{model_name}_compress_features.csv
```

Example format of the extracted deep learning feature file:

```csv
ID,DL_0,DL_1,DL_2,...,DL_n
case001.png,0.123,0.045,0.876,...,0.321
case002.png,0.234,0.067,0.765,...,0.432
```

The default feature compression dimension is 8:

```python
cm_features = compress_df_feature(
    features=features,
    dim=8,
    prefix='DL_',
    not_compress='ID'
)
```

Compressed features are saved to:

```text
features/{model_name}_compress_features.csv
```

## Step 4. Transformer Fusion

The notebook `Step4. Transformer Fusion.ipynb` is used for Transformer-based feature fusion and classification.

This step is designed to integrate multiple deep learning features, multi-sequence features, or features extracted from multiple models into a final classification model.

Main parameters include:

```python
model_root = r''
train = r''
val = r''
tests = [r'']
target_file = r''
input_dim = 128
bags_size = 4
normalize = True
header = 0
```

Parameter descriptions:

| Parameter | Description |
| --------- | ----------- |
| `model_root` | Output directory of the Transformer fusion model |
| `train` | Training feature file |
| `val` | Validation feature file |
| `tests` | List of testing feature files |
| `target_file` | Label file |
| `input_dim` | Input feature dimension |
| `bags_size` | Number of feature bags per sample |
| `normalize` | Whether to normalize features |
| `header` | Header row of the feature file |

The default code performs 100 repeated training runs:

```python
for i in range(100):
    model_root = os.path.join(model_root, f"try-{i}")
```

Each run is saved into a corresponding `try-*` subdirectory.

## Requirements

The project is designed to run in a Python environment. The main dependencies include:

```text
python
pandas
numpy
torch
monai
matplotlib
onekey_algo
```

Common dependencies can be installed using:

```bash
pip install pandas numpy torch monai matplotlib
```

The `onekey_algo` package is part of the Onekey framework and should be configured according to the local Onekey environment.

## Data Preparation

Before running the pipeline, the following files should be prepared.

### 1. Image Data

Images should be prepared in `.jpg` or `.png` format:

```text
data/
├── case001.png
├── case002.png
├── case003.png
└── ...
```

### 2. Label File

The label file should contain at least two columns:

```csv
ID,label
case001.png,0
case002.png,1
case003.png,0
```

For multi-center data or pre-defined train/test splits, a `group` column can be added:

```csv
ID,label,group
case001.png,0,train
case002.png,1,train
case003.png,0,test
```

### 3. labels.txt

The `labels.txt` file defines the class labels. For a binary classification task:

```text
0
1
```

## Running Order

The recommended running order is:

```text
1. Step0.Generate Data Splits.ipynb
2. Step1. Deep Learning Model Training.ipynb
3. Step2. Model Visualization with Grad-CAM.ipynb
4. Step3. Deep Learning Feature Extraction.ipynb
5. Step4. Transformer Fusion.ipynb
```

### Step 0: Generate Data Splits

Set the label file path:

```python
label_path = r'path/to/label.csv'
```

This step generates:

```text
split_info/train-RND-0.txt
split_info/val-RND-0.txt
```

### Step 1: Train Deep Learning Models

Set the required paths:

```python
train_f = r'path/to/train-RND-0.txt'
val_f = r'path/to/val-RND-0.txt'
labels_f = r'path/to/labels.txt'
data_pattern = r'path/to/image_dir'
```

Then run the training notebook.

### Step 2: Generate Grad-CAM Visualizations

Set the following parameters:

```python
sample_dir = r'path/to/image_dir'
model_root = r'path/to/trained_model'
target_layer = 'target_layer_name'
```

The Grad-CAM heatmaps will be generated after execution.

### Step 3: Extract Deep Learning Features

Set the following parameters:

```python
model_name = 'ViT'
model_root = r'path/to/model_root'
data_pattern = r'path/to/image_dir'
feature_name = 'avgpool'
```

This step generates:

```text
features/ViT_features.csv
features/ViT_compress_features.csv
```

### Step 4: Train Transformer Fusion Model

Set the feature files and label file:

```python
train = r'path/to/train_features.csv'
val = r'path/to/val_features.csv'
tests = [r'path/to/test_features.csv']
target_file = r'path/to/label.csv'
```

Then run the Transformer fusion model.

## Outputs

The main outputs of this project include:

```text
split_info/
├── label-RND-*.csv
├── train-RND-*.txt
└── val-RND-*.txt

dl_models/
└── trained deep learning models

features/
├── *_features.csv
└── *_compress_features.csv

model_root/
├── Grad-CAM/
└── try-*/
```

Descriptions:

- `train-RND-*.txt` and `val-RND-*.txt` are used for deep learning model training.
- `dl_models/` stores trained deep learning models.
- `Grad-CAM/` stores visualization results.
- `features/` stores extracted deep learning features.
- `try-*/` stores Transformer fusion model outputs from repeated training runs.

## Notes

1. Replace all empty paths `r''` in the notebooks with actual local paths before running.
2. For Windows paths, raw string format is recommended, for example `r'C:\path\to\data'`.
3. The `train.txt` and `val.txt` files should be Tab-separated.
4. `data_pattern` should point to the common image directory.
5. If absolute paths are already used in `train.txt` and `val.txt`, `data_pattern` can be set to `None`.
6. The Grad-CAM `target_layer` should be selected according to the specific model architecture.
7. The feature compression dimension `dim` should be smaller than the sample size.
8. In Transformer fusion, `input_dim` and `bags_size` should match the actual input feature format.

## Citation

If this pipeline is used in academic research, please cite the corresponding Onekey framework, deep learning framework, and relevant methodological references according to the actual implementation.
