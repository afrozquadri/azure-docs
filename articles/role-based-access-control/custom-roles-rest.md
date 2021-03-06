---
title: Create custom roles for Azure resources using the REST API - Azure | Microsoft Docs
description: Learn how to create custom roles with role-based access control (RBAC) for Azure resources using the REST API. This includes how to list, create, update, and delete custom roles.
services: active-directory
documentationcenter: na
author: rolyon
manager: mtillman
editor: ''

ms.assetid: 1f90228a-7aac-4ea7-ad82-b57d222ab128
ms.service: role-based-access-control
ms.workload: multiple
ms.tgt_pltfrm: rest-api
ms.devlang: na
ms.topic: conceptual
ms.date: 02/20/2019
ms.author: rolyon
ms.reviewer: bagovind

---
# Create custom roles for Azure resources using the REST API

If the [built-in roles for Azure resources](built-in-roles.md) don't meet the specific needs of your organization, you can create your own custom roles. This article describes how to create and manage custom roles using the REST API.

## List roles

To list all roles or get information about a single role using its display name, use the [Role Definitions - List](/rest/api/authorization/roledefinitions/list) REST API. To call this API, you must have access to the `Microsoft.Authorization/roleDefinitions/read` operation at the scope. Several [built-in roles](built-in-roles.md) are granted access to this operation.

1. Start with the following request:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter={filter}
    ```

1. Within the URI, replace *{scope}* with the scope for which you want to list the roles.

    | Scope | Type |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | Subscription |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Resource group |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | Resource |

1. Replace *{filter}* with the condition that you want to apply to filter the role list.

    | Filter | Description |
    | --- | --- |
    | `$filter=atScopeAndBelow()` | List roles available for assignment at the specified scope and any of its child scopes. |
    | `$filter=roleName%20eq%20'{roleDisplayName}'` | Use the URL encoded form of the exact display name of the role. For instance, `$filter=roleName%20eq%20'Virtual%20Machine%20Contributor'` |

### Get information about a role

To get information about a role using its role definition identifier, use the [Role Definitions - Get](/rest/api/authorization/roledefinitions/get) REST API. To call this API, you must have access to the `Microsoft.Authorization/roleDefinitions/read` operation at the scope. Several [built-in roles](built-in-roles.md) are granted access to this operation.

To get information about a single role using its display name, see previous [List roles](custom-roles-rest.md#list-roles) section.

1. Use the [Role Definitions - List](/rest/api/authorization/roledefinitions/list) REST API to get the GUID identifier for the role. For built-in roles, you can also get the identifier from [Built-in roles](built-in-roles.md).

1. Start with the following request:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. Within the URI, replace *{scope}* with the scope for which you want to list the roles.

    | Scope | Type |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | Subscription |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Resource group |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | Resource |

1. Replace *{roleDefinitionId}* with the GUID identifier of the role definition.

## Create a custom role

To create a custom role, use the [Role Definitions - Create Or Update](/rest/api/authorization/roledefinitions/createorupdate) REST API. To call this API, you must have access to the `Microsoft.Authorization/roleDefinitions/write` operation on all the `assignableScopes`. Of the built-in roles, only [Owner](built-in-roles.md#owner) and [User Access Administrator](built-in-roles.md#user-access-administrator) are granted access to this operation. 

1. Review the list of [resource provider operations](resource-provider-operations.md) that are available to create the permissions for your custom role.

1. Use a GUID tool to generate a unique identifier that will be used for the custom role identifier. The identifier has the format: `00000000-0000-0000-0000-000000000000`

1. Start with the following request and body:

    ```http
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

    ```json
    {
      "name": "{roleDefinitionId}",
      "properties": {
        "roleName": "",
        "description": "",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
    
            ],
            "notActions": [
    
            ]
          }
        ],
        "assignableScopes": [
          "/subscriptions/{subscriptionId}"
        ]
      }
    }
    ```

1. Within the URI, replace *{scope}* with the first `assignableScopes` of the custom role.

    | Scope | Type |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | Subscription |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Resource group |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | Resource |

1. Replace *{roleDefinitionId}* with the GUID identifier of the custom role.

1. Within the request body, in the `assignableScopes` property, replace *{roleDefinitionId}* with the GUID identifier.

1. Replace *{subscriptionId}* with your subscription identifier.

1. In the `actions` property, add the operations that the role allows to be performed.

1. In the `notActions` property, add the operations that are excluded from the allowed `actions`.

1. In the `roleName` and `description` properties, specify a unique role name and a description. For more information about the properties, see [Custom roles](custom-roles.md).

    The following shows an example of a request body:

    ```json
    {
      "name": "88888888-8888-8888-8888-888888888888",
      "properties": {
        "roleName": "Virtual Machine Operator",
        "description": "Can monitor and restart virtual machines.",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
              "Microsoft.Storage/*/read",
              "Microsoft.Network/*/read",
              "Microsoft.Compute/*/read",
              "Microsoft.Compute/virtualMachines/start/action",
              "Microsoft.Compute/virtualMachines/restart/action",
              "Microsoft.Authorization/*/read",
              "Microsoft.ResourceHealth/availabilityStatuses/read",
              "Microsoft.Resources/subscriptions/resourceGroups/read",
              "Microsoft.Insights/alertRules/*",
              "Microsoft.Support/*"
            ],
            "notActions": []
          }
        ],
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000"
        ]
      }
    }
    ```

## Update a custom role

To update a custom role, use the [Role Definitions - Create Or Update](/rest/api/authorization/roledefinitions/createorupdate) REST API. To call this API, you must have access to the `Microsoft.Authorization/roleDefinitions/write` operation on all the `assignableScopes`. Of the built-in roles, only [Owner](built-in-roles.md#owner) and [User Access Administrator](built-in-roles.md#user-access-administrator) are granted access to this operation. 

1. Use the [Role Definitions - List](/rest/api/authorization/roledefinitions/list) or [Role Definitions - Get](/rest/api/authorization/roledefinitions/get) REST API to get information about the custom role. For more information, see the earlier [List roles](custom-roles-rest.md#list-roles) section.

1. Start with the following request:

    ```http
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. Within the URI, replace *{scope}* with the first `assignableScopes` of the custom role.

    | Scope | Type |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | Subscription |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Resource group |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | Resource |

1. Replace *{roleDefinitionId}* with the GUID identifier of the custom role.

1. Based on the information about the custom role, create a request body with the following format:

    ```json
    {
      "name": "{roleDefinitionId}",
      "properties": {
        "roleName": "",
        "description": "",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
    
            ],
            "notActions": [
    
            ]
          }
        ],
        "assignableScopes": [
          "/subscriptions/{subscriptionId}"
        ]
      }
    }
    ```

1. Update the request body with the changes you want to make to the custom role.

    The following shows an example of a request body with a new diagnostic settings action added:

    ```json
    {
      "name": "88888888-8888-8888-8888-888888888888",
      "properties": {
        "roleName": "Virtual Machine Operator",
        "description": "Can monitor and restart virtual machines.",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
              "Microsoft.Storage/*/read",
              "Microsoft.Network/*/read",
              "Microsoft.Compute/*/read",
              "Microsoft.Compute/virtualMachines/start/action",
              "Microsoft.Compute/virtualMachines/restart/action",
              "Microsoft.Authorization/*/read",
              "Microsoft.ResourceHealth/availabilityStatuses/read",
              "Microsoft.Resources/subscriptions/resourceGroups/read",
              "Microsoft.Insights/alertRules/*",
              "Microsoft.Insights/diagnosticSettings/*",
              "Microsoft.Support/*"
            ],
            "notActions": []
          }
        ],
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000"
        ]
      }
    }
    ```

## Delete a custom role

To delete a custom role, use the [Role Definitions - Delete](/rest/api/authorization/roledefinitions/delete) REST API. To call this API, you must have access to the `Microsoft.Authorization/roleDefinitions/delete` operation on all the `assignableScopes`. Of the built-in roles, only [Owner](built-in-roles.md#owner) and [User Access Administrator](built-in-roles.md#user-access-administrator) are granted access to this operation. 

1. Use the [Role Definitions - List](/rest/api/authorization/roledefinitions/list) or [Role Definitions - Get](/rest/api/authorization/roledefinitions/get) REST API to get the GUID identifier of the custom role. For more information, see the earlier [List roles](custom-roles-rest.md#list-roles) section.

1. Start with the following request:

    ```http
    DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. Within the URI, replace *{scope}* with the scope that you want to delete the custom role.

    | Scope | Type |
    | --- | --- |
    | `subscriptions/{subscriptionId}` | Subscription |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Resource group |
    | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/ providers/Microsoft.Web/sites/mysite1` | Resource |

1. Replace *{roleDefinitionId}* with the GUID identifier of the custom role.

## Next steps

- [Custom roles for Azure resources](custom-roles.md)
- [Manage access to Azure resources using RBAC and the REST API](role-assignments-rest.md)
- [Azure REST API Reference](/rest/api/azure/)