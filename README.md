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

The pipeline consists of **three processors connected sequentially**, forming a simple yet complete ETL workflow:

#### 1. GetFile – Data Ingestion

#### 2. ExecuteScript – Data Transformation

#### 3. PutFile – Data Output

### Result
Once the flow is started:
- Dropping a `.gpx` file into `gpx_input/`
- Automatically produces a corresponding `.csv` file in `gpx_output/`
- No manual intervention is required

This minimal example demonstrates how Apache NiFi can be used to build an **automated, scalable, and reproducible data ingestion pipeline**.



## Screenshot proving execution
Ypu can find a [`Screenshot`](./Flow_NIFI_execute.png) proving the execution went well.

## Explanation of how this tool fits into a Big Data ecosystem


## Challenges encountered and how we solved them


## My Setup Notes 


