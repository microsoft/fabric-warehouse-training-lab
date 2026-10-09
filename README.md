# Fabric Warehouse training lab

A hands-on introduction to loading, transforming, recovering, monitoring, tuning,
and auditing data in Microsoft Fabric Warehouse using a TPC-H sample dataset.

## Start here: run the notebooks in order

Deploying the lab creates the Fabric items; it does **not** generate the sample
data or run the exercises for you.

1. **Run `generate_tpc-h_data` first.** Open this notebook in your Fabric workspace and confirm that `warehouse_training_lab_sample_data` is its default Lakehouse. Run the package-installation cell, then the data-generation cell, and wait for generation to finish. The notebook writes the sample data to the Lakehouse's `Files/tpch_sf100` folder.
2. **Then open `warehouse_training_lab`.** Select the deployed `warehouse_training_lab` Warehouse as the notebook's data item and start with the **T-SQL** runtime. Work through the instructions and cells in order, beginning with **Getting started** and **Lab setup**. The setup cells create the warehouse tables and load the generated files; the later sections use those tables for the exercises below.

**Work through the training notebook cell by cell rather than selecting Run all.**
Some exercises require portal actions, a runtime change, a wait for query history,
or intentionally produce an error to demonstrate a recovery scenario.

> Use a dedicated training workspace and sample data only. The notebook drops,
> truncates, and renames tables, and the custom SQL pool exercises change
> workspace-wide settings.

## Lab contents

| Item | Purpose |
| --- | --- |
| [`generate_tpc-h_data`](generate_tpc-h_data.Notebook) | Python notebook that generates the sample data. Run this first. |
| [`warehouse_training_lab_sample_data`](warehouse_training_lab_sample_data.Lakehouse) | Lakehouse containing the generated Parquet files used by the ingestion exercises. |
| [`warehouse_training_lab`](warehouse_training_lab.Warehouse) | Warehouse in which you create tables and run the SQL exercises. |
| [`warehouse_training_lab`](warehouse_training_lab.Notebook) | Guided training notebook covering the sections described below. |

You need a Fabric workspace with capacity and permissions to run notebooks, read
and write the sample data, and manage the settings used in the exercises. If you
deployed these items through Jumpstart, use the existing Warehouse; you do not
need to create another one when following the notebook's manual setup guidance.

## Data-generation notebook overview

The `generate_tpc-h_data` notebook has two executable cells:

| Cell | What it does |
| --- | --- |
| Install dependencies | Installs LakeBench `1.2.0` with its data-generation extras. |
| Generate TPC-H data | Runs `TPCHDataGenerator` with `scale_factor=100`, writing to `/lakehouse/default/Files/tpch_sf100`. |

Allow generation to complete before starting the warehouse notebook. Scale factor
100 can take time and consume capacity and storage. If you change the output
folder, also update the training notebook's ingestion paths.

## Warehouse training notebook: section guide

The main notebook uses the sample dataset to walk through the warehouse lifecycle:
prepare tables, ingest data, recover from changes, measure query behavior, tune
the workload, and inspect audit activity. It is primarily T-SQL, with a Python
section for configuring and exercising custom SQL pools.

### Getting started

Review the workspace collation guidance, choose the T-SQL runtime, and attach the
Warehouse in the notebook explorer. The notebook recommends case-insensitive
collation for a smoother lab experience.

### Lab setup

Create the eight core TPC-H tables: `customer`, `lineitem`, `nation`, `orders`,
`part`, `partsupp`, `region`, and `supplier`. Setup also creates additional
`lineitem` tables for ingestion and clustering comparisons, loads the initial
data from the sample Lakehouse, and runs preparation queries.

### Data ingestion

Load data with `COPY INTO`, copy existing warehouse data with `INSERT INTO ...
SELECT`, and read files with `OPENROWSET`. An optional activity uses a Fabric Data
Factory copy pipeline. Compare the ingestion methods through
`queryinsights.exec_requests_history`, including elapsed time, allocated CPU time,
rows, and data scanned. Run the three labeled sample queries here: later tuning
exercises use them as a baseline.

### Data transformation

Explore recovery and ETL patterns through deliberate changes to the `nation`
table.

| Subsection | What you explore |
| --- | --- |
| Warehouse snapshots | Create a snapshot named `MySnapshot` in the portal, compare snapshot data with the live table after a truncate, and review recovery from a snapshot. |
| Recovering from ETL failures or accidental changes | Query historical data with `FOR TIMESTAMP AS OF`, create a point-in-time recovery clone, and restore rows to the live table. |
| Restore points | Create a restore point, drop a table, observe the intentionally failing recovery attempts, and restore the Warehouse through the portal. |
| Table clones | Explore history and retention boundaries, transform a cloned table, swap table names, and roll back to the previous version while retaining the changed data for investigation. |
| Comparing snapshots, time travel and clones | Review the effects of schema changes and the differences between the recovery mechanisms demonstrated in the lab. |

Follow the snapshot, restore-point, and timestamp instructions before executing
the dependent cells. Historical queries need a suitable time within the table's
available history.

### Monitoring and tuning

Use Query Insights to inspect the labeled ingestion and sample queries, then
compare different resource-allocation and table-layout choices.

| Subsection | What you explore |
| --- | --- |
| Custom SQL Pools | Switch to the Python runtime, inspect the current configuration, and configure example `ETL`, `Reporting`, and `Adhoc` pools. Run SQL through ODBC with and without the `Reports` application name to exercise request classification. A reference example shows how to disable custom pools afterwards. |
| Data clustering | Switch back to T-SQL. Compare unclustered `lineitem` data with tables clustered on `l_shipdate` and `l_linenumber`. Examine query benefits against the additional load-time and CPU cost. |
| Reviewing the impact of custom SQL pools | Return to Query Insights to compare pool routing, runtime, CPU, and data scanned for the application-name examples. |
| Collecting query consumption information | Supply a start timestamp and inspect queries overlapping a 30-second window. The results show query activity and execution metrics, not a direct capacity bill. |

Before running the Python connection examples, change the sample
`warehouse_name = "day_02_warehouse"` to your deployed Warehouse name, normally
`warehouse_training_lab`, and use its actual SQL connection details. Allow time
for completed queries to appear in Query Insights; the optional pipeline exercise
suggests waiting 5-10 minutes.

### SQL analytics endpoint

Review how the SQL analytics endpoint exposes Lakehouse tables and why metadata
synchronization and file layout matter for query performance.

| Subsection | What you explore |
| --- | --- |
| Metadata Sync | Review synchronization behavior, inspect table status with `sys.dm_db_external_tables_log_status`, and examine refresh examples using `sys.sp_dw_refresh_ext_table` and the Fabric REST API. Use examples appropriate to the endpoint's supported metadata-sync version. |
| Optimizing lake file sizes | Learn about small-file overhead and file-size trade-offs, and inspect the `sys.sp_get_table_health_metrics` example for table storage health. |

These are reference examples to adapt to your Lakehouse and table names. They
require Lakehouse tables: the generator writes Parquet files under **Files**, not
Delta tables under **Tables**, so generating the sample files alone does not
provide tables for these endpoint exercises.

### Securing your warehouse

Enable SQL audit events in the Warehouse settings, run the labeled sample queries
to generate activity, and inspect audit records with `sys.fn_get_audit_file_v2`.
The final query joins `sys.dm_audit_actions` to show readable action names alongside
the event time, principal, client IP, application name, and SQL statement.

Update the sample audit-log URL to the target Warehouse's actual audit location
before running the final query.

## Deployment parameterization

The root [`parameter.yml`](parameter.yml) is loaded automatically by Fabric
Jumpstart's `fabric-cicd` deployment. For every target environment, it replaces the
source workspace GUID with `$workspace.$id` and the source lakehouse GUID with the
deployed ID of `warehouse_training_lab_sample_data`.

These replacements cover `COPY INTO` and `OPENROWSET` paths and the data
generator's Lakehouse attachment. Deploy the Lakehouse alongside the notebooks
and keep `parameter.yml` at the deployment repository root.

The parameter file does not configure every example-specific connection or
endpoint: review the Python SQL connection details, audit-log location, and any
environment-specific OneLake hostnames before running those exercises.

Direct notebook import or Fabric Git synchronization does not apply this
`fabric-cicd` parameter file; those workflows require separate parameterization.
