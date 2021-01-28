Step-by-step: End to end analytics in Synapse (*under development)

**1. Environment setup**

1a: Create a new resource group

1b: Create Azure Synapse Analytics workspace
 - Set ADLS as account (Basics)
 - Set default as file system name (Basics)
 - Set password for SQL Admin (Security)
 - Don't allow connections from all IP addresses (Networking)

1c: Create a storage account with hierarchical namespace (ADLS)
 - Create containers for data (raw, curated, dev, staging, prod)

1d: Create Azure Key Vault
 - Give Synapse Workspace GET access permission in access policies

1e: Create an SQL Pool (SQLPool01)

1f: Change networking on Synapse Workspace in "Firewalls"-tab to allow for Azure Services and your own IP

1g: Ensure you have Owner role on the Synapse Workspace

1h: Create secret for ADLS access key (Key1) and for the password of the SQL Admin

**2. Configuring initial Synapse environment**

2a: Set up Git configuration

2b: Create linked service for NYCTLC trip data
 - Base url: https://s3.amazonaws.com/nyc-tlc/

2c: Set up Azure Key Vault linked servicekv

2d: Set up adls dataset
 - Parameters: *directory* and *file*

2e: Set up nyctlc dataset
 - Parameters: *relativeUrl*

2f: Group datasets in folder "Parameterized"

**3. Setting up some initial extract pipelines**

3a. Set up folder structure for pipeline sequence (01-03)

3b. Create a pipeline in each bucket; *Master* (01), *Extract year* (02), and *Extract month* (03)

3c. Set up necessary parameters for *Master* (01); ArrayMonth (Array), ArrayMonth (Array)

3d. Set up necessary parameters for *Extract year* (02); ArrayMonth (Array), ArrayMonth (Array)

3e. Set up necessary parameters for *Extract month* (03); Year (String), ArrayMonth (Array)

3f. Set up necessary variable for *Extract year* (02); Year (String)

3g. Set up necessary variables for *Extract month* (03); relativeUrl (String), directory (String), and file (string)

3h. In *Master*, create an *Execute Pipeline* activity and attach *Extract year* and belonging parameters to activity

3i. In *Extract year* create a ForEach activity with sequential settings for ArrayYear, and add two inner activities where you Set variable (Year) and upon succeeded an Execute pipeline activity that triggers *Extract month* with corresponding parameters and variables

3j. In *Extract month* create a ForEach activity with sequential setting for ArrayMonth, and add four innter activities where you three Set variable activities; relativeUrl = @concat('trip+data/yellow_tripdata_', pipeline().parameters.Year, '-', item(), '.csv'), directory = @concat('raw/nyctlc/tripdata/', pipeline().parameters.Year, '/', item()), and file = @concat('yellow_tripdata_', pipeline().parameters.Year, '-', item(), '.csv'). Lastly, add an Copy Data activity with nyctlc as source and adls as target.

3k. Get lookup data by creating another separate pipeline with relativeUrl = misc/taxi+_zone_lookup.csv, directory = raw/nyctlc/misc, and file = taxi+_zone_lookup.csv. Include the pipeline in folder *99 Utilities*.

3l. Publish and trigger for the wanted periods.

</br>

**4. Preparing the load and transform stage**

4a. Create schema nyctlc in SQLPool01

4b. We need to create a master key in our SQL environment - "CREATE MASTER KEY"

4c. Create table nyctlc.YellowTaxiTripRecords and nyctlc.SummaryYellowTaxiTripRecords. SQL-scripts in folder.

4d. Create linked service for ADLS that we are going to use for staging in our mapping data flow. Use key vault secret for authentication.

4e. Create Synapse Analytics linked service with Azure Key Vault for SQL Authentication for sqladminuser

4f. Create SQL Pool dataset for the newly created linked service SummaryYellowTaxiTripRecords and YellowTaxiTripRecords, and add them to a folder SQLPool01

4g. Create ADLS dataset CSV for trip records and lookup table and add them to a folder NYCTLC *storage account name*

4h. Add an activity in *Master* (01); "Load and Transform"

4i. Add a new pipeline *Load and Transform Year* (02) and add parameter ArrayYear. Add ForEach actvity and add Execute Pipeline inside with current item as parameter value

4j. Add a new pipeline *Load and Transform* with parameter Year (String) and add a mapping data flow inside. Initiate data flow debug

4k. Set staging linked service and staging storage folder in *Settings* for Data Flow (data/staging) and set wanted Core Count

**5. Building the data flow**

5a. Add Year as parameter, add this parameter to the data flow in the pipeline.

5b. Two csv source inputs. In YellowTaxiTripRecords.csv add a wildcard path concat('raw/nyctlc/tripdata/', $Year, '/*/*.csv'). For the lookup table use the direct path raw/nyctlc/misc/taxi+_zone_lookup.csv

5c. Set all input values to string type

5d. Create a left inner join activity - PULocationID (YellowTaxiTripRecords) == LocationID (YellowTaxiLookupLocation)

5e. Select wanted columns and rename accordingly:
- VendorID
- Pickup_Datetime
- Dropoff_Datetime
- PassengerCount
- TripDistance
- PickupLocationID
- PaymentType
- FareAmount
- ExtraAmount
- MtaTaxAmount
- TipAmount
- TollsAmount
- ImprovementSurchargeAmount
- TotalAmount
- PickupBorough
- PickupZone

5f. Ensuring data consistency
- TripDistance: toDecimal(TripDistance)
- FareAmount: toDecimal(FareAmount)
- ExtraAmount: toDecimal(ExtraAmount)
- MtaTaxAmount: toDecimal(MtaTaxAmount)
- TipAmount: toDecimal(TipAmount)
- TollsAmount: toDecimal(TollsAmount)
- ImprovementSurchargeAmount: toDecimal(ImprovementSurchargeAmount)
- TotalAmount: toDecimal(TotalAmount)
- Pickup_Datetime: toTimestamp(Pickup_Datetime, 'yyyy-MM-dd HH:mm:ss')
- Dropoff_Datetime: toTimestamp(Dropoff_Datetime, 'yyyy-MM-dd HH:mm:ss')

5g. Sink - YellowTaxiTripRecords Synapse SQL dataset (preview data to ensure everything is alright)

**Bonus exercises (before last tasks on Power BI)**

5h. SelectAggData
- Pickup_Datetime: TripDate
- PassengerCount
- TripDistance
- FareAmount
- TipAmount
- TotalAmount
- PickupBorough
- PickupZone

5i. TransformAggData
 - TripDate: toDate(TripDate)
 - PassengerCount: toInteger(PassengerCount)

 5j. Aggregate
 - Group by: TripDate, PickupBorough, and PickupZone
 - Aggregates: 
 - TotalTripCount: count()
 - TotalPassengerCount: sum(PassengerCount)
 - TotalDistanceTravelled: sum(TripDistance)
 - TotalTipAmount: sum(TipAmount)
 - TotalFareAmount: sum(FareAmount)
 - TotalTripAmount: sum(TotalAmount)

 5k. Sink - SummaryYellowTaxiTripRecords dataset

 5l. Publish and run master pipeline for wanted timeframe

 **6. Connecting Power BI and creating a simple report**

 6a. Create a linked service for Power BI

 6b. Create a new Power BI dataset on SQLPool01 from Synapse Studio, and download the .pbids file provided. Open the file in Power BI Desktop.

 6c. Enter credentials, choose DirectQuery, save the file, and publish it to your wanted Power BI workspace.

 6d. Go to app.powerbi.com. Log in, go to the workspace you published the dataset, find the dataset, go to settings, update data source credentialsby using OAuth2 and use your account.

 6d. When successfully published, go back to Synapse Studio, and start developing your Power BI reports