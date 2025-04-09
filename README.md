# Data Platform Team Lead - Technical Challenge - Proposal

## Architecture

### Overview

The proposed solution can be divided in the following stages:

1. Data Sources: Identify and document all heterogeneous data sources (BigQuery, GCS, PostgreSQL on CloudSQL, etc).
1. Ingestion Layer: Implement a data ingestion layer/engine capable of handling both real-time and batch data from various sources.
1. Processing Layer: Design a processing framework to handle ETL (Extract, Transform, Load) operations, ensuring efficient data processing.
1. Storage Layer: Utilize a centralized data lake or warehouse (like BigQuery) for harmonized data storage.
1. Query Layer: Enable federated querying capabilities across different GCP projects.

### Ingestion Layer

This layer must be able to extract data from real-time and batch data.  Real Time and Batch data can be stored in project defined buckets which can match the data source region in order to facilitate the data extraction.  After the data is extracted, all the data is copied to the same region to be able to processed it.

Depending on the type of data source several strategies can be used:

1. BigQuery Data Source: In this case the easiest way is to allow access to a service account that can be able to pull the data directly from the sources.
1. CloudSQL: BigQuery External sources can be used to extract the data directly.
1. Streaming sources: Pub/Sub can be used to stream the data from the origin, but this must be accomplished by the source data team
1. GCS: Just copy the data to predefined buckets

The real-time processing can be event-driven, but the batch ingestion needs to be orchestrated.  Cloud Composer (Apache Airflow) can be used to pull data from various sources at defined intervals.

### Processing Layer

This layer must handle ETL operations in order to transform the data into the required structures.  The weapon of choice here is Cloud Dataflow in order to handle real-time and batch processing, if the data is too massive DataProc is an alternative.

The main goal of this layer is to unify and harmonize the required data.  To reach this goal an Unified Data Model must be develop, it must standarized the schema across all the different data sources.

Also Data Quality must be added to this layer to ensure that incoming data adheres to predefined standards.

Data is cataloged using Dataplex, which registers datasets in a centralized metadata repository, providing visibility and governance over data assets.  It also allows us to maintain metadata, track data lineage and facilitate data discovery.

### Storage Layer

Stores the result of the Processing layer, allowing to find the required data in an easy way.  BigQuery is an excellent choice to handle a large volumes of data in a relational way if needed.

### Query Layer

This is the layer that the external users will be interacting with.  Copies of the data to the required zones or regions can be generated, also access can be provided views or predefined datasets.  BQ cross-project querying capabilities can be leverage to enable federated queries seamlessly.

From this point onwards, the end-user can use the data for whatever purpose he or she needs: dashboards (Looker Studio), schedule queries (Cloud Composer or BQ Schedule Queries) or even analytical pipelines (Composer or Vertex Pipelines) can be build on top of this layer.

## Schema Consistency and Versioning

### Schema Management

As it was mentioned before a unified data model must be developed to standardize schemas across various sources. Dataflow transformations will enforce this schema during ingestion.

### Versioning

Version control can be implemented using BigQuery’s table versioning features. Maintain a record of schema changes in Dataplex to track historical data structures and ensure backward compatibility.

## Cross-Region Query Performance and Cost Optimization

### Performance Optimization

Partitioning and clustering in BigQuery can be used to improve query performance. Leverage BigQuery's caching capabilities to speed up repeated queries.  Also the data stored in GCS can have a lifecycle predefined and can be deleted or moved to cheaper storage if it won't be used after a period of time (e.g 30 days).

### Cost Optimization

Monitor the entire platform costs is a must!

Monitor query costs using BigQuery's built-in cost analysis tools. Implement materialized views for frequently accessed data to reduce query execution costs and latency.

Monitor data size (datasets and buckets) is also needed in order to defined clean-up strategies.

And finally monitor data process and transformation costs is needed, usually the number and types of nodes used. A good analysis can optimize the data pipelines performance, cost and efficiency.

## Trade-offs

The major trade-off is related to RT ingestion and processing, because they are really expensive (for data producers and consumers).  It is a good idea to evaluate if it is really a necessity or micro-batch ingestion can be used to meet the desired requirements.  

To minimize repeated query costs and improve response times, caching strategies using BigQuery's result caching can be implemented.

Also materialized views for complex aggregations that are frequently queried can be created, balancing the need for real-time data against the costs of maintaining these views.

## Fallback Strategies

For data ingestion failures, Cloud Functions to trigger alerts and fallback mechanisms can be used, such as reprocessing data from a reliable backup source or using cached data for analytics.

Also a dedicated operation team must be in place to monitor, alert and fix data ingestion errors/failures that will happen.  The size and dedication of this team depends on how critical is the data for the company.

## Final remarks

It is not mention before but documentation of the entire platform is a necessary and important task.  Besides a clean and tested code, architecture diagrams, data models, and usage guidelines must be documented.

And the development of the platform must have stakeholders engagement and support.  In order to accomplish this, awareness and training sessions must be conducted.  This sessions must be directed to end users and also to the input data owners.
