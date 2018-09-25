---
title: "What's new in SQL Server 2019 | Microsoft Docs"
ms.custom: ""
ms.date: "09/24/2018"
ms.prod: "sql-server-2018"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "server-general"
ms.tgt_pltfrm: ""
ms.topic: "article"
author: "MikeRayMSFT"
ms.author: "mikeray"
manager: "craigg"
monikerRange: ">=sql-server-ver15||=sqlallproducts-allversions"
---
# What's new in SQL Server 2019

[!INCLUDE[tsql-appliesto-ssver15-xxxx-xxxx-xxx](../includes/tsql-appliesto-ssver15-xxxx-xxxx-xxx.md)]

[!INCLUDE[sql-server-2019](..\includes\sssqlv15-md.md)] builds on previous releases to grow SQL Server as a platform that gives you choices of development languages, data types, on-premises or cloud, and operating systems. This article summarizes what is new for SQL Server 2019. For more information and known issues, see the [SQL Server 2019 Release Notes](sql-server-ver15-release-notes.md).

**Try SQL Server 2019!**
- [![Download from Evaluation Center](../includes/media/download2.png)](http://go.microsoft.com/fwlink/?LinkID=862101) [Download SQL Server 2019 to install on Windows](http://go.microsoft.com/fwlink/?LinkID=862101)
- Install on Linux for [Red Hat Enterprise Server](../linux/quickstart-install-connect-red-hat.md), [SUSE Linux Enterprise Server](../linux/quickstart-install-connect-suse.md), and [Ubuntu](../linux/quickstart-install-connect-ubuntu.md).
- [Run SQL Server 2019 on Docker](../linux/quickstart-install-connect-docker.md).

## CTP 2.0 

Community technology preview (CTP) 2.0 is the first public release of [!INCLUDE[sql-server-2019](..\includes\sssqlv15-md.md)]. The following features are added or enhanced for [!INCLUDE[sql-server-2019](..\includes\sssqlv15-md.md)] CTP 2.0.

- [Big Data Clusters](#bigdatacluster)
  - Deploy a Big Data cluster with SQL and Spark Linux containers on Kubernetes
  - Access your big data from HDFS
  - Run Advanced analytics and machine learning with Spark
  - Use Spark streaming to data to SQL data pools
  - Use Azure Data Studio to run Query books that provide a notebook experience

- [Database engine](#databaseengine)
  - UTF-8 support
  - Resumable online index create allows index create to resume after interruption
  - Clustered columnstore online index build and rebuild
  - Always Encrypted with secure enclaves
  - Intelligent query processing
  - Java language programmability extension
  - SQL Graph features
  - Database scoped configuration setting for online and resumable DDL operations
  - Always On Availability Groups - secondary replica connection redirection
  - Data discovery and classification - natively built into SQL Server
  - Expanded support for persistent memory devices
  - Support for columnstore statistics in `DBCC CLONEDATABASE`
  - New options added to `sp_estimate_data_compression_savings`
  - SQL Server Machine Learning Services failover clusters
  - Lightweight query profiling infrastructure enabled by default
  - New Polybase connectors
  - New `sys.dm_db_page_info` system function returns page information

- [SQL Server on Linux](#sqllinux)
  - Replication support
  - Support for the Microsoft Distributed Transaction Coordinator (MSDTC)
  - Always On Availability Group on Docker containers with Kubernetes
  - OpenLDAP support for third-party AD providers
  - Machine Learning on Linux
  - New container registry
  - New RHEL-based container images
  - Memory pressure notification

- [Master Data Services](#mds)
  - Silverlight controls replaced

- [Security](#security)
  - Certificate management in SQL Server Configuration Manager

- [Tools](#tools)
  - SQL Server Management Studio (SSMS) 18.0 (preview)
  - Azure Data Studio

Continue reading for more details about these features.

## <a id="bigdatacluster"></a>Big Data Clusters

[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] [Big Data Clusters](../big-data-cluster/big-data-cluster-overview.md) enables new scenarios including the following:

- Deploy a Big Data cluster with [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] and Spark Linux containers on Kubernetes
- Access your big data from HDFS
- Run Advanced analytics and machine learning with Spark
- Use Spark streaming to data to SQL data pools
- Run Query books that provide a notebook experience in [**Azure Data Studio**](../sql-operations-studio/what-is.md).
  
[!INCLUDE [Big Data Clusters preview](../includes/big-data-cluster-preview-note.md)]

## <a id="databaseengine"></a> Database Engine

CTP 2.0 introduces or enhances the following new features for [!INCLUDE[ssdeNoVersion](../includes/ssdenoversion_md.md)].

### Database compatibility level

Database **COMPATIBILITY_LEVEL 150** is added. To enable for a specific user database, execute:

   ```sql
   ALTER DATABASE database_name SET COMPATIBILITY_LEVEL =  150;
   ```

### UTF-8 support

Full support for the widely used UTF-8 character encoding as an import or export encoding, or as database-level or column-level collation for text data. UTF-8 is allowed in the `CHAR` and `VARCHAR` datatypes, and is enabled when creating or changing an object’s collation to a collation with the `UTF8` suffix. 

For example,`LATIN1_GENERAL_100_CI_AS_SC` to `LATIN1_GENERAL_100_CI_AS_SC_UTF8`. UTF-8 is only available to Windows collations that support supplementary characters, as introduced in SQL Server 2012. `NCHAR` and `NVARCHAR` allow UTF-16 encoding only, and remain unchanged.

This feature may provide significant storage savings, depending on the character set in use. For example, changing an existing column data type with ASCII strings from `NCHAR(10)` to `CHAR(10)` using an UTF-8 enabled collation, translates into nearly 50% reduction in storage requirements. This reduction is because `NCHAR(10)` requires 22 bytes for storage, whereas `CHAR(10)` requires 12 bytes for the same Unicode string.

### Resumable online index create

- **Resumable online index create** allows an index create operation to pause and resume later from where the operation was paused or failed, instead of restarting from the beginning.

  Resumable online index create supports the follow scenarios:
  - Resume an index create operation after an index create failure, such as after a database failover or after running out of disk space.
  - Pause an ongoing index create operation and resume it later allowing to temporarily free system resources as required and resume this operation later.
  - Create large indexes without using as much log space and a long-running transaction that blocks other maintenance activities and allowing log truncation.

  In case of an index create failure, without this feature an online index create operation must be executed again and the operation must be restarted from the beginning.

With this release, we extend the resumable functionality adding this feature to available [resumable online index rebuild](http://azure.microsoft.com/blog/modernize-index-maintenance-with-resumable-online-index-rebuild/).

In addition, this feature can be set as the default for a specific database using [database scoped default setting for online and resumable DDL operations](../t-sql/statements/alter-database-scoped-configuration-transact-sql.md).

For more information, see [Resumable Online Index Create](../t-sql/statements/create-index-transact-sql.md#resumable-indexes).

### Build and rebuild clustered columnstore indexes online

Convert row-store tables into columnstore format. Creating clustered columnstore indexes (CCI) was an offline process in the previous versions of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] - requiring all changes stop while the CCI is created. With [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] and 
[!INCLUDE[ssSDSfull](../includes/sssdsfull-md.md)] you can create or re-create CCI online. Workload will not be blocked and all changes made on the underlying data are transparently added into the target columnstore table. Examples of new [!INCLUDE[tsql](../includes/tsql-md.md)] statements that can be used are:

  ```sql
  CREATE CLUSTERED COLUMNSTORE INDEX cci
    ON <tableName>
    WITH (ONLINE = ON);
  ```

  ```sql
  ALTER INDEX cci
    ON <tableName>
    REBUILD WITH (ONLINE = ON);
  ```

### Always Encrypted with secure enclaves

Expands upon Always Encrypted with in-place encryption and rich computations. The expansions come from the enabling of computations on plaintext data, inside a secure enclave on the server side.

Cryptographic operations include the encryption of columns, and the rotating of column encryption keys. These operations can now be issued by using [!INCLUDE[tsql](../includes/tsql-md.md)], and they do not require that data be moved out of the database. Secure enclaves provide Always Encrypted to a broader set of scenarios that have both of the following requirements:  

- The demand that sensitive data are protected from high-privilege, yet unauthorized users, including database administrators, system administrators,  cloud operators, or malware.
- The requirement that rich computations on protected data be supported within the database system.

For details, see [Always Encrypted with secure enclaves](../relational-databases/security/encryption/always-encrypted-enclaves.md).

> [!NOTE]
> Always Encrypted with secure enclaves is only available on Windows OS.

### Intelligent query processing

- **Row mode memory grant feedback** expands on the memory grant feedback feature introduced in [!INCLUDE[ssSQL17](../includes/sssql17-md.md)] by adjusting memory grant sizes for both batch and row mode operators.  For an excessive memory grant condition, if the granted memory is more than two times the size of the actual used memory, memory grant feedback will recalculate the memory grant. Consecutive executions will then request less memory. For an insufficiently sized memory grant that results in a spill to disk, memory grant feedback will trigger a recalculation of the memory grant. Consecutive executions will then request more memory. This feature is enabled by default under database compatibility level 150.

- **Approximate COUNT DISTINCT** returns the approximate number of unique non-null values in a group. This function is designed for use in big data scenarios. This function is optimized for queries where all the following conditions are true:
   - Accesses data sets of at least millions of rows.
   - Aggregates a column or columns that have a large number of distinct values.
   - Responsiveness is more critical than absolute precision.
      - `APPROX_COUNT_DISTINCT` returns results that are typically within 2% of the precise answer.
      - And it returns the approximate answer in a small fraction of the time needed for the precise answer.

- **Batch mode on rowstore** no longer requires a columnstore index to process a query in batch mode. Batch mode allows query operators to work on a set of rows, instead of just one row at a time. This feature is enabled by default under database compatibility level 150. Batch mode improves the speed of queries that access rowstore tables when all the following are true:
   - The query uses analytic operators such as joins or aggregation operators.
   - The query involves 100,000 or more rows.
   - The query is CPU bound, rather than input/output data bound.
   - Creation and use of a columnstore index would have one of the following drawbacks:
      - Would add too much overhead to the query.
      - Or, is not feasible because your application depends on a feature that is not yet supported with columnstore indexes.

- **Table variable deferred compilation** improves plan quality and overall performance for queries referencing table variables. During optimization and initial compilation, this feature will propagate cardinality estimates that are based on actual table variable row counts.  This accurate row count information will be used for optimizing downstream plan operations. This feature is enabled by default under database compatibility level 150.

To use intelligent query processing features, set database `COMPATIBILITY_LEVEL = 150`.

### <a id="programmability"></a> Java language programmability extensions

- **Java language extension (preview)**: Use the Java language extension to execute Java code in [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)]. In [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)], this extension is installed when you add the feature 'Machine Learning Services (in-database)' to your SQL Server instance.

### <a id="sqlgraph"></a> SQL Graph features

- **Match support in `MERGE` DML** allows you to specify graph relationships in a single statement, instead of separate `INSERT`, `UPDATE`, or `DELETE` statements. Merge your current graph data from node or edge tables with new data using the `MATCH` predicates in the `MERGE` statement. This feature enables `UPSERT` scenarios on edge tables. Users can now use a single merge statement to insert a new edge or update an existing one between two nodes.

- **Edge Constraints** are introduced for edge tables in SQL Graph. Edge tables can connect any node to any other node in the database. With introduction of edge constraints, you can now apply some restrictions on this behavior. The new `CONNECTION` constraint can be used to specify the type of nodes a given edge table will be allowed to connect to in the schema.

### Database scoped default setting for online and resumable DDL operations 

- **Database scoped default setting for online and resumable DDL operations** allows a default behavior setting for `ONLINE` and `RESUMABLE` index operations at the database level, rather than defining these options for each individual index DDL statement such as index create or rebuild.

- Set these defaults using the `ELEVATE_ONLINE` and `ELEVATE_RESUMABLE` database scoped configuration options. Both options will cause the engine to automatically elevate supported operations to index online or resumable execution. You can enable the following behaviors using these options:

  - `FAIL_UNSUPPORTED` option allows all index operations online or resumable and fail index operations that are not supported for online or resumable.
  - `WHEN_SUPPPORTED` option allows supported operations online or resumable and run index unsupported operations offline or non-resumable.
  - `OFF` option allows the current behavior of executing all index operations offline and non-resumable unless explicitly specified in the DDL statement.

To override the default setting, include the ONLINE or RESUMABLE option in the index create and rebuild commands.  

Without this feature you have to specify the online and resumable options directly in the index DDL statement such as index create and rebuild.

More information:
For more information on index resumable operations see [Resumable Online Index Create](http://azure.microsoft.com/blog/resumable-online-index-create-is-in-public-preview-for-azure-sql-db/).

### <a id="ha"></a>Always On Availability Groups - more synchronous replicas 

- **Up to five synchronous replicas**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] increases the maximum number of synchronous replicas to 5, up from 3 in [!INCLUDE[ssSQL17](../includes/sssql17-md.md)] . You can configure this group of 5 replicas to have automatic failover within the group. There is 1 primary replica, plus 4 synchronous secondary replicas.

- **Secondary-to-primary replica connection redirection**: Allows client application connections to be directed to the primary replica regardless of the target server specified in the connection string. This capability allows connection redirection without a listener. Use secondary-to-primary replica connection redirection in the following cases:

  - The cluster technology does not offer a listener capability.
  - A multi subnet configuration where redirection becomes complex.
  - Read scale-out or disaster recovery scenarios where cluster type is `NONE`.

For details, see [Secondary to primary replica read/write connection redirection (Always On Availability Groups)](../database-engine/availability-groups/windows/secondary-replica-connection-redirection-always-on-availability-groups.md).

### Data discovery and classification

Data discovery and classification provides advanced capabilities that are natively built into [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)]. Classifying and labeling your most sensitive data provides the following benefits:
- Helps meet data privacy standards and regulatory compliance requirements.
- Supports security scenarios, such as monitoring (auditing), and alerting on anomalous access to sensitive data.
- Makes it easier to identify where sensitive data resides in the enterprise, so that administrators can take the right steps to secure the database.

For more information, see [SQL Data Discovery and Classification](../relational-databases/security/sql-data-discovery-and-classification.md).

[Auditing](../relational-databases/security/auditing/sql-server-audit-database-engine.md) has also been enhanced to include a new field in the audit log called `data_sensitivity_information`, which logs the sensitivity classifications (labels) of the actual data that was returned by the query. For details and examples, see [Add sensitivity classification](../t-sql/statements/add-sensitivity-classification-transact-sql.md).

>[!NOTE]
>There are no changes in terms of how audit is enabled. There is a new field added to the audit records, `data_sensitivity_information`, which logs the sensitivity classifications (labels) of the actual data that was returned by the query. See [Auditing access to sensitive data](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-data-discovery-and-classification#subheading-3).

### Expanded support for persistent memory devices

Any [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] file that is placed on a persistent memory device can now operate in *enlightened* mode. [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] directly accesses the device, bypassing the storage stack of the operating system using efficient memcpy operations. This mode improves performance because it allows low latency input/output against such devices.
    - Examples of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] files include:
        - Database files
        - Transaction log files
        - In-Memory OLTP checkpoint files
    - Persistent memory is also known as storage class memory.
    - Persistent memory is occasionally referred to informally as *pmem* on some non-Microsoft websites.

> [!NOTE]
> For this preview release, enlightenment of files on persistent memory devices is only available on Linux. SQL Server on Windows supports persistent memory devices starting with SQL Server 2016.

### Support for columnstore statistics in DBCC CLONEDATABASE

`DBCC CLONEDATABASE` creates a schema-only copy of a database that includes all the elements necessary to troubleshoot query performance issues without copying the data.  In previous versions of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)], the command did not copy the statistics necessary to accurately troubleshoot columnstore index queries and manual steps were required to capture this information. Now in [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)], DBCC CLONEDATABASE automatically captures the stats blobs for columnstore indexes, so no manual steps will be required.

### New options added to sp_estimate_data_compression_savings

`sp_estimate_data_compression_savings` returns the current size of the requested object and estimates the object size for the requested compression state.  Currently this procedure supports three options: `NONE`, `ROW`, and `PAGE`. SQL Server 2019 introduces two new options: `COLUMNSTORE` and `COLUMNSTORE_ARCHIVE`. These new options will allow you to estimate the space savings if a columnstore index is created on the table using either standard or archive columnstore compression.

### <a id="ml"></a> SQL Server Machine Learning Services failover clusters and partition based modeling

- **Partition-based modeling**: Process external scripts per partition of your data using the new parameters added to `sp_execute_external_script`. This functionality supports training many small models (one model per partition of data) instead of one large model.

- **Windows Server Failover Cluster**: Configure high availability for Machine Learning Services on a Windows Server Failover Cluster.

For detailed information, see [What's new in SQL Server Machine Learning Services](../advanced-analytics/what-s-new-in-sql-server-machine-learning-services.md).

### Lightweight query profiling infrastructure enabled by default

The lightweight query profiling infrastructure (LWP) provides query performance data more efficiently than standard profiling technologies. Lightweight profiling is now enabled by default. It was introduced in [!INCLUDE[ssSQL15](../includes/sssql15-md.md)] SP1. Lightweight profiling offers a query execution statistics collection mechanism with an expected overhead of 2% CPU, compared with an overhead of up to 75% CPU for the standard query profiling mechanism. On previous versions, it was OFF by default. Database administrators could enable it with [trace flag 7412](../t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql.md). 

For more information on lightweight profiling, see [Developers Choice: Query progress – anytime, anywhere](http://blogs.msdn.microsoft.com/sql_server_team/query-progress-anytime-anywhere/).

### <a id="polybase"></a>New Polybase connectors

- **New connectors for [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)], Oracle, Teradata, and MongoDB**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] introduces new connectors to external data for [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)], Oracle, Teradata, and MongoDB.

### New sys.dm_db_page_info system function returns page information

`sys.dm_db_page_info(database_id, file_id, page_id, mode)` returns information about a page in a database. The function returns a row that contains the header information from the page, including the `object_id`, `index_id`, and `partition_id`. This function replaces the need to use `DBCC PAGE` in most cases.  

In order to facilitate troubleshooting of page-related waits, a new column called page_resource was also added to `sys.dm_exec_requests` and `sys.sysprocesses`. This new column allows you to join `sys.dm_db_page_info` to these views via another new system function - `sys.fn_PageResCracker`. See the following script as an example:

```sql
SELECT page_info.* 
FROM sys.dm_exec_requests AS d 
  CROSS APPLY sys.fn_PageResCracker(d.page_resource) AS r
  CROSS APPLY sys.dm_db_page_info(r.db_id, r.file_id, r.page_id,'DETAILED')
    AS page_info;
```

## <a id="sqllinux"></a> SQL Server on Linux

- **Replication support**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] supports SQL Server Replication on Linux. A Linux virtual machine with SQL Agent can be a publisher, distributor, or subscriber. 

  Create the following types of publications:
  - Transactional
  - Snapshot
  - Merge

  Configure replication [!INCLUDE[ssManStudioFull](../includes/ssmanstudiofull-md.md)] or use [replication stored procedures](../relational-databases/system-stored-procedures/replication-stored-procedures-transact-sql.md).

- **Support for the Microsoft Distributed Transaction Coordinator (MSDTC)**: SQL Server 2019 on Linux supports the Microsoft Distributed Transaction Coordinator (MSDTC). For details, see [How to configure MSDTC on Linux](../linux/sql-server-linux-configure-msdtc.md).

- **Always On Availability Group on Docker containers with Kubernetes**: Kubernetes can orchestrate containers running [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] instances to provide a highly available set of databases with SQL Server Always On Availability Groups. A Kubernetes operator deploys a StatefulSet including a container with **mssql-server container** and a health monitor.

- **OpenLDAP support for third-party AD providers**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] on Linux supports OpenLDAP, which allows third-party providers to join Active Directory.

- **Machine Learning on Linux**: SQL Server 2019 Machine Learning Services (In-Database) is now supported on Linux. Support includes `sp_execute_external_script` stored procedure. For instructions on how to install Machine Learning Services on Linux, see [Install SQL Server 2019 Machine Learning Services R and Python support on Linux](../linux/sql-server-linux-setup-machine-learning.md).

- **New container registry**: All container images for [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] as well as [!INCLUDE[ssSQL17](../includes/sssql17-md.md)] are now located in the Microsoft Container Registry. Microsoft Container Registry is the official container registry for the distribution of Microsoft product containers. In addition, certified RHEL-based images are now published.

  - Microsoft Container Registry: `mcr.microsoft.com/mssql/server:vNext-CTP2.0`
  - Certified RHEL-based container images: `mcr.microsoft.com/mssql/rhel/server:vNext-CTP2.0`

## <a id="mds"></a> Master Data Services (MDS)

- **Silverlight controls replaced with HTML**: The Master Data Services (MDS) portal no longer depends on Silverlight. All the former Silverlight components have been replaced with HTML controls.

## <a id="security"></a>Security

- **Certificate management in SQL Server Configuration Manager**: SSL/TLS certificates are widely used to secure access to SQL Server instances. Certificate management is now integrated into the SQL Server Configuration Manager, simplifying common tasks such as:

  - Viewing and validating certificates installed in a [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] instance. 
  - Viewing certificates close to expiration.
  - Deploy certificates across machines participating in Always On Availability Groups (from the node holding the primary replica).
  - Deploy certificates across machines participating in a failover cluster instance (from the active node).

  > [!NOTE]
  > User must have administrator permissions on all the cluster nodes.

## <a id="tools"></a>Tools

- [**Azure Data Studio**](../azure-data-studio/what-is.md): Previously released under the preview name SQL Operations Studio, Azure Data Studio is a lightweight, modern, open source, cross-platform desktop tool for the most common tasks in data development and administration. With Azure Data Studio you can connect to SQL Server on premises and in the cloud on Windows, macOS, and Linux. Azure Data Studio allows you to:

  - Edit and run queries in a modern development environment with lightning fast Intellisense, code snippets, and source control integration.  
  - Quickly visualize data with built-in charting of your result sets.  
  - Create custom dashboards for your servers and databases using customizable widgets.  
  - Easily manage your broader environment with the built-in terminal.  
  - Analyze data in an integrated notebook experience built on Jupyter.  
  - Enhance your experience with custom theming and extensions.  
  - And explore your Azure resources with a built-in subscription and resource browser.
  - Supports scenarios using SQL Server Big Data Cluster.


- [**SQL Server Management Studio (SSMS) 18.0 (preview)**](../ssms/sql-server-management-studio-ssms.md)

  - Support for [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)].
  - Support for Always Encrypted with secure enclaves.
  - Smaller download size.
  - Now based on the Visual Studio 2017 Isolated Shell.
  - For a complete list, see the [SSMS changelog](../ssms/sql-server-management-studio-changelog-ssms.md).

## Other services

[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] CTP 2.0 does not introduce new features for the following services:

- [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] Analysis Services (SSAS)
- [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] [!INCLUDE[ssISnoversion](../includes/ssisnoversion-md.md)] (SSIS)
- [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] [!INCLUDE[ssRSnoversion](../includes/ssrsnoversion-md.md)] (SSRS)

## Next steps

See the [SQL Server 2019 Release Notes](sql-server-ver15-release-notes.md).

[!INCLUDE[get-help-options](../includes/paragraph-content/get-help-options.md)]
