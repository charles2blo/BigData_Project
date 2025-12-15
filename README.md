# BigData_Project: Sailing Data Pipeline – From GPX to Insights
**Autors:** Hugo BASSAGET & Charles DE BLAUWE


## Short description
This project demonstrates an end-to-end data pipeline designed to process, transform, and visualize GPS telemetry data collected from a Garmin watch during a sailing trip.   
This pipeline allows sailors to gain insights into their performance and optimize navigation strategies.   
By leveraging Apache NiFi for automated ETL (Extract, Transform, Load) and Python for geospatial analytics, the project converts raw GPX files into structured CSV datasets. We decided to include interactive visualizations of sailing performance, speed analysis, and navigation strategies using Folium and Matplotlib.


## The chosen tool
**Apache NiFi** (Version: latest via Docker)


## Why NIFI?
We selected Apache NiFi for this project due to its specific suitability for IoT and telemetry data ingestion:
- **Visual Dataflow Management:** NiFi provides a low-code, drag-and-drop interface that makes it easy to visualize the flow of data from the watch (raw files) to the analytics layer (CSV).
- **Handling Unstructured Data:** GPX files are nested XML structures. NiFi's **ExecuteScript** processor allows for powerful, custom parsing logic (using **Groovy**) to flatten this hierarchy into a usable CSV format without complex external preprocessing.
- **Automation & Scalability:** Unlike manual conversion tools, NiFi creates an "always-on" pipeline. As soon as a new `.gpx` file is dropped into the input folder, it is automatically processed.
- **Docker Integration:** NiFi allows for a clean, portable installation that does not clutter the local OS, which is crucial for reproducibility in academic projects.


## High-Level Architecture
The following diagram illustrates the global architecture of the project and the role played by Apache NiFi in the data pipeline:

```text
[ Garmin Watch ]
       |
       v
[ Raw GPX Files ]
       |
       v
[ Apache NiFi ]
 (Ingestion & ETL)
       |
       v
[ Structured CSV ]
       |
       v
[ Python Analytics ]
(Folium, Matplotlib, Jupyter)
```

## Installation steps
### 1. Prerequisites
Before starting, ensure the following are installed:
- **Operating System:** macOS (Also works in Windows)  
- **Container Engine:** Docker Desktop  
- **Analytics Environment:** Python 3.13, Jupyter Notebook  
---
### 2. Docker Infrastructure Setup
We created a dedicated project directory `Projet-Voile` and defined the infrastructure in a [`docker-compose.yml`](./Projet-Voile/docker-compose.yml) file.  
- **Service Name:** `nifi-ece-2025`  
- **Port Mapping:** Host port `8443` → Container port `8443` (HTTPS)  
- **Authentication:** Single-user credentials configured via environment variables  
- **Volume Mapping:** Local directories mounted to the container to allow file exchange (critical for ETL)
**Command used to launch the container:**
bash
docker compose up -d
Note: Your browser may show a security warning due to a self-signed certificate; you can safely proceed.  
Command used to launch:
```
Bash
docker compose up -d
```
---
### 3. Verification
After approximately 2 minutes, the NiFi UI should be accessible at:
https://localhost:8443/nifi  
**Username:** CHARLO  
**Password:** AZERTYUIOPQS  


## A minimal working example 
Our **"Hello World"** for this Big Data tool is a fully functional NiFi dataflow that automatically converts a sailing track recorded in **GPX format** into a structured **CSV dataset**, ready for analytics.

### The NiFi Flow

The pipeline consists of three processors connected sequentially, forming a simple yet complete ETL workflow:

#### 1. GetFile – Data Ingestion
The GetFile processor continuously monitors a specified directory inside the container (`/gpx_input`) and creates a FlowFile for each detected `.gpx` file.  
Its main configuration ensures automated ingestion of new GPX files without deleting the original source files.
**Key settings:**
- `Input Directory`: [`gpx_input`](./Projet-Voile/gpx_input)
- `Keep Source File`: `true`
- `File Filter`: `.*\.gpx`
- Relationships:
  - `success` → connected to `ExecuteScript`
---
#### 2. ExecuteScript – Data Transformation
The ExecuteScript processor performs the core ETL transformation by parsing the GPX XML content using a Groovy script.  
It extracts latitude, longitude, elevation, and timestamp from each track point and converts the hierarchical XML structure into a flat CSV-compatible format.
**Key settings:**
- `Script Engine`: `Groovy`
- `Script File`: [`gpx_to_csv`](./gpx_to_csv)
- Relationships:
  - `success` → connected to `PutFile`
  - `failure` → auto-terminated
---
#### 3. PutFile – Data Output
The PutFile processor writes the transformed data to disk as CSV files in the output directory (`/gpx_output`).  
Each processed GPX file generates a corresponding CSV file, making the data immediately available for analysis.
**Key settings:**
- `Directory`: [`gpx_output`](./Projet-Voile/gpx_output)
- `Conflict Resolution Strategy`: `replace`
- `Create Missing Directories`: `true`
### Result
Once the flow is started:
- Dropping a `.gpx` file into `gpx_input/`
- Automatically produces a corresponding `.csv` file in `gpx_output/`
- No manual intervention is required
This minimal example demonstrates how Apache NiFi can be used to build an **automated, scalable, and reproducible data ingestion pipeline**.


## Proof of execution
You can find a [`Screenshot`](./Flow_NIFI_execute.png) proving the execution went well.  
This screenshot confirms that the data pipeline was successfully executed:
- **GetFile** has ingested the GPX files from the input directory, as shown by the high number of processed tasks and the total volume of data read.
- **ExecuteScript** has correctly transformed each GPX track point into CSV rows, producing 619 FlowFiles without errors.
- **PutFile** has written all transformed data to disk, confirming that the ETL process completed successfully end-to-end.
- All queues between processors are empty (`Queued 0`), indicating that the flow has fully finished processing and no data is stuck in transit.

You can see the execution logs in this container: [`nifi_data`](./Projet-Voile/nifi_data)  

As a result, the following CSV files were successfully generated in the output directory:
- [`activity_16613729286.csv`](./Projet-Voile/gpx_output/activity_16613729286.csv)
- [`activity_20110577449.csv`](./Projet-Voile/gpx_output/activity_20110577449.csv)


## How Apache NiFi Fits into a Big Data Ecosystem
Apache NiFi acts as the ingestion and routing layer in a modern Big Data or Data Lake architecture. Its primary role is to reliably collect, transform, and route data from diverse sources to downstream processing systems.
### Data Acquisition
In a real-world scenario, sailing boats (or IoT devices in general) continuously generate telemetry data such as GPS positions, speed, or sensor measurements. These data streams are often transmitted using lightweight protocols like **MQTT**, **UDP**, or **HTTP**.  
Apache NiFi can ingest these data streams in real time, acting as the *front door* of the data platform.
### ETL Offloading
NiFi is particularly well suited for handling the early stages of data processing:
- Parsing unstructured or semi-structured formats (e.g., GPX, XML, JSON)
- Cleaning and validating incoming data
- Converting raw data into structured formats such as CSV or JSON
By handling the "dirty work" of parsing XML/GPX into CSV/JSON, NiFi prepares clean data for downstream processing engines like Apache Spark (for machine learning) or Kafka (for real-time streaming).  
### Data Provenance and Governance
One of NiFi’s key strengths in a Big Data context is data provenance. NiFi automatically tracks the lineage of every FlowFile:
- Source file
- Applied transformations
- Destination  
This capability is essential for data governance, debugging, and auditability.
Overall, Apache NiFi provides a robust, transparent, and scalable foundation for building reliable Big Data pipelines.


## Challenges encountered and how we solved them
### Challenge 1: Docker Container Name Conflicts
**Issue:** Docker keeps stopped containers in cache. Trying to run (`docker-compose up`) resulted in: (`Conflict. The container name "/nifi-ece-2025" is already in use`).
**Solution:** We implemented a strict cleanup routine before launching:  
```
docker rm -f nifi-ece-2025
docker compose up -d
```
## Challenge 2: Learn Groovy programming langage for ExecuteScript
We first planned to create a python code which could not fit with the ExecuteScript processor, so with the help of Chat GPT, we managed to tranform our python code into a Groovy code. 
## Challenge 3: Visualization Color Formats
**Issue:** When visualizing speed in Python, the map would not render.
**Analysis:** The `matplotlib` library generates colors in RGBA tuples (e.g., `(0.1, 0.5, 1.0, 1.0)`), but the `folium` mapping library requires HEX strings (e.g., `#1A85FF`).
**Solution:** We added a conversion step in the Python notebook using `matplotlib.colors.to_hex()`.


## Our Setup Notes 
**Specific problem solving process regarding Data Parsing Robustness**
One specific technical hurdle we struggled with was handling "dirty" data in the GPX files. Some track points recorded by the watch lacked an elevation (`<ele>`) tag.
**Initial Failure:**  
The initial Groovy script failed with `NullPointerException` whenever it encountered a point without elevation, stopping the entire pipeline.
**The Fix:**  
We had to learn how to write "defensive" code in Groovy for NiFi. We modified the script to check for the existence of the tag before accessing it.
**Code Snippet (Before):**  
```groovy
// Caused crashing
def ele = trkpt.ele.text()
```
**Code Snippet (After):**
```groovy
// Checks size > 0. If empty, returns empty string ""
def ele = (trkpt.'ele'?.size() > 0) ? trkpt.ele.text().trim() : ""
```
This ensured that even if the GPS signal was degraded and missed metadata, the CSV row would still be generated (with a blank value), preserving the rest of the valuable data.

## How to Run the Visualization (Python)
1. **Ensure Python 3.13 is installed.**
2. **Install dependencies:**  
```bash
pip install pandas folium matplotlib jupyter
```
3. **Open the notebook:**
```bash
jupyter notebook Analyse_CSV.ipynb
```
4. **Run all cells to generate the interactive map.**
