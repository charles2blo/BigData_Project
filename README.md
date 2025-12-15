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
--
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
As a result, the following CSV files were successfully generated in the output directory:
- [`activity_16613729286.csv`](./gpx_output/activity_16613729286.csv)
- [`activity_20110577449.csv`](./gpx_output/activity_20110577449.csv)


## Explanation of how this tool fits into a Big Data ecosystem


## Challenges encountered and how we solved them


## My Setup Notes 


