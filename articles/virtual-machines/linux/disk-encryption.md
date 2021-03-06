---
title: Criptografia do lado do servidor do Azure Managed Disks-CLI do Azure
description: O armazenamento do Azure protege seus dados criptografando-os em repouso antes de mantê-los para clusters de armazenamento. Você pode contar com chaves gerenciadas pela Microsoft para a criptografia de seus discos gerenciados ou pode usar chaves gerenciadas pelo cliente para gerenciar a criptografia com suas próprias chaves.
author: roygara
ms.date: 04/21/2020
ms.topic: conceptual
ms.author: rogarana
ms.service: virtual-machines-linux
ms.subservice: disks
ms.openlocfilehash: a9266f20d276b0b8d315cc83cd96fc235c79e773
ms.sourcegitcommit: a8ee9717531050115916dfe427f84bd531a92341
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/12/2020
ms.locfileid: "83198949"
---
# <a name="server-side-encryption-of-azure-managed-disks"></a>Criptografia do lado do servidor de Azure Managed disks

O Azure Managed disks criptografa automaticamente os dados por padrão ao mantê-los para a nuvem. A criptografia do lado do servidor protege seus dados e ajuda a atender aos compromissos de segurança e conformidade da organização. Os dados nos Managed disks do Azure são criptografados de forma transparente usando a [criptografia AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)de 256 bits, uma das codificações de bloco mais fortes disponíveis e é compatível com o FIPS 140-2.   

A criptografia não afeta o desempenho dos discos gerenciados. Não há nenhum custo adicional para a criptografia.

Para obter mais informações sobre os módulos de criptografia subjacentes ao Azure Managed disks, consulte [API de criptografia: próxima geração](https://docs.microsoft.com/windows/desktop/seccng/cng-portal)

## <a name="about-encryption-key-management"></a>Sobre o gerenciamento de chaves de criptografia

Você pode contar com chaves gerenciadas por plataforma para a criptografia do seu disco gerenciado ou pode gerenciar a criptografia usando suas próprias chaves. Se você optar por gerenciar a criptografia com suas próprias chaves, poderá especificar uma *chave gerenciada pelo cliente* a ser usada para criptografar e descriptografar todos os dados em discos gerenciados. 

As seções a seguir descrevem cada uma das opções de gerenciamento de chaves mais detalhadamente.

## <a name="platform-managed-keys"></a>Chaves gerenciadas pela plataforma

Por padrão, os discos gerenciados usam chaves de criptografia gerenciadas pela plataforma. A partir de 10 de junho de 2017, todos os novos discos gerenciados, instantâneos, imagens e novos dados gravados em discos gerenciados existentes são automaticamente criptografados em repouso com chaves gerenciadas pela plataforma.

## <a name="customer-managed-keys"></a>Chaves gerenciadas pelo cliente

Você pode optar por gerenciar a criptografia no nível de cada disco gerenciado com suas próprias chaves. A criptografia do lado do servidor para discos gerenciados com chaves gerenciadas pelo cliente oferece uma experiência integrada com o Azure Key Vault. Você pode importar [suas chaves RSA](../../key-vault/keys/hsm-protected-keys.md) para seu Key Vault ou gerar novas chaves rsa no Azure Key Vault. 

O Azure Managed disks lida com a criptografia e a descriptografia de uma maneira totalmente transparente usando a [criptografia de envelope](../../storage/common/storage-client-side-encryption.md#encryption-and-decryption-via-the-envelope-technique). Ele criptografa dados usando uma DEK (chave de criptografia de dados) baseada em [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256, que é, por sua vez, protegida usando suas chaves. O serviço de armazenamento gera chaves de criptografia de dados e as criptografa com chaves gerenciadas pelo cliente usando a criptografia RSA. A criptografia de envelope permite que você gire (altere) suas chaves periodicamente de acordo com suas políticas de conformidade sem afetar suas VMs. Quando você gira suas chaves, o serviço de armazenamento criptografa novamente as chaves de criptografia de dados com as novas chaves gerenciadas pelo cliente. 

Você precisa conceder acesso a discos gerenciados em seu Key Vault para usar suas chaves para criptografar e descriptografar o DEK. Isso permite o controle total de seus dados e chaves. Você pode desabilitar suas chaves ou revogar o acesso a discos gerenciados a qualquer momento. Você também pode auditar o uso da chave de criptografia com monitoramento Azure Key Vault para garantir que apenas os discos gerenciados ou outros serviços confiáveis do Azure estejam acessando suas chaves.

Para SSDs Premium, SSDs padrão e HDDs padrão: quando você desabilita ou exclui sua chave, todas as VMs com discos que usam essa chave serão desligadas automaticamente. Depois disso, as VMs não serão utilizáveis, a menos que a chave seja habilitada novamente ou você atribua uma nova chave.

Para ultra discos, quando você desabilita ou exclui uma chave, qualquer VM com ultra disks usando a chave não será desligada automaticamente. Depois que você desalocar e reiniciar as VMs, os discos deixarão de usar a chave e, em seguida, as VMs não ficarão online novamente. Para colocar as VMs online novamente, você deve atribuir uma nova chave ou habilitar a chave existente.

O diagrama a seguir mostra como os discos gerenciados usam Azure Active Directory e Azure Key Vault para fazer solicitações usando a chave gerenciada pelo cliente:

![Fluxo de trabalho de chaves gerenciadas pelo cliente e disco gerenciado. Um administrador cria um Azure Key Vault e, em seguida, cria um conjunto de criptografia de disco e configura o conjunto de criptografia de disco. O conjunto está associado a uma VM, o que permite que o disco use o Azure AD para autenticar](media/disk-storage-encryption/customer-managed-keys-sse-managed-disks-workflow.png)


A lista a seguir explica o diagrama ainda mais detalhadamente:

1. Um administrador de Azure Key Vault cria recursos do Key Vault.
1. O administrador do cofre de chaves importa suas chaves RSA para Key Vault ou gerar novas chaves RSA no Key Vault.
1. Esse administrador cria uma instância do recurso de conjunto de criptografia de disco, especificando uma ID de Azure Key Vault e uma URL de chave. O conjunto de criptografia de disco é um novo recurso introduzido para simplificar o gerenciamento de chaves para discos gerenciados. 
1. Quando um conjunto de criptografia de disco é criado, uma [identidade gerenciada atribuída pelo sistema](../../active-directory/managed-identities-azure-resources/overview.md) é criada no Azure Active Directory (AD) e associada ao conjunto de criptografia de disco. 
1. O administrador do cofre de chaves do Azure concede a permissão de identidade gerenciada para executar operações no cofre de chaves.
1. Um usuário da VM cria discos associando-os ao conjunto de criptografia de disco. O usuário da VM também pode habilitar a criptografia do lado do servidor com chaves gerenciadas pelo cliente para recursos existentes associando-as ao conjunto de criptografia de disco. 
1. Os discos gerenciados usam a identidade gerenciada para enviar solicitações para o Azure Key Vault.
1. Para ler ou gravar dados, o Managed disks envia solicitações para Azure Key Vault criptografar (encapsular) e descriptografar (desencapsular) a chave de criptografia de dados para executar criptografia e descriptografia dos dados. 

Para revogar o acesso às chaves gerenciadas pelo cliente, consulte [Azure Key Vault PowerShell](https://docs.microsoft.com/powershell/module/azurerm.keyvault/) e [Azure Key Vault CLI](https://docs.microsoft.com/cli/azure/keyvault). Revogar o acesso efetivamente bloqueia o acesso a todos os dados na conta de armazenamento, pois a chave de criptografia é inacessível pelo armazenamento do Azure.

### <a name="supported-regions"></a>Regiões com suporte

[!INCLUDE [virtual-machines-disks-encryption-regions](../../../includes/virtual-machines-disks-encryption-regions.md)]

### <a name="restrictions"></a>Restrições

Por enquanto, as chaves gerenciadas pelo cliente têm as seguintes restrições:

- Se esse recurso estiver habilitado para o disco, você não poderá desabilitá-lo.
    Se você precisar solucionar esse erro, deverá [copiar todos os dados](disks-upload-vhd-to-managed-disk-cli.md#copy-a-managed-disk) para um disco gerenciado totalmente diferente que não esteja usando chaves gerenciadas pelo cliente.
- Somente [as chaves RSA "soft" e "Hard"](../../key-vault/keys/about-keys.md) do tamanho 2048 têm suporte, sem outras chaves ou tamanhos.
- Os discos criados a partir de imagens personalizadas que são criptografadas usando criptografia do lado do servidor e chaves gerenciadas pelo cliente devem ser criptografados usando as mesmas chaves gerenciadas pelo cliente e devem estar na mesma assinatura.
- Os instantâneos criados a partir de discos criptografados com criptografia do lado do servidor e chaves gerenciadas pelo cliente devem ser criptografados com as mesmas chaves gerenciadas pelo cliente.
- Todos os recursos relacionados às chaves gerenciadas pelo cliente (cofres de chaves do Azure, conjuntos de criptografia de disco, VMs, discos e instantâneos) devem estar na mesma assinatura e região.
- Discos, instantâneos e imagens criptografadas com chaves gerenciadas pelo cliente não podem passar para outra assinatura.
- Se você usar o portal do Azure para criar o conjunto de criptografia de disco, não poderá usar instantâneos por enquanto.
- Os discos gerenciados criptografados usando chaves gerenciadas pelo cliente também não podem ser criptografados com Azure Disk Encryption.
- Para obter informações sobre como usar chaves gerenciadas pelo cliente com galerias de imagens compartilhadas, consulte [Visualização: usar chaves gerenciadas pelo cliente para criptografar imagens](../image-version-encryption.md).

### <a name="cli"></a>CLI
#### <a name="setting-up-your-azure-key-vault-and-diskencryptionset"></a>Configurando seu Azure Key Vault e DiskEncryptionSet

1. Certifique-se de que você instalou a versão mais recente da [CLI do Azure](/cli/azure/install-az-cli2) e entrou em uma conta do Azure com [az login](/cli/azure/reference-index).

1. Crie uma instância de Azure Key Vault e a chave de criptografia.

    Ao criar a instância de Key Vault, você deve habilitar a exclusão reversível e a proteção de limpeza. A exclusão reversível garante que a Key Vault mantenha uma chave excluída para um determinado período de retenção (padrão de 90 dias). A proteção de limpeza garante que uma chave excluída não possa ser excluída permanentemente até que o período de retenção se espuser. Essas configurações protegem você contra a perda de dados devido à exclusão acidental. Essas configurações são obrigatórias ao usar um Key Vault para criptografar discos gerenciados.

    > [!IMPORTANT]
    > Não use o camel case da região, se você fizer isso, poderá enfrentar problemas ao atribuir discos adicionais ao recurso no portal do Azure.

    ```azurecli
    subscriptionId=yourSubscriptionID
    rgName=yourResourceGroupName
    location=westcentralus
    keyVaultName=yourKeyVaultName
    keyName=yourKeyName
    diskEncryptionSetName=yourDiskEncryptionSetName
    diskName=yourDiskName

    az account set --subscription $subscriptionId

    az keyvault create -n $keyVaultName -g $rgName -l $location --enable-purge-protection true --enable-soft-delete true

    az keyvault key create --vault-name $keyVaultName -n $keyName --protection software
    ```

1.    Crie uma instância de um DiskEncryptionSet. 
    
        ```azurecli
        keyVaultId=$(az keyvault show --name $keyVaultName --query [id] -o tsv)
    
        keyVaultKeyUrl=$(az keyvault key show --vault-name $keyVaultName --name $keyName --query [key.kid] -o tsv)
    
        az disk-encryption-set create -n $diskEncryptionSetName -l $location -g $rgName --source-vault $keyVaultId --key-url $keyVaultKeyUrl
        ```

1.    Conceda ao recurso DiskEncryptionSet acesso ao cofre de chaves. 

        > [!NOTE]
        > Pode levar alguns minutos para o Azure criar a identidade de seu DiskEncryptionSet em seu Azure Active Directory. Se você receber um erro como "não é possível localizar o objeto Active Directory" ao executar o comando a seguir, aguarde alguns minutos e tente novamente.

        ```azurecli
        desIdentity=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [identity.principalId] -o tsv)
    
        az keyvault set-policy -n $keyVaultName -g $rgName --object-id $desIdentity --key-permissions wrapkey unwrapkey get
    
        az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId
        ```

#### <a name="create-a-vm-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>Criar uma VM usando uma imagem do Marketplace, criptografando o sistema operacional e os discos de dados com chaves gerenciadas pelo cliente

```azurecli
rgName=yourResourceGroupName
vmName=yourVMName
location=westcentralus
vmSize=Standard_DS3_V2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskencryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az vm create -g $rgName -n $vmName -l $location --image $image --size $vmSize --generate-ssh-keys --os-disk-encryption-set $diskEncryptionSetId --data-disk-sizes-gb 128 128 --data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```


#### <a name="encrypt-existing-managed-disks"></a>Criptografar discos gerenciados existentes 

Os discos existentes não devem ser anexados a uma VM em execução para que você possa criptografá-los usando o seguinte script:

```azurecli
rgName=yourResourceGroupName
diskName=yourDiskName
diskEncryptionSetName=yourDiskEncryptionSetName
 
az disk update -n $diskName -g $rgName --encryption-type EncryptionAtRestWithCustomerKey --disk-encryption-set $diskEncryptionSetId
```

#### <a name="create-a-virtual-machine-scale-set-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>Criar um conjunto de dimensionamento de máquinas virtuais usando uma imagem do Marketplace, criptografando o sistema operacional e os discos de dados com chaves gerenciadas pelo cliente

```azurecli
rgName=yourResourceGroupName
vmssName=yourVMSSName
location=westcentralus
vmSize=Standard_DS3_V2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskencryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)
az vmss create -g $rgName -n $vmssName --image UbuntuLTS --upgrade-policy automatic --admin-username azureuser --generate-ssh-keys --os-disk-encryption-set $diskEncryptionSetId --data-disk-sizes-gb 64 128 --data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

#### <a name="create-an-empty-disk-encrypted-using-server-side-encryption-with-customer-managed-keys-and-attach-it-to-a-vm"></a>Criar um disco vazio criptografado usando a criptografia do lado do servidor com chaves gerenciadas pelo cliente e anexá-lo a uma VM

```azurecli
vmName=yourVMName
rgName=yourResourceGroupName
diskName=yourDiskName
diskSkuName=Premium_LRS
diskSizeinGiB=30
location=westcentralus
diskLUN=2
diskEncryptionSetName=yourDiskEncryptionSetName


diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az disk create -n $diskName -g $rgName -l $location --encryption-type EncryptionAtRestWithCustomerKey --disk-encryption-set $diskEncryptionSetId --size-gb $diskSizeinGiB --sku $diskSkuName

diskId=$(az disk show -n $diskName -g $rgName --query [id] -o tsv)

az vm disk attach --vm-name $vmName --lun $diskLUN --ids $diskId 

```

#### <a name="change-the-key-of-a-diskencryptionset-to-rotate-the-key-for-all-the-resources-referencing-the-diskencryptionset"></a>Altere a chave de um DiskEncryptionSet para girar a chave para todos os recursos que fazem referência a DiskEncryptionSet

```azurecli

rgName=yourResourceGroupName
keyVaultName=yourKeyVaultName
keyName=yourKeyName
diskEncryptionSetName=yourDiskEncryptionSetName


keyVaultId=$(az keyvault show --name $keyVaultName--query [id] -o tsv)

keyVaultKeyUrl=$(az keyvault key show --vault-name $keyVaultName --name $keyName --query [key.kid] -o tsv)

az disk-encryption-set update -n keyrotationdes -g keyrotationtesting --key-url $keyVaultKeyUrl --source-vault $keyVaultId

```

#### <a name="find-the-status-of-server-side-encryption-of-a-disk"></a>Localizar o status da criptografia do lado do servidor de um disco

```azurecli

az disk show -g yourResourceGroupName -n yourDiskName --query [encryption.type] -o tsv

```

> [!IMPORTANT]
> As chaves gerenciadas pelo cliente dependem de identidades gerenciadas para recursos do Azure, um recurso do Azure Active Directory (Azure AD). Quando você configura chaves gerenciadas pelo cliente, uma identidade gerenciada é automaticamente atribuída aos seus recursos nos bastidores. Se, subsequentemente, você mover a assinatura, o grupo de recursos ou o disco gerenciado de um diretório do Azure AD para outro, a identidade gerenciada associada a discos gerenciados não será transferida para o novo locatário, portanto, as chaves gerenciadas pelo cliente poderão deixar de funcionar. Para obter mais informações, consulte [transferindo uma assinatura entre diretórios do Azure ad](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

[!INCLUDE [virtual-machines-disks-encryption-portal](../../../includes/virtual-machines-disks-encryption-portal.md)]

> [!IMPORTANT]
> As chaves gerenciadas pelo cliente dependem de identidades gerenciadas para recursos do Azure, um recurso do Azure Active Directory (Azure AD). Quando você configura chaves gerenciadas pelo cliente, uma identidade gerenciada é automaticamente atribuída aos seus recursos nos bastidores. Se, subsequentemente, você mover a assinatura, o grupo de recursos ou o disco gerenciado de um diretório do Azure AD para outro, a identidade gerenciada associada a discos gerenciados não será transferida para o novo locatário, portanto, as chaves gerenciadas pelo cliente poderão deixar de funcionar. Para obter mais informações, consulte [transferindo uma assinatura entre diretórios do Azure ad](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

## <a name="server-side-encryption-versus-azure-disk-encryption"></a>Criptografia do lado do servidor versus criptografia de disco do Azure

[Azure Disk Encryption para máquinas virtuais e conjuntos de dimensionamento de máquinas virtuais](../../security/fundamentals/azure-disk-encryption-vms-vmss.md) aproveita o recurso [BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview) do Windows e o recurso [DM-cript](https://en.wikipedia.org/wiki/Dm-crypt) do Linux para criptografar discos gerenciados com chaves gerenciadas pelo cliente na VM convidada.  A criptografia do lado do servidor com chaves gerenciadas pelo cliente melhora o ADE, permitindo que você use qualquer tipo de sistema operacional e imagens para suas VMs criptografando os dados no serviço de armazenamento.

## <a name="next-steps"></a>Próximas etapas

- [Explore os modelos de Azure Resource Manager para criar discos criptografados com chaves gerenciadas pelo cliente](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [O que é o Azure Key Vault?](../../key-vault/general/overview.md)
- [Replicar máquinas com discos habilitados para chaves gerenciadas pelo cliente](../../site-recovery/azure-to-azure-how-to-enable-replication-cmk-disks.md)
- [Configurar a recuperação de desastre de VMs VMware para o Azure usando o PowerShell](../../site-recovery/vmware-azure-disaster-recovery-powershell.md#replicate-vmware-vms)
- [Configurar a recuperação de desastres para o Azure para máquinas virtuais do Hyper-V usando o PowerShell e o Azure Resource Manager](../../site-recovery/hyper-v-azure-powershell-resource-manager.md#step-7-enable-vm-protection)
