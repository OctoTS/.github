<div align="right">
  🇬🇧 English | <a href="time_series_storage.md">🇵🇱 Polski</a>
</div>

# Time Series Storage - Research

## Introduction

Time series are data recorded over time, such as CI/CD metrics, performance, and monitoring.

The purpose of this document is to provide an overview of how to store them in the context of the OctoTS project, but not limited to it.

## 1. Plain text

#### Description
Plain text is the simplest possible way to store time series data, where each line represents a single measurement. There is no imposed strict structure; the format depends on the implementation.

#### Advantages
- Maximum simplicity.
- Versatility – every operating system and programming language can read and write text files.
- Minimal overhead – no tags, quotes, or separators; the file can be as small as necessary

#### Disadvantages
- No structural information – the file itself does not indicate what the numbers represent
- Requires custom formatting conventions
- No validation – errors can easily go undetected.

#### Sources
- https://en.wikipedia.org/wiki/Plain_text

---

## 2. Text-based file formats

### 2.1 CSV (Comma-Separated Values)

#### Description
CSV is the simplest text-based data exchange format, in which each line represents a single record, and fields are separated by commas (or another separator, such as a semicolon).

#### Example
```csv
timestamp,value1,value2
2026-03-01 14:30:00,120,22.4
2026-03-02 14:30:00,133,13.2
```

#### Advantages
- Versatility – readable by virtually any analytical tool, spreadsheets, Python scripts, R, etc.
- Readability – can be opened in a text editor to quickly check the data.
- Ease of generation and parsing.

#### Disadvantages
- Lack of efficient compression - files grow quickly, especially at high sampling rates.
- Issues with data containing separators (e.g., commas in text)
- Does not support metadata (e.g., units, series descriptions) - these must be stored separately.

#### Sources
- https://en.wikipedia.org/wiki/Comma-separated_values

---

### 2.2 JSON (JavaScript Object Notation)

#### Description
JSON is a text format for representing structured data. It is widely used in APIs and web applications.

#### Example
```json
[
  {
    "timestamp": "2026-03-01 14:30:00",
    "value1": 120,
    "value2": 22.4,
    "tags": {
      "host": "server01",
      "region": "eu"
    }
  },
  {
    "timestamp": "2026-03-02 14:30:00",
    "value1": 133,
    "value2": 13.2,
    "tags": {
      "host": "server01",
      "region": "eu"
    }
  }
]
```

#### Advantages
- Readability and versatility - a text format that is easy for humans to read, yet machine-parseable in virtually any programming language.
- Schema flexibility - new fields (e.g., additional metadata, tags) can be added without the need to migrate existing data.
- Support for nested structures - allows complex metadata to be stored directly with measurements (e.g., tags, locations, sensor information).

#### Disadvantages
- Larger file sizes than CSV - field names are repeated in every record, and there is more punctuation
- Less readable diffs in Git - due to formatting and nesting, minor changes can result in large differences in the diff, making it harder to track history in version control systems.
- More difficult to append - appending a new record requires parsing the entire file, closing the bracket, and adding a comma before the new element.

#### Sources
- https://www.json.org/json-en.html
- https://en.wikipedia.org/wiki/JSON

---

### 2.3 JSON Lines (JSONL / NDJSON)

#### Description
JSON Lines (JSONL), also known as NDJSON (Newline Delimited JSON), is a text format in which each line of the file is a separate JSON object. The format is designed for streaming processing and easy data appending (append-only). It is often used in logs, big data systems, and data pipelines.

#### Example
```json
{"timestamp": "2026-03-01 14:30:00", "value1": 120, "value2": 22.4, "tags": {"host": "server01", "region": "eu"}}
{"timestamp": "2026-03-02 14:30:00", "value1": 133, "value2": 23.2, "tags": {"host": "server01", "region": "eu"}}
```

#### Advantages
- All the advantages of JSON.
- Efficient appending - simply add a line.
- More readable diffs - changes are visible as a single line, not an entire block.

#### Disadvantages
- Still larger file size than CSV
- No file-level consistency validation - each line is interpreted independently, so schema inconsistencies between records are possible.

#### Sources
- https://jsonlines.org/

---

### 2.4 XML (eXtensible Markup Language)

#### Description
XML is a text-based data format used for storing and transmitting structured data. It allows for the definition of custom tags, enabling the creation of complex data structures.

#### Example
```xml
<timeseries>
  <point>
    <time>2026-03-01 14:30:00</time>
    <value1>120</value1>
    <value2>22.4</value2>
  </point>
  <point>
    <time>2026-03-02 14:30:00</time>
    <value1>133</value1>
    <value2>13.2</value2>
  </point>
</timeseries>
```

#### Advantages
- Readability.
- Hierarchical structure – natural representation of complex relationships (e.g., sensor → series → observation).
- Rich metadata – ability to include any descriptive information (units, locations, notes) directly within the data structure.
- Validation – ability to define a schema (DTD, XML Schema, RelaxNG) and automatically validate data.

#### Disadvantages
- High data overhead - opening and closing tags repeatedly duplicate field names, resulting in even larger file sizes than CSV or JSON
- Less readable diffs.
- More difficult to append.
- Difficulty with manual editing - manually editing large files is cumbersome, and it is easy to make syntax errors.

#### Sources
- https://en.wikipedia.org/wiki/XML

---

### 2.5 TimeSeries.JSON

#### Description
TimeSeries.io is an open-source project provided by Geosoft that defines how to store and transmit time series data. It is based on the TimeSeries.JSON format, a JSON extension designed specifically for time series data.
The file has the structure of an array of objects, each consisting of three parts:
- header - metadata describing the entire dataset.
- signals - an array of signal definitions.
- data - an array containing the actual measurement data.

#### Example
```json
[
  {
    "header": {
      "TimeSeries.JSON": "1.0",
      "name": "New York City Marathon",
      "description": "R. Chepngetich",
      "source": "Garmin Fenix 8",
      "organization": "Runners World",
      "license": "Creative Commons BY-NC",
      "location": [40.785091, -73.968285],
      "timeStart": "2024-12-02T00:00:00.000Z",
      "timeEnd": "2024-12-02T02:09:56.000Z",
      "timeStep": null
    },
    "signals": [
      {
        "name": "time",
        "description": null,
        "quantity": "time",
        "unit": "s",
        "valueType": "datetime",
        "dimensions": 1
      },
      {
        "name": "latitude",
        "description": null,
        "quantity": "angle",
        "unit": "degA",
        "valueType": "float",
        "dimensions": 1
      },
      {
        "name": "longitude",
        "description": null,
        "quantity": "angle",
        "unit": "degA",
        "valueType": "float",
        "dimensions": 1
      },
      {
        "name": "elevation",
        "description": null,
        "quantity": "length",
        "unit": "m",
        "valueType": "float",
        "dimensions": 1
      }
    ],
    "data": [
      ["2024-12-02T00:00:00.000Z", 40.60305, -74.0556192, 27],
      ["2024-12-02T00:03:07.000Z", 40.60490, -74.0499759, -2],
      ["2024-12-02T00:07:04.000Z", 40.60721, -74.0428090, -2],
      ["2024-12-02T00:11:18.000Z", 40.60965, -74.0351057,  4],
      ["2024-12-02T00:12:44.000Z", 40.61053, -74.0325308, 16],
      ["2024-12-02T00:13:51.000Z", 40.61162, -74.0308571, 11],
      ["2024-12-02T00:14:53.000Z", 40.61294, -74.0298057, 18],
      ["2024-12-02T00:16:15.000Z", 40.61452, -74.0280890, 16],
      ["2024-12-02T00:17:43.000Z", 40.61627, -74.0263295, 12],
      ["2024-12-02T00:19:22.000Z", 40.61771, -74.0289688, 24]
    ]
 }
]
```

#### Advantages
- Readability.
- Flexibility – support for univariate and multivariate signals, various data types, metadata, units, and missing values.
- Tool ecosystem – ready-made libraries in several popular languages, integration with Excel and MATLAB.

#### Disadvantages
- Disadvantages of standard JSON
- Niche nature.
- Unnecessary complexity for simple cases.

#### Sources
- https://geosoft.no/
- https://github.com/geosoft-as/timeseries-io

---

### 2.6 MRTG Log Format (Multi Router Traffic Grapher)

#### Description
MRTG is one of the oldest network traffic monitoring tools, which stores time series data in simple text files (logs) and generates HTML graphs based on them. It was designed to save space by using a mechanism that gradually reduces the resolution of historical data. The file has a fixed structure: timestamp, incoming traffic, outgoing traffic.

#### Example
```
1710681600 125000 98000
1710678000 118000 92000
1710674400 130000 100000
1710670800 110000 87000
1710667200 105000 86000
1710663600 99000 80000
```

#### Advantages
- Very simple, text-based format.
- Automatic downsampling of older data, which saves space.

#### Disadvantages
- Specific only to counter-type metrics.
- Lack of flexibility—fixed number of columns.
- Format difficult to integrate with modern tools.

#### Sources
- https://oss.oetiker.ch/mrtg/
- https://en.wikipedia.org/wiki/Multi_Router_Traffic_Grapher

---

### 2.7 .ts (sktime)

#### Description
The .ts format was created specifically for the sktime library—a Python machine learning tool for time series data.
A .ts file consists of a header and a data section. The header contains the following metadata: the dataset name, whether the data contains explicit timestamps, whether the data is univariate, and whether the data contains class labels (with a list of values if true).

#### Example
```
@problemName MultivariateExample
@timeStamps false
@univariate false
@classLabel true 0 1
@data
2,3,2,4:4,3,2,2:0
13,12,32,12:22,23,12,32:1
4,?,5,4:3,2,3,2:0
```

#### Advantages
- Supports complex cases (multivariate, varying lengths).
- Integration with the Python ecosystem—direct loading into sktime structures and further processing

#### Disadvantages
- Niche—used mainly in the sktime environment and related research projects. Outside this context, it is rarely encountered.
- Large file sizes.
- More difficult to append.

#### Sources
- https://www.sktime.net/en/v0.24.2/examples/data/loading_data.html#Representing-data-with-.ts-files

---

### 2.8 TSD/DAT (Innovyze Time Series Data Format)

#### Description
The TSD/DAT format was developed by Innovyze to store large amounts of time series data from various sources, particularly in applications related to water management and water supply network monitoring.
It consists of two file types:
- TSD file - contains metadata and definitions of measurement points
- DAT files - contain the actual time-series data for a given day (file name in the format YYYY-MM-DD.dat)

#### Example
TSD:
```
[TSD_VERSION=3.0]
[SYSTEM_TYPE=Radcom logger]
FO120716,FO12 STATION FLOW,Flow,m3/h,USED,0,500
FO120717,FO12 BOREHOLE FLOW,Flow,m3/h,USED,0,500
```

DAT:
```
_00:00
FO120716,238.0952, 1
FO120717,102.3199, 1
_00:21
FO120716,236.0195, 1
FO120717,102.3199, 2
FO120718,3.2451, 1
FO120719,2.2073, 1
```

#### Advantages
- Separation of metadata and data - changing the channel description, units, or validity thresholds does not require modifying large files containing measurements.
- Efficient storage - data is grouped into daily files, which facilitates archiving, searching, and the eventual deletion of old data.
- Space savings - the channel key (8 characters) is short compared to repeating full names in every record (as in JSON).
- Data validation capabilities (min/max, flags)

#### Disadvantages
- Format tailored specifically to the company’s needs; not suitable for broader use
- Two interrelated structures—requires maintaining consistency between TSD and DAT files (keys must match), managing the directory structure

#### Sources
- https://help2.innovyze.com/infoworkswspro/Content/HTML/WS/r_time_series_data_file_format.htm
- https://help2.innovyze.com/infoworkswspro/Content/HTML/WS/a_time_series_data_files.htm

---

## 3. Binary formats
Binary formats are designed for high performance and space efficiency. They are the best choice for large-scale production and analytical systems, but are not suitable for Git-based projects where readability and the ability to track changes are critical.

---

### 3.1 HDF5 (Hierarchical Data Format version 5)

#### Description
An advanced, hierarchical binary format capable of storing massive, complex scientific datasets, including multidimensional arrays with rich metadata.

#### Features
- Very high performance.
- Rich metadata and a self-describing structure.
- Widely used in the natural sciences, engineering, and industry.
- Requires specialized libraries.

#### Sources
- https://www.hdfgroup.org/solutions/hdf5/
- https://en.wikipedia.org/wiki/Hierarchical_Data_Format

---

### 3.2 Apache Parquet

#### Description
An open-source, columnar, binary file format popular in the big data ecosystem (Apache Spark, Hadoop) for efficient data storage, including time series, with an emphasis on compression and fast queries.

#### Features
- Very good compression (various methods, e.g., Snappy, Gzip).
- Columnar storage—reduces the amount of data read during aggregate queries.
- Support for complex data types and nested structures.

#### Sources
- https://parquet.apache.org/
- https://github.com/apache/parquet-format

---

### 3.3 Feather

#### Description
A lightweight, columnar binary format designed for maximum read and write performance and easy data exchange between different data analysis languages, particularly Python and R.

#### Features
- Maximum I/O speed.
- Ideal for temporary storage and fast data exchange.
- No support for large analytical systems.

#### Sources
- https://arrow.apache.org/docs/python/feather.html
- https://github.com/wesm/feather

---

### 3.4 ORC (Optimized Row Columnar)

#### Description
An open-source columnar file format, created specifically for the Hadoop ecosystem as an optimization for Apache Hive.

#### Features
- Highest compression efficiency.
- High query performance.
- Optimized for Hive and the Hadoop ecosystem.

#### Sources
- https://orc.apache.org/
- https://en.wikipedia.org/wiki/Apache_ORC

---

### 3.5 NetCDF (Network Common Data Format)

#### Description
A scientific data format, particularly popular in meteorology, oceanography, and climatology for storing multidimensional time- and space-oriented data.

#### Features
- Support for multidimensional data (e.g., time × space).
- Built-in metadata (variable descriptions, units).
- Highly efficient for large datasets.
- A rich ecosystem of tools (NCO, CDO, xarray) and integration with programming languages.

#### Sources
- https://www.unidata.ucar.edu/software/netcdf/
- https://en.wikipedia.org/wiki/NetCDF

---

### 3.6 TsFile (Time Series File)

#### Description
A format designed and developed by Apache IoTDB, created to meet the specific requirements of the Industrial Internet of Things (IIoT).

#### Features
- Optimized for time-series data.
- Very high read and write performance.
- Support for large volumes of sensor data.
- Good compression to reduce storage costs.

#### Sources
- https://tsfile.apache.org/
- https://github.com/apache/tsfile

---

### 3.7 TeaFile

#### Description
A flat file format for storing large time-series datasets, designed with simplicity and maximum speed in mind.

#### Features
- Very fast data access.
- Simple APIs available for C#, C++, Python, and R.
- Self-describing header storing the data schema.

#### Sources
- https://github.com/discretelogics/TeaFiles.Net-Time-Series-Storage-in-Files

---

## 4. Time Series Databases

Time series databases (TSDBs) are optimized for storing time-ordered data, enabling fast queries, aggregations, and scaling with large data volumes.

---

### 4.1 InfluxDB

#### Description
A popular open-source TSDB, optimized for metrics and logs from IoT and system monitoring.

#### Features
- Columnar architecture based on Arrow/Parquet.
- Support for queries in InfluxQL, a language similar to SQL.
- Data compression and historical retention.

#### Sources
- https://www.influxdata.com/
- https://docs.influxdata.com/influxdb/

---

### 4.2 TimescaleDB

#### Description
A PostgreSQL extension for time-series data that combines a relational structure with the performance of TSDB.

#### Features
- Full compatibility with PostgreSQL and standard SQL.
- Advanced time-series features: hypertables, continuous aggregates, columnar compression.
- Hybrid storage engine (Hypercore) combining rows and columns.
- Performance limitations resulting from the PostgreSQL architecture.

#### Sources
- https://www.timescale.com/
- https://docs.timescale.com/

---

### 4.3 OpenTSDB

#### Description
A distributed database on Hadoop/HBase for storing large amounts of real-time metrics.

#### Features
- Scalability thanks to HBase-based architecture.
- Requires maintaining an HBase cluster – high operational complexity.
- Proprietary, non-SQL query language.
- Older generation of TSDB, less commonly chosen for new projects.

#### Sources
- http://opentsdb.net/
- https://github.com/OpenTSDB/opentsdb

---

### 4.4 Prometheus

#### Description
An open-source TSDB, popular for monitoring applications and systems, particularly in Kubernetes environments.

#### Features
- Pull model—the server automatically fetches metrics from exporters.
- Built-in PromQL query language.
- Tight integration with Kubernetes and the Cloud Native ecosystem.
- Ideal for monitoring, but not for long-term archiving.

#### Sources
- https://prometheus.io/
- https://prometheus.io/docs/introduction/overview/

---

### 4.5 VictoriaMetrics

#### Description
A lightweight, high-performance TSDB designed as an efficient monitoring solution that can serve both as long-term storage for Prometheus and as a standalone replacement for it.

#### Features
- Extremely fast writes and reads.
- Vertical and horizontal scalability.
- Broad protocol compatibility.
- Low memory consumption.

#### Sources
- https://victoriametrics.com/
- https://github.com/VictoriaMetrics/VictoriaMetrics

---

### 4.6 Kdb+/q

#### Description
A commercial database for high-frequency data analysis, popular in finance and trading.

#### Features
- Extreme performance for large data volumes.
- Proprietary query language q.
- Built-in aggregation and statistical analysis functions.
- Commercial license; niche in scientific applications.

#### Sources
- https://kx.com/
- https://en.wikipedia.org/wiki/Kdb%2B