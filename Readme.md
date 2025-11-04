# S3 Data Viewer - ECHO S3 Data Browser and Processor

An interactive Jupyter Notebook tool for browsing, downloading, and processing X-ray imaging data from ECHO S3 storage with built-in HDF5 file viewer and advanced image processing capabilities.

---

## Features

- **S3 Backend**: Secure credential management and bucket access
- **File Browser**: Interactive file selection, filtering, and batch download
- **HDF5 Viewer**: Direct S3 streaming and dataset inspection
- **Image Processing**: X-ray enhancement with flat field correction, histogram equalization, and Gaussian filtering

---

## Setup

1. **Create and activate conda environment**:
```bash
conda env create -f echo_s3_env.yml
conda activate echo_s3_env
```

2. **Configure credentials** (Required to use this notebook):
   - Credentials file is required. Modify `DEFAULT_JSON` path in Cell 1 to point to your credentials file
   - Credentials file format:
```json
{
  "access_key": "YOUR_ACCESS_KEY",
  "secret_key": "YOUR_SECRET_KEY"
}
```

---

## Usage

Open the notebook in VSCode or Jupyter.

### Workflow

**Step 0: Select Kernel**
- In VSCode, select kernel as `echo_s3_env`

---

**Step 1: Initialize Backend (Cells 1-3)**
- Run Cell 1: Initialize S3 backend and load credentials
- Run Cell 2: Load core functions
- Run Cell 3: Additional setup

---

**Step 2: HDF5 File Browser & Viewer (Cell 4)**
- Run Cell 4: Display HDF5 file browser interface
- Select bucket from dropdown menu, then press "List Objects" button
- Apply filters to browse H5 files:
  - Select directory from dropdown (optional)
  - Enter keyword to search (optional)
  - Choose sort order (by name, size, or date)
- Select files using checkboxes
- Optional: Click "Download Selected" to download files locally
- Optional: View the last frame of selected file

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
- Save Format: PNG (8-bit) / TIFF (32-bit float) / NPY (numpy array)
- Save Directory: Output path for processed images

**Actions:**
- Click "Preview Last Frame" to view 5-panel visualization with histograms at each processing step
- Once satisfied with preview, click "Batch Process & Save" to process all frames

**Processing Pipeline**: Original (uint16) → Normalized [0,1] → Flat Field Correction → Linear Stretch (1%/99%) → Gaussian Filter → Save

---

## Configuration

- **ECHO Endpoint**: `https://s3.echo.stfc.ac.uk` (modify `ECHO_ENDPOINT` in Cell 1)
- **Credentials Path**: `~/Project/S3/echo_creds.json` (modify `DEFAULT_JSON` in Cell 1)
- **Object Limit**: 5000 objects per listing (modify `MAX_LIST_OBJECTS` in Cell 2)

---

## Notes

- HDF5 files are streamed directly from S3 (no local download needed)
- Batch processing saves frames to disk without loading all frames into memory

---

## Troubleshooting

- **Credentials not found**: Check JSON file path or environment variables
- **Connection error**: Verify network access to ECHO S3 endpoint
- **HDF5 read error**: File may require hdf5plugin compression support

---

## Security

- Never commit credentials to version control
