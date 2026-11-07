# 10.9. EDA and Reporting

- [EDA services on AWS](#eda-services-on-aws)
- [SageMaker Data Wrangler](#sagemaker-data-wrangler)
- [SageMaker notebooks](#sagemaker-notebooks)
- [Amazon Athena](#amazon-athena)
- [AWS Glue DataBrew](#aws-glue-databrew)
- [Amazon QuickSight](#amazon-quicksight)
- [SageMaker Unified Studio](#sagemaker-unified-studio)

---

## EDA services on AWS

| Need | Service |
|---|---|
| Visual EDA for ML data | SageMaker Data Wrangler (in Canvas) |
| Code-based EDA | SageMaker Studio notebooks |
| SQL on data in S3 | Amazon Athena |
| Data profiles for analysts | AWS Glue DataBrew |
| Business dashboards | Amazon QuickSight |
| One place for data, SQL, and ML | SageMaker Unified Studio |

```mermaid
flowchart LR
    S3["Data in S3"] --> GC["Glue Data Catalog"]
    GC --> ATH["Athena SQL"]
    GC --> DW["Data Wrangler"]
    GC --> DB["Glue DataBrew"]
    ATH --> QS["QuickSight dashboards"]
    DW --> NB["Studio notebooks / training"]
```

## SageMaker Data Wrangler

<img src="../icons/Amazon-SageMaker-AI.svg" alt="Amazon SageMaker AI" width="48">

- Data Wrangler is part of SageMaker Canvas.
- It imports data from S3, Athena, Redshift, Snowflake, and more.
- The Data Quality and Insights report shows:
  - Missing values and duplicates
  - Outliers and class imbalance
  - Target leakage
  - Quick model feature importance
- It gives visual analyses, for example histograms, scatter plots, and correlation.
- It gives more than 300 built-in transforms.
- You export the flow to a SageMaker Processing job, Pipeline, or Feature Store.
- You can use natural language to ask for transforms.

## SageMaker notebooks

<img src="../icons/Amazon-SageMaker-AI.svg" alt="Amazon SageMaker AI" width="48">

- Use JupyterLab or Code Editor in SageMaker Studio.
- Use pandas, matplotlib, and seaborn for EDA.
- Use Amazon EMR or AWS Glue interactive sessions for Spark at scale.
- Use the SageMaker Python SDK to query Feature Store and Athena.

## Amazon Athena

<img src="../icons/Amazon-Athena.svg" alt="Amazon Athena" width="48">

- Athena is serverless SQL on data in S3.
- It uses the AWS Glue Data Catalog for table schemas.
- You pay for the data that each query scans.
- Use Parquet and partitions to reduce cost.
- Athena for Apache Spark runs notebooks for Spark-based EDA.

## AWS Glue DataBrew

<img src="../icons/AWS-Glue-DataBrew.svg" alt="AWS Glue DataBrew" width="48">

- DataBrew is a visual tool for data cleaning and profiling.
- Profile jobs show statistics, missing values, and correlations.
- Recipes record transforms. You run recipes as jobs on full datasets.
- Data quality rules check data in profile jobs.

## Amazon QuickSight

<img src="../icons/Amazon-Quick.svg" alt="Amazon Quick" width="48">

- QuickSight is the AWS BI service. AWS now offers it as part of Amazon Quick Suite.
- It connects to Athena, Redshift, S3, RDS, and other sources.
- SPICE is an in-memory engine for fast dashboards.
- Amazon Q in QuickSight answers natural language questions and builds visuals.
- You can embed dashboards in applications.
- ML insights give anomaly detection and forecasts in visuals.

## SageMaker Unified Studio

<img src="../icons/Amazon-SageMaker.svg" alt="Amazon SageMaker" width="48">

- Unified Studio is part of the next generation of Amazon SageMaker.
- It gives one place for SQL, notebooks, data processing, and ML.
- It uses SageMaker Lakehouse to access S3 and Redshift data.
- It uses SageMaker Catalog (built on Amazon DataZone) for data discovery and governance.
- It includes Amazon Q Developer to help write SQL and code.
