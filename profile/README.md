<div align="right">
  🇬🇧 English | <a href="README-pl.md">🇵🇱 Polski</a>
</div>

# Welcome to OctoTS!


## Project OctoTS: Technical Description
# Problem Statement
Traditional time-series databases (such as InfluxDB or kdb+) require dedicated infrastructure, paid licensing, and constant monitoring. For smaller projects, internal tools, or public datasets, deploying such complex systems creates a disproportionate operational overhead. Furthermore, data stored in external cloud environments often becomes a "black box" - without sophisticated audit logs, it is difficult to precisely track who modified a specific entry and when. This leads to the decoupling of metrics from the source code, hindering time-based analysis in relation to specific software versions.

# The Solution: Project OctoTS
OctoTS is a solution aimed at developing a standardized protocol and a set of tools for storing time-series data in text format directly within a Git repository. The core premise of the system is to leverage version control mechanisms as a native data store with a full change history. This allows technical metrics to be collected in the same location as the source code, ensuring total data consistency with the software development history and eliminating the need for external databases.
Architecture and Automation
The system relies on GitHub Actions CI/CD pipelines, which trigger measurement scripts periodically or upon specific events. Results are automatically appended to text files and committed to the repository as individual updates. Every data point is permanently linked to a unique change identifier (commit hash), ensuring the historical consistency and integrity of the entire dataset.

# Visualization
The solution is complemented by a data visualization layer built with React. It is hosted and served directly within the GitHub ecosystem via GitHub Pages. This module is responsible for fetching raw information from the repository files and transforming it into interactive charts and dashboards. As a result, users can analyze software parameter changes in real-time, compare data across different timeframes, and instantly detect trends or regressions - all without leaving the GitHub platform.