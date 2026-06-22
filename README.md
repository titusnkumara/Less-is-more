# Less Is More: Intelligent Sparse Sampling Preserves R-Peak detection and HRV Accuracy Down to 12 Hz

## ECG Database Download, Compression Benchmarking, and QRS Detection Sweep

This repository provides a complete pipeline to:

- Download three standard ECG databases (MIT‑BIH, QTDB, NSTDB) using Python.
- Convert the signals and annotations to simple text files.
- Run a MATLAB analysis that:
  - Evaluates QRS detection performance and HRV metrics across multiple target sampling frequencies using four sparse representation methods.
  - Benchmarks compression efficiency (sparse representation vs. original, with LZ4/Heatshrink/Gzip).

## 1. Prerequisites

### 1.1 Python (≥3.6)
Install the required packages:
pip install wfdb numpy requests

### 1.2 MATLAB (≥R2018b recommended)
The MATLAB script requires the following toolboxes:

- Signal Processing Toolbox (for butter, filtfilt, resample, etc.)
- Statistics and Machine Learning Toolbox (for prctile, signrank)

It also requires the BioSigKit class for Pan‑Tompkins QRS detection. Download it from:

- [BioSigKit on MATLAB Central][https://mathworks.com/matlabcentral/fileexchange/67805-biosigkit-a-toolkit-for-bio-signal-analysis]

Place the BioSigKit folder on your MATLAB path, e.g.:
addpath(genpath('path/to/BioSigKit'));

### 1.3 External Compression Tools (Optional)
The compression benchmark calls external executables. They are **not required** for the detection sweep.

- LZ4 – brew install lz4 (macOS) or use your system package manager.
- Heatshrink – brew install heatshrink (macOS) or build from source.
- Gzip – usually pre‑installed on macOS/Linux.

If these tools are missing, the MATLAB script will skip the corresponding compression steps and still run the detection evaluation.

---

## 2. Data Download (Python Scripts)

Run the three Python scripts to download the databases. They will create folders (mitdb-mat/, qtdb-mat/, nstdb-mat/) containing:

- <record>.txt – the first channel (MLII) in mV, one sample per line.
- <record>-ann.txt – QRS annotation indices (1‑based) for selected beat types.

Important: The scripts use USE_ONE_BASED_INDICES = True to align with MATLAB’s 1‑based indexing. Do not change this unless you also adapt the MATLAB script.

python downloadMITBIH.py
python downloadQT.py
python downloadNSTDB.py

---

## 3. Running the MATLAB Analysis

The main MATLAB script is "BiokitRunAllMultiDB.mlx". Before running, set the database at the top of the script:

database = 'mitdb';   % alternatives: 'qtdb', 'nstdb'

Then execute the script in MATLAB. It will:

1. Load or cache the data – look for the .txt files in the corresponding folder and save a .mat cache for faster future loading.
2. Perform the frequency sweep – for each target frequency (divisors of the processed sampling rate, e.g., 120, 80, 60, …, 16 Hz), it:
   - Builds sparse signals using four methods: Naive, Max‑hold, Min+Max (Intel), and Adaptive.
   - Optionally up‑samples to a fixed detection rate (45 Hz) when the target frequency is below 45 Hz.
   - Runs Pan‑Tompkins detection on the reconstructed signal.
   - Maps detected R‑peak positions back to the native sampling rate (250 Hz for QTDB, 360 Hz for MITDB/NSTDB).
   - Computes F1 score, sensitivity, precision, timing error, and HRV errors (ΔMeanRR, ΔSDNN, ΔRMSSD) relative to the native ground truth.
3. Produce output – summary tables in the command window, a CSV file with per‑record per‑target results, and three figures saved as PNG/PDF.
2. Run the compression benchmark "compressAndSave30Hz.mlx" to see the compression performance – compare original 16‑bit signals with sparse (30 Hz) representation, and compress both with LZ4, Heatshrink, and Gzip. Results and compressed files are saved in mit-compressed/.

---

## 4. Outputs

After a successful run, you will find:

- CSV file: per_record_results_<database>.csv – one row per record, target frequency, and method, containing:
  - F1 (%), timing error (ms), ΔMeanRR (ms), ΔSDNN (ms), ΔRMSSD (ms).
- Figures (saved as both PNG and vector PDF):
  - detection_performance_<database>.png/.pdf – F1, sensitivity, precision, and timing error vs. frequency.
  - computational_metrics_<database>.png/.pdf – speedup and processing time vs. frequency.
  - hrv_errors_<database>.png/.pdf – HRV errors vs. frequency.
- Compressed files (if compression benchmark ran): mit-compressed/ contains .bin files for original and sparse streams.

---

## 5. Customization

- To change the target frequencies, edit the divisors array in the MATLAB script.
- To adjust the tolerance for QRS matching, modify tol_ms (default 150 ms).
- To change the detection up‑sampling rate, modify Fs_pt (default 45 Hz).

---

## 6. Notes

- The MATLAB script uses Makima interpolation for up‑sampling; you can change to 'pchip' or 'spline' by editing reconMethod.
- The database‑specific handling (resampling for QTDB) is automatic – do not manually resample the text files.
- If you encounter missing file errors, verify that the Python scripts completed without errors and that the folder names match exactly.

---

## 7. Troubleshooting

| Issue | Solution |
|-------|----------|
| BioSigKit not found | Add its folder to MATLAB path or install via the File Exchange. |
| Missing .txt files | Re‑run the Python download scripts. |
| LZ4/Heatshrink not found | Install the tools or comment out the compression benchmark section. |
| Out of memory | Reduce the number of records processed by editing the records list. |

---

## 8. License

This project is for research and educational purposes. The databases are subject to their own licenses (PhysioNet). Please cite the appropriate references when using this code.


## 9. Contact

For issues with the code, please open an issue on the repository (if available) or contact the author.
