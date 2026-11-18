# DWG / DXF Batch RegApp & Item Purger

A fast, standalone Windows desktop utility written in Rust to recursively batch-purge unreferenced Registered Applications (`RegApps`) and unused named objects from DWG and DXF files—without requiring AutoCAD or OpenCADStudio to be open.

Built with [`acadrust`](https://github.com) and [`eframe` / `egui`](https://github.com), it mirrors your folder hierarchy, safely skips the output folder when placed inside the input directory, optionally follows and cleans attached Xrefs, converts Xref paths to relative paths, and generates a detailed CSV audit report.

---

## Features

- **Batch RegApp Cleanup:** Strips unreferenced `AppId` entries from DWG and DXF files while preserving built-in system IDs (`ACAD`, `ACAD_MLEADERVER`) and any `AppId` referenced by decoded `XData` records or raw DWG Extended Entity Data (`EED`).
- **Selective Item Purging:** Toggle individual cleanup categories from the right-hand **Select type** panel:
  - **Registered Apps (RegApps)**
  - **Blocks** (unreferenced block definitions)
  - **Xrefs** (unreferenced external reference definitions)
  - **Images** (unreferenced `IMAGEDEF` raster image definitions and dictionary entries)
  - **Layers** (preserving `0` and `DEFPOINTS`)
  - **Linetypes** (preserving `BYBLOCK`, `BYLAYER`, and `CONTINUOUS`)
  - **Text Styles** (preserving `STANDARD` and styles used by `TEXT` / `MTEXT`)
  - **Dimension Styles** (preserving `STANDARD` and styles used by `DIMENSION` entities)
- **Nested Fixpoint Purging:** Automatically runs up to 10 passes so removing a parent block or Xref also cleans up newly orphaned nested blocks, layers, linetypes, and styles.
- **Recursive Xref Processing & Relative Paths:**
  - **Also open & purge attached Xref files:** Follows attached Xref paths recursively (including external Xrefs outside the input root, saved safely under `_External_Xrefs\`) with circular-reference protection.
  - **Convert Xref paths to relative:** Rewrites attached Xref paths to clean relative paths (e.g. `.\Subfolder\Base.dwg`) before saving.
- **Safe Folder Traversal:** Recursively scans subfolders with `WalkDir` while automatically ignoring the output directory even when it is created as a subfolder of the input directory (e.g. `<Input>\CLEAN`).
- **Dry Run & CSV Audit Report:** Preview changes without writing CAD files and export `purge_report.csv` with before/after file sizes (bytes and KB), total bytes saved, per-category purge counts, and a full list of removed item names.
- **Quick-Open Actions:** Open the output folder or launch `purge_report.csv` directly from the GUI once a batch finishes.

---

## Prerequisites & Building

1. **Install Rust:** Ensure you have the Rust toolchain (`rustc` and `cargo`) installed via [rustup.rs](https://rustup.rs).
2. **Clone the Repository:**
   ```bash
   git clone https://github.com<your-username>/OCS_Plugin_Clean.git
   cd OCS_Plugin_Clean
   ```
3. **Build the Release Executable:**
   ```bash
   cargo build --release
   ```
   The standalone Windows GUI binary will be created at:
   ```text
   target\release\dwg-purge-regapps.exe
   ```

---

## Usage

1. Launch `target\release\dwg-purge-regapps.exe`.
2. Click **Browse...** next to **Input Folder** to select the root directory containing your `.dwg` and `.dxf` files (the **Output Folder** defaults automatically to `<Input Folder>\CLEAN`).
3. In the right-hand **Select type** sidebar, choose which item categories and options to enable.
4. *(Recommended)* Leave **Dry Run (preview & CSV report only)** checked for your first pass to inspect `purge_report.csv`.
5. Click **Start Batch Purge**.
6. Once finished, click **Open Output Folder** or **Open CSV Report** to inspect the results.

---

## CSV Report Columns

Every run writes `purge_report.csv` inside the output directory with the following columns:

| Column | Description |
| :--- | :--- |
| `File` | Relative path of the processed DWG/DXF file (prefixed with `[XREF]` for external Xrefs) |
| `Status` | `DRY_RUN`, `CLEANED`, or `ERROR` |
| `Size_Before_Bytes` | Original file size in bytes |
| `Size_After_Bytes` | Cleaned file size in bytes |
| `Saved_Bytes` | Total byte reduction achieved |
| `Size_Before_KB` | Original file size in kilobytes |
| `Size_After_KB` | Cleaned file size in kilobytes |
| `Total_Candidates` | Total number of items scanned across enabled categories |
| `Total_Purged` | Total number of unreferenced items removed |
| `RegApps_Purged` | Count of purged `AppId` entries |
| `Blocks_Purged` | Count of purged unreferenced blocks |
| `Xrefs_Purged` | Count of purged unreferenced Xrefs |
| `Images_Purged` | Count of purged unreferenced raster image definitions |
| `Layers_Purged` | Count of purged unreferenced layers |
| `Linetypes_Purged` | Count of purged unreferenced linetypes |
| `TextStyles_Purged` | Count of purged unreferenced text styles |
| `DimStyles_Purged` | Count of purged unreferenced dimension styles |
| `Purged_Item_Names` | Semicolon-separated list of every removed item (`Type:Name`) |
| `Error` | Error details if a file failed to read or write |

---

## Safety Notes

- **Non-Destructive Output:** Original files are never overwritten; cleaned files are written to the separate output directory.
- **Inspect Complex Drawings:** Because `acadrust` is pre-1.0, always verify complex drawings containing custom Civil 3D proxy objects or ACIS 3D solids in OpenCADStudio or your target CAD viewer after cleaning.

---

## Dependencies

- [`acadrust`](https://crates.io) — DWG/DXF reading, manipulation, and writing
- [`eframe`](https://crates.io) — Native desktop GUI framework
- [`rfd`](https://crates.io) — Native OS folder picker dialogs
- [`rayon`](https://crates.io) — Parallel multi-threaded batch processing
- [`walkdir`](https://crates.io) — Recursive directory traversal
