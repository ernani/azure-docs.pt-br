---
title: Adicionar ou remover atribuições de função do Azure usando o CLI do Azure-RBAC do Azure
description: Saiba como conceder acesso aos recursos do Azure para usuários, grupos, entidades de serviço ou identidades gerenciadas usando o CLI do Azure e o Azure RBAC (controle de acesso baseado em função) do Azure.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 3483ee01-8177-49e7-b337-4d5cb14f5e32
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/25/2019
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 3a66482aeee7832baa91fe98357b870e2a280912
ms.sourcegitcommit: 4499035f03e7a8fb40f5cff616eb01753b986278
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/03/2020
ms.locfileid: "82735769"
---
# <a name="add-or-remove-azure-role-assignments-using-azure-cli"></a>Adicionar ou remover atribuições de função do Azure usando CLI do Azure

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control-definition-grant.md)]Este artigo descreve como atribuir funções usando CLI do Azure.

## <a name="prerequisites"></a>Pré-requisitos

Para adicionar ou remover atribuições de função, você deve ter:

- as permissões `Microsoft.Authorization/roleAssignments/write` e `Microsoft.Authorization/roleAssignments/delete`, como [Administrador de Acesso do Usuário](built-in-roles.md#user-access-administrator) ou [Proprietário](built-in-roles.md#owner)
- [Bash em Azure cloud Shell](/azure/cloud-shell/overview) ou [CLI do Azure](/cli/azure)

## <a name="get-object-ids"></a>Obter IDs de objeto

Para adicionar ou remover atribuições de função, talvez seja necessário especificar a ID exclusiva de um objeto. A ID tem o formato: `11111111-1111-1111-1111-111111111111`. Você pode obter a ID usando o portal do Azure ou CLI do Azure.

### <a name="user"></a>Usuário

Para obter a ID de objeto para um usuário do Azure AD, você pode usar [AZ ad User show](/cli/azure/ad/user#az-ad-user-show).

```azurecli
az ad user show --id "{email}" --query objectId --output tsv
```

### <a name="group"></a>Agrupar

Para obter a ID de objeto para um grupo do Azure AD, você pode usar [AZ ad Group show](/cli/azure/ad/group#az-ad-group-show) ou [AZ ad Group List](/cli/azure/ad/group#az-ad-group-list).

```azurecli
az ad group show --group "{name}" --query objectId --output tsv
```

### <a name="application"></a>Aplicativo

Para obter a ID de objeto para uma entidade de serviço do Azure AD (identidade usada por um aplicativo), você pode usar [AZ ad SP List](/cli/azure/ad/sp#az-ad-sp-list). Para uma entidade de serviço, use a ID de objeto e **não** a ID do aplicativo.

```azurecli
az ad sp list --display-name "{name}" --query [].objectId --output tsv
```

## <a name="add-a-role-assignment"></a>Adicionar uma atribuição de função

No RBAC do Azure, para conceder acesso, você adiciona uma atribuição de função.

### <a name="user-at-a-resource-group-scope"></a>Usuário em um escopo de grupo de recursos

Para adicionar uma atribuição de função para um usuário em um escopo de grupo de recursos, use [AZ role Assignment Create](/cli/azure/role/assignment#az-role-assignment-create).

```azurecli
az role assignment create --role <role_name_or_id> --assignee <assignee> --resource-group <resource_group>
```

O exemplo a seguir atribui a função *colaborador da máquina virtual* ao *usuário\@patlong contoso.com* no escopo do grupo de recursos *Pharma-Sales* :

```azurecli
az role assignment create --role "Virtual Machine Contributor" --assignee patlong@contoso.com --resource-group pharma-sales
```

### <a name="using-the-unique-role-id"></a>Usando a ID de função exclusiva

Há algumas ocasiões em que um nome de função pode ser alterado, por exemplo:

- Você está usando sua própria função personalizada e decide alterar o nome.
- Você está usando uma função de visualização que tem **(visualização)** no nome. Quando a função é liberada, a função é renomeada.

> [!IMPORTANT]
> Uma versão de visualização é fornecida sem um contrato de nível de serviço e não é recomendada para cargas de trabalho de produção. Alguns recursos podem não ter suporte ou podem ter restrição de recursos.
> Para obter mais informações, consulte [Termos de Uso Complementares de Versões Prévias do Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Mesmo que uma função seja renomeada, a ID da função não será alterada. Se você estiver usando scripts ou automação para criar atribuições de função, é uma prática recomendada usar a ID de função exclusiva em vez do nome da função. Portanto, se uma função for renomeada, os scripts provavelmente funcionarão.

Para adicionar uma atribuição de função usando a ID de função exclusiva em vez do nome da função, use [AZ role Assignment Create](/cli/azure/role/assignment#az-role-assignment-create).

```azurecli
az role assignment create --role <role_id> --assignee <assignee> --resource-group <resource_group>
```

O exemplo a seguir atribui a função [colaborador da máquina virtual](built-in-roles.md#virtual-machine-contributor) ao usuário *patlong\@contoso.com* no escopo do grupo de recursos *Pharma-Sales* . Para obter a ID de função exclusiva, você pode usar a [lista de definição de função AZ](/cli/azure/role/definition#az-role-definition-list) ou ver [funções internas do Azure](built-in-roles.md).

```azurecli
az role assignment create --role 9980e02c-c2be-4d73-94e8-173b1dc7cf3c --assignee patlong@contoso.com --resource-group pharma-sales
```

### <a name="group-at-a-subscription-scope"></a>Agrupar em um escopo de assinatura

Para adicionar uma atribuição de função a um grupo, use [criar atribuição de função AZ](/cli/azure/role/assignment#az-role-assignment-create). Para obter informações sobre como obter a ID de objeto do grupo, consulte [obter IDs de objeto](#get-object-ids).

```azurecli
az role assignment create --role <role_name_or_id> --assignee-object-id <assignee_object_id> --resource-group <resource_group> --scope </subscriptions/subscription_id>
```

O exemplo a seguir atribui a função *leitor* ao grupo de *equipe Ana Mack* com ID 22222222-2222-2222-2222-222222222222 em um escopo de assinatura.

```azurecli
az role assignment create --role Reader --assignee-object-id 22222222-2222-2222-2222-222222222222 --scope /subscriptions/00000000-0000-0000-0000-000000000000
```

### <a name="group-at-a-resource-scope"></a>Agrupar em um escopo de recurso

Para adicionar uma atribuição de função a um grupo, use [criar atribuição de função AZ](/cli/azure/role/assignment#az-role-assignment-create). Para obter informações sobre como obter a ID de objeto do grupo, consulte [obter IDs de objeto](#get-object-ids).

O exemplo a seguir atribui a função *colaborador de máquina virtual* ao grupo de *equipe Ana Mack* com ID 22222222-2222-2222-2222-222222222222 em um escopo de recurso para uma rede virtual denominada *Pharma-Sales-Project-Network*.

```azurecli
az role assignment create --role "Virtual Machine Contributor" --assignee-object-id 22222222-2222-2222-2222-222222222222 --scope /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/pharma-sales/providers/Microsoft.Network/virtualNetworks/pharma-sales-project-network
```

### <a name="application-at-a-resource-group-scope"></a>Aplicativo em um escopo de grupo de recursos

Para adicionar uma atribuição de função para um aplicativo, use [AZ role Assignment Create](/cli/azure/role/assignment#az-role-assignment-create). Para obter informações sobre como obter a ID de objeto do aplicativo, consulte [obter IDs de objeto](#get-object-ids).

```azurecli
az role assignment create --role <role_name_or_id> --assignee-object-id <assignee_object_id> --resource-group <resource_group>
```

O exemplo a seguir atribui a função *colaborador da máquina virtual* a um aplicativo com a ID de objeto 44444444-4444-4444-4444-444444444444 no escopo do grupo de recursos *Pharma-Sales* .

```azurecli
az role assignment create --role "Virtual Machine Contributor" --assignee-object-id 44444444-4444-4444-4444-444444444444 --resource-group pharma-sales
```

### <a name="user-at-a-subscription-scope"></a>Usuário em um escopo de assinatura

Para adicionar uma atribuição de função para um usuário em um escopo de assinatura, use [AZ role Assignment Create](/cli/azure/role/assignment#az-role-assignment-create). Para obter a ID da assinatura, você pode encontrá-la na folha **assinaturas** no portal do Azure ou pode usar a lista de [contas AZ](/cli/azure/account#az-account-list).

```azurecli
az role assignment create --role <role_name_or_id> --assignee <assignee> --subscription <subscription_name_or_id>
```

O exemplo a seguir atribui a função *leitor* ao usuário *annm\@example.com* em um escopo de assinatura.

```azurecli
az role assignment create --role "Reader" --assignee annm@example.com --subscription 00000000-0000-0000-0000-000000000000
```

### <a name="user-at-a-management-group-scope"></a>Usuário em um escopo do grupo de gerenciamento

Para adicionar uma atribuição de função para um usuário em um escopo de grupo de gerenciamento, use [AZ role Assignment Create](/cli/azure/role/assignment#az-role-assignment-create). Para obter a ID do grupo de gerenciamento, você pode encontrá-la na folha **grupos de gerenciamento** no portal do Azure ou pode usar [AZ Account Management-grupo List](/cli/azure/account/management-group#az-account-management-group-list).

```azurecli
az role assignment create --role <role_name_or_id> --assignee <assignee> --scope /providers/Microsoft.Management/managementGroups/<group_id>
```

O exemplo a seguir atribui a função de *leitor de cobrança* ao usuário *Alain\@example.com* em um escopo de grupo de gerenciamento.

```azurecli
az role assignment create --role "Billing Reader" --assignee alain@example.com --scope /providers/Microsoft.Management/managementGroups/marketing-group
```

### <a name="new-service-principal"></a>Nova entidade de serviço

Se você criar uma nova entidade de serviço e tentar atribuir imediatamente uma função a essa entidade de serviço, essa atribuição de função poderá falhar em alguns casos. Por exemplo, se você usar um script para criar uma nova identidade gerenciada e, em seguida, tentar atribuir uma função a essa entidade de serviço, a atribuição de função poderá falhar. O motivo dessa falha é provavelmente um atraso de replicação. A entidade de serviço é criada em uma região; no entanto, a atribuição de função pode ocorrer em uma região diferente que ainda não tenha replicado a entidade de serviço. Para resolver esse cenário, você deve especificar o tipo de entidade de segurança ao criar a atribuição de função.

Para adicionar uma atribuição de função, use [AZ role Assignment Create](/cli/azure/role/assignment#az-role-assignment-create), especifique um valor `--assignee-object-id`para e, em `--assignee-principal-type` seguida `ServicePrincipal`, defina como.

```azurecli
az role assignment create --role <role_name_or_id> --assignee-object-id <assignee_object_id> --assignee-principal-type <assignee_principal_type> --resource-group <resource_group> --scope </subscriptions/subscription_id>
```

O exemplo a seguir atribui a função *colaborador de máquina virtual* à identidade gerenciada de *teste de MSI* no escopo do grupo de recursos *Pharma-Sales* :

```azurecli
az role assignment create --role "Virtual Machine Contributor" --assignee-object-id 33333333-3333-3333-3333-333333333333 --assignee-principal-type ServicePrincipal --resource-group pharma-sales
```

## <a name="remove-a-role-assignment"></a>Excluir uma atribuição de função

No RBAC do Azure, para remover o acesso, você remove uma atribuição de função usando [AZ role Assignment Delete](/cli/azure/role/assignment#az-role-assignment-delete):

```azurecli
az role assignment delete --assignee <assignee> --role <role_name_or_id> --resource-group <resource_group>
```

O exemplo a seguir remove a atribuição de função de *colaborador de máquina virtual* do usuário *\@patlong contoso.com* no grupo de recursos *Pharma-Sales* :

```azurecli
az role assignment delete --assignee patlong@contoso.com --role "Virtual Machine Contributor" --resource-group pharma-sales
```

O exemplo a seguir remove a função *leitor* do grupo de *equipe Ana Mack* com ID 22222222-2222-2222-2222-222222222222 em um escopo de assinatura. Para obter informações sobre como obter a ID de objeto do grupo, consulte [obter IDs de objeto](#get-object-ids).

```azurecli
az role assignment delete --assignee 22222222-2222-2222-2222-222222222222 --role "Reader" --subscription 00000000-0000-0000-0000-000000000000
```

O exemplo a seguir remove a função de *leitor de cobrança* do usuário *Alain\@example.com* no escopo do grupo de gerenciamento. Para obter a ID do grupo de gerenciamento, você pode usar [AZ Account Management – Group List](/cli/azure/account/management-group#az-account-management-group-list).

```azurecli
az role assignment delete --assignee alain@example.com --role "Billing Reader" --scope /providers/Microsoft.Management/managementGroups/marketing-group
```

## <a name="next-steps"></a>Próximas etapas

- [Listar atribuições de função do Azure usando CLI do Azure](role-assignments-list-cli.md)
- [Use a CLI do Azure para gerenciar recursos e grupos de recursos do Azure](../azure-resource-manager/cli-azure-resource-manager.md)
