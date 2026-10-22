# Amazon S3 Tables: The Future of AWS Lakehouses

- Date: April 7th, 2026
- Category: Technical
- Authors: Carolina Piro; Uriel Muñoz
- URL: https://www.marvik.ai/blog/amazon-s3-tables-the-future-of-aws-lakehouses

---

Recently introduced by AWS, S3 Tables provides fully managed Apache Iceberg tables directly within S3.

In this post, we will explain briefly what this means and its importance, as well as explore some of its main features and limitations. Then, we will offer some useful guidelines to analyse if this tool is appropriate for your Data Stack, and how to interact with the data saved in it.

And, after reading this, you can continue with the following post in the series: Amazon S3 Tables: Quick setup example (a more practical one), to start interacting with S3 Tables in your own AWS account.

## Introducing Amazon S3 Tables

Amazon S3 Tables provide S3 storage for tabular data, represented in columns and rows, like in an SQL database table. This data is stored in a new bucket type: a table bucket , where tables are subresources.

Table buckets support storing tables in the Apache Iceberg format, enabling the use of  standard SQL to query its tables with any of the engines that support Iceberg, such as Amazon Athena, Amazon Redshift, and Apache Spark.

Before we dive into S3 Tables themselves, let’s review some key concepts related to them: Apache Iceberg and Data Lakehouses.

#### What is Apache Iceberg?

Apache Iceberg is an open source table format for big analytic datasets. It adds an abstraction layer to Object Storage , such as S3, allowing the data to be reliably and performantly queryable by many different engines.

This abstraction layer enables features in a Data Lake that are typical of Data Warehouses and Relational Databases, which is often referred to as a Data Lakehouse :

- Data Lakes enable users to store large volumes of data at a low cost, while providing scalability and flexibility. However, they lack some crucial capabilities to manage this data, often becoming data swamps . They difficult Data Governance and Data Quality enforcement, and they are not optimized for query performance.

- Data Lakehouses add the abstraction layer in order to provide, among other features, ACID transaction support and schema evolution, while still supporting storage on cost-effective systems such as Amazon S3.

While there are some proprietary competitors, such as Databricks’ Delta Lake, Iceberg has become the standard for Data Lakehouse table formats . It offers true interoperability, being vendor-agnostic and widely adopted by leading platforms like Snowflake, Databricks and Dremio.

In addition to allowing the use of SQL statements on the underlying data and schema evolution, Apache Iceberg supports data partitioning and it manages the versioning of the data, allowing time-travel and rollbacks to previous states. It is also designed for scale, well-suited for managing tables that are over hundreds of gigabytes.

#### Data Lakehouse implementation options

When designing and implementing a Data Lakehouse, traditionally there were two main options:

- Self-Managed Data Lakehouses : Which include a Data Storage layer, often on standard S3 or Google Cloud Storage, and an open table format, usually Apache Iceberg. This option offers flexibility and low storage costs. However, it requires a high degree of operational maturity to manage the catalog, optimize file sizes, and handle concurrent writes.

- External Proprietary Platforms : They abstract the maintenance and offer incredible performance, but they often come with licensing fees, high compute costs and a degree of vendor lock-in. Some examples of these platforms are Databricks and Snowflake.

## S3 Tables Main Features

What sets these new Amazon S3 buckets apart is not just that they support tabular data, but that they actively manage it. Traditionally, maintaining a Data Lake meant carefully managing metadata and file sizes, and S3 Tables change that dynamic.

That management is a key factor in modern lakehouses, as their most important bottleneck is the degradation of data layout, often referred to as the small files problem . When streaming continuous data or running frequent micro-batch ingestion pipelines, thousands of fragmented files are written per hour. This hyper-fragmentation creates a cascading failure across the architecture: it inflates the size of the Iceberg manifest lists, overwhelms the metadata catalog, and imposes massive file-open overhead on the query engine. This is exactly why AWS introduced S3 Tables, integrating automated table maintenance natively at the storage layer.

So, essentially, Amazon S3 Tables offers a sweet spot between implementing Self-Managed Iceberg and leveraging External Platforms , which use proprietary formats and addons.

The following are some of its main features:

- Built-in support for Apache Iceberg The tables in these buckets are natively stored in Iceberg format, eliminating the need to manually add the abstraction layer on top of plain files in S3. As we mentioned previously, Iceberg brings a variety of features to optimize query performance and it is the standard open table format, consumable by the majority of providers.

- Managed table maintenance and optimization S3 continuously performs the following maintenance operations automatically: Compaction : combination of small files into larger ones to reduce metadata overhead and render query scan more efficient. Snapshot management : expire old snapshots based on configurable retention periods, making the corresponding data files as noncurrent and cleaning them up. Unreferenced file removal : delete data and metadata files no longer referenced by any snapshot. Record expiration : removal of expired records based on a timestamp column, available with S3 Tables Intelligent-Tiering. These operations increase table performance (keeping the metadata lean so query engines can easily consume it), while reducing storage cost. The automation  streamlines the operation of data lakes at scale by reducing the need for manual management and optimization.

- Compaction : combination of small files into larger ones to reduce metadata overhead and render query scan more efficient.

- Snapshot management : expire old snapshots based on configurable retention periods, making the corresponding data files as noncurrent and cleaning them up.

- Unreferenced file removal : delete data and metadata files no longer referenced by any snapshot.

- Record expiration : removal of expired records based on a timestamp column, available with S3 Tables Intelligent-Tiering.

- Storage-Compute decoupling S3 Tables exposes an Iceberg REST Catalog endpoint, which is the emerging standard for Iceberg catalog interoperability. Every query engine that supports Iceberg can directly read and write to your tables. This allows to separate the storage layer from compute engines entirely.

- High-Throughput S3 Tables is optimized for the high-concurrency demands of modern analytics, providing higher transactions per second (TPS) and better query throughput compared to self-managed tables in S3 general purpose buckets. It also offers the same durability, availability, and scalability as regular S3 buckets. ‍

- Access management and security Access and permissions to table buckets, table namespaces and individual tables can be managed with AWS Identity and Access Management (IAM), facilitating Data Governance, and integrating it to the existing Security Management system for other AWS cloud resources.

## S3 Tables Limitations

While Amazon S3 Tables provides a variety of strong features, it also comes with trade-offs in control, cost, and flexibility, compared to self-managed Iceberg Catalogs or proprietary Lakehouse Platforms (like Databricks or Snowflake). The following are some of its main limitations:

- Regional availability is not guaranteed Not all AWS Regions support this service yet, you should check the current availability for your target regions.

- Limited Storage Class flexibility One of S3's greatest strengths is its variety of storage tiers, but S3 Tables are more restrictive, supporting only S3 Standard and Intelligent-Tiering. There is no support for cold tiers individually.

- Single file format In table buckets, Iceberg is the only format enabled, you cannot store raw logs, CSVs, or images in them.

- Less access methods supported Unlike standard S3, you cannot generate a presigned URL to give a third-party temporary access to a specific data file. Access must go through the Iceberg REST Catalog. Also, table buckets do not support public access policies , all data must be accessed via authenticated AWS APIs or integrated engines. ‍

- Case sensitivity enforcement To ensure compatibility with all AWS Analytics services, table names and definitions must use all lowercase letters.

## Uses cases where S3 Tables shine

We have described Data Lakehouses, Iceberg and S3 Tables characteristics but, in practice: How to identify if a specific Data Solution will benefit from the implementation of a Data Lakehouse, and more specifically using S3 Tables? Here are some guidelines to help you answer these questions.

#### When do you need a Data Lakehouse?

You need an abstraction layer when your Data Lake starts feeling like a data swamp : you encounter corrupted data because of partial writes, and there is redundant or unvalidated data in your files. Also, if you need to manage files row by row instead of as a complete unit and your schemas are often evolving, an abstraction layer with a Table Format becomes essential.

A Data Lakehouse is ideal to implement the Bronze or Raw Data storage in your Data Platform, as it simplifies data governance and reduces data duplication and infrastructure costs. It enables organizations to perform high-performance SQL analytics alongside machine learning and real-time streaming.

#### When is S3 Tables the right choice?

Having identified the need of a Data Lakehouse, what makes S3 Tables specially suitable? Here are a few characteristics that can make a specific case a good candidate for S3 Tables implementation:

- AWS as the main Cloud Provider If your organization’s Cloud Infrastructure is mainly built on AWS, S3 Tables is a first-class AWS resource that integrates seamlessly into your core infrastructure. You can create specific VPC Interface Endpoints, apply IAM policies, implement zero-code ingestion pipelines using tools like Amazon Data Firehose, and more.

- Regulatory Compliance and Auditing needs Industries like Fintech or Healthcare often require point-in-time reporting or the ability to time travel and see the state of certain tables on a specific date. By leveraging Iceberg’s native versions management within a managed S3 environment, you get built-in versioning and auditing. You can query historical data states without complex manual archiving processes.

- High-Velocity Streaming Ingestion S3 Tables delivers measurably better performance for analytics and streaming workloads. It supports up to 10x higher transactions per second than a self-managed Iceberg Lakehouse on standard S3 buckets, which is critical for workloads with frequent concurrent writes from multiple pipelines. As query performance depends heavily on table maintenance, businesses with high-velocity data ingestion pipelines (such as streaming or frequent ETL updates) will benefit greatly from S3 Tables' self-optimization.

- It supports up to 10x higher transactions per second than a self-managed Iceberg Lakehouse on standard S3 buckets, which is critical for workloads with frequent concurrent writes from multiple pipelines.

- As query performance depends heavily on table maintenance, businesses with high-velocity data ingestion pipelines (such as streaming or frequent ETL updates) will benefit greatly from S3 Tables' self-optimization.

- Multi-provider ecosystems In organizations where many different query engines and programming languages are used, the interoperability given by the Iceberg REST Catalog endpoint ensures data access to all users while maintaining a single version of the data.For example: the Data Engineering team uses Spark, the Business Intelligence team uses Athena or Snowflake, and the Data Science team uses Python, and they all connect to the same Data Lakehouse.

#### When S3 Tables may not be the best option?

There are also some situations in which self-managed Lakehouses or proprietary tools may be a best choice, for example:

- Maintenance control and flexibility needs If you need full control over file layout, storage classes, lifecycle policies, and Iceberg table properties, without the constraints managed services impose. In a self-managed setup, you can decide exactly when to run compaction, for example.

- Cost-sensitive archival workloads If you have a great amount of cold data, which is rarely accessed but must be kept for years, general purpose buckets offer more storage class options, such as Glacier Instant Retrieval or Standard-IA. The cost of keeping it in an S3 Table Bucket will be significantly higher.

- Non-Iceberg data co-location requirements If you need to store Iceberg tables alongside non-tabular data (images, logs, ML artifacts) in the same bucket, general purpose buckets are required.

## Understanding Costs

When evaluating S3 Tables as a component for your Data Platform, it is important to consider the Total Cost of Ownership (TCO) rather than just storage cost. While the storage pricing for S3 Tables is structured differently than standard S3 (and is slightly higher per GB), the real difference comes from compute and engineering time.

With S3 Tables, you pay a nominal fee for the serverless table optimization executed by AWS. However, this eliminates the need to spin up and pay for your own EMR or Glue clusters purely to run OPTIMIZE, VACUUM, or compaction jobs. This is especially important in cases when there are high frequency writes and multiple tables, as more compaction is needed and the operational cost of maintenance rises.

When you factor in the saved compute costs and the engineering hours won back by not having to build and monitor your own maintenance pipelines, S3 Tables often results in a lower overall TCO.

## Comments on query performance

Another important consideration when deciding on the infrastructure of your Data Lakehouse is the resulting query performance of its data. While this topic is complex and would require an extensive post on its own, we have gathered some important comments about it.

The first thing to know is that query performance and compute costs on Apache Iceberg depends on a combination of factors such as: the execution engine's architecture, the efficiency of the underlying file structures, the metadata catalog implementation, and the maintenance routines applied to the Lakehouse.

To select the query engine that will consume your Data Lakehouse, there are two main options:

- External Proprietary Data Platforms or Warehouses As we saw, these platforms, such as Databricks and Snowflake, can host your entire Data Lakehouse, but they also provide the technology to query it. Each engine has its primary execution and resource allocation models, as well as concurrency handling mechanisms, and optimization philosophies. Consequently, each has target workloads for which they are best suited for. They can also consume data residing on external Lakehouses, such as S3 Tables and self-managed options. But, in some cases, they cannot offer the same performance on these sources that in internal ones.

- Federated Query Engines These tools allow a fully decentralized data architecture where data remains exclusively in self-managed Lakehouses or S3 Tables, and they provide direct SQL access. The most prominent engines in this category are Amazon Athena and Trino. While both engines are designed for distributed SQL execution over object storage, their operational models are vastly different. These engines are highly cost-effective for infrequent ad-hoc exploration. For BI scenarios, when you want to build sophisticated business reports from historical data, a Data Warehouse tool, like Amazon Redshift, is a more suitable choice.

Regardless of the selected tool, the ultimate performance of the query engine heavily depends on the physical state of the data resting in the lake. As we mentioned before, modern Lakehouses tend to suffer from the small files problem . A managed solution minimizes the efforts to prevent this issue while, for organizations operating self-managed Iceberg tables on general-purpose storage, implementing a rigorous, automated maintenance routine is non-negotiable.

## How to interact with Amazon S3 Tables

In order to access the tables stored in a table bucket, you need to integrate it with analytics applications that support Apache Iceberg. There are two main ways to do so:

- Using AWS Glue Data Catalog , to integrate mainly with AWS analytics services and connect to other third party Iceberg clients.

- Leveraging the Amazon S3 Tables Iceberg REST endpoint to directly connect to open-source query engines and other services.

#### AWS Glue Data Catalog integration

You can integrate table buckets with Data Catalog and AWS analytics services using IAM access controls by default, or optionally use Lake Formation access controls.

To perform this integration, you need to enable the “Integration with AWS analytics services” for your table buckets, and Amazon S3 adds the catalog named s3tablescatalog as a Federated Catalog in AWS Glue, in the current Region. Then, all current and future table buckets, namespaces, and tables are populated to the AWS Glue Data Catalog in that Region, following this equivalence: Table Buckets are mapped as Catalogs, Namespaces as Databases, and Tables remain Tables.

- You can create Apache Iceberg tables in table buckets and access them via the following AWS analytics engines: Amazon Athena, Amazon Redshift, Amazon EMR, Amazon Data Firehose, AWS Glue ETL, Quick and Querying S3 Tables with SageMaker Unified Studio.

- And you can use third-party analytics engines that support Iceberg as well, via the AWS Glue Iceberg REST endpoint , using any Iceberg client, including Spark, PyIceberg, and more.

#### Amazon S3 Tables Iceberg REST endpoint

You can use the Amazon S3 Tables Iceberg REST endpoint to access your tables directly from any Iceberg REST compatible clients through HTTP endpoints, to create, update, or query tables in S3 table buckets.

The endpoint implements a set of standardized Iceberg REST APIs specified in the Apache Iceberg REST Catalog Open API specification. The endpoint works by translating Iceberg REST API operations into corresponding S3 Tables operations.

The following AWS analytics services and query engines can access tables this way: any Iceberg client, including Spark, PyIceberg, and more, as well as Amazon EMR and AWS Glue ETL.

To use the Amazon S3 Tables Iceberg REST endpoint with PyIceberg, specify the following application configuration properties:

```
1rest_catalog = load_catalog(
2catalog_name,
3**{
4	"type" : "rest",
5	"warehouse" : "arn:aws:s3tables:<Region>:<accountID>:bucket/<bucketname>",
6    "uri": "https://s3tables.<Region>.amazonaws.com/iceberg",
7    "rest.sigv4-enabled": "true",
8    "rest.signing-name": "s3tables",
9    "rest.signing-region": "<Region>"
10	}
11)
```

Then, you can use the PyIceberg scan function to read data from your Iceberg tables, for example. You can filter rows, select specific columns, and limit the number of returned records:

```
1table = rest_catalog.load_table(f"{database_name}.{table_name}")
2scan_df = table.scan(
3    row_filter=(
4        f"city = 'Amsterdam'"
5    ),
6    selected_fields=("city", "lat"),
7    limit=100,
8).to_pandas()
9
10print(scan_df)
```

## Conclusion

For modern Data Stacks, the implementation of Data Lakehouses is often a requisite, mainly to implement the Raw Data layer, and Apache Iceberg has become the standard format for that.

By turning Apache Iceberg into a fully managed, serverless storage layer, AWS is allowing teams to reclaim the hours spent on database administration and focus on deriving actual value from their data. If your team is struggling with Iceberg maintenance, or if you are looking to build a high-performance, cost-effective lakehouse from scratch, S3 Tables should be at the top of your evaluation list.

Looking to modernize your data stack? At Marvik.ai, we help teams architect and implement scalable, future-proof data solutions. Get in touch to see how we can help you leverage S3 Tables and Apache Iceberg for your specific use cases.

## Sources

- Working with Amazon S3 Tables and table buckets - Amazon Simple Storage Service

- Apache Iceberg

- Modern data lakes - AWS Prescriptive Guidance

- Amazon S3 Tables vs Self-Managed Apache Iceberg on S3: A Technical Deep-Dive for Startups | AWS Builder Center

- Table buckets - Amazon Simple Storage Service

- Accessing table data - Amazon Simple Storage Service

- Amazon S3 Tables integration with AWS analytics services overview

- Integrating Amazon S3 Tables with AWS analytics services

- Accessing tables using the Amazon S3 Tables Iceberg REST endpoint - Amazon Simple Storage Service

- Working with Iceberg tables by using PyIceberg - AWS Prescriptive Guidance ‍

- When should I use Athena? - Amazon Athena
