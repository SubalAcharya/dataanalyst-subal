# dataanalyst-subal
AWS-Based Data Analytic Platform (DAP) for the City of Vancouver

Project Description

Building a scalable Data Analytic Platform (DAP) on AWS to analyze City of Vancouver funding data, with a focus on housing and childcare allocations. This portfolio project demonstrates how cloud tools can ingest and transform open city data into insights. By leveraging AWS services, the platform efficiently processes the 2023–2026 Capital Plan and 2025 Capital Budget dataset to highlight funding trends in key areas like housing and childcare, enabling data-driven decisions for city planning.

Objective

Enable city planners and stakeholders to make data-driven decisions by providing a cloud-based analytics platform. The DAP uses AWS tools to turn raw open data into actionable insights, illustrating trends in capital funding. In particular, it helps visualize how resources are allocated to critical categories (e.g. housing and childcare) over the 2023–2026 plan, supporting strategic planning and investment decisions.

Dataset

Source: 

City of Vancouver Open Data Portal – “2023–2026 Capital Plan and 2025 Capital Budget with proposed four-year Capital Plan allocations.” This public dataset details the City’s four-year capital investment plan, including original 2023–2026 plans, updates in the 2025 budget, funding sources, and planned allocations. It links new 2025 capital budget requests to previously approved multi-year projects, reflecting Council-approved changes in December 2024. The data covers 12 major service categories (e.g. Housing, Childcare, Parks, Transportation, etc.), but our analysis concentrates on Housing and Childcare funding trends.

Size & Format: 

The dataset contains 17 columns and includes aggregated funding figures (original plan, adjustments, revised plan) and categorizations for each project or program. For example, it records the 2023–2026 Original Capital Plan amount, any changes made by 2025, and the Revised Capital Plan total for each service category. It also provides a breakdown of allocations across the years 2023, 2024, 2025, and 2026, enabling year-by-year trend analysis (housing investments peaked in 2024, while childcare funding grows steadily through 2026).

License: 

The dataset is released under the Open Government Licence – Vancouver, a permissive open-data license. This means it can be freely used, modified, and shared for any purpose with proper attribution (e.g. including the statement “Contains information licensed under the Open Government Licence – Vancouver.”). The license prohibits use of any personal or confidential information and disallows implying City endorsement. In short, the data is open and royalty-free for public use, aligning with the City’s open data policy.

Methodology

The DAP follows a structured ETL/ELT pipeline to convert raw data into meaningful insights. The overall architecture is illustrated below, showing how data flows from ingestion to analysis in AWS:
Architecture of the AWS-based DAP: Raw open data is stored in Amazon S3, cleaned with AWS Glue DataBrew, cataloged in AWS Glue Data Catalog via a crawler, and queried using Amazon Athena. Processed results and queries can be visualized or saved back to S3. This serverless pipeline is scalable and cost-efficient.

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

Tools and Technologies

The project utilized the following AWS services and tools:

•	Amazon S3: Cloud storage for raw and processed data. S3 provides secure, durable storage for the datasets (both input CSVs and output results). It was used to stage the capital plan data and store Athena query outputs.

•	AWS Glue DataBrew: Data preparation service for profiling and cleaning data without code. DataBrew was used to visually inspect the dataset, handle missing values and outliers, and apply transformations via a recipe. It greatly simplified data cleaning tasks.

•	AWS Glue Data Catalog & Crawler: A centralized metadata repository to store dataset schema. The Glue Crawler was used to automatically infer the schema of the cleaned data in S3 and register a table in the Data Catalog. This made the data queryable in Athena by defining its structure (columns, types, partitions).

•	Amazon Athena: A serverless interactive query service to analyze data in S3 using SQL. Athena was used to run SQL queries on the cleaned capital plan dataset (via the Glue Catalog) to produce summary statistics and insights (e.g. aggregating funding by year and category).

•	Amazon QuickSight: AWS business intelligence tool for creating visualizations and dashboards. QuickSight (or similar BI tools) can connect to Athena to generate charts. In this project, QuickSight or manual plotting was used to create a bar chart of funding trends for housing vs. childcare.

•	AWS CLI: The command-line interface was used for tasks like transferring data to S3 and running AWS services commands (e.g., launching crawlers). It facilitated scriptable automation of the pipeline steps.

•	AWS Pricing Calculator: An online tool used to estimate the AWS costs of the solution. It helped calculate the expected monthly and annual expenses for S3 storage, DataBrew, Glue, Athena, etc., to ensure the platform remains cost-effective.

Deliverables

The AWS-based DAP produced several deliverables demonstrating the successful implementation and insights:
Annual funding allocations for Housing vs. Childcare (2023–2026). This visualization highlights that Housing projects received the highest funding in 2024 (then stabilized), while Childcare funding rose steadily each year, nearly doubling by 2026.

•	Cleaned & Structured Dataset: A refined version of the capital plan data, free of inconsistencies. This is stored in S3 (in a structured folder hierarchy) and cataloged for easy access. It can be re-used for further analysis or combined with other city datasets.

•	SQL Query Outputs (Summaries): A set of Athena query results providing key summaries – for example, total capital plan allocations by service category, year-over-year changes, and comparisons of housing vs. childcare funding. These queries confirm trends such as the peak and stabilization of housing funds and the significant surge in childcare funding by 2026.

•	Data Visualizations: Charts and graphs were generated to communicate the findings. For instance, a combined bar chart (as shown above) compares Housing and Childcare funding each year. Such visuals help city officials quickly grasp the resource allocation trends over the 4-year period.

•	Cost Estimate Report: An estimated budget for running the DAP on AWS, broken down by service. This includes the projected storage costs, data processing costs (DataBrew, Glue, Athena query scanning), etc., for a year of operation. The costs were found to be minimal (on the order of only a few dollars per month) due to the serverless, on-demand nature of the services used.

Budget

These costs assume light usage consistent with this project (e.g. storing a few megabytes of data on S3, running DataBrew jobs for profiling/cleaning a small dataset, and executing several Athena queries per month). The platform was designed with cost-efficiency in mind – by using serverless and on-demand AWS services, the monthly cost is kept around $1–$2, making it highly affordable for the City. There were no upfront infrastructure costs, and charges scale with usage. This low operating cost (~$17/year in this scenario) demonstrates that even small municipal teams can leverage cloud analytics within limited budgets.

Overall, the AWS-Based DAP for the City of Vancouver successfully showcases how open data can be transformed into valuable insights with minimal cost and effort. It provides a template for future city analytics projects, combining open data and cloud technology to inform policy and investment decisions.
