---
title: Set up a data science client for R development on SQL Server | Microsoft Docs
ms.prod: sql
ms.technology: machine-learning

ms.date: 04/15/2018  
ms.topic: conceptual
author: HeidiSteen
ms.author: heidist
manager: cgronlun
---
# Set up a data-science client for R development on SQL Server
[!INCLUDE[appliesto-ss-xxxx-xxxx-xxx-md-winonly](../../includes/appliesto-ss-xxxx-xxxx-xxx-md-winonly.md)]

After you have configured an instance of [!INCLUDE[ssCurrent](../../includes/sscurrent-md.md)] to support machine learning, you should set up a development environment that is capable of connecting to the server for remote execution and deployment.

This article describes some typical client scenarios, including configuration of the free Visual Studio Community edition to run R code in SQL Server.

## 1 - Install R libraries

Your client environment must include Microsoft R Open, as well as RevoScaleR and other product-specific R packages that support distributed execution of R on SQL Server. Standard distributions of R do not have the packages that support remote compute contexts or parallel execution of R tasks.

To get the full complement of libraries, install any of the following, but choose an option that most closely matches your SQL Server installation. Your client libraries should closely match equivalent libraries on the server. 

| Library provider | Usage  |
|------------------|--------------------------------|
| [SQL Server 2017 Machine Learning Server (Standalone)](../install/sql-machine-learning-standalone-windows-install.md) | For best compatibility with a SQL Server 2017 instance, install the free developer edition of SQL Server 2017, choosing the "Standalone server" feature under **Shared features** in SQL Server Setup. |
| [SQL Server 2016 R Server (Standalone)](../install/sql-r-standalone-windows-install.md) | Recommended for use with SQL Server 2016 R Services. Install the free developer edition of SQL Server 2016, choosing the "Standalone R Server" feature under **Shared features** in SQL Server Setup. |
| [Microsoft R Client](http://aka.ms/rclient/download) |  This free download is a limited version of R Server: it provides RevoScaleR and other R packages, but is capped at two threads and in-memory data. However, by installing R Client, you can create R solutions that start locally, and shift execution (referred to as *compute context*) to access data and the computational power of a remote SQL Server instance. For more information, see [What is Microsoft R Client](https://docs.microsoft.com/machine-learning-server/r-client/what-is-microsoft-r-client). |

## 2 - Use built-in R tools

When you install R with SQL Server, you get the same R tools that are installed with any base installation of R, such as RGui, Rterm, and so forth. These tools are lightweight, useful for checking package and library information, running ad hoc commands or script, or stepping through tutorials.

Base R in Microsoft products is provided by Microsoft R Open, included in SQL Server Setup, built on open-source R. The following standard R tools are therefore installed by default. 

| Tool | Description | 
|------|-------------|
| **RTerm**: | A command-line terminal for running R scripts | 
| **RGui.exe** | A simple interactive editor for R. The command-line arguments are the same for RGui.exe and RTerm. |
|**RScript** | A command-line tool for running R scripts in batch mode. |

Tools are located in **bin** folder for base R as installed SQL Server or R Client. The following paths are valid locations for the tools, depending on which product version and feature you installed:

+ R Client: `~\Program Files\Microsoft\R Client\R_SERVER\bin\x64`
+ R Services: `~\Program Files\Microsoft SQL Server\MSSQL13.<instancename>\R_SERVICES\bin\x64`
+ R Server Standalone: `~\Program Files\Microsoft SQL Server\130\R_SERVER\bin\x64`
+ Machine Learning Services: `~\Program Files\Microsoft SQL Server\MSSQL14.<instancename>\R_SERVICES\bin\x64`
+ Machine Learning Server (Standalone): `~\Program Files\Microsoft SQL Server\140\R_SERVER\bin\x64`

## 3 - Install an IDE

For sustained and serious development projects, you should install an integrated development environment (IDE).

### RStudio

When using [RStudio](https://www.rstudio.com/), you can configure the environment to use the R libraries and executables that correspond to those on a remote SQL Server.

1. Check R package versions installed on SQL Server. For more information, see [Get R package information](determine-which-packages-are-installed-on-sql-server.md#get-the-r-library-location).

1. Install Microsoft R Client or one of the standalone server options to add RevoScaleR and other R packages, including the base R distribution used by your SQL Server instance. Choose a version at the same level or lower (packages are backward compatible) that provides the same package versions as those on the server. For version information, see the version map in this article: [Upgrade R and Python components](use-sqlbindr-exe-to-upgrade-an-instance-of-sql-server.md).

1. In RStudio, [update your R path](https://support.rstudio.com/hc/articles/200486138-Using-Different-Versions-of-R) to point to the R environment providing RevoScaleR, Microsoft R Open, and other Microsoft packages. Depending on how you obtained RevoScaleR and other libraries, it is most likely one of the following paths:

  + C:\Program Files\Microsoft SQL Server\130\R_SERVER\Library
  + C:\Program Files\Microsoft SQL Server\140\R_SERVER\Library
  + C:\Program Files\Microsoft\R Client\R_SERVER\bin\x64

### R Tools for Visual Studio (RTVS)

If you don't already have a preferred IDE for R, we recommend **R Tools for Visual Studio**.

+ [Download R Tools for Visual Studio (RTVS)](https://visualstudio.microsoft.com/vs/features/rtvs/)
+ [Installation instrructions](https://docs.microsoft.com/visualstudio/rtvs/installing-r-tools-for-visual-studio) - RTVS is available in several versions of Visual Studio.
+ [Get started with R Tools for Visual Studio](https://docs.microsoft.com/visualstudio/rtvs/getting-started-with-r)

### Connect to SQL Server from RTVS

This example uses Visual Studio 2017 Community Edition, with the data science workload installed.

1. From the **File** menu, select **New** and then select **Project**.

2. The -hand pane contains a list of preinstalled templates. Click **R**, and select **R Project**. In the **Name** box, type `dbtest` and click **OK**.

3. Visual Studio creates a new project folder and a default script file, `Script.R`. 

4. Type `.libPaths()` on the first line of the script file, and then press CTRL + ENTER.

5. The current R library path should be displayed in the **R Interactive** window. 

6. Click the **R Tools** menu and select **Windows** to see a list of other R-specific windows that you can display in your workspace.
 
    + View help on packages in the current library by pressing CTRL + 3.
    + See R variables in the **Variable Explorer**, by pressing CTRL + 8.

7. Create a connection string to a SQL Server instance, and use the connection string in the RxInSqlServer constructor to create a SQL Server data source object. 

    ```r
    connStr <- "Driver=SQL Server;Server=MyServer;Database=MyTestDB;Uid=;Pwd="
    sqlShareDir <- paste("C:\\AllShare\\", Sys.getenv("USERNAME"), sep = "")
    sqlWait <- TRUE
    sqlConsoleOutput <- FALSE
    cc <- RxInSqlServer(connectionString = connStr, shareDir = sqlShareDir, wait = sqlWait, consoleOutput = sqlConsoleOutput)
    sampleDataQuery <- "SELECT TOP 100 * from [dbo].[MyTestTable]"
    inDataSource <- RxSqlServerData(sqlQuery = sampleDataQuery, connectionString = connStr, rowsPerRead = 500)
    ```
    > [!TIP]
    > To run a batch, select the lines you want to run and press CTRL + ENTER.

8. Set the compute context to the server, and then run some simple R code on the data.

    ```r
    rxSetComputeContext(cc)
    rxGetVarInfo(data = inDataSource)
    ```

    Results are returned in the **R Interactive** window.
    
    If you want to assure yourself that the code is being executed on the SQL Server instance, you can use Profiler to create a trace.

## Next steps

Two different tutorials include exercises for switching the compute context from local to a remote SQL Server instance.

+ [Tutorial: Use RevoScaleR R functions with SQL Server data](../tutorials/deepdive-data-science-deep-dive-using-the-revoscaler-packages.md)
+ [Data Science End-to-End Walkthrough](../tutorials/walkthrough-data-science-end-to-end-walkthrough.md)