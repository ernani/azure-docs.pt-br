---
title: Início Rápido – Backup de VM do modelo do Resource Manager
description: Saiba como fazer backup de máquinas virtuais com o modelo do Azure Resource Manager
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 05/14/2019
ms.custom: mvc,subject-armqs
ms.openlocfilehash: d6fb73801f0f460daf2ed70f8dc88187e41ea887
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/29/2020
ms.locfileid: "81458838"
---
# <a name="back-up-a-virtual-machine-in-azure-with-resource-manager-template"></a>Fazer backup de uma máquina virtual no Azure com o modelo do Resource Manager

O [Backup do Azure](backup-overview.md) faz backup de aplicativos e computadores locais e VMs do Azure. Este artigo mostra como fazer backup de uma VM do Azure com o modelo do Resource Manager e Azure PowerShell. Este início rápido concentra-se no processo de implantação de um modelo do Resource Manager para criar um cofre dos Serviços de Recuperação. Para obter mais informações sobre como desenvolver modelos do Resource Manager, confira [documentação do Resource Manager](/azure/azure-resource-manager/) e a [referência de modelo](/azure/templates/microsoft.recoveryservices/allversions).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Como alternativa, é possível fazer backup de uma VM usando o [Azure PowerShell](./quick-backup-vm-powershell.md), a [CLI do Azure](quick-backup-vm-cli.md) ou no [portal do Azure](quick-backup-vm-portal.md).

## <a name="create-a-vm-and-recovery-services-vault"></a>Criar uma VM e o cofre dos Serviços de Recuperação

Um [cofre dos Serviços de Recuperação](backup-azure-recovery-services-vault-overview.md) é um contêiner lógico que armazena dados de backup para recursos protegidos, como VMs do Azure. Quando um trabalho de backup executa, ele cria um ponto de recuperação dentro do cofre dos Serviços de Recuperação. Você pode usar um desses pontos de recuperação para restaurar dados para um determinado ponto no tempo.

### <a name="review-the-template"></a>Examinar o modelo

O modelo usado neste início rápido é proveniente dos [modelos de Início Rápido do Azure](https://azure.microsoft.com/resources/templates/101-recovery-services-create-vm-and-configure-backup/). Este modelo permite que você implante a VM do Windows simples e o cofre dos Serviços de Recuperação configurados com o DefaultPolicy for Protection.

:::code language="json" source="~/quickstart-templates/101-recovery-services-create-vm-and-configure-backup/azuredeploy.json" range="1-247" highlight="221-245":::

Os recursos definidos no modelo são:

- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Compute/virutalMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft.RecoveryServices/vaults**](/azure/templates/microsoft.recoveryservices/2016-06-01/vaults)
- [**Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems**](/azure/templates/microsoft.recoveryservices/2016-06-01/vaults/backupfabrics/protectioncontainers/protecteditems)

### <a name="deploy-the-template"></a>Implantar o modelo

Para implantar o modelo, selecione **Experimentar** para abrir o Azure Cloud Shell e, em seguida, cole o script do PowerShell a seguir na janela do shell. Para colar o código, clique com o botão direito do mouse na janela do shell e, em seguida, selecione **Colar**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name (limited to eight characters) that is used to generate Azure resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$adminUsername = Read-Host -Prompt "Enter the administrator username for the virtual machine"
$adminPassword = Read-Host -Prompt "Enter the administrator password for the virtual machine" -AsSecureString
$dnsPrefix = Read-Host -Prompt "Enter the unique DNS Name for the Public IP used to access the virtual machine"

$resourceGroupName = "${projectName}rg"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-recovery-services-create-vm-and-configure-backup/azuredeploy.json"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -projectName $projectName -adminUsername $adminUsername -adminPassword $adminPassword -dnsLabelPrefix $dnsPrefix
```

Neste início rápido, o Azure PowerShell é usado para implantar o modelo do Resource Manager. O [Portal do Azure](../azure-resource-manager/templates/deploy-portal.md), a [CLI do Azure](../azure-resource-manager/templates/deploy-cli.md) e a [API Rest](../azure-resource-manager/templates/deploy-rest.md) também podem ser usados para implantar modelos.

## <a name="validate-the-deployment"></a>Validar a implantação

### <a name="start-a-backup-job"></a>Iniciar um trabalho de backup

O modelo cria uma VM e ativa a VM novamente. Após implantar o modelo, você precisa iniciar um trabalho de backup. Para saber mais, confira [Iniciar um trabalho de backup](./quick-backup-vm-powershell.md#start-a-backup-job).

### <a name="monitor-the-backup-job"></a>Monitorar o trabalho de backup

Para monitorar o trabalho de backup, confira [Monitorar o trabalho de backup](./quick-backup-vm-powershell.md#monitor-the-backup-job).

## <a name="clean-up-the-deployment"></a>Limpar a implantação

Se você não precisar mais fazer backup da VM, poderá limpá-la.

- Se você quiser experimentar a restauração da VM, ignore a limpeza.
- Se você usou uma VM existente, poderá ignorar o cmdlet [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) final para deixar o grupo de recursos e a VM implementados.

Desabilite a proteção, remova os pontos de restauração e o cofre. Em seguida, exclua o grupo de recursos e os recursos de VM associados, da seguinte maneira:

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints
$vault = Get-AzRecoveryServicesVault -Name "myRecoveryServicesVault"
Remove-AzRecoveryServicesVault -Vault $vault
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Próximas etapas

Neste início rápido, você criou um cofre dos Serviços de Recuperação, habilitou a proteção em uma VM e criou o ponto de recuperação inicial.

- [Saiba como](tutorial-backup-vm-at-scale.md) fazer backup de VMs no portal do Azure.
- [Saiba como](tutorial-restore-disk.md) restaurar rapidamente uma VM
- [Saiba como](../azure-resource-manager/templates/template-tutorial-create-first-template.md) criar modelos do Azure Resource Manager.
