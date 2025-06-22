# dataanalyst-subal
📊 AWS-Based Data Analytic Platform (DAP) for the City of Vancouver

Project Description

Building a scalable Data Analytic Platform (DAP) on AWS to analyze City of Vancouver funding data, with a focus on housing and childcare allocations. This portfolio project demonstrates how cloud tools can ingest and transform open city data into insights. By leveraging AWS services, the platform efficiently processes the 2023–2026 Capital Plan and 2025 Capital Budget dataset to highlight funding trends in key areas like housing and childcare, enabling data-driven decisions for city planning.

🎯Objective

Enable city planners and stakeholders to make data-driven decisions by providing a cloud-based analytics platform. The DAP uses AWS tools to turn raw open data into actionable insights, illustrating trends in capital funding. In particular, it helps visualize how resources are allocated to critical categories (e.g. housing and childcare) over the 2023–2026 plan, supporting strategic planning and investment decisions.

🧾 Dataset

Source: 

City of Vancouver Open Data Portal – “2023–2026 Capital Plan and 2025 Capital Budget with proposed four-year Capital Plan allocations.” This public dataset details the City’s four-year capital investment plan, including original 2023–2026 plans, updates in the 2025 budget, funding sources, and planned allocations. It links new 2025 capital budget requests to previously approved multi-year projects, reflecting Council-approved changes in December 2024. The data covers 12 major service categories (e.g. Housing, Childcare, Parks, Transportation, etc.), but our analysis concentrates on Housing and Childcare funding trends.

![Image](https://github.com/user-attachments/assets/64f14aec-d6c1-4263-a320-d26dfc30cf2d)
![Image](https://github.com/user-attachments/assets/a422e756-4100-4333-90d4-7afb83d9a6ff)

Size & Format: 

The dataset contains 17 columns and includes aggregated funding figures (original plan, adjustments, revised plan) and categorizations for each project or program. For example, it records the 2023–2026 Original Capital Plan amount, any changes made by 2025, and the Revised Capital Plan total for each service category. It also provides a breakdown of allocations across the years 2023, 2024, 2025, and 2026, enabling year-by-year trend analysis (housing investments peaked in 2024, while childcare funding grows steadily through 2026).

License: 

The dataset is released under the Open Government Licence – Vancouver, a permissive open-data license. This means it can be freely used, modified, and shared for any purpose with proper attribution (e.g. including the statement “Contains information licensed under the Open Government Licence – Vancouver.”). The license prohibits use of any personal or confidential information and disallows implying City endorsement. In short, the data is open and royalty-free for public use, aligning with the City’s open data policy.

🔍 Methodology

The DAP follows a structured ETL/ELT pipeline to convert raw data into meaningful insights. The overall architecture is illustrated below, showing how data flows from ingestion to analysis in AWS:
Architecture of the AWS-based DAP: Raw open data is stored in Amazon S3, cleaned with AWS Glue DataBrew, cataloged in AWS Glue Data Catalog via a crawler, and queried using Amazon Athena. Processed results and queries can be visualized or saved back to S3. This serverless pipeline is scalable and cost-efficient.

![Image](https://github.com/user-attachments/assets/f8076d31-e90b-4b69-b912-d0cb7f202904)

The implementation consisted of several key steps:

•	Data Ingestion: 

The raw CSV dataset was downloaded from the City’s Open Data Portal and uploaded to Amazon S3 for storage. An S3 bucket (e.g. s3://capital-plan-raw) was created to hold the data, organized by year (.../year=2025/). This secure, durable storage ensures the City’s capital plan data is centralized for analysis. Initial cataloging with AWS Glue Crawler was run on the raw data to quickly infer its schema and make the data discoverable. Example: Using the AWS CLI to upload the file:

•	Data Profiling: 

Before cleaning, the dataset was profiled using AWS Glue DataBrew to understand its structure and quality. A DataBrew project was created to examine all 17 columns across the dataset (24 records in this subset focusing on housing and childcare). Data profiling identified issues such as missing values in certain fields (e.g. some entries lacked a “Service Category 3” sub-classification) and potential outliers or inconsistencies in funding amounts. This step ensured awareness of data quality problems that could bias results, guiding the cleaning process.

•	Data Cleaning: 

Using AWS Glue DataBrew, a cleaning recipe was applied to fix the detected issues. Missing category values were imputed (filled with the most frequent valid category to maintain consistency), and obvious outliers in numeric fields (e.g. unusually high “Changes to Capital Plan” amounts) were adjusted to reasonable values. DataBrew’s no-code transformations (such as filtering nulls and standardizing formats) helped normalize the data without writing custom code. The cleaned output was then saved back to S3 (under a cleaned/ prefix in the bucket) in CSV format for further use. (In a broader pipeline, this could also be output to Parquet for efficiency.) A sample DataBrew transformation might be replacing nulls in a column: “Fill missing Service_Category_3 with ‘General’”.

•	Data Cataloging: 

After cleaning, an AWS Glue Crawler was run on the cleaned S3 data to update the Glue Data Catalog with the refined schema. The crawler (capital-plan-crawler-clean) created or updated a table (e.g. capital_plan_data) in a Glue database, registering all columns (such as Service_Category_1, Project_Program_Name, Original_Plan, Revised_Plan, etc.). This metadata catalog makes the dataset queryable by Athena using standard SQL. The cataloging step is crucial for enabling fast data discovery and conforms to governance standards by providing a central schema definition for the dataset.

•	Data Summarization (Analysis): 

Finally, Amazon Athena was used to query the cataloged data and summarize key insights. Athena, being serverless, allowed running SQL queries directly on the S3-hosted data. A results bucket was configured (e.g. s3://athena-capital-plan-results) for query outputs. The team wrote SQL queries to aggregate funding by category and year. For instance, one query aggregated the total Revised Capital Plan allocations for Housing and Childcare from 2023 through 2026. An example Athena query (simplified) to get total allocations by category:

Athena quickly scanned the data and returned the results, confirming that housing investments were highest in 2024, then leveled off, whereas childcare funding grew each year and peaked in 2026. These query results were then used for visualization and further interpretation.

🧰 Tools and Technologies

The project utilized the following AWS services and tools:

•	Amazon S3: Cloud storage for raw and processed data. S3 provides secure, durable storage for the datasets (both input CSVs and output results). It was used to stage the capital plan data and store Athena query outputs.

•	AWS Glue DataBrew: Data preparation service for profiling and cleaning data without code. DataBrew was used to visually inspect the dataset, handle missing values and outliers, and apply transformations via a recipe. It greatly simplified data cleaning tasks.

•	AWS Glue Data Catalog & Crawler: A centralized metadata repository to store dataset schema. The Glue Crawler was used to automatically infer the schema of the cleaned data in S3 and register a table in the Data Catalog. This made the data queryable in Athena by defining its structure (columns, types, partitions).

•	Amazon Athena: A serverless interactive query service to analyze data in S3 using SQL. Athena was used to run SQL queries on the cleaned capital plan dataset (via the Glue Catalog) to produce summary statistics and insights (e.g. aggregating funding by year and category).

•	Amazon QuickSight: AWS business intelligence tool for creating visualizations and dashboards. QuickSight (or similar BI tools) can connect to Athena to generate charts. In this project, QuickSight or manual plotting was used to create a bar chart of funding trends for housing vs. childcare.

•	AWS CLI: The command-line interface was used for tasks like transferring data to S3 and running AWS services commands (e.g., launching crawlers). It facilitated scriptable automation of the pipeline steps.

•	AWS Pricing Calculator: An online tool used to estimate the AWS costs of the solution. It helped calculate the expected monthly and annual expenses for S3 storage, DataBrew, Glue, Athena, etc., to ensure the platform remains cost-effective.

📦 Deliverables

The AWS-based DAP produced several deliverables demonstrating the successful implementation and insights:
Annual funding allocations for Housing vs. Childcare (2023–2026). This visualization highlights that Housing projects received the highest funding in 2024 (then stabilized), while Childcare funding rose steadily each year, nearly doubling by 2026.

![Image](https://github.com/user-attachments/assets/d746ade9-6060-4e2c-a121-32f36374dbe7)

•	Cleaned & Structured Dataset: A refined version of the capital plan data, free of inconsistencies. This is stored in S3 (in a structured folder hierarchy) and cataloged for easy access. It can be re-used for further analysis or combined with other city datasets.

•	SQL Query Outputs (Summaries): A set of Athena query results providing key summaries – for example, total capital plan allocations by service category, year-over-year changes, and comparisons of housing vs. childcare funding. These queries confirm trends such as the peak and stabilization of housing funds and the significant surge in childcare funding by 2026.

•	Data Visualizations: Charts and graphs were generated to communicate the findings. For instance, a combined bar chart (as shown above) compares Housing and Childcare funding each year. Such visuals help city officials quickly grasp the resource allocation trends over the 4-year period.

•	Cost Estimate Report: An estimated budget for running the DAP on AWS, broken down by service. This includes the projected storage costs, data processing costs (DataBrew, Glue, Athena query scanning), etc., for a year of operation. The costs were found to be minimal (on the order of only a few dollars per month) due to the serverless, on-demand nature of the services used.

Budget

These costs assume light usage consistent with this project (e.g. storing a few megabytes of data on S3, running DataBrew jobs for profiling/cleaning a small dataset, and executing several Athena queries per month). The platform was designed with cost-efficiency in mind – by using serverless and on-demand AWS services, the monthly cost is kept around $1–$2, making it highly affordable for the City. There were no upfront infrastructure costs, and charges scale with usage. This low operating cost (~$17/year in this scenario) demonstrates that even small municipal teams can leverage cloud analytics within limited budgets.

Overall, the AWS-Based DAP for the City of Vancouver successfully showcases how open data can be transformed into valuable insights with minimal cost and effort. It provides a template for future city analytics projects, combining open data and cloud technology to inform policy and investment decisions.


📋 Data Quality Control Analysis

Data Security

AWS KMS Encryption for S3

To protect data at rest, we enabled server-side encryption on the S3 bucket using AWS Key Management Service (KMS). AWS KMS manages encryption keys and integrates with S3 so that all objects are encrypted using a customer-managed KMS key "docs.aws.amazon.com". This ensures that only authorized principals with access to the KMS key can decrypt the data, enhancing confidentiality and integrity. We created a symmetric CMK (Customer Master Key) for S3 encryption. For example, using the AWS CLI, a new KMS key can be created with default settings for encryption and decryption:

![Image](https://github.com/user-attachments/assets/176b635a-2488-415a-bbd5-6d8402163597)

After creation, this KMS key was configured as the default encryption key (SSE-KMS) for the S3 bucket. All data written to the bucket is automatically encrypted with this KMS key. This approach meets compliance requirements by ensuring encrypted storage of sensitive information "docs.aws.amazon.com", and we can audit key usage (via CloudTrail) to track any encrypt/decrypt events on the data.

S3 Bucket Versioning

To preserve data states and provide an additional layer of protection, we enabled S3 Bucket Versioning on the primary data bucket. Versioning allows us to retain multiple versions of each object, so any updates or deletions do not permanently remove data "docs.aws.amazon.com". With versioning turned on, if a file is accidentally deleted or overwritten, the previous version remains available and can be restored "docs.aws.amazon.com". This feature safeguards against accidental data loss during data cleaning or analysis processes. For example, if an ETL job inadvertently modifies or deletes files, we can easily roll back to an earlier version. Enabling versioning was a straightforward but crucial step to ensure data preservation and support data recovery workflows.

![Image](https://github.com/user-attachments/assets/928c9596-adb3-4b56-9e4c-cce966bacb4f)

S3 Bucket Replication
We also implemented S3 Bucket Replication to achieve cross-bucket (and cross-region) redundancy. Using S3’s replication feature, all objects in the primary bucket are automatically and asynchronously copied to a secondary backup bucket. This secondary bucket serves as a real-time backup, improving fault tolerance and meeting disaster recovery objectives. In practice, we configured a replication rule with an IAM role that allows S3 to replicate objects to the target bucket. Once enabled, any new object uploaded to the primary bucket is replicated to the designated backup bucket (in the same or a different AWS region) without manual intervention "linkedin.com". This protects the data against regional outages or catastrophic failures; even if the primary bucket faces an issue, an up-to-date copy exists in the backup location. (Note: For existing objects created before replication was enabled, S3 Batch Replication can be used to backfill the backup bucket.)

![Image](https://github.com/user-attachments/assets/c1a19b0e-f975-4735-aebb-d4e7311e50a6)

![Image](https://github.com/user-attachments/assets/268147be-f5f3-4b5c-9889-c66649e5ceea)

Overall, these security enhancements (encryption, versioning, and replication) ensure that the City of Vancouver’s DAP data is securely stored and highly durable against loss or corruption.

Data Governance

Maintaining high data quality was critical for the Data Analytic Platform. We established several data quality rules and built them into the data pipeline to enforce Data Governance standards:

Completeness:

Ensure that key fields (such as project name or other mandatory attributes) are at least 95% non-null. In other words, allow a maximum of 5% missing values per critical column. This threshold guarantees that analyses aren’t skewed by excessive null or empty entries.

Uniqueness:

Ensure that at least 99% of records have a unique identifier (e.g. a project ID). This means duplicate IDs should be under 1%, affirming that each project is almost always represented once. Any significant duplicate entries would fail this rule, preventing double-counting or overestimation in analytics.


Freshness: 

Ensure data recency by requiring that records are no more than 1000 days (~2.7 years) old. This rule filters out stale data so that the dataset reflects current or recent information, aligning with the city’s need for up-to-date analytics.

![Image](https://github.com/user-attachments/assets/639cbe37-de4b-4488-8d0c-d13e4a3ad9c2)

![Image](https://github.com/user-attachments/assets/c0ce1d40-be5f-4c8e-8886-dadb24c368a4)

These rules were implemented in the ETL pipeline using AWS Glue. The Glue job applied validations to each incoming dataset: any record that did not meet all of the above criteria was flagged. For instance, the ETL script (or Glue Data Quality node) would count nulls in critical fields and drop records if the 5% null threshold was breached, check for duplicate IDs and isolate/remove them beyond the allowed count, and compare dates to ensure they fall within the last 1000 days. By enforcing these conditions, we ensured that only high-quality data proceeds through the pipeline. The thresholds (95% completeness, 99% uniqueness, etc.) can be tuned as needed, but they were chosen as reasonable standards for this project. This data quality evaluation process guarantees that downstream analyses and visualizations are based on reliable, clean data.

Data Pipeline Governance Features


Figure:
Data quality ETL pipeline in AWS Glue, separating records into “Passed” (curated) and “Failed” (rejected) datasets based on validation rules. To incorporate the above governance rules, we designed a robust data quality check pipeline using AWS Glue Studio (visual ETL). The pipeline workflow is as follows:

Data Ingestion (S3 to Glue:

The raw City of Vancouver dataset (stored as CSV in S3) is first ingested into an AWS Glue ETL job. A crawler or direct schema definition informs Glue about the data structure.

Data Quality Validation: 

We added an Evaluate Data Quality step (using either Glue Data Quality features or custom PySpark logic) that applies the three rules – completeness, uniqueness, freshness – to each record. This step evaluates every row against the criteria defined in the Data Governance section. Records that pass all rules are tagged as valid, while any record failing one or more rules is tagged as invalid. (For example, a record with a missing project name would be marked invalid for failing the completeness rule.)

![Image](https://github.com/user-attachments/assets/eda3b42a-ce5a-430b-a531-23dfc206ec39)

Conditional Routing (Branching): 

Utilizing Glue’s branching capabilities, the pipeline then splits into two branches based on the validation outcome. One branch collects all Passed records (those that met the quality thresholds), and the other branch collects all Failed records (those that did not). This conditional split cleanly isolates bad data from good data."aws.amazon.com"

Schema Adjustment:

In the “failed” branch, we preserved all original columns for review. In the “passed” branch, we applied a Change Schema step to drop any fields that were only needed for validation checks (if any). This produces a refined schema containing only the useful analysis columns, ensuring the curated dataset is lean and focused.

Partition Management:

We leveraged Glue’s Autobalance (or coalesce) feature on each branch to control the number of output partitions. The goal was to avoid generating too many small files. We set the output to a single partition in this case, so that each branch’s result is consolidated into one output file (this simplifies downstream consumption and ensures all “passed” records end up in one CSV, and similarly for “failed” records).

Output to S3: 

![Image](https://github.com/user-attachments/assets/e0d08eeb-c186-45be-aaea-939e68641ad1)

Finally, both branches write out the results as CSV files to S3. We configured the Glue sinks so that the Passed records are written to an S3 prefix/folder named “Passed/” and Failed records to a “Failed/” folder. Each run of the pipeline will update these locations. The passed dataset in S3 serves as the cleansed, high-quality data ready for analysis, while the failed dataset can be reviewed separately to identify data issues or to trigger remediation if needed (e.g., ask data providers to fix missing values or remove duplicates).

This ETL pipeline design introduces governance by automatically separating low-quality data from high-quality data "aws.amazon.com". It provides visibility into data quality: stakeholders can inspect the “Failed” bucket to see why records were rejected (e.g., which rule failed) and maintain trust in the “Passed” data. The use of AWS Glue’s visual workflow made it easy to implement and adjust these steps, and the entire process runs in a serverless manner.

Monitoring

To ensure the platform runs smoothly and to catch any anomalies, we incorporated robust monitoring using AWS services.

Amazon CloudWatch

![Image](https://github.com/user-attachments/assets/8fca2806-729a-40e7-a7d8-5e81cdbd30e9)

We used Amazon CloudWatch for real-time monitoring, metric visualization, and alerting on our AWS resources. CloudWatch provides system-wide observability of AWS applications by collecting metrics and logs, and it allows creation of dashboards and alarms "docs.aws.amazon.com". In our project, we set up a CloudWatch Dashboard to track key S3 metrics over time – for example, the total bucket storage size and number of objects were plotted on a line graph to observe growth. This dashboard updates automatically, giving us a quick view of how the DAP data is expanding or changing.

More importantly, we defined a CloudWatch Alarm on the S3 bucket size. The alarm watches the metric BucketSizeBytes (StandardStorage type) for our bucket on a daily interval. We chose a threshold (e.g., 100,000 bytes) that should not be exceeded under normal conditions for our demo dataset. If the average bucket size goes beyond this limit, the alarm will trigger. The alarm is configured to send an email notification (using an SNS topic subscription) to alert the team. Below is an example of how one could create such an alarm via the AWS CLI:

In the above snippet, the alarm monitors the bucket’s size (in bytes) daily and sends a notification if the size goes over 100,000 bytes. This kind of alert helps us proactively manage storage (for instance, cleaning up if a job writes an unexpected large output) and ensures we don’t unknowingly run up storage costs or run out of expected space. CloudWatch alarms can also trigger automated actions, but in our case an email notification was sufficient for awareness.

AWS CloudTrail

![Image](https://github.com/user-attachments/assets/1d043c4d-4fd9-4a2c-b8af-a81064dde442)

For auditing and governance, we enabled AWS CloudTrail on the account. CloudTrail records all API calls and events made in the AWS environment
nops.io. This means every action taken on our AWS resources (S3, Glue, KMS, etc.) is logged, including details on which user or service made the call, the time it occurred, which resources were affected, and other request parameters. CloudTrail is essential for security and compliance – it provides a comprehensive audit trail of changes and access to the data platform. If something goes wrong or if we need to investigate any anomaly (like an unexpected data deletion or an unauthorized access attempt), CloudTrail logs will show the history of actions leading up to that event. We have CloudTrail configured to deliver logs to an S3 bucket for long-term retention and also to integrate with CloudWatch (as a CloudWatch Log) for easier querying. By reviewing CloudTrail logs, we can verify that our data pipeline steps and security measures are being used as intended. In summary, CloudTrail enables governance, compliance, and risk auditing by giving a record of every action on the DAP infrastructure "nops.io". 

![Image](https://github.com/user-attachments/assets/d60c0b52-060a-48c1-8b3e-81d3f19b8b4b)

Overall, the combination of CloudWatch and CloudTrail forms a robust monitoring and auditing framework for the DAP. CloudWatch focuses on performance metrics and operational thresholds (allowing us to react to issues in real time), while CloudTrail focuses on transparency and auditability of user/actions over time. These services ensure that the City of Vancouver’s data platform not only stays performant but also remains secure and compliant through continuous oversight.
