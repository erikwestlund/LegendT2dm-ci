Large-scale Evidence Generation and Evaluation across a Network of Databases for Type 2 Diabetes Mellitus (LEGEND-T2DM)
=============================================================================

<img src="https://img.shields.io/badge/Study%20Status-Results%20Available-yellow.svg" alt="Study Status: Results Available">

- Analytics use case(s): **Population-Level Estimation**
- Study type: **Clinical Application**
- Tags: **-**
- Study lead: **Marc A. Suchard**
- Study lead forums tag: **[msuchard](https://forums.ohdsi.org/u/msuchard)**
- Study start date: **1 November 2021**
- Study end date: **-**
- Protocol: **[HTML document](https://ohdsi-studies.github.io/LegendT2dm/Protocol.html)**
- Publications: **-**
- Results explorer: **-**

Requirements
============

- A database in [Common Data Model version 5](https://github.com/OHDSI/CommonDataModel) in one of these platforms: SQL Server, Oracle, PostgreSQL, IBM Netezza, Apache Impala, Amazon RedShift, Google BigQuery, or Microsoft APS.
- R version 4.0.5
- On Windows: [RTools](http://cran.r-project.org/bin/windows/Rtools/)
- [Java](http://java.com)
- 100 GB of free disk space

⫚ Fork - LegendT2DM Custom Data Extraction (CDE) Package
========================================================

This fork has been modified to extract LegendT2DM for methods development and testing. The abbreviation and prefix `CDE` has been used throughout to indicate changes.

This fork has the same requirements as the original package (namely, R 4.0.5) but requires some modifications to the original package to run.

To use the CDE functionality, follow the directions in the R scripts in the `extra/CDE/` subdirectory of the package:

* `0 - Prepare Environment.R` - Install the required packages and set up the environment. Note that there are instructions for two changes to the renv.lock file to assure all packages can be downloaded.
* `1 - Cohort Extraction.R` - Run the CDE process to extract the data from the database. You will need to modify the connection details to your database in the beginning of this file.
* `2 - Data Preparation.R` - This file provides examples for how to work with the extracted data.

This fork by default works with all the main drug classes for the ITT, OT1, and OT2 analyses. This is reflected in the `extra/cdeCohorsToCreate.csv`, `extra/cdeOutcomesOfInterest.csv`, and `extra/cdeTcosOfInterest.csv` files. These files can be modified to work with a subset of the drug classes. 

This fork depends on a modified version of the `CohortMethod` package (see the `DESCRIPTION` file) which allows for skipping all the propensity score generation and modeling steps. This saves compute time when running the package.


How to run
==========
1. Follow [these instructions](https://ohdsi.github.io/Hades/rSetup.html) for setting up your R environment, including RTools and Java.

2. Open your study package in RStudio. Use the following code to install all the dependencies:

	```r
	install.packages("renv")
	renv::activate()
	renv::restore()
	```

3. In RStudio, select 'Build' then 'Install and Restart' to install the `LegendT2dm` package.

4. For **class v.s. class** studies, follow the instructions in Steps 6-11 below. For your convenience, all code is also provided under `extras/CodeToRun.R`.

5. For **drug v.s. drug** studies across all drug classes, follow the instructions in Steps 6-11. Replace the argument `indicationId = "class"` with `indicationId = "drug"` and also replace `cohorts = "class"` with `cohorts = "drug"` to view/upload diagnostics and results. It is _strongly recommended_ to run all the code provided under `extras/CodeToRunAllDrugs.R` for more reliable execution. 

6. Once the `LegendT2dm` package is installed, you can execute the feasibility assessment portion of the study by modifying and using the code below. You may also refer to the code provided under `extras/CodeToRun.R` (or `indicationId = "drug"` for drug-level studies):

	```r
	library(LegendT2dm)
	
	Sys.setenv(DATABASECONNECTOR_JAR_FOLDER="s:/DatabaseDrivers")
	
	# Run-once: set-up your database driver
	DatabaseConnector::downloadJdbcDrivers(dbms = "postgresql")
	
	# Optional: specify where the temporary files (used by the Andromeda package) will be created:
	options(andromedaTempFolder = "s:/AndromedaTemp")

	# Maximum number of cores to be used:
	maxCores <- min(4, parallel::detectCores()) # Or more depending on your hardware

	# Minimum cell count when exporting data:
	minCellCount <- 5

	# The folder where the study intermediate and result files will be written:
	outputFolder <- "s:/LegendT2dmStudy"

	# Details for connecting to the server:
	# See ?DatabaseConnector::createConnectionDetails for help
	connectionDetails <- DatabaseConnector::createConnectionDetails(dbms = "postgresql",
									server = "some.server.com/ohdsi",
									user = "joe",
									password = "secret")

	# The name of the database schema where the CDM data can be found:
	cdmDatabaseSchema <- "cdm_synpuf"

	# The name of the database schema and table where the study-specific cohorts will be instantiated:
	cohortDatabaseSchema <- "scratch.dbo"
	tablePrefix <- "legendt2dm_study"

	# Some meta-information that will be used by the export function:
	databaseId <- "Synpuf"
	databaseName <- "Medicare Claims Synthetic Public Use Files (SynPUFs)"
	databaseDescription <- "Medicare Claims Synthetic Public Use Files (SynPUFs) were created to allow interested parties to gain familiarity using Medicare claims data while protecting beneficiary privacy. These files are intended to promote development of software and applications that utilize files in this format, train researchers on the use and complexities of Centers for Medicare and Medicaid Services (CMS) claims, and support safe data mining innovations. The SynPUFs were created by combining randomized information from multiple unique beneficiaries and changing variable values. This randomization and combining of beneficiary information ensures privacy of health information."

	# For some database platforms (e.g. Oracle): define a schema that can be used to emulate temp tables:
	options(sqlRenderTempEmulationSchema = NULL)

	# Feasibility assessment ---------------------------------------------------------
	assessPhenotypes(connectionDetails = connectionDetails,
					 cdmDatabaseSchema = cdmDatabaseSchema,
					 oracleTempSchema = oracleTempSchema,
					 cohortDatabaseSchema = cohortDatabaseSchema,
					 outputFolder = outputFolder,
					 tablePrefix = tablePrefix,
					 indicationId = 'class',
					 databaseId = databaseId,
					 databaseName = databaseName,
					 databaseDescription = databaseDescription,
					 createExposureCohorts = TRUE,
					 runExposureCohortDiagnostics = TRUE,
					 createOutcomeCohorts = TRUE,
					 runOutcomeCohortDiagnostics = TRUE)

	```

7. Upload the files ```class/cohortDiagnosticsExport/Results_class_exposures_<DatabaseId>.zip``` and ```outcome/cohortDiagnosticsExport/Results_outcomes_<DatabaseId>.zip``` in the output folder to the study coordinator:

	```r
	uploadPhenotypeResults(cohorts = "class",
	                       outputFolder, privateKeyFileName = "<file>", userName = "<name>"
	uploadPhenotypeResults(cohorts = "outcome",
	                       outputFolder, privateKeyFileName = "<file>", userName = "<name>")
  	```
  	
	where `<file>` and `<name>` are the credentials provided to you personally by the study coordinator.
  
8. View your cohort diagnostics locally via:

	```r
	CohortDiagnostics::preMergeDiagnosticsFiles(dataFolder = file.path(outputFolder, 
	                                                                   "class/cohortDiagnosticsExport"))
	LegendT2dmCohortExplorer::launchCohortExplorer(cohorts = "class",
	                                               dataFolder = file.path(outputFolder, 
						                              "class/cohortDiagnosticsExport"))

	CohortDiagnostics::preMergeDiagnosticsFiles(dataFolder = file.path(outputFolder, 
	                                                                   "outcome/cohortDiagnosticsExport"))
	LegendT2dmCohortExplorer::launchCohortExplorer(cohorts = "outcome",
	                                               dataFolder = file.path(outputFolder, 
						                              "outcome/cohortDiagnosticsExport"))
	```

9. Complete the feasibility assessment by constructing sample-restricted propensity models: 
  	```r
	assessPropensityModels(connectionDetails = connectionDetails,
	                       cdmDatabaseSchema = cdmDatabaseSchema,
	                       oracleTempSchema = oracleTempSchema,
	                       cohortDatabaseSchema = cohortDatabaseSchema,
	                       outputFolder = outputFolder,
	                       indicationId = "class",
	                       tablePrefix = tablePrefix,
	                       databaseId = databaseId,
	                       maxCores = maxCores)
	```
  	and uploading the file ```class/assessmentOfPropensityScores/Results_class_ps_<DatabaseId>.zip``` in the output folder to the study coordinator:
  
	```r
  	uploadPsAssessmentResults(cohorts = "class",
	                          outputFolder, privateKeyFileName = "<file>", userName = "<name>")
	```

10. To prepare to execute the class-vs-class comparative effectiveness and safety (CES) study, first update your `LegendT2dm` to version `>= 2.0.0`.  You can accomplish this via a `git pull` in RStudio and then select 'Build' and 'Install and Restart'.  To check your package version:
  	```
  	packageVersion("LegendT2dm")
  	```

11. Execute the CES study via:
	```
	execute(connectionDetails = connectionDetails,
        	cdmDatabaseSchema = cdmDatabaseSchema,
        	oracleTempSchema = oracleTempSchema,
        	cohortDatabaseSchema = cohortDatabaseSchema,
        	outputFolder = outputFolder,
        	indicationId = "class",
        	databaseId = databaseId,
        	databaseName = databaseName,
        	databaseDescription = databaseDescription,
        	tablePrefix = tablePrefix,
        	createExposureCohorts = TRUE, 
        	createOutcomeCohorts = TRUE,  
        	fetchAllDataFromServer = TRUE,
        	generateAllCohortMethodDataObjects = TRUE,
        	runCohortMethod = TRUE,
        	computeCovariateBalance = TRUE,
        	exportToCsv = TRUE,
        	maxCores = maxCores)
	```
	and upload the file ```class/export/Results_class_study_<DatabaseId>.zip``` in the output folder to the study coordinator:
	```
	uploadStudyResults(cohorts = "class",
	                   outputFolder, privateKeyFileName = "<file>", userName = "<name>")
	```

License
=======
The `LegendT2dm` package is licensed under Apache License 2.0

Development
===========
`LegendT2dm` was developed in ATLAS and R Studio.

### Development status

Collecting cohort diagnostics from data partners.
