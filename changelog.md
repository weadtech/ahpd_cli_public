# Changelog

# 0.1.1 - 2025-10-24

### Changed

* **Criteria Display:** The **normalization logic for breakdown data** (`alternatives_contribution.by_criteria`) has been improved. It now applies a two-step normalization that ensures that:
1. The lowest negative value is correctly mapped to `min` (e.g., **0**).
2. The **sum of normalized values** for each alternative is always equal to the `total sum` (e.g., **100**).

This ensures a better and more consistent display of criteria performance, especially those derived from original data that include negative values.


# CHANGELOG

## 1.0.0 - 2025-10-19

This is the initial production release of the **AHPd CLI (Command Line Interface)**, providing a high-performance, data-driven, and objective multi-criteria decision system built on an optimized core.

This version enables seamless integration of the AHPd decision engine into automated pipelines, scripting environments, and real-time systems.

### ✨ Features

* **Initial Core Release:** Introduction of the AHPd CLI application for Linux (`ahpd`) and Windows (`ahpd.exe`).
* **Data-Driven Decision Engine:** Full implementation of the AHPd algorithm, providing $100\%$ objective criteria weighting based on statistical data dispersion.
* **Multiple Input Methods:** Support for loading JSON input data via three methods:
    * Reading from a **File** (`ahpd json -f <file>`).
    * Receiving via **Standard Input (Pipe)** (`cat file | ahpd json`).
    * Providing data **Inline** (`ahpd inline -i '<json>'`).
* **Structured JSON Input:** Enforces a mandatory input structure (`data`, `criteria`, `options`) and validates criteria preferences (`min` or `max`).
* **Comprehensive Output Control:** Users can define the level of output detail using the `--level` flag:
    * `r`: Final **Rank** (normalized priority vector).
    * `g`: **Global Contribution** (automatic criteria weights).
    * `d`: **Detailed Contribution** (audit trail/explainability).
* **Flexible Output Destination:** Support for printing results to **Standard Output** (`--output inline`) or saving to a **File** (`--output <filename>`).
* **Auditability:** Output data is consistently represented in **f64 (double-precision floating-point)** for high precision and full decision auditability.

### 🛠️ Improvements

* **High Performance:** Core compiled for maximum processing speed, suitable for high-volume analysis.
* **Debugging Tools:** Added a **Verbose Mode** (`-v`) for progress messages and a **Validation Mode** (`--check`) to quickly verify JSON structure without running the calculation.
* **Readability:** Added the `--pretty` flag to format JSON output with indentation.