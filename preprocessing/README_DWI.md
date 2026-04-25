# DWI/IVIM Parameter Map Processing

## Overview

This project provides a Jupyter Notebook workflow for processing diffusion-weighted MRI (DWI) and intravoxel incoherent motion (IVIM) data. The notebook is designed to calculate diffusion-related parameter maps, organize generated NIfTI files, and split multi-b-value DWI images into separate b-value volumes.

The workflow mainly includes:

- ADC map calculation using a mono-exponential diffusion model.
- IVIM-related parameter map calculation using a bi-exponential or microvascular diffusion model.
- Automatic organization of generated parameter maps into dedicated folders.
- Splitting of 4D DWI/IVIM images according to predefined b values.
- Interpretation of microvascular imaging parameters, including `vm`, `vs`, and `ANB`.
## Installation
Install the required dependencies using:
```
pip install -r requirements.txt

```
## Main Functions

The notebook uses functions from the `pixelmed_calc` package:

```python
from pixelmed_calc.medical_imaging.RadiologyComponents.diffusion import adc_map_dir
from pixelmed_calc.medical_imaging.RadiologyComponents.diffusion import ivim_param_maps_dir
from pixelmed_calc.medical_imaging.RadiologyComponents.diffusion import split_dwi_by_bvalue
```

### 1. ADC Map Calculation

The ADC map is calculated from DWI data using a mono-exponential diffusion model.

```python
cases_dir = r''      # Directory containing DWI cases
output_dir = r''     # Directory for saving ADC maps

adc_map_dir(cases_dir, output_dir)
```

### 2. IVIM Parameter Map Calculation

IVIM-related parameter maps are calculated from multi-b-value DWI data.

```python
cases_dir = r''      # Directory containing IVIM/DWI cases
output_dir = r''     # Directory for saving IVIM parameter maps

ivim_param_maps_dir(cases_dir, output_dir)
```

### 3. Result Organization

The notebook includes scripts to organize generated `.nii.gz` files according to their suffixes.

Files containing the following suffixes are copied into corresponding folders:

| File suffix | Output folder |
|---|---|
| `_ANB.nii.gz` | `ANB/` |
| `_vm.nii.gz` | `vm/` |
| `_vs.nii.gz` | `vs/` |
| `_ADC.nii.gz` | `ADC/` |
| Other `.nii.gz` files | `others/` |

Example output structure:

```text
output/
├── ANB/
├── vm/
├── vs/
├── others/
│   └── ADC/
```

The current script copies files into the target folders using `shutil.copy()`. Original files are therefore preserved in the source directory.

### 4. Multi-b-value DWI Splitting

The notebook also provides an example for splitting a 4D DWI/IVIM image into separate files according to b values.

```python
root_dir = '/path/to/sample_data/IVIM'

b_values = [0, 10, 50, 100, 400, 500, 800, 1000, 1200, 1500, 2000]
filename = 'IVIM.nii.gz'

split_dwi_by_bvalue(root_dir, b_values, filename=filename)
```

The `b_values` list must match the volume order of the original 4D DWI/IVIM image.

## Requirements

### Python Packages

The notebook requires the following Python components:

```text
pixelmed_calc
os
shutil
```

`os` and `shutil` are built-in Python libraries. The key external dependency is `pixelmed_calc`.

### Recommended Environment

A recommended environment includes:

```text
Python >= 3.8
pixelmedGAN
Jupyter Notebook or JupyterLab
pixelmed_calc
```

The exact installation method for `pixelmed_calc` should follow the package documentation or the local project environment in which it is provided.

## Input Data

The expected input data include DWI or IVIM images, typically stored as NIfTI files such as:

```text
.nii
.nii.gz
```

For ADC calculation, the input should contain DWI data suitable for mono-exponential model fitting.

For IVIM calculation, the input should contain multi-b-value DWI data suitable for IVIM or microvascular parameter estimation.

Before running the notebook, users should set:

```python
cases_dir = r'/path/to/input_cases'
output_dir = r'/path/to/output_results'
```

## Output Data

The workflow can generate the following parameter maps:

| Parameter | Description |
|---|---|
| `ADC` | Apparent diffusion coefficient map derived from the mono-exponential diffusion model. |
| `vm` | Mean microvascular velocity, reflecting the overall dynamic level of microvascular blood flow. |
| `vs` | Standard deviation of microvascular velocity, reflecting spatial heterogeneity of blood flow velocity. |
| `ANB` | Apparent number of branches or branch-related index, reflecting microvascular structural complexity. |

All output parameter maps are saved as NIfTI files, typically in `.nii.gz` format.

## Parameter Interpretation

### ADC

ADC reflects the apparent diffusion of water molecules in tissue. In tumor imaging, ADC is commonly used to evaluate tissue cellularity and diffusion restriction.

### Mean Velocity (`vm`)

`vm` represents the weighted average velocity across the microvascular network. It reflects the overall level of microvascular blood flow dynamics.

### Velocity Standard Deviation (`vs`)

`vs` describes the standard deviation of blood flow velocity across the vascular network. It can be used to quantify intratumoral heterogeneity in microvascular flow.

### Apparent Number of Branches (`ANB`)

`ANB` is a branch-related microvascular index. It characterizes the structural complexity of the microvascular network and may reflect abnormal vascular architecture within tumors.

## Recommended Workflow

A typical workflow is:

1. Prepare DWI or IVIM data in the required directory structure.
2. Set the input directory and output directory in the notebook.
3. Run ADC map calculation if mono-exponential diffusion parameters are needed.
4. Run IVIM parameter map calculation if microvascular parameters are needed.
5. Organize generated `.nii.gz` files into parameter-specific folders.
6. If needed, split the original 4D DWI/IVIM image into separate b-value images.
7. Use the generated parameter maps for downstream radiomics, deep learning, or statistical analysis.

## Example Directory Structure

```text
project/
├── DWI.ipynb
├── input_cases/
│   ├── case_001/
│   ├── case_002/
│   └── case_003/
└── output/
    ├── ANB/
    ├── vm/
    ├── vs/
    ├── ADC/
    └── others/
```

## Notes Before Running

Before running the notebook, please check the following:

- `cases_dir` points to the correct input image directory.
- `output_dir` points to the desired output directory.
- The input DWI/IVIM files are compatible with the `pixelmed_calc` functions.
- The b-value list matches the actual order of volumes in the 4D image.
- The output directory has write permission.
- The file naming convention is consistent with the suffix-based organization scripts.

## Suggested Use Cases

The generated parameter maps can be used for:

- Quantitative diffusion MRI analysis.
- Tumor microvascular heterogeneity assessment.
- Radiomics feature extraction.
- Deep learning model construction.
- Treatment response prediction.
- Pathological risk factor prediction.
- Prognostic modeling.

## File Description

```text
DWI.ipynb
```

This notebook contains the complete workflow for ADC calculation, IVIM parameter map calculation, result organization, and multi-b-value DWI splitting.
