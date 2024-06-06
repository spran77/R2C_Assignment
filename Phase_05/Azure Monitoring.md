1. **Azure Monitor**:

- This service provides a unified view of our application and infrastructure performance and health.

2. **Application Insights**:

- This is an application performance management service for web applications that provides real-time monitoring and analytics.

**Terraform configuration for Monitoring**

```hcl

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
