# S3 Data Viewer - ECHO S3 Data Browser and Processor

An interactive Jupyter Notebook tool for browsing, downloading, and processing X-ray imaging data from ECHO S3 storage with built-in HDF5 file viewer and advanced image processing capabilities.

---

## Features

- **S3 Backend**: Secure credential management and bucket access
- **File Browser**: Interactive file selection, filtering, and batch download
- **Metadata Filtering**: Filter experiments using CSV metadata files with flexible experiment ID matching
- **HDF5 Viewer**: Direct S3 streaming and dataset inspection
- **Batch Frame Export**: Export all frames from multiple HDF5 files in PNG, TIFF, or NPY format
- **Image Processing**: X-ray enhancement with flat field correction, histogram equalization, and Gaussian filtering

---

## Setup

### 1. Create and activate conda environment

```bash
conda env create -f echo_s3_env.yml
conda activate echo_s3_env
```

### 2. Configure credentials (Required)

**Step 1:** Edit `echo_creds.json` and fill in your ECHO S3 credentials:
```json
{
  "access_key": "your_actual_access_key",
  "secret_key": "your_actual_secret_key"
}
```

**Step 2:** In the notebook Cell 1, update the `DEFAULT_JSON` variable (around line 37) to point to your `echo_creds.json` file location:
```python
DEFAULT_JSON = os.path.expanduser(r"C:\path\to\your\echo_creds.json")
```

**⚠️ Security Warning:**
- Never commit credentials to version control
- If sharing this repo, make sure to clear your credentials from `echo_creds.json` before committing

---

## Usage

Open the notebook in VSCode or Jupyter.

### Workflow

**Step 0: Select Kernel**
- In VSCode, select kernel as `echo_s3_env`

---

**Step 1: Initialize Backend (Cells 1-3)**
- Run Cell 1: Initialize S3 backend and load credentials
- Run Cell 2: Load core functions (includes metadata filtering support)
- Run Cell 3: Load visualization functions

---

**Step 2: HDF5 File Browser & Viewer (Cell 4)**
- Run Cell 4: Display HDF5 file browser interface
- Select bucket from dropdown menu, then press "List Objects" button
- Apply basic filters to browse H5 files:
  - Select directory from dropdown (optional)
  - Enter keyword(s) to search (supports multiple keywords separated by spaces or commas)
  - Choose sort order (by name, size, or date)

**File Actions:**
- Select files using checkboxes
- Optional: Click "Download Selected" to download files locally
- Optional: Click "Visualize Last Frame" to preview the last frame of selected H5 file
- Optional: Click "Batch Save Selected Frames" to export all frames from selected files

**Batch Save Frames:**
- Select one or more H5 files
- Choose save format:
  - **8-bit PNG**: Normalized to 0-255 (smallest file size, for visualization)
  - **16-bit Float TIFF**: Preserves intensity values with good precision (balanced size/quality)
  - **NumPy Array (.npy)**: Raw array format (for further processing)
- Specify output directory
- Dataset name field: Leave blank for auto-detection or specify (such as "narrowfov")
- Click "Batch Save Selected Frames" to process all files
- Each H5 file will create its own subdirectory with all frames saved

---

**Step 2.5: Metadata Filtering (Optional)**

If you want to filter experiments by metadata (Material, Alloy, Condition, etc.):

- **Locate metadata file**: Find `metadata_summary.csv` in the network location `\\psg-ds2422plus\d1\[experiment_folder]\`
  - Each experiment folder contains its own `metadata_summary.csv` file
- **Load metadata**: In Cell 4 UI, enter the path to `metadata_summary.csv` and click "Load" button
- **Apply filters**: 
  - Expand the "Metadata Filters" accordion to see available filter options
  - Select desired values from dropdown menus for each metadata column
  - Click "Apply Metadata Filters" button to filter objects based on selected criteria

---

**Step 3: H5 Frame Browser (Cell 5)**

**Requirements:**
- Must run Cell 4 first and select an .h5 file

- Run Cell 5: Display H5 frame browser interface
- Click "Load Selected H5 File" button to load file info
- Navigate through frames:
  - Use frame slider to scrub through frames smoothly
  - Enter frame number to jump to specific frame
  - Use First/Previous/Next/Last buttons for navigation
- View frames with histogram equalization and large visualization

---

**Step 4: Advanced Frame Processing Pipeline (Cell 6)**

**Requirements:**
- Must run Cell 4 first and select an .h5 file
- Start frame must be ≥ 20 (needed for flat field computation)

- Run Cell 6: Display image processor interface

**Processing Pipeline (MATLAB Style - Fixed Steps):**
1. Normalize to [0,1]: Converts uint16 to float32
2. Flat Field Correction: Uses mean of 20 frames before start frame
3. Linear Stretch (1%/99% Percentiles): Preserves histogram shape and contrast
4. Gaussian Smoothing: Adjustable σ parameter (default: 4.0)

**Configure Parameters:**
- Start Frame (min: 20)
- End Frame
- Step: Process every Nth frame
- Gaussian σ: Smoothing strength (default: 4.0)
- Save Format: PNG (8-bit) / TIFF (16-bit float) / NPY (numpy array)
- Save Directory: Output path for processed images

**Actions:**
- Click "Preview Last Frame" to view 5-panel visualization with histograms at each processing step
- Once satisfied with preview, click "Batch Process & Save" to process all frames

**Processing Pipeline**: Original (uint16) → Normalized [0,1] → Flat Field Correction → Linear Stretch (1%/99%) → Gaussian Filter → Save

---

## Configuration

- **ECHO Endpoint**: `https://s3.echo.stfc.ac.uk` (modify `ECHO_ENDPOINT` in Cell 1)
- **Credentials Path**: Modify `DEFAULT_JSON` in Cell 1 to point to your `echo_creds.json` file
- **Object Limit**: 5000 objects per listing (modify `MAX_LIST_OBJECTS` in Cell 2)

---

## Metadata CSV Format

The `metadata_summary.csv` file contains experiment information with the following structure:

```csv
Experiment id,Material,Alloy,Condition,...
mg39628-1_0092,Al,Al-10Si-0.5Mg,As-cast,...
mg39628-1_0093,Al,Al-10Si-0.5Mg,T6,...
```

**Key Requirements:**
- Must include an `Experiment id` column
- Column names are case-sensitive (except Comments which is excluded from filters)

---

## Notes

- HDF5 files are streamed directly from S3 (no local download needed for viewing)
- Metadata filtering uses case-insensitive matching for experiment IDs
- Keyword search supports multiple keywords (space or comma separated)
- Batch processing saves frames to disk without loading all frames into memory
- TIFF format uses 16-bit float precision, providing good balance between file size and data quality

---

## Troubleshooting

- **Credentials not found**: Check JSON file path or environment variables
- **Connection error**: Verify network access to ECHO S3 endpoint
- **HDF5 read error**: File may require hdf5plugin compression support
- **Metadata filters not working**: Ensure CSV is loaded successfully and "Apply Metadata Filters" button is clicked

---

## Security

- Never commit credentials to version control
- Clear credentials from `echo_creds.json` before sharing or committing the repo
