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

# monitoring.tf
resource "azurerm_log_analytics_workspace" "main" {
  name                = "myLogAnalyticsWorkspace"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                 = "PerGB2018"
}

resource "azurerm_application_insights" "main" {
  name                = "myAppInsights"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  application_type    = "web"
  workspace_id        = azurerm_log_analytics_workspace.main.id
}

resource "azurerm_monitor_diagnostic_setting" "app_service" {
  name               = "diagAppService"
  target_resource_id = azurerm_app_service.main.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.main.id

  log {
    category = "AppServiceHTTPLogs"
    enabled  = true
    retention_policy {
      enabled = true
      days    = 30
    }
  }

  metric {
    category = "AllMetrics"
    enabled  = true
    retention_policy {
      enabled = true
      days    = 30
    }
  }
}

resource "azurerm_monitor_diagnostic_setting" "sql_database" {
  name               = "diagSqlDatabase"
  target_resource_id = azurerm_sql_database.main.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.main.id

  log {
    category = "SQLInsights"
    enabled  = true
    retention_policy {
      enabled = true
      days    = 30
    }
  }

  metric {
    category = "AllMetrics"
    enabled  = true
    retention_policy {
      enabled = true
      days    = 30
    }
  }
}
```
