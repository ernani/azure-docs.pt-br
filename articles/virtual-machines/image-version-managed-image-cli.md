---
title: Migrar de uma imagem gerenciada para uma versão de imagem com o CLI do Azure
description: Saiba como migrar de uma imagem gerenciada para uma versão de imagem em uma galeria de imagens compartilhada usando o CLI do Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 6f49ece874ea52227e6531193fc53b3bea525702
ms.sourcegitcommit: e0330ef620103256d39ca1426f09dd5bb39cd075
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/05/2020
ms.locfileid: "82796597"
---
# <a name="migrate-from-a-managed-image-to-an-image-version-using-the-azure-cli"></a>Migrar de uma imagem gerenciada para uma versão de imagem usando o CLI do Azure
Se você tiver uma imagem gerenciada existente que deseja migrar para uma galeria de imagens compartilhada, poderá criar uma imagem da Galeria de imagens compartilhada diretamente da imagem gerenciada. Depois de testar a nova imagem, você pode excluir a imagem gerenciada de origem. Você também pode migrar de uma imagem gerenciada para uma galeria de imagens compartilhada usando o [PowerShell](image-version-managed-image-powershell.md).

As imagens em uma galeria de imagens têm dois componentes, que serão criados neste exemplo:
- Uma **definição de imagem** contém informações sobre a imagem e os requisitos para usá-la. Isso inclui se a imagem é Windows ou Linux, especializada ou generalizada, notas de versão e requisitos mínimos e máximos de memória. É uma definição de um tipo de imagem. 
- Uma **versão de imagem** é usada para criar uma VM ao usar uma galeria de imagens compartilhada. Você pode ter diversas versões de uma imagem conforme necessário para seu ambiente. Quando você cria uma VM, a versão da imagem é usada para criar novos discos para a VM. Versões de imagem podem ser usadas várias vezes.


## <a name="before-you-begin"></a>Antes de começar

Para concluir este artigo, você deve ter uma [Galeria de imagens compartilhada](shared-images-cli.md)existente. 

Para concluir o exemplo neste artigo, você precisa ter uma imagem gerenciada existente de uma VM generalizada. Para obter mais informações, consulte [capturar uma imagem gerenciada](./linux/capture-image.md). Se a imagem gerenciada contiver um disco de dados, o tamanho do disco de dados não poderá ser superior a 1 TB.

Ao trabalhar com este artigo, substitua o grupo de recursos e os nomes de VM quando for necessário.



## <a name="create-an-image-definition"></a>Criar uma definição de imagem

Como as imagens gerenciadas são sempre imagens generalizadas, você criará uma definição de `--os-state generalized` imagem usando para uma imagem generalizada.

Os nomes de definição de imagem podem ser compostos de letras maiúsculas ou minúsculas, dígitos, pontos, traços e pontos. 

Para obter mais informações sobre os valores que você pode especificar para uma definição de imagem, consulte [definições de imagem](https://docs.microsoft.com/azure/virtual-machines/linux/shared-image-galleries#image-definitions).

Crie uma definição de imagem na Galeria usando [AZ SIG Image-Definition Create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

Neste exemplo, a definição de imagem é chamada *myImageDefinition*e é para uma imagem [generalizada](./linux/shared-image-galleries.md#generalized-and-specialized-images) do SO Linux. Para criar uma definição para imagens usando um sistema operacional Windows, `--os-type Windows`use. 

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
imageDef=myImageDefinition
az sig image-definition create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state generalized
```


## <a name="create-the-image-version"></a>Criar a versão da imagem

Crie versões usando [AZ Image Gallery Create-Image-Version](/cli/azure/sig/image-version#az-sig-image-version-create). Você precisará passar a ID da imagem gerenciada para usar como uma linha de base para criar a versão da imagem. Você pode usar [AZ Image List](/cli/azure/image?view#az-image-list) para obter as IDs para suas imagens. 

```azurecli-interactive
az image list --query "[].[name, id]" -o tsv
```

Caracteres permitidos para a versão da imagem são números e pontos. Os números devem estar dentro do intervalo de um inteiro de 32 bits. Formato: *MajorVersion*. *MinorVersion*. *Patch*.

Neste exemplo, a versão da nossa imagem é a *1.0.0* e vamos criar uma réplica na região *do Sul EUA Central* e uma réplica na região *leste dos EUA 2* usando o armazenamento com redundância de zona. Ao escolher regiões de destino para replicação, lembre-se de que você também precisa incluir a região de *origem* como um destino para replicação.

Passe a ID da imagem gerenciada no `--managed-image` parâmetro.


```azurecli-interactive 
imageID="/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
az sig image-version create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus2=1=standard_zrs" \
   --replica-count 2 \
   --managed-image $imageID
```

> [!NOTE]
> Você precisa aguardar que a versão da imagem termine completamente de ser compilada e replicada antes de poder usar a mesma imagem gerenciada para criar outra versão de imagem.
>
> Você também pode armazenar todas as réplicas de versão da imagem no [armazenamento com redundância](https://docs.microsoft.com/azure/storage/common/storage-redundancy-zrs) de `--storage-account-type standard_zrs` zona adicionando ao criar a versão da imagem.
>

## <a name="next-steps"></a>Próximas etapas

Crie uma VM com base em uma [versão de imagem generalizada](vm-generalized-image-version-cli.md).