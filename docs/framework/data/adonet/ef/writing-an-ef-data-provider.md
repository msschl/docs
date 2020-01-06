---
title: "Writing an Entity Framework Data Provider"
ms.date: "03/30/2017"
ms.assetid: 092e88c4-a301-453a-b5c3-5740c6575a9f
---
# Writing an Entity Framework Data Provider
This section discusses how to write an Entity Framework provider to support a data source other than SQL Server. The Entity Framework includes a provider that supports SQL Server.  
  
## Introducing the Entity Framework Provider Model  
 The Entity Framework is database independent, and you can write a provider by using the ADO.NET Provider Model to connect to a diverse set of data sources.  
  
 The Entity Framework data provider (built using the ADO.NET Data Provider model) performs the following functions:  
  
- Maps Entity Data Model (EDM) primitive types to provider types.  
  
- Exposes provider-specific functions.  
  
- Generates provider-specific commands for a given DbQueryCommandTree to support Entity Framework queries.  
  
- Generates provider-specific update commands for a given DbModificationCommandTree to support updates through the Entity Framework.  
  
- Exposes mapping files for the store schema definition, to support generation of a model based on a database.  
  
- Exposes metadata (tables and views, for example) via a conceptual model.  
  
 ![b42a7a5c&#45;0ac0&#45;4911&#45;86be&#45;0460a78760ba](./media/b42a7a5c-0ac0-4911-86be-0460a78760ba.gif "b42a7a5c-0ac0-4911-86be-0460a78760ba")  
  
## Sample  
 See the [Entity Framework Sample Provider](https://code.msdn.microsoft.com/windowsdesktop/Entity-Framework-Sample-6a9801d0) for a sample of an Entity Framework provider that supports a data source other than SQL Server.  
  
## In This Section  
 [SQL Generation](sql-generation.md)  
  
 [Modification SQL Generation](modification-sql-generation.md)  
  
 [Provider Manifest Specification](provider-manifest-specification.md)  
  
## See also

- [Working with Data Providers](working-with-data-providers.md)
