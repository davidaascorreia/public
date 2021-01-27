Step-by-step: End to end analytics in Synapse

1. Environment setup

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

2. Synapse setup

