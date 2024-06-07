The service connection is explained on the below and also methods to create each of them manually

1. **Azure Key Vault Configuration**:

- Creating the Azure Key Vault

```sh

az keyvault create --name R2C-vault --resource-group R2C-Resource_grp --location US EAST

```

- Configuring Access Policies

  - Access policies >> Add Access Policy >> Secret Management >> Service principal used by Azure Pipelines (from the service connection)

- Access for Azure App Service

  - access policy >> Secret Management >> Identity of Azure App Service.

- Configuring Firewall and VNet
  - Key Vault >> Firewalls and virtual networks >> Enable "Allow trusted Microsoft services to bypass this firewall" >> Add VNet Rules

2. **Azure SQL Database Configuration**:

```sh
az sql server create --name r2cdev.database.windows.net --resource-group R2C-Resource_grp --location US EAST --admin-user Ranji*** --admin-password ********
az sql db create --resource-group R2C-Resource_grp --server r2cdev.database.windows.net --name R2CDatabase_1 --service-objective S0

```

- Adding Specific IP Ranges if we are not using a Vnet

```sh
az sql server firewall-rule create --resource-group R2C-Resource_grp --server r2cdev.database.windows.net --name AllowAzureIPs --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

```

- Enabling VNet Service Endpoint.
  > > Azure Portal >> SQL server >> Virtual networks >> VNet rule to allow access for VNet >> Subnet is configured

3. **Azure App Services Configuration**:

- Azure App Service.

```sh
az webapp create --name R2CApp --resource-group R2C-Resource_grp --plan ASP-R2CResourcegrp-86b1 (F1: 1)

```

- > > Identity >> VNet Integration >> "Networking", enable VNet integration .

4. **Azure API Management Configuration**:

- API Management Service

```sh
az apim create --name R2CAPI --resource-group R2C-Resource_grp --publisher-email sp.ranjith@xyz.com --publisher-name Ranjith

```

- Configuring the API
  - > > Add APIs >> Configure Policies >> Configure Networking >>

5. **Firewall Configuration Scripts**

- Azure Services

```sh

az sql server firewall-rule create --resource-group R2C-Resource_grp --server r2cdev.database.windows.net --name AllowAzureServices --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

```

- Specific IP Ranges

```sh
az sql server firewall-rule create --resource-group R2C-Resource_grp --server r2cdev.database.windows.net --name AllowPipelinesIP --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

```

Configured the below as well

- Azure Key Vault Firewall Rules
- VNet Rules
