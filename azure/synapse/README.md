Step-by-step: End to end analytics in Synapse

1. Environment setup

1a: Create a new resource group <br />
1b: Create Azure Synapse Analytics workspace <br />
    - Set ADLS as account (Basics) <br />
    - Set default as file system name (Basics) <br />
    - Set password for SQL Admin (Security) <br />
    - Don't allow connections from all IP addresses (Networking) <br />
1c: Create a storage account with hierarchical namespace (ADLS) <br />
    - Create containers for data (raw, curated, dev, staging, prod) <br />
1d: Create Azure Key Vault <br />
    - Give Synapse Workspace GET access permission in access policies <br />
1e: Create an SQL Pool (SQLPool01) <br />
1f: Change networking on Synapse Workspace in "Firewalls"-tab to allow for Azure Services and your own IP <br />
1g: Ensure you have Owner role on the Synapse Workspace <br />
1h: Create secret for ADLS access key (Key1) and for the password of the SQL Admin <br />

2. Configuring initial Synapse environment

2a: Set up Git configuration <br />
2b: Create linked service for NYCTLC trip data <br />
    - Base url: https://s3.amazonaws.com/nyc-tlc/ <br />
2c: Set up Azure Key Vault linked servicekv <br />
2d: Set up adls dataset <br />
    - Parameters: *directory* and *file* <br />
2e: Set up nyctlc dataset <br />
    - Parameters: *relativeUrl* <br />
2f: Group datasets in folder "Parameterized" <br />

3. Setting up some initial extract pipelines

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
3k. Get lookup data by creating another separate pipeline with relativeUrl = misc/taxi+_zone_lookup.csv, directory = nyctlc/misc, and file = taxi+_zone_lookup.csv. Include the pipeline in folder *99 Utilities*.
3l. Publish and trigger for the wanted periods.


