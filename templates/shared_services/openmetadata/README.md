# OpenMetadata Shared Service Porter Bundle

## Overview

This Porter bundle deploys [OpenMetadata](https://open-metadata.org/), a comprehensive platform for data discovery, lineage, governance, and collaboration. It is designed to be deployed as a shared service within a TRE (Trusted Research Environment).

The bundle utilizes the official OpenMetadata Helm chart to deploy the application to a Kubernetes cluster. It requires external instances for a PostgreSQL (or MySQL) database and an Elasticsearch service.

## Prerequisites

1.  **Porter**: Ensure Porter CLI is installed and configured.
2.  **Kubernetes Cluster**: An existing Kubernetes cluster where OpenMetadata will be deployed. The `kubeconfig` for this cluster should be available in your environment or configured for Porter.
3.  **Database**: An external PostgreSQL or MySQL database accessible from the Kubernetes cluster. You will need the host, port, username, password, and database name.
4.  **Elasticsearch**: An external Elasticsearch instance accessible from the Kubernetes cluster. You will need the host, port, and connection scheme (http/https).
5.  **Azure Credentials**: The bundle requires Azure credentials (tenant ID, subscription ID, client ID, client secret) for interacting with Azure services, primarily for Terraform state management. These should be configured as Porter credentials.

## Bundle Structure

*   `porter.yaml`: The main Porter manifest.
*   `Dockerfile.tmpl`: Template for the invocation image.
*   `parameters.json`: Default values for certain parameters.
*   `template_schema.json`: JSON schema defining all configurable parameters, their types, descriptions, and defaults. This provides a comprehensive list of what can be configured.
*   `README.md`: This file.

## Parameters

This bundle requires several parameters for configuration. Key parameters include:

*   `tre_id`: The ID of the parent TRE instance.
*   `id`: A unique ID for this OpenMetadata service instance.
*   `tfstate_resource_group_name`: Azure resource group for Terraform state.
*   `tfstate_storage_account_name`: Azure storage account for Terraform state.
*   `openmetadata_kubernetes_namespace`: Kubernetes namespace for deployment (default: `openmetadata`).
*   `openmetadata_helm_chart_version`: Version of the OpenMetadata Helm chart (see `parameters.json` for default).
*   `database_host`, `database_port`, `database_user`, `database_password`, `database_name`, `database_type`: Configuration for the external database.
*   `elasticsearch_host`, `elasticsearch_port`, `elasticsearch_scheme`: Configuration for Elasticsearch.
*   `airflow_enabled`: Whether to deploy the bundled Airflow (default: `true`).

For a full list of parameters, their descriptions, types, and default values, please refer to the `template_schema.json` file.

## Credentials Required

The following Porter credentials must be configured:

*   `azure_tenant_id`
*   `azure_subscription_id`
*   `azure_client_id`
*   `azure_client_secret`

You can set these using `porter credentials edit azure_tenant_id` (and so on for each).

## Usage

### Install

To install the OpenMetadata bundle:

```bash
porter install <installation-name> --bundle templates/shared_services/openmetadata:latest --param-file path/to/your/params.json --cred azure
```

Replace `<installation-name>` with a name for this installation (e.g., `my-openmetadata`).
You will need a `params.json` file (or use individual `--param` flags) to provide values for the required parameters (see `template_schema.json` for required fields and create a JSON file with your values).

Example `my-params.json`:
```json
{
  "tre_id": "mytre-dev-1234",
  "id": "openmetadata-prod",
  "tfstate_resource_group_name": "rg-porter-tfstate",
  "tfstate_storage_account_name": "stportertfstate123",
  "database_host": "mydb.postgres.database.azure.com",
  "database_user": "om_user",
  "database_password": "YOUR_DB_PASSWORD",
  "elasticsearch_host": "myes.elasticsearch.azure.com"
}
```

### Upgrade

To upgrade an existing installation (e.g., to a new bundle version or with changed parameters):

```bash
porter upgrade <installation-name> --bundle templates/shared_services/openmetadata:latest --param-file path/to/your/params.json --cred azure
```

### Uninstall

To uninstall the OpenMetadata service:

```bash
porter uninstall <installation-name> --cred azure
```

## Outputs

After successful installation/upgrade, the bundle may provide the following outputs:

*   `openmetadata_ui_url`: The URL to access the OpenMetadata UI. (Note: The mechanism to retrieve this reliably from the Helm chart needs to be verified and implemented in `porter.yaml`).
*   `openmetadata_api_endpoint`: The API endpoint for OpenMetadata. (Similar to UI URL, retrieval needs verification).
*   `openmetadata_namespace`: The Kubernetes namespace where OpenMetadata is deployed.

These can be viewed using `porter installation output show <output-name> -i <installation-name>`.

## Important Notes

*   **Helm Chart Values**: The `porter.yaml` passes configuration to the OpenMetadata Helm chart. If you need to customize the deployment further (e.g., resource limits, advanced OpenMetadata server settings), you might need to modify the `values` section within the `helm3` mixin steps in `porter.yaml` or expose more parameters.
*   **External Dependencies**: This bundle assumes you have pre-provisioned and manage the lifecycle of the database and Elasticsearch instances.
*   **Terraform State**: The bundle is configured to use Azure Blob Storage for Terraform state, managed by Porter. Ensure the `tfstate_*` parameters are correctly set.
```
