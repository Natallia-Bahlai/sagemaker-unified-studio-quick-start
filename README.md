# Demystifying Data Analytics in Amazon SageMaker Unified Studio
> [!NOTE]
> - Mental Model to understand the structure of Amazon SageMaker Unified Studio and Amazon SageMaker Lakehouse in no time
> - Quick Start demo provisioned through CloudFormation resources

## Amazon SageMaker Unified Studio
![Amazon SageMaker Unified Studio](visuals/SageMakerUnifiedStudio.png)
[Amazon SageMaker Unified Studio](https://aws.amazon.com/sagemaker/unified-studio/) provides an integrated environment for domain-specific data projects. The environment is provisioned through AWS-managed blueprints powered by CloudFormation templates and organized into specialized projects ranging from SQL analytics, data exploration and processing, AI model development and training to GenAI applicarion development.

Users authenticated via IAM or SSO can work within these projects to unlock the value of data by:
- Connecting multiple data sources, including Amazon S3 data lakes, Amazon Redshift data warehouses and managed storage, and federated sources
- Unifying these sources through the Amazon SageMaker Lakehouse and registering in data catalogs
- Analyzing data using query engines like Amazon Athena or Amazon Redshift Query Editor v2, or exploring and processing data programmatically using JupyterLab Notebooks
- Governing unified and shared data access through assets exposed via a business catalog

Projects in Amazon SageMaker Unified Studio also serve as collaboration and permission boundaries, with consistent access policies using a single permission model with granular controls.

Amazon SageMaker Unified Studio makes it easy for customers to find and access data from across their organization and brings together purpose-built AWS analytics, AI/ML capabilities so customers can act on their data using the best tool for the job across all types of common data use cases, assisted by Amazon Q Developer along the way.


## Amazon SageMaker Lakehouse
![Amazon SageMaker Lakehouse](visuals/SageMakerLakehouse.png)
**Amazon SageMaker Lakehouse** is a capability that unifies data across Amazon S3 data lakes, Amazon Redshift data warehouses and Redshift managed storage (RMS catalogs), enabling you to build powerful analytics and AI/ML applications on a single copy of data. In addition to seamlessly accessing data from these sources, you can connect to operational databases and third-party data sources and query data in-place with federated query capabilities. Through AWS Glue zero-ETL replication, you can bring data from operational databases (such as Amazon Aurora, Amazon RDS for MySQL, Amazon DynamoDB), and SaaS sources (like Salesforce and SAP), and load data into Amazon Redshift data warehouse or Redshift managed storage without writing any ETL job.

Amazon SageMaker Lakehouse offers three major benefits:

- **Unified data access** - Amazon SageMaker Lakehouse uses a technical data catalog powered by AWS Glue Data Catalog as the primary interface for interaction with selected data sources.
- **Integrated access control** - Amazon SageMaker Lakehouse supports fine-grained access control to your data, regardless of the underlying storage formats or query engines used.
- **Open source compatibility** - Amazon SageMaker Lakehouse leverages open-source Apache Iceberg, enabling data interoperability across various Apache Iceberg compatible query engines and tools. Data in SageMaker Lakehouse can be accessed from Apache Iceberg–compatible engine such as Apache Spark, Athena, or Amazon EMR.

There are two ways to connect to Amazon Redshift data warehouse:
- As a federated data source, connecting to a selected native database that can be either a local database or a destination database where data is replicated through zero-ETL.
- As a compute engine to execute queries, providing access to the entire workgroup/node with both native databases and auto-mounted data catalogs such as AWSDataCatalog and Redshift Managed Storage (RMS). These catalogs are automatically discovered as external databases in Amazon Redshift data warehouses once the necessary permissions are established. This allows you to analyze lakehouse data using Redshift Query Editor v2. 

## Demo Solution Architecture
![Solution Architecture](visuals/DemoArchitecture.png)

We have demo data residing in three places:
-	Customer data in Amazon Aurora PostgreSQL
-	Sales data with the inventory data in Amazon Redshift Serverless
-	Financial data such as invoices residing in Amazon DynamoDB

For simplicity of demo provisioning, infrastructure resources such VPC, subnets, security groups will be reused and both Amazon SageMaker domain and data sources will be provisioned along each other. Connectivity to the data sources will be achieved using admin database users and federated through out-of-the-box project role.

Data access is restricted by applying fine-grained permissions using AWS Lake Formation.

## Demo Data Model
![Data Model](visuals/DemoDataModel.png)

## Zero-ETL integrations
In this post we will cover the following zero-ETL integration options:
-	Federated Query of data in Amazon Aurora and Amazon DynamoDB through Amazon Athena. This direction queries data in place
-	Zero-ETL replication from Amazon Aurora and Amazon DynamoDB based on a given interval automatically. This direction queries replicated data within Amazon Redshift
-	Zero-ETL replication from Amazon DynamoDB by running COPY command

![Zero-ETL integrations](visuals/DemoZero-ETL.png)

Here is the feature comparison of Zero-ETL options in context of Amazon SageMaker Unified Studio:

![Zero-ETL Options - Comparison](visuals/Zero-ETLComparison.png)

## Deployment

### Prerequisites

These prerequisites must be completed in AWS Management Console
1.	Register current user as Lake Formation admin. This will be required to manage and apply additional fine-grained permissions
2.	Create Amazon SageMaker Domain in AWS Mngt console
- Open AWS Mngt console and go to Amazon SageMaker
- Create a Unified Studio domain
- Select Quick Setup and select VPC with 3 subnets. If you choose to *Create VPC*, CloudFormation template provisions properly-configured VPC with necessary VPC endpoints. 
- Create IAM Identity Center user with a given email and accept invitation to activate user
- Copy Amazon SageMaker Unified Studio URL

Next steps must be completed from Amazon SageMaker Unified Studio
- Open Amazon SageMaker Unified Studio URL and login as a given user
- Create project with project profile: Data analytics and AI-ML model development
- Go to Project overview and copy Project ID and Project IAM Role ARN containing: datazone_usr_role_{ProjectID}_{EnvironmentID}

### Deployment

1.	Collect the following inputs which will be required as inputs in the CloudFormation template:
- [x] VPC ID where we provisioned SageMaker Domain
- [x] Subnet IDs
- [x] Security Group ID from Security Group with name datazone-{ProjectID}-dev
- [x] Project ID
- [x] Project IAM Role ARN
2.	Deploy provided CloudFormation Templates: [StackSMUSDataSources.yaml](cloudformation/StackSMUSDataSources.yaml) and specify parameters. If you have not created VPC as part of Domain setup, then run [StackSMUSVPCEndpoints.yaml](cloudformation/StackSMUSVPCEndpoints.yaml) to create necessary VPC Endpoints: STS, Secrets Manager, Glue, RDS Data, Redshift Data, Redshift Serverless Interface endpoints and S3 Gateway endpoint.
3.	Review Output parameters

### Post Deployment
1.	In AWS Mngt Console, go to Amazon SageMaker AI, open the provisioned notebook and run scripts in [InitDataSources.ipynb](InitDataSources.ipynb) to init Aurora PostgreSQL and DynamoDB
2.	In AWS Mngt Console, go to Amazon SageMaker, open Domain URL, login into Amazon SageMaker Unified Studio and go to the previously created Project
3.  Open Data tab, click on '+' and then choose Add Data → Add Connection → Select connection type and specify configuration parameters where {x} are located in the CloudFormation Outputs:

| Connection Type  | Configuration parameters ({x} from CloudFormation Outputs) |
| ------------- | ------------- |
| Amazon Aurora PostgreSQL | Name: demo-aurorapg <br> Host: {AuroraPGHost} <br> Port: {AuroraPGPort} <br> Database: {AuroraPGDatabase} <br> Authentification : AWS Secrets Manager: {AuroraPGSecretArn} |
| Amazon Redshift Serverless  | Name: demo-redshift <br> Host: {RedshiftHost} <br> Port: {RedshiftPort} <br> Database: {RedshiftDatabase} <br> Authentification : AWS Secrets Manager: {RedshiftSecretArn} |
| Amazon DynamoDB | Name: demo-ddb |
   
5.	Open Compute tab and connect the existing compute resource:
- Add Compute → Connect to existing compute resources → Amazon Redshift Serverless
- Endter the following configuration parameters:
- On **demo-wg.redshift** compute details page, select *Actions* → *Open Query Editor* and ensure selected data source in the right top corner is *Redshift (demo-wg.redshift) → dev → public*
- Run DDL + DML from [redshift.sql](sql/redshift.sql) to populate data in the redshift local dev database

| Compute Type | Configuration parameters (from CloudFormation Outputs) |
| ------------- | ------------- |
| Amazon Redshift Serverless | Redshift compute: demo-wg <br/> Authentication : AWS Secrets Manager: {RedshiftSecretArn} <br/> Name: demo |


### Create Zero-ETL Integrations
#### Zero-ETL Integration between Redshift and Aurora PostgreSQL
Open Query Editor and select connection to the custom Redshift compute
- Run the following commands to create zero-etl database
```sql
SELECT integration_id FROM SVV_INTEGRATION;
-- copy integration_id
CREATE DATABASE "zetlpg" FROM INTEGRATION 'integration_id' DATABASE "postgres";
```
#### Zero-ETL Integration between Redshift and DynamoDB
- Copy Invoices data from Amazon DynamoDB into Redshift by running the following commands:
```sql
CREATE TABLE invoices (
order_id integer not null,
invoice_number varchar(200) not null,
total integer not null,
status varchar(10) not null,
primary key(invoice_number)
);
COPY invoices from 'dynamodb://invoices'
IAM_ROLE default
readratio 50;
```
![Zero-ETL Integrations](visuals/Zero-ETLSetup.png)

## SQL Analytics via Redshift Query Editor v2

Open Project and navigate to the Query Editor. Select Redshift connection pointing to our custom compute demo.redshift. Enter the following SQL to find an answer what are top 5 customers with maximum orders.
Below SQL command joins local tables with the customer table from replicated from Amazon Aurora PostgreSQL database to nominated Redshift database zetlpg via Zero-ETL integration.
```sql
SELECT
  o.customer_id, c.customer_name,
  SUM(od.quantity) AS total_quantity
FROM
  public.orders o
  JOIN public.order_details od ON o.order_id = od.order_id
  JOIN public.products p ON od.product_id = p.product_id
  JOIN "zetlpg"."public"."customers" c ON c.customer_id = o.customer_id
GROUP BY
  o.customer_id, c.customer_name
ORDER BY
  total_quantity DESC
LIMIT
  5;
```
Review the results:

![Redshift Query Editor v2](visuals/Redshift%20Query%20Editor%20v2.png)

### Generative SQL

Now open Amazon Q and type the following question: 

**Question:** *What are the most ordered products?*

Amazon Q will generate SELECT statement similar to this one:

```sql
SELECT
  p."product_name",
  SUM(od."quantity") AS "total_quantity"
FROM
  public.products p
  JOIN public.order_details od ON p."product_id" = od."product_id"
GROUP BY
  p."product_name"
ORDER BY
  "total_quantity" DESC;
```
Click 'Add to querybook' and execute to confirm the results.

![Redshift Query Editor v2](visuals/Generative%20SQL.png)

**Question:** *Identify all product categories associated with invoices that are currently in draft status. Include the outstanding amounts for these invoices.*

Amazon Q will generate SELECT statement similar to this one:

```sql
SELECT
  c."category_name",
  SUM(i."total") AS "outstanding_amount"
FROM
  "public"."categories" c
  JOIN "public"."products" p ON c."category_id" = p."category_id"
  JOIN "public"."order_details" od ON p."product_id" = od."product_id"
  JOIN "public"."invoices" i ON od."order_id" = i."order_id"
WHERE
  i."status" = 'DRAFT'
GROUP BY
  c."category_name";
```

**Questions:** *Show a list of customers with unpaid invoices? Statuses of unpaid invoises are SUBMITTED and AUTHORISED*

Amazon Q will generate SELECT statement similar to this one:

```sql
SELECT DISTINCT
  o.customer_id
FROM
  "public".orders o
  JOIN "public".invoices i ON o.order_id = i.order_id
WHERE
  i.status IN ('SUBMITTED', 'AUTHORISED');
```

## SQL Analytics via Amazon Athena

Now let’s query the federated data sources such as Amazon DynamoDB, Aurora PostgreSQL, Redshift Serverless. Once connections to the federated sources are established successfully, expand connection, select target table and click on '⋮' to query with Amazon Athena

![Redshift Query Editor v2](visuals/Athena.png)

Here are sample queries to try:
```sql
select * from "demo-aurorapg"."public"."customers" limit 10;
select * from "demo-redshift"."public"."invoices" limit 10;
select * from "demo-dynamodb"."default"."invoices" limit 10;
```

### Connecting to Snowflake

Create demo warehouse, user, role and sample data by running these commands in Snowflake:

```sql
-- Create warehouse
CREATE WAREHOUSE "demo_wh" WITH 
    WAREHOUSE_SIZE = 'X-SMALL'
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 1
    AUTO_SUSPEND = 600  -- 10 minutes
    AUTO_RESUME = TRUE
    INITIALLY_SUSPENDED = TRUE
    SCALING_POLICY = 'STANDARD';

-- Grant usage
GRANT USAGE ON WAREHOUSE "demo_wh" TO ROLE PUBLIC;

-- Create a specific role
CREATE ROLE demo_role;

-- Create user
CREATE USER demo_user
    PASSWORD = '******'  -- use a strong password
    DEFAULT_ROLE = demo_role
    DEFAULT_WAREHOUSE = "demo_wh"
    MUST_CHANGE_PASSWORD = FALSE;

-- Assign the role to user
GRANT ROLE demo_role TO USER demo_user;

-- Grant warehouse access
GRANT USAGE ON WAREHOUSE "demo_wh" TO ROLE demo_role;

--ALTER USER demo_user SET DEFAULT_WAREHOUSE = "demo_wh";

CREATE DATABASE "demo_db";
CREATE SCHEMA "demo_schema";

-- Create customer table
CREATE TABLE "demo_db"."demo_schema"."customer" (
    "customer_id" INTEGER,
    "first_name" VARCHAR(50),
    "email" VARCHAR(100),
    "city" VARCHAR(50),
    "total_orders" INTEGER
);

-- Insert sample data
INSERT INTO "demo_db"."demo_schema"."customer" 
VALUES 
    (1, 'John', 'john.doe@email.com', 'New York', 5),
    (2, 'Emma', 'emma.smith@email.com', 'Los Angeles', 3),
    (3, 'Michael', 'michael.brown@email.com', 'Chicago', 7),
    (4, 'Sarah', 'sarah.wilson@email.com', 'Houston', 2),
    (5, 'David', 'david.lee@email.com', 'Seattle', 4),
    (6, 'Lisa', 'lisa.anderson@email.com', 'Boston', 6),
    (7, 'James', 'james.taylor@email.com', 'Miami', 1),
    (8, 'Maria', 'maria.garcia@email.com', 'Denver', 8),
    (9, 'Robert', 'robert.miller@email.com', 'Phoenix', 3),
    (10, 'Jennifer', 'jennifer.davis@email.com', 'Portland', 5);

-- Grant access
GRANT USAGE ON DATABASE "demo_db" TO ROLE demo_role;
GRANT USAGE ON SCHEMA "demo_db"."demo_schema" TO ROLE demo_role;
GRANT SELECT ON TABLE "demo_db"."demo_schema"."customer" TO ROLE demo_role;
```

Open Amazon SageMaker Unified Studio Project and go to Data tab. Click on '+' and then choose Add Data → Add Connection → Select connection type Snowflake  and specify configuration parameters:

| Connection Type  | Configuration parameters |
| ------------- | ------------- |
| Snowflake | Name: demo-snowflake <br> Host: {account}.snowflakecomputing.com <br> Port: 443 <br> Database: demo_db <br> Warehouse: demo_wh <br> Authentification : AWS Secrets Manager: demo_user |

Once connection is established successfully, expand connection, select target table and click on '⋮' to query with Amazon Athena

## References
[Amazon SageMaker Unified Studio](https://aws.amazon.com/sagemaker/unified-studio/)

[Amazon SageMaker Lakehouse](https://aws.amazon.com/sagemaker/lakehouse/)

[What is zero-ETL?](https://aws.amazon.com/what-is/zero-etl/)

[Amazon DynamoDB Zero-ETL integrations](https://aws.amazon.com/dynamodb/integrations/)

### Feature releases
[Amazon DynamoDB zero-ETL integration with Amazon SageMaker Lakehouse](https://aws.amazon.com/about-aws/whats-new/2024/12/amazon-dynamo-db-zero-etl-integration-sagemaker-lakehouse/)

[Amazon DynamoDB zero-ETL integration with Amazon Redshift](https://aws.amazon.com/about-aws/whats-new/2024/10/amazon-dynamodb-zero-etl-integration-redshift/)

[Amazon Aurora PostgreSQL zero-ETL integration with Amazon Redshift](https://aws.amazon.com/about-aws/whats-new/2024/10/amazon-aurora-postgresql-zero-etl-integration-redshift-generally-available/)
