# Azure Data Pipeline – Local → Azure VM → ADLS Gen2

## Overview
This project demonstrates an end-to-end data ingestion workflow where CSV files stored on a local machine are transferred to an Azure Virtual Machine (VM) and then ingested into Azure Data Lake Storage Gen2 (ADLS Gen2) using Azure Data Factory (ADF).

The local folder structure contains three subfolders, each holding different CSV files. The ADF pipeline is designed to automatically detect active folders using a SQL configuration table, validate their existence, and copy their contents into ADLS Gen2.

---

## Architecture Workflow
1. *Local Machine → Azure Virtual Machine*  
   The main folder containing all subfolders and CSV files is manually copied from the local computer to the Azure Virtual Machine.

2. *SQL Server Configuration Table*  
   Inside the Azure VM, SQL Server hosts a configuration table (tblconfig) that controls ingestion.  
   The table contains:
   - *src_folder* – name of the source folder on the Virtual Machine 
   - *sink_container* – target container in ADLS Gen2  
   - *sink_folder* – target folder path  
   - *is_active* – flag (1/0) indicating whether the folder should be ingested  

   The ADF script activity queries this table to dynamically identify which folders are active and should be processed.

3. *Azure Virtual Machine → Azure Data Lake Gen2 (via ADF Pipeline)*  
   The ADF pipeline performs:
   - *Script activity*: Connects to SQL Server to fetch a list of active folders.
   - *ForEach activity*: Iterates over each active folder returned by the script.
   - *GetMetadata activity*: Checks whether the folder exists on the VM path.
   - *If Condition*: Continues only if the folder exists.
   - *Copy Data activity*: Copies all CSV files from the VM folder to the target path in ADLS Gen2.

---

## SQL Configuration Table Example

tblconfig
| src_folder | sink_container | sink_folder                  | is_active |
|-----------|----------------|------------------------------|-----------|
| sample1   | global         | india/landing/sample1out     | 1         |
| sample2   | global         | india/landing/sample2out     | 1         |
| sample3   | global         | india/landing/sample3out     | 1         |
| sample4   | global         | india/landing/sample4out     | 0         |
| sample5   | global         | india/landing/sample5out     | 0         |

Only rows where *is_active = 1* are processed.

---

## Pipeline Components
### *1. Script Activity*
- Executes a SQL query on the configuration table.
- Returns only the active folders.
- Output feeds into the ForEach activity.

### *2. ForEach Activity*
Loops over each active folder returned by the script.

### *3. GetMetadata Activity*
Checks if the folder physically exists on the VM file system.

### *4. If Condition*
Ensures copy happens only for valid/existing folders.

### *5. Copy Data Activity*
Moves CSV files from the VM path (through SHIR) into ADLS Gen2.

---

## Features
- Metadata-driven design using SQL Server  
- Automated folder selection using is_active flag  
- Validates folder existence before copying  
- Supports multiple subfolders dynamically  
- Uses Self-Hosted Integration Runtime on Azure VM  
- Ensures reliable ingestion into ADLS Gen2  

---

## Purpose
This project can be used as a template for:
- Moving on-premise files to Azure  
- Metadata-driven ingestion patterns  
- Hybrid pipelines using Self-Hosted IR  
- Dynamic folder and file ingestion automation  

---

## Technologies Used
- *Azure Data Factory*  
- *Azure Virtual Machine*  
- *Self-Hosted Integration Runtime (SHIR)*  
- *SQL Server on Azure VM* (configuration-driven pipeline)
- *Azure Data Lake Storage Gen2*
