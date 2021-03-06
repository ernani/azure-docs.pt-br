---
title: Autorizar o acesso a dados com uma identidade gerenciada
titleSuffix: Azure Storage
description: Saiba como usar identidades gerenciadas para recursos do Azure para autorizar o acesso a dados de BLOB e de fila de aplicativos em execução em máquinas virtuais do Azure, aplicativos de funções, conjuntos de dimensionamento de máquinas virtuais e outros.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: f3bac0d47a53da1ec4d1fa08b5f0933f5f65dc56
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "79255334"
---
# <a name="authorize-access-to-blob-and-queue-data-with-managed-identities-for-azure-resources"></a>Autorizar o acesso a dados de BLOB e de fila com identidades gerenciadas para recursos do Azure

Os armazenamentos de blobs e de filas do Azure dão suporte à autenticação do Azure AD (Active Directory) com [identidades gerenciadas para recursos do Azure](../../active-directory/managed-identities-azure-resources/overview.md). Identidades gerenciadas para recursos do Azure podem autorizar o acesso a dados de BLOB e de fila usando as credenciais do Azure AD de aplicativos em execução em VMs (máquinas virtuais) do Azure, aplicativos de funções, conjuntos de dimensionamento de máquinas virtuais e outros serviços. Usando identidades gerenciadas para recursos do Azure junto com a autenticação do Azure AD, você pode evitar o armazenamento de credenciais com seus aplicativos que são executados na nuvem.  

Este artigo mostra como autorizar o acesso a dados de BLOB ou de fila de uma VM do Azure usando identidades gerenciadas para recursos do Azure. Ele também descreve como testar seu código no ambiente de desenvolvimento.

## <a name="enable-managed-identities-on-a-vm"></a>Habilitar identidades gerenciadas em uma VM

Antes de poder usar identidades gerenciadas para recursos do Azure para autorizar o acesso a BLOBs e filas de sua VM, você deve primeiro habilitar identidades gerenciadas para recursos do Azure na VM. Para saber como habilitar identidades gerenciadas para Recursos do Azure, confira um dos seguintes artigos:

- [Azure portal](https://docs.microsoft.com/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm)
- [PowerShell do Azure](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [CLI do Azure](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Modelo do Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Bibliotecas de cliente Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

Para obter mais informações sobre identidades gerenciadas, consulte [identidades gerenciadas para recursos do Azure](../../active-directory/managed-identities-azure-resources/overview.md).

## <a name="authenticate-with-the-azure-identity-library"></a>Autenticar com a biblioteca de identidades do Azure

A biblioteca de cliente de identidade do Azure fornece suporte à autenticação de token do Azure AD para o [SDK do Azure](https://github.com/Azure/azure-sdk). As versões mais recentes das bibliotecas de cliente de armazenamento do Azure para .NET, Java, Python e JavaScript se integram com a biblioteca de identidade do Azure para fornecer um meio simples e seguro de adquirir um token 2,0 do OAuth para autorização de solicitações de armazenamento do Azure.

Uma vantagem da biblioteca de cliente de identidade do Azure é que ela permite que você use o mesmo código para autenticar se seu aplicativo está em execução no ambiente de desenvolvimento ou no Azure. A biblioteca de cliente de identidade do Azure para .NET autentica uma entidade de segurança. Quando seu código está em execução no Azure, a entidade de segurança é uma identidade gerenciada para recursos do Azure. No ambiente de desenvolvimento, a identidade gerenciada não existe, portanto, a biblioteca de cliente autentica o usuário ou uma entidade de serviço para fins de teste.

Após a autenticação, a biblioteca de cliente de identidade do Azure Obtém uma credencial de token. Essa credencial de token é encapsulada no objeto de cliente de serviço que você cria para executar operações no armazenamento do Azure. A biblioteca lida com isso para você com perfeição obtendo a credencial de token apropriada.

Para obter mais informações sobre a biblioteca de cliente de identidade do Azure para .NET, consulte [biblioteca de cliente de identidade do Azure para .net](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity). Para obter a documentação de referência para a biblioteca de cliente de identidade do Azure, consulte [namespace do Azure. Identity](/dotnet/api/azure.identity).

### <a name="assign-role-based-access-control-rbac-roles-for-access-to-data"></a>Atribuir funções RBAC (controle de acesso baseado em função) para acesso aos dados

Quando uma entidade de segurança do Azure AD tenta acessar dados de BLOB ou fila, essa entidade de segurança deve ter permissões para o recurso. Se a entidade de segurança é uma identidade gerenciada no Azure ou uma conta de usuário do Azure AD executando código no ambiente de desenvolvimento, a entidade de segurança deve ser atribuída a uma função de RBAC que concede acesso a dados de BLOB ou de fila no armazenamento do Azure. Para obter informações sobre a atribuição de permissões via RBAC, consulte a seção intitulada **atribuir funções RBAC para direitos de acesso** em [autorizar o acesso a BLOBs e filas do Azure usando o Azure Active Directory](../common/storage-auth-aad.md#assign-rbac-roles-for-access-rights).

### <a name="authenticate-the-user-in-the-development-environment"></a>Autenticar o usuário no ambiente de desenvolvimento

Quando seu código está em execução no ambiente de desenvolvimento, a autenticação pode ser manipulada automaticamente ou pode exigir um logon do navegador, dependendo de quais ferramentas você está usando. Por exemplo, Microsoft Visual Studio dá suporte ao SSO (logon único), para que a conta de usuário ativa do Azure AD seja usada automaticamente para autenticação. Para obter mais informações sobre o SSO, consulte [logon único para aplicativos](../../active-directory/manage-apps/what-is-single-sign-on.md).

Outras ferramentas de desenvolvimento podem solicitar que você faça logon por meio de um navegador da Web.

### <a name="authenticate-a-service-principal-in-the-development-environment"></a>Autenticar uma entidade de serviço no ambiente de desenvolvimento

Se seu ambiente de desenvolvimento não oferecer suporte a logon único ou logon por meio de um navegador da Web, você poderá usar uma entidade de serviço para autenticar a partir do ambiente de desenvolvimento.

#### <a name="create-the-service-principal"></a>Criar a entidade de serviço

Para criar uma entidade de serviço com CLI do Azure e atribuir uma função de RBAC, chame o comando [AZ ad SP Create-for-RBAC](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) . Forneça uma função de acesso de dados do armazenamento do Azure para atribuir à nova entidade de serviço. Além disso, forneça o escopo para a atribuição de função. Para obter mais informações sobre as funções internas fornecidas para o armazenamento do Azure, consulte [funções internas para recursos do Azure](../../role-based-access-control/built-in-roles.md).

Se você não tiver permissões suficientes para atribuir uma função à entidade de serviço, talvez seja necessário solicitar ao proprietário da conta ou ao administrador para executar a atribuição de função.

O exemplo a seguir usa o CLI do Azure para criar uma nova entidade de serviço e atribuir a função de **leitor de dados de blob de armazenamento** a ela com o escopo da conta

```azurecli-interactive
az ad sp create-for-rbac \
    --name <service-principal> \
    --role "Storage Blob Data Reader" \
    --scopes /subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

O `az ad sp create-for-rbac` comando retorna uma lista de propriedades de entidade de serviço no formato JSON. Copie esses valores para que você possa usá-los para criar as variáveis de ambiente necessárias na próxima etapa.

```json
{
    "appId": "generated-app-ID",
    "displayName": "service-principal-name",
    "name": "http://service-principal-uri",
    "password": "generated-password",
    "tenant": "tenant-ID"
}
```

> [!IMPORTANT]
> As atribuições de função do RBAC podem levar alguns minutos para serem propagadas.

#### <a name="set-environment-variables"></a>Definir variáveis de ambiente

A biblioteca de cliente de identidade do Azure lê valores de três variáveis de ambiente em tempo de execução para autenticar a entidade de serviço. A tabela a seguir descreve o valor a ser definido para cada variável de ambiente.

|Variável de ambiente|Valor
|-|-
|`AZURE_CLIENT_ID`|A ID do aplicativo para a entidade de serviço
|`AZURE_TENANT_ID`|A ID de locatário do Azure AD da entidade de serviço
|`AZURE_CLIENT_SECRET`|A senha gerada para a entidade de serviço

> [!IMPORTANT]
> Depois de definir as variáveis de ambiente, feche e abra novamente a janela do console. Se você estiver usando o Visual Studio ou outro ambiente de desenvolvimento, talvez seja necessário reiniciar o ambiente de desenvolvimento para que ele registre as novas variáveis de ambiente.

Para obter mais informações, consulte [criar identidade para o aplicativo do Azure no portal](../../active-directory/develop/howto-create-service-principal-portal.md).

[!INCLUDE [storage-install-packages-blob-and-identity-include](../../../includes/storage-install-packages-blob-and-identity-include.md)]

## <a name="net-code-example-create-a-block-blob"></a>Exemplo de código .NET: crie um blob de blocos

Adicione as seguintes `using` diretivas ao seu código para usar as bibliotecas de cliente de armazenamento do Azure e identidade do Azure.

```csharp
using Azure;
using Azure.Identity;
using Azure.Storage.Blobs;
using System;
using System.IO;
using System.Text;
using System.Threading.Tasks;
```

Para obter uma credencial de token que seu código pode usar para autorizar solicitações para o armazenamento do Azure, crie uma instância da classe [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) . O exemplo de código a seguir mostra como obter a credencial de token autenticado e usá-la para criar um objeto de cliente de serviço e, em seguida, usar o cliente de serviço para carregar um novo blob:

```csharp
async static Task CreateBlockBlobAsync(string accountName, string containerName, string blobName)
{
    // Construct the blob container endpoint from the arguments.
    string containerEndpoint = string.Format("https://{0}.blob.core.windows.net/{1}",
                                                accountName,
                                                containerName);

    // Get a credential and create a client object for the blob container.
    BlobContainerClient containerClient = new BlobContainerClient(new Uri(containerEndpoint),
                                                                    new DefaultAzureCredential());

    try
    {
        // Create the container if it does not exist.
        await containerClient.CreateIfNotExistsAsync();

        // Upload text to a new block blob.
        string blobContents = "This is a block blob.";
        byte[] byteArray = Encoding.ASCII.GetBytes(blobContents);

        using (MemoryStream stream = new MemoryStream(byteArray))
        {
            await containerClient.UploadBlobAsync(blobName, stream);
        }
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

> [!NOTE]
> Para autorizar solicitações em dados de BLOB ou de fila com o Azure AD, você deve usar HTTPS para essas solicitações.

## <a name="next-steps"></a>Próximas etapas

- [Gerenciar direitos de acesso aos dados de armazenamento com o RBAC](storage-auth-aad-rbac.md).
- [Use o Azure AD com aplicativos de armazenamento](storage-auth-aad-app.md).
- [Execute comandos do CLI do Azure ou do PowerShell com as credenciais do Azure ad para acessar dados de BLOB ou fila](authorize-active-directory-powershell.md).
