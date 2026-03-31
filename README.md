# Semi-automated Scan Reorientation Workflow

A Jupyter notebook workflow for orientation quality control and manual reorientation of 3D NIfTI scans.

This repository provides a practical semi-automated pipeline for datasets in which a small subset of scans do not match the expected orientation relative to a reference template or the rest of the cohort. The workflow is designed for rapid visual review, selective intervention, and reproducible tracking of manual corrections.

## What this workflow does

The notebook implements a three-step workflow:

1. **Batch QC visualization**
   - Scans a folder of NIfTI images
   - Extracts the center slice in the axial, coronal, and sagittal planes
   - Builds paged contact-sheet figures with **20 subjects per figure**
   - Places **one subject per row** and **three views per row**
   - Saves each page as both **PNG** and **PDF**
   - Displays each page directly in the notebook for immediate review

2. **Identify problematic scans**
   - Review the exported QC figures
   - Decide which scans appear mis-oriented relative to the template or the majority of the dataset
   - Copy those scans into a dedicated folder for manual handling

3. **Interactive manual reorientation**
   - Opens one scan at a time alongside a template reference
   - Lets the user apply flip operations through simple keyboard commands
   - Saves corrected outputs using the **template affine and header**
   - Tracks progress in a CSV file so the workflow can be paused and resumed

## Typical use case

This workflow is useful when:
- most scans are already correctly oriented
- only a subset of scans need manual correction
- you want a lightweight visual QC step before editing anything
- you want to keep a record of which files were changed and how they were handled

## Requirements

Python packages:

```bash
pip install numpy pandas nibabel matplotlib jupyter ipython
```

```txt
numpy
pandas
nibabel
matplotlib
jupyter
ipython
```

## Input data

The notebook is designed for **3D NIfTI scans** (`.nii` or `.nii.gz`).

Supported behavior:
- 3D NIfTI files are used directly
- 4D files with a singleton last dimension are squeezed to 3D
- 4D files with multiple volumes use the **first volume** for QC display

## Workflow details

### Step 1. Batch visualization for orientation QC

The first code cell generates paged contact-sheet QC figures for all scans in an input folder.

#### Key characteristics
- **20 subjects per figure** by default
- **1 subject per row**
- **3 columns per row**
  - axial center slice
  - coronal center slice
  - sagittal center slice
- each page is:
  - displayed directly in the notebook
  - saved as a `.png`
  - saved as a `.pdf`

#### Editable paths at the top of the cell
```python
SCAN_DIR = Path("/path/to/files/to/inspect")
OUTPUT_FIG_DIR = Path("/path/to/output/qc_figures")
FILE_PATTERN = "*.nii.gz*"
```

#### Useful display settings
```python
SUBJECTS_PER_FIGURE = 20
DOWNSAMPLE = 1
ROW_HEIGHT = 2.2
FIGURE_WIDTH = 12
USE_PERCENTILE_WINDOW = True
PERCENTILE_RANGE = (1, 99)
SAVE_DPI = 200
```

#### Output
Each saved page contains:
- scan name
- image shape
- affine axis codes
- center slices in three planes

This makes it easy to quickly review many subjects and flag scans that need manual correction.

### Step 2. Collect scans that need correction

After reviewing the exported figures:
- identify scans whose orientation appears inconsistent
- **copy** those scans into a dedicated folder
- keep the original dataset unchanged
- use the dedicated folder as the input for Step 3

This keeps manual editing focused, safer, and easier to document.

### Step 3. Interactive manual reorientation

The third step loads one problematic scan at a time and displays it next to the template reference.

#### Editable paths at the top of the cell
```python
IN_DIR = Path("/path/to/problematic/scans/to/reorient")
OUT_DIR = Path("/path/to/output/reoriented_scans")
CSV_PATH = Path("/path/to/output/scan_reorientation_info.csv")
TEMPLATE_PATH = Path("/path/to/reference/template.nii.gz")
```

#### Keyboard controls
- `a`: flip LR + AP
- `d`: flip LR + SI
- `s`: save corrected scan and continue
- `e`: stop and resume later
- `help` or `?`: show help text

#### What gets saved
Corrected scans are written using the **template affine and header**, which helps standardize orientation metadata across outputs.

#### What gets tracked
A CSV file records:
- `file name`
- `subject ID`
- `reoriented`
- `AP_opposite`
- `VD_opposite`

This allows:
- resuming after interruption
- tracking which scans were already processed
- recording whether each flip mode was toggled

## Example usage

### 1. Run batch QC export
Open the notebook and update the Step 1 paths:

```python
SCAN_DIR = Path("/data/my_scans")
OUTPUT_FIG_DIR = Path("/data/qc_figures")
```

Run the Step 1 cell. The notebook will:
- search the input folder for NIfTI files
- create paged contact sheets
- save and display each page

### 2. Review QC pages
Look through the saved PNG or PDF files and identify scans that appear mis-oriented.

### 3. Copy problematic scans
Copy those scans to a separate folder, for example:

```text
/data/problematic_scans_for_reorientation
```

### 4. Run interactive reorientation
Update the Step 3 paths:

```python
IN_DIR = Path("/data/problematic_scans_for_reorientation")
OUT_DIR = Path("/data/reoriented_scans")
CSV_PATH = Path("/data/reorientation_log.csv")
TEMPLATE_PATH = Path("/data/template.nii.gz")
```

Then run the interactive cell and correct scans one at a time.

## Design choices

This workflow intentionally uses a **semi-automated** design rather than a fully automated reorientation rule because:
- some orientation issues are easier to identify visually than algorithmically
- manual review is often needed for heterogeneous legacy datasets
- only a subset of files may require intervention
- a notebook-based workflow is convenient for collaborative review and ad hoc correction

## Author

**Zongyu Li**

Guo Lab

Biomedical Engineering Department

Columbia University