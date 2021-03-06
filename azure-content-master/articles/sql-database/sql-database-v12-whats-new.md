<properties
	pageTitle="What's new in SQL Database V12 | Microsoft Azure"
	description="Describes why business systems that are using Azure SQL Database in the cloud will benefit by upgrading to version V12 now."
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/19/2016"
	ms.author="genemi"/>


# What's new in SQL Database V12


This topic describes the many advantages that the new V12 version of Azure SQL Database has over version V11.


We continue to add features to V12. So we encourage you to visit our Service Updates webpage for Azure, and to use its filters:


- Filtered to the [SQL Database service](https://azure.microsoft.com/updates/?service=sql-database).
- Filtered to General Availability [(GA) announcements](http://azure.microsoft.com/updates/?service=sql-database&update-type=general-availability) for SQL Database features.


The latest information about resource limits for SQL Database is documented at:<br/>[Azure SQL Database Resource Limits](sql-database-resource-limits.md).


## Increased application compatibility with SQL Server


A key goal for SQL Database V12 was to improve the compatibility with Microsoft SQL Server 2014, and to maintain the compatibility as new versions of SQL Server are released. Among other areas, V12 achieves parity with SQL Server in the important area of programmability. For example:

- [Built-in JSON support](https://msdn.microsoft.com/library/dn921897.aspx)

- [Window functions](http://msdn.microsoft.com/library/ms189798.aspx), with [OVER](http://msdn.microsoft.com/library/ms189461.aspx)

- [XML indexes](http://msdn.microsoft.com/library/bb934097.aspx) and [selective XML indexes](http://msdn.microsoft.com/library/jj670104.aspx)

- [Change tracking](http://msdn.microsoft.com/library/bb933875.aspx)

- [SELECT...INTO](http://msdn.microsoft.com/library/ms188029.aspx)

- [Full-text search](http://msdn.microsoft.com/library/ms142571.aspx)

- [ALTER DATABASE SCOPED CONFIGURATION (Transact-SQL)](http://msdn.microsoft.com/library/mt629158.aspx)

Please see [here](sql-database-transact-sql-information.md) for the small set of features not yet supported in SQL Database.


### Compatibility level 130


> [AZURE.IMPORTANT] Starting in **June 2016**, *newly* created databases on Azure SQL Database V12 have their compatibility level start at 130, which matches Microsoft SQL Server 2016 GA.
> 
> Of course, you can use `ALTER DATABASE YourDatabase SET COMPATIBILITY_LEVEL = 120` if you prefer.
> 
> Databases created before June 2016 do not have their compatibility level changed by this change of default. Nor is the level of a database changed by upgrading it from V11 to V12.



For an explanation of how you can compare your most important queries between the latest versus previous compatibility level, see:

- [Improved Query Performance with Compatibility Level 130 in Azure SQL Database](sql-database-compatibility-level-query-performance-130.md)



## More premium performance, new performance levels


In V12, we increased the Database Transaction Units (DTUs) allocated to all Premium performance levels by 25% at no additional cost. Even greater performance gains can be achieved with new features like:


- Support for in-memory [columnstore indexes](http://msdn.microsoft.com/library/gg492153.aspx).
- [Table partitioning by rows](http://msdn.microsoft.com/library/ms187802.aspx) with related enhancements to [TRUNCATE TABLE](http://msdn.microsoft.com/library/ms177570.aspx).
- The availability of dynamic management views [(DMVs)](http://msdn.microsoft.com/library/ms188754.aspx) to help monitor and tune performance.


### Reliable performance


If your client program connects to SQL Database V12 while your client runs on an Azure virtual machine (VM), you must open the following port ranges on the VM:

- 11000-11999
- 14000-14999


Click [here](sql-database-develop-direct-route-ports-adonet-v12.md) for details about the ports for SQL Database V12. The ports are needed by performance enhancements in SQL Database V12.


## Better support for cloud SaaS vendors


Only in V12, we released the new Standard performance level S3 and the public preview of [elastic database pools](sql-database-elastic-pool.md).
This is a solution specifically designed for cloud SaaS vendors.  With elastic database pools, you can:


- Share DTUs amongst databases to reduce costs for large numbers of databases.
- Execute [elastic database jobs](sql-database-elastic-jobs-overview.md) to manage databases at scale.


## Security enhancements


Security is a primary concern for anyone who runs their business in the cloud. The latest security features released in V12 include:


- [Row-level security](http://msdn.microsoft.com/library/dn765131.aspx) (RLS)
- [Dynamic Data Masking](sql-database-dynamic-data-masking-get-started.md)
- [Contained databases](http://msdn.microsoft.com/library/ff929188.aspx)
- [Application roles](http://msdn.microsoft.com/library/ms190998.aspx) managed with GRANT, DENY, REVOKE
- [Transparent Data Encryption](http://msdn.microsoft.com/library/0bf7e8ff-1416-4923-9c4c-49341e208c62.aspx) (TDE)
- [Connecting to SQL Database By Using Azure Active Directory Authentication](sql-database-aad-authentication.md)
 - SQL Database now supports Azure Active Directory authentication, a mechanism of connecting to SQL Database by using identities in Azure Active Directory (Azure AD). With Azure Active Directory authentication you can centrally manage the identities of database users and other Microsoft services in one central location.
- [Always Encrypted](https://msdn.microsoft.com/library/mt163865.aspx) (in preview) makes encryption transparent to applications and allows clients to encrypt sensitive data inside client applications without sharing the encryption keys with SQL Database.


## Increased business continuity when recovery is needed


V12 offers significantly improved recovery point objectives (RPOs) and estimated recovery times (ERTs):


| Business continuity feature | Earlier version | V12 |
| :-- | :-- | :-- |
| Geo-restore | ??? RPO < 24 hours.<br/>??? ERT <  12 hours. | ??? RPO < 1 hour.<br/>??? ERT < 12 hours. |
| Active Geo-Replication | ??? RPO < 5 minutes.<br/>??? ERT < 1 hour. | ??? RPO < 5 seconds.<br/>??? ERT < 30 seconds. |


See [SQL Database business continuity](sql-database-business-continuity.md) for more information.


## More reasons to upgrade now


There are many good reasons why customers should upgrade now to Azure SQL Database V12 from V11:


- SQL Database V12 has a long list of features beyond those of V11.
- We continue to add new features to V12, but no new features will be added to V11.
- Most new features are released on SQL Database V12 before they are being released for Microsoft SQL Server.


## Are you using V12 already?


One easy way to see if you have a database or logical server running on an earlier version of the SQL Database service is to do the following:


1. Go to the [Azure Portal](https://portal.azure.com/).
2. Click **Browse**.
3. Click **SQL Servers**.
4. The icon next to your server or database tells the story:
 - ![Icon for a v12 server](./media/sql-database-v12-whats-new/v12_icon.png) **V12 logical server**
 - ![Icon for earlier version server](./media/sql-database-v12-whats-new/earlier_icon.png) **Earlier version logical server**


Another technique to ascertain the version is to run the `SELECT @@version;` statement in your database, and view the results similar to:


- **12**.0.2000.10 &nbsp; *(version V12)*
- **11**.0.9228.18 &nbsp; *(version V11)*


A V12 database can be hosted only on a V12 logical server. And a V12 server can host only V12 databases.


If you are not yet running on V12, you can upgrade your logical server by following the steps in [Upgrade to SQL Database V12 in place](sql-database-v12-plan-prepare-upgrade.md).


## <a name="V12AzureSqlDbPreviewGaTable"></a> General Availability regions


- By July 31, 2015, all regions had been promoted to General Availability (GA).
- V12 was released in December 2014, but only at the status of Preview.

[Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
