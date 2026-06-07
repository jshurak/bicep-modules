# Storage Account

Core storage account for shared blob data and artifacts. Wraps the [Azure Verified Module (AVM)](https://github.com/Azure/bicep-registry-modules/tree/main/avm/res/storage/storage-account) for `Microsoft.Storage/storageAccounts`.

## Usage

Reference `main.bicep` from your deployment template:

```bicep
module storageAccount 'br/path/to/storage-account/main.bicep' = {
  name: 'storageAccountDeployment'
  params: {
    namePrefix: 'sa'
    storageSku: 'Standard_LRS'
    containerNames: [
      'deployments'
    ]
    roleAssignments: [
      {
        principalId: '<principal-object-id>'
        principalType: 'ServicePrincipal'
        roleDefinitionIdOrName: 'Storage Blob Data Contributor'
      }
    ]
  }
}

output storageAccountName string = storageAccount.outputs.resStorageName
output storageAccountId string = storageAccount.outputs.resStorageID
output blobContainerUrl string = storageAccount.outputs.blobContainerURL
```

When consuming from this repository locally, point the module path at this directory (for example, `'../modules/storage/storage-account/main.bicep'`). When publishing to a registry, use the appropriate `br/` reference for your registry and version.

## Parameters

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `namePrefix` | `string` | `'sa'` | Prefix used in the storage account name when `storageAccountName` is not set. |
| `storageAccountName` | `string` | `''` | Explicit storage account name. When empty, the name is generated as `{namePrefix}{uniqueString(resourceGroup().id)}`. |
| `storageSku` | `string` | — | Azure Storage replication SKU (`Standard_LRS` or `Standard_ZRS`). **Required.** |
| `storageKind` | `string` | `'StorageV2'` | Storage account kind. |
| `blobPublicAccess` | `bool` | `false` | Whether anonymous public read access is allowed for blobs. |
| `containerNames` | `string[]` | `[]` | Blob container names to create in the storage account. |
| `roleAssignments` | `array` | `[]` | RBAC assignments on the storage account. Each item uses `principalId`, `principalType`, and `roleDefinitionIdOrName`. |

## Outputs

| Name | Description |
| --- | --- |
| `resStorageName` | Name of the deployed storage account. |
| `resStorageID` | Full ARM resource ID of the deployed storage account. |
| `blobContainerURL` | Primary blob endpoint, or the URL of the first container when `containerNames` is set. |

## Notes

- Public network access is enabled with network ACLs defaulting to `Allow` and `AzureServices` bypass.
- Shared key access is enabled (`allowSharedKeyAccess: true`).
- Blob services and containers are only configured when `containerNames` is non-empty.
