All these resourses are can be deployed in azure portal using GUI and can be configured manually.

1. **SQL Database**:

- Service: Azure SQL Database
- Purpose: Store application data
- Configuration: Single Database or Elastic Pool depending on the load and usage patterns

2. **Key Vault**:

- Service: Azure Key Vault
- Purpose: Store sensitive information like connection strings, API keys, and certificates
- Configuration: Enable logging and access policies for applications and developers

3. **Webapp Services**:

- Service: Azure App Service
- Purpose: Host the web application
- Configuration: Use App Service Plan to scale the app and deploy multiple instances for high availability

4. **Integration with External Services**:

- Services: Azure API Management, Logic Apps, or Azure Functions
- Purpose: Manage and secure APIs, orchestrate workflows, or run serverless functions to integrate with external services
- Configuration: Configure API endpoints, security, and monitoring

**These resources can also be deployed using terraform and the services can be configured**

The below is the example terraform code

```hcl
# provider.tf
provider "azurerm" {
  features {}
}

# sql_database.tf
resource "azurerm_resource_group" "main" {
  name     = "myResourceGroup"
  location = "West US"
}

resource "azurerm_sql_server" "main" {
  name                         = "mySqlServer"
  resource_group_name          = azurerm_resource_group.main.name
  location                     = azurerm_resource_group.main.location
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = "P@ssword1234"
}

resource "azurerm_sql_database" "main" {
  name                = "myDatabase"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  server_name         = azurerm_sql_server.main.name
  edition             = "Standard"
  requested_service_objective_name = "S0"
}

# key_vault.tf
resource "azurerm_key_vault" "main" {
  name                        = "myKeyVault"
  resource_group_name         = azurerm_resource_group.main.name
  location                    = azurerm_resource_group.main.location
  sku_name                    = "standard"
  tenant_id                   = "YOUR_TENANT_ID"
  soft_delete_enabled         = true
  purge_protection_enabled    = true
}

resource "azurerm_key_vault_secret" "sql_connection_string" {
  name         = "SqlConnectionString"
  value        = "Server=tcp:${azurerm_sql_server.main.name}.database.windows.net,1433;Initial Catalog=${azurerm_sql_database.main.name};Persist Security Info=False;User ID=sqladmin;Password=P@ssword1234;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
  key_vault_id = azurerm_key_vault.main.id
}

# app_service.tf
resource "azurerm_app_service_plan" "main" {
  name                = "myAppServicePlan"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku {
    tier = "Standard"
    size = "S1"
  }
}

resource "azurerm_app_service" "main" {
  name                = "myAppService"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  app_service_plan_id = azurerm_app_service_plan.main.id

  app_settings = {
    "WEBSITE_RUN_FROM_PACKAGE" = "1"
    "ConnectionStrings:DefaultConnection" = "@Microsoft.KeyVault(SecretUri=https://${azurerm_key_vault.main.vault_uri}/secrets/SqlConnectionString)"
  }
}

# api_management.tf
resource "azurerm_api_management" "main" {
  name                = "myApiManagement"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  publisher_name      = "My Company"
  publisher_email     = "email@mycompany.com"
  sku_name            = "Developer_1"
}

resource "azurerm_api_management_api" "example" {
  name                = "example-api"
  resource_group_name = azurerm_resource_group.main.name
  api_management_name = azurerm_api_management.main.name
  revision            = "1"
  display_name        = "Example API"
  path                = "example"
  protocols           = ["https"]
}

```
