---
title: Criar e criptografar um conjunto de dimensionamento de máquinas virtuais com modelos do Azure Resource Manager
description: Neste início rápido, você aprenderá a usar os modelos do Azure Resource Manager para criar e criptografar um conjunto de dimensionamento de máquinas virtuais
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 10/10/2019
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: c2b49a7e7e14bfbefcca64133ff23fdfabe53e7b
ms.sourcegitcommit: a8ee9717531050115916dfe427f84bd531a92341
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/12/2020
ms.locfileid: "83198434"
---
# <a name="encrypt-virtual-machine-scale-sets-with-azure-resource-manager"></a>Criptografar os Conjuntos de Dimensionamento de Máquinas Virtuais com o Azure Resource Manager

Criptografe ou descriptografe Conjuntos de Dimensionamento de Máquinas Virtuais do Linux usando modelos do Azure Resource Manager.

## <a name="deploying-templates"></a>Como implantar modelos

Primeiro, selecione o modelo que se adapta ao seu cenário.

- [Habilitar a criptografia de disco em um conjunto de dimensionamento de máquinas virtuais do Linux em execução](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-vmss-linux)

- [Habilitar a criptografia de disco em um conjunto de dimensionamento de máquinas virtuais do Windows em execução](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-vmss-windows)

  - [Implantar um conjunto de dimensionamento de máquinas virtuais de VMs do Linux com um jumpbox e habilitar a criptografia em Conjuntos de Dimensionamento de Máquinas Virtuais do Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-linux-jumpbox)

  - [Implantar um conjunto de dimensionamento de máquinas virtuais de VMs do Windows com um jumpbox e habilitar a criptografia em Conjuntos de Dimensionamento de Máquinas Virtuais do Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-windows-jumpbox)

- [Desabilitar a criptografia de disco em um conjunto de dimensionamento de máquinas virtuais do Linux em execução](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-vmss-linux)

- [Desabilitar a criptografia de disco em um conjunto de dimensionamento de máquinas virtuais do Windows em execução](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-vmss-windows)

Depois, siga estas etapas:

     1. Click **Deploy to Azure**.
     2. Fill in the required fields then agree to the terms and conditions.
     3. Click **Purchase** to deploy the template.

## <a name="next-steps"></a>Próximas etapas

- [Azure Disk Encryption para Conjuntos de Dimensionamento de Máquinas Virtuais](disk-encryption-overview.md)
- [Criptografar um conjunto de dimensionamento de máquinas virtuais usando a CLI do Azure](disk-encryption-cli.md)
- [Criptografar um conjunto de dimensionamento de máquinas virtuais usando o Azure PowerShell](disk-encryption-powershell.md)
- [Criar e configurar um cofre de chaves para o Azure Disk Encryption](disk-encryption-key-vault.md)
- [Usar o Azure Disk Encryption com o sequenciamento de extensão do conjunto de dimensionamento de máquinas virtuais](disk-encryption-extension-sequencing.md)
