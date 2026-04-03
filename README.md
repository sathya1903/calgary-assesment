# Web Server Log Analysis — University of Calgary HTTP Dataset

## Overview
Analysis of the University of Calgary HTTP web server access log (1994–1995), one of the earliest publicly available web server log datasets. The dataset contains 724,836 HTTP requests logged in NCSA Common Log Format.

---

## Tools & Libraries
| Tool | Purpose |
|---|---|
| Python 3 | Core language |
| pandas | Data loading, cleaning, and analysis |
| re | Regex-based log line parsing |
| gzip | Decompressing the `.gz` dataset |
| datetime | Timestamp parsing and conversion |

---

## Approach

The solution is structured in three clean layers:

**1. Data Acquisition**
The dataset is downloaded programmatically and decompressed from `.gz` format — making the pipeline fully reproducible with a single notebook run.

**2. Parsing**
A `parse_line()` function processes each log line using a compiled regex pattern, extracting and type-converting all fields. Four additional features are engineered during parsing: `date`, `hour`, `method`, and `extension`.

**3. Analysis**
Each analytical question is implemented as an independent, well-documented function — making the code modular, readable, and easy to test.

---

## Challenges & Solutions

**1. File Format, Encoding & Scale**
The dataset arrived as a compressed `.gz` file encoded in **Latin-1** (not UTF-8), which is common for data from the early 1990s. Opening it without specifying `encoding='latin-1'` caused decoding errors. The file was also read **line by line** rather than all at once to handle its 700,000+ lines efficiently without memory issues.

**2. Timestamp Inconsistencies**
Some log entries included timezone offsets (e.g., `-0600`) while others did not, resulting in a mix of timezone-aware and timezone-naive datetime objects. Pandas cannot store these together as `datetime64[ns]` — it falls back to slow Python object types. This was resolved by converting all timestamps to UTC first, then stripping timezone info using `.dt.tz_localize(None)`.

**3. Missing Values & Type Conversion**
58,035 byte entries recorded a dash (`-`) instead of a number. Rather than dropping these records, they were stored as `None` using pandas **nullable integer types** (`Int16` for `http_code`, `Int64` for `bytes`) — preserving records for all non-byte analysis without upcasting to float. Low-cardinality columns (`method`, `extension`, `date`) were cast to **categorical dtype** to reduce memory usage and speed up `groupby` operations.

**4. Query Strings in Filenames**
Many filenames contained query parameters after a `?` (e.g., `/search.php?user=john`). Without stripping the query string first, file extensions would be incorrectly extracted as `php?user=john`. This was resolved by splitting on `?` and using only the path portion before extracting the extension.

**5. Performance at Scale**
With 724,836 records, all analysis operations use pandas **vectorized methods** — avoiding Python loops entirely to keep the notebook fast and efficient.

---

## Key Findings
- **724,836** total HTTP requests analyzed
- **78%** of all requests returned HTTP 200 (success)
- **index.html** was the most requested file with 140,074 hits
- **2 PM** was consistently the peak traffic hour
- **23,517** requests resulted in 404 (Not Found) errors
- Only **42** server errors (HTTP 500) — a remarkably stable server for 1994
