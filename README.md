## AHPd CLI — Data-Driven Multi-Criteria Decision System

The **AHPd Command Line Interface (CLI)** provides a high-performance, platform-agnostic way to integrate the **AHPd** (Analytic Hierarchy Process Data-Driven) decision engine into automated pipelines, scripts, and real-time data processing workflows.

Built on an optimized core, the CLI ensures **100% objective, consistent, and auditable** multi-criteria decision-making based purely on statistical data analysis.

## 🚀 Getting Started

The AHPd CLI executable is named **`ahpd`** (Linux) or **`ahpd.exe`** (Windows).

### Show Help

To see the list of available commands and options, use the help flag:

```bash
ahpd --help
# or
ahpd.exe --help
```

### Basic Execution (Examples)

The CLI supports three primary methods for data input: loading from a **file**, receiving via **pipe (standard input)**, or providing data **inline**. In all cases, the input must be a valid AHPd **JSON structure**.

| Input Method | Linux Example | Windows Example |
| :--- | :--- | :--- |
| **File** | `ahpd json -f phones.json` | `ahpd.exe json -f phones.json` |
| **Pipe (Stdin)** | `cat phones.json \| ahpd json` | `cat phones.json \| ahpd.exe json` |
| **Inline Data** | `ahpd inline -i '{"data":{...}}'` | `ahpd.exe inline -i '{"data":{...}}'` |

## ⚙️ Usage Details and Options

The CLI utilizes subcommands (`json` or `inline`) to specify the data ingestion method, followed by various options to control input, output, and execution mode.

### Data Input Methods

| Command | Description | Input Option |
| :--- | :--- | :--- |
| **`json`** | Used for reading JSON data from a **file** (`-f`) or from **Standard Input (pipe)**. | `-f <file_path>` |
| **`inline`** | Used for providing the complete JSON data structure directly as an **inline string**. | `-i '<json_string>'` |

#### Mandatory JSON Structure

The AHPd CLI requires the input data to adhere to a strict structure containing the mandatory keys: **`data`**, **`criteria`**, and **`options`**.

```json
{
    "data": {
        "criteria": {
            "price US$": "min",
            "storage GB": "max",
            "memory GB": "max",
            "camera Mpx": "max",
            "battery mAh": "max"
        },
        "options": {
            "Phone A": [9494, 128, 6, 48, 4323],
            "Phone B": [4139, 256, 8, 50, 4500],
            "Phone C": [4429, 256, 8, 50, 4300],
            "Phone D": [1885, 128, 6, 64, 5065]
        }
    }
}
```

**Structure Rules:**

1.  **`criteria` (Object):** Defines the quantitative criteria for the decision.

      * **Keys (Criterion Name):** Must be a **string**.
      * **Values (Preference):** Must be a **string** and restricted to only two values:
          * **`"min"`:** Indicates that a lower value for this criterion is better (e.g., price, risk).
          * **`"max"`:** Indicates that a higher value for this criterion is better (e.g., performance, quality).

2.  **`options` (Object):** Contains the raw, quantitative data for each alternative being compared.

      * **Keys (Alternative Name):** Must be a **string** (e.g., "Product A," "Supplier XYZ").
      * **Values (Data Array):** Must be an **array of numbers**.
          * **Numeric Values:** All values must be **numbers** (integers or floats). Negative numbers are accepted.
          * **Positional Coherence:** The order and count of values in this array **must strictly match** the order and count of the criteria defined in the **`criteria`** object. (e.g., the first number in the array is the value for the first criterion defined, the second for the second criterion, and so on).

#### Full Inline Example

This example shows the required structure for the inline JSON data:

```bash
# Complete example with inline JSON data
ahpd inline -i '{"data":{"criteria":{"price US$":"min","storage GB":"max","memory GB":"max","camera Mpx":"max","battery mAh":"max"},"options":{"Phone A":[9494,128,6,48,4323],"Phone B":[4139,256,8,50,4500],"Phone C":[4429,256,8,50,4300],"Phone D":[1885,128,6,64,5065]}}}'
```

### Output Formatting

You have control over the level of detail (`--level`) and the destination (`--output`) of the final results.

#### Output Levels (`--level`)

This option defines which parts of the decision analysis are included in the JSON output. The default level is **`rank`**.

| Level Name | Abbreviation | Description |
| :--- | :--- | :--- |
| **`rank`** | `r` | The final percentage ranking of all alternatives (default). |
| **`contribution-global`** | `g` or `global` | The automatically calculated weight (importance) of each criterion. |
| **`contribution-detailed`** | `d` or `detailed` | The percentage contribution of each criterion to the score of each alternative (Auditability/Explainability). |

**Usage Examples:**

```bash
# Output only the final rank (default)
ahpd json -f phones.json

# Explicitly define desired output levels (Rank + Global Contribution + Detailed Contribution)
ahpd json -f phones.json --level rank contribution-global contribution-detailed

# Use combined or abbreviated forms
ahpd json -f phones.json --level r global
ahpd json -f phones.json --level rgd
```

#### Output Destination (`--output`)

| Value | Description |
| :--- | :--- |
| **`inline`** | Writes the result to **Standard Output (stdout)**. This is the **default behavior**. |
| **`<filename>`** | Writes the result to the specified file. If no extension is provided, `.json` is added. |

**Usage Examples (Production):**

```bash
# Pretty-print to standard output (useful for visual inspection)
ahpd json -f phones.json --pretty --output inline

# Output to a file named 'results.json' with only the detailed contribution data
ahpd json -f phones.json --level detailed --output results.json
```

#### Pretty-Printing (`--pretty`)

Adds formatting (indentation and line breaks) to the JSON output for human readability.

### Execution Control

| Option | Description |
| :--- | :--- |
| **`--check`** | Performs structural validation of the input JSON data only. **The AHPd calculation is skipped.** |
| **`-v`** | **Verbose Mode.** Displays progress messages and additional non-result information during execution (useful for debugging). |

**Validation Example:**

```bash
# Structural validation only via pipe
cat phones.json | ahpd json --check
```

## Output Explanation and Execution Documentation

### Understanding the AHPd Output

The AHPd output is designed to be **consistent, auditable, and highly precise**, relying on **f64 (double-precision floating-point)** numbers. This precision is essential to ensure that the mathematical integrity of the original AHP method, combined with the data-driven analysis, is maintained, especially in high-stakes decision-making.

The JSON output provides three main results, corresponding to the levels you can request:

1.  **`criteria_weights` (Global Contribution - `g`):** The **automatic weights** assigned to each criterion. The AHPd determines this importance based purely on the **statistical dispersion** of the raw data.
      * **Interpretation:** A higher weight means that criterion showed a greater difference between the alternatives, thus having a more significant impact on the final ranking. The sum of all weights equals $1.0$.
2.  **`rank` (Rank - `r`):** The final, normalized **priority vector** (ranking) of each alternative.
      * **Interpretation:** This is the core result. The values represent the overall performance score of each alternative, where the sum of all scores equals $1.0$. The alternative with the highest value is the optimal choice.
3.  **`alternatives_contribution` (Detailed Contribution - `d`):** This section provides the full **audit trail** for the decision.
      * **`by_criteria`:** Shows exactly how much each criterion contributed (in percentage) to the total score of that specific alternative. This is the **explainability** component.
      * **`total_percentage`:** **This field is NOT the final rank.** It shows the sum of all individual contributions from all criteria for each alternative, normalized to $100\%$. It is primarily used for visualization purposes and understanding the relative spread of total influence across all options.

### Documenting the Execution Command

#### Command Executed

This command processes the data from `phones.json`, requests all possible output detail levels, saves the result to a specific file, and formats the output for readability.

```bash
ahpd.exe json -f phones.json --level rdg --output results.json --pretty
```

**Output file:**

```json
{
  "contribution": {
    "alternatives_contribution": {
      "by_criteria": {
        "Phone A": {
          "battery mAh": 25.27140468399798,
          "camera Mpx": 24.073235745226988,
          "memory GB": 22.7835981160184,
          "price US$": 10.151185142297878,
          "storage GB": 17.720576312458753
        },
        "Phone B": {
          "battery mAh": 18.725026772638852,
          "camera Mpx": 17.849621957062656,
          "memory GB": 21.623542027984474,
          "price US$": 16.57434354299879,
          "storage GB": 25.227465699315214
        },
        "Phone C": {
          "battery mAh": 18.24259984226682,
          "camera Mpx": 18.198574261252137,
          "memory GB": 22.046272819345443,
          "price US$": 15.791901454565918,
          "storage GB": 25.720651622569683
        },
        "Phone D": {
          "battery mAh": 19.309583049401244,
          "camera Mpx": 20.932567729107106,
          "memory GB": 14.85838512914299,
          "price US$": 33.34294232523744,
          "storage GB": 11.556521767111215
        }
      },
      "total_percentage": {
        "Phone A": 18.810524412740325,
        "Phone B": 26.42622428319317,
        "Phone C": 25.91950921187671,
        "Phone D": 28.843742092189796
      }
    },
    "criteria_weights": {
      "battery mAh": 0.05517554629857757,
      "camera Mpx": 0.09811746760930982,
      "memory GB": 0.11601983189243667,
      "price US$": 0.4599742131173236,
      "storage GB": 0.27071294108235233
    }
  },
  "rank": {
    "Phone A": 0.14922568130663832,
    "Phone B": 0.260912125126531,
    "Phone C": 0.25370960371891654,
    "Phone D": 0.336152589847914
  }
}
```

| Parameter | Description |
| :--- | :--- |
| **`ahpd.exe`** | The AHPd CLI executable for Windows. |
| **`json`** | Subcommand indicating the input type is JSON. |
| **`-f phones.json`** | Specifies that the JSON data should be read from the file `phones.json`. |
| **`--level rdg`** | **Output Levels:** Requests the generation of the **R**ank, **D**etailed contribution, and **G**lobal criteria weights. |
| **`--output results.json`** | Specifies the output destination as the file `results.json` (instead of standard output). |
| **`--pretty`** | Formats the output JSON with indentation for human readability. |

#### Output Analysis (`results.json`)

The output is structured into two main objects: **`contribution`** and **`rank`**.

##### Criteria Weights (Global Importance)

The **`criteria_weights`** object shows which attributes were mathematically considered the most important by the AHPd engine, due to their higher statistical dispersion in the input data.

| Criterion | Weight | Interpretation |
| :--- | :--- | :--- |
| **`price US$`** | **0.459974** | **Highest Weight:** Price was the most important factor in this decision, indicating a very significant difference between the phones' prices (Phone D's low price vs. Phone A's high price). |
| **`storage GB`** | **0.270713** | Second most important factor. |
| `memory GB` | 0.116020 | |
| `camera Mpx` | 0.098117 | |
| `battery mAh` | 0.055176 | Lowest weight. |

##### Final Ranking (`rank`)

This is the definitive result, showing the optimal alternative based on all criteria and their calculated weights. The values sum to 1.0.

| Alternative | Rank Score | Final Position |
| :--- | :--- | :--- |
| **`Phone D`** | **0.336153** | **Winner** |
| `Phone B` | 0.260912 | 2nd Place |
| `Phone C` | 0.253710 | 3rd Place |
| `Phone A` | 0.149226 | 4th Place |

##### Detailed Contribution (Auditability)

The **`alternatives_contribution`** section explains *why* each alternative achieved its score.

**Focus on the Winner (`Phone D`):**

The high rank of Phone D ($0.336153$) is explained by its performance on the most important criterion, **`price US$`** (Weight: $0.459974$).

```json
"Phone D": {
  ...
  "price US$": 33.34294232523744, // 33.34% of Phone D's total performance comes from its low price
  ...
}
```

This clearly demonstrates: **Phone D won primarily because its significantly lower price gave it a massive advantage, which outweighed the importance of its weaker attributes** (like lower `storage GB` and `memory GB`), following the objective framework set by AHPd.

### Example visual (with Chart.js)

This simple example demonstrates the decision to purchase a device, comparing the features that the **user considers relevant**.

| Option  | price US$ (min) | storage GB (max) | memory GB (max) | camera Mpx (max) | battery mAh (max) |
|----------|-----------------|------------------|------------------|------------------|-------------------|
| Phone A  | 9494            | 128              | 6                | 48               | 4323              |
| Phone B  | 4139            | 256              | 8                | 50               | 4500              |
| Phone C  | 4429            | 256              | 8                | 50               | 4300              |
| Phone D  | 1885            | 128              | 6                | 64               | 5065              |

Note that the data was passed **without** any transformation, **without** normalization, **exactly as it is in the real world**.

The user only needed to indicate whether a lower "price" is better or a larger "battery" is better.

The screenshot below intuitively shows the results, allowing you to see **precisely how much each feature (criterion)** contributed to the final ranking score. **This visually validates the weights calculated by AHPd**.

![./docs/print-chart.png](./docs/print-chart.png)

## 🧾 Practical Use Cases

| Area | Application | Strategic Outcome |
| :--- | :--- | :--- |
| **Finance** | Comparing investments based on return, risk, and liquidity. | Optimized portfolio construction and risk alignment. |
| **IT & Engineering** | Selecting vendors, software architectures, or technologies. | Reduced deployment costs and increased system efficiency. |
| **Operations** | Choosing optimal equipment, routes, or maintenance strategies. | Streamlined efficiency and reduced operational overhead. |
| **Product & Marketing** | Prioritizing features, analyzing competitor products, or setting prices. | Data-driven product roadmaps and competitive advantage. |
| **HR & Procurement** | Evaluating candidate suitability or selecting raw material suppliers. | Consistent, measurable selection criteria. |

## 🧩 Integration Options

AHPd is designed to be platform-agnostic, supporting several integration paths:

| Type | Description | Link |
| :--- | :--- | :--- |
| **PHP Native Extension** | Native C/Rust implementation for maximum performance within PHP systems. | [View PHP Documentation](https://github.com/weadtech/ahpd_lib) |
| **REST API** | JSON-compatible web service for integration with any programming language or BI tool. | [Online Service](https://ahpbi.wead.tech/api-rest) |
| **CLI Application** | Command-line tool for direct use in automated pipelines and scripts. | [This Repo](./README.md) |
| **GUI Application** | Desktop application for end-user analysis and reporting. | *(Planned)* |