# Ingest Real-time Data with Azure Stream Analytics and Azure Synapse Analytics

# Overview
This lab demonstrates how to ingest and analyze real-time data using Azure Stream Analytics, Azure Event Hubs, and Azure Synapse Analytics. You'll simulate sales order data streams, ingest the data into a dedicated SQL pool in Synapse Analytics, and perform real-time aggregations into Azure Data Lake.

# Prerequisites
 - An active Azure subscription
 - Basic understanding of Azure Synapse Analytics and Event Hubs
 - Azure Cloud Shell access
 - Node.js pre-installed in Cloud Shell environment

# Scenario
This lab simulates an online retail application generating continuous sales orders. You'll:
 - Send order data to Azure Event Hubs.
 - Use Azure Stream Analytics to:
 - Ingest the stream into an Azure Synapse Analytics SQL table.
 - Aggregate the data and write it as CSV to Azure Data Lake.

# Architecture
 - Event Source: Simulated orders sent via Node.js script
 - Stream Processor: Azure Stream Analytics job
 - Destinations:
   - Dedicated SQL Pool (table: FactOrder)
   - Azure Data Lake (CSV files)

# Steps
# 1. Provision Azure Resources
  Run the setup script in Azure Cloud Shell to provision the following in a resource group:
  - Azure Synapse Workspace
  - Dedicated SQL Pool
  - Azure Event Hub Namespace
  - Azure Data Lake Storage
# 2. Send Simulated Order Data
Run the following command in Cloud Shell to send 100 simulated orders:
node ~/dp-203/Allfiles/labs/18/orderclient
# 3. Verify Table in Synapse Studio
  - Open Synapse Studio
  - Resume the SQL Pool if needed
  - Navigate to Tables > dbo.FactOrder
  - Run SELECT TOP 100 * FROM dbo.FactOrder
# 4. Create Azure Stream Analytics Job – Ingest Orders
  - Job name: ingest-orders
  - Input: Event Hub (orders)
  - Output: Azure Synapse Table (FactOrder)
  - Query:
  - 
SELECT

    EventProcessedUtcTime AS OrderDateTime,
    
    ProductID,
    
    Quantity
    
INTO

    [FactOrder]
    
FROM

    [orders]
    

  - Start the job
  - Re-run the Node.js order client script
  - Check the table for ingested data
# 5. Create Azure Stream Analytics Job – Aggregate Orders
  - Job name: aggregate-orders
  - Input: Event Hub (orders)
  - Output: Azure Data Lake Storage (datalake)
  - Output Format: CSV, append mode
  - Query:
  - 
SELECT

    DateAdd(second,-5,System.TimeStamp) AS StartTime,
    
    System.TimeStamp AS EndTime,
    
    ProductID,
    
    SUM(Quantity) AS Orders
    
INTO

    [datalake]
    
FROM

    [orders] TIMESTAMP BY EventProcessedUtcTime
    
GROUP BY ProductID, TumblingWindow(second, 5)

HAVING COUNT(*) > 1

  - Start the job
  - Re-run the order client script
  - Explore Data Lake > files container > year/month/day folders

# 6. Query Aggregated CSV Data in Synapse Studio
SELECT
    TOP 100 *
FROM
    OPENROWSET(
        BULK 'https://<your-storage>.dfs.core.windows.net/files/2023/**',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        HEADER_ROW = TRUE
    ) AS [result]

# Clean Up
To avoid additional charges, stop all running Stream Analytics jobs and delete the resource group when finished.

👤 Yazar
Sefa Öztürk
IT Trainee | Azure Data Engineer in progress
📇 LinkedIn: https://www.linkedin.com/in/sefa-ozturk1972

