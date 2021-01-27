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

2. Synapse setup

