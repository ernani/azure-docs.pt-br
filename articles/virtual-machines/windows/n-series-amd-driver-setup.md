---
title: Instalação do driver AMD GPU da série N do Azure para Windows
description: Como configurar drivers de GPU AMD para VMs da série N que executam o Windows Server ou Windows no Azure
author: vikancha
manager: jkabat
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 12/4/2019
ms.author: vikancha
ms.openlocfilehash: 1bcc13db3f503c80fda71a2104d0ff8d99e67df6
ms.sourcegitcommit: a8ee9717531050115916dfe427f84bd531a92341
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/12/2020
ms.locfileid: "83198006"
---
# <a name="install-amd-gpu-drivers-on-n-series-vms-running-windows"></a>Instalar drivers do AMD GPU em VMs da série N que executam o Windows

Para aproveitar os recursos de GPU das novas VMs da série Azure NVv4 que executam o Windows, os drivers de GPU AMD devem ser instalados. A [extensão de driver de GPU AMD](../extensions/hpccompute-amd-gpu-windows.md) instala drivers de GPU AMD em uma VM da série NVv4. Instale ou gerencie a extensão usando o portal do Azure ou ferramentas, como Azure PowerShell ou modelos do Azure Resource Manager. Consulte a [documentação de extensão do driver do AMD GPU](../extensions/hpccompute-amd-gpu-windows.md) para obter os sistemas operacionais e as etapas de implantação com suporte.

Se você optar por instalar manualmente os drivers do AMD GPU, este artigo fornecerá os sistemas operacionais, drivers e etapas de instalação e verificação com suporte.

Somente os drivers de GPU publicados pela Microsoft têm suporte em VMs NVv4. Não instale drivers de GPU de nenhuma outra fonte.

Para especificações básicas, capacidades de armazenamento e detalhes de disco, consulte [tamanhos de VM Windows da GPU](sizes-gpu.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).



## <a name="supported-operating-systems-and-drivers"></a>Sistemas operacionais e drivers com suporte

| Sistema operacional | Driver |
| -------- |------------- |
| Windows 10 EVD-Build 1903 <br/><br/>Windows 10-Build 1809<br/><br/>Windows Server 2016<br/><br/>Windows Server 2019 | [20. q 1.1](https://download.microsoft.com/download/3/8/9/3893407b-e8aa-4079-8592-735d7dd1c19a/Radeon-Pro-Software-for-Enterprise-GA.exe) (. exe) |


## <a name="driver-installation"></a>Instalação do driver

1. Conecte-se por Área de Trabalho Remota a cada VM da série NVv4.

2. Se você for um cliente do NVv4 Preview, interrompa a VM e aguarde até que ela se mova para o estado parado (desalocado).

3. Inicie a VM e baixe o utilitário de [limpeza AMD](https://download.microsoft.com/download/4/f/1/4f19b714-9304-410f-9c64-826404e07857/AMDCleanupUtilityni.exe)mais recente. Desinstale o driver existente executando "amdcleanuputility-x64. exe". Não use nenhum utilitário de limpeza existente instalado com o driver anterior.  

4. Baixe e instale o driver mais recente.

5. Reinicialize a VM.

## <a name="verify-driver-installation"></a>Verificar a instalação de drivers

É possível verificar a instalação de drivers no Gerenciador de Dispositivos. O exemplo a seguir mostra a configuração bem-sucedida da placa MI25 do Radeon instinto em uma VM NVv4 do Azure.
<br />
![Propriedades do driver GPU](./media/n-series-amd-driver-setup/device-manager.png)

Você pode usar o DxDiag para verificar as propriedades de exibição de GPU, incluindo a RAM de vídeo. O exemplo a seguir mostra uma partição 1/2 do cartão MI25 do Radeon instinto em uma VM NVv4 do Azure.
<br />
![Propriedades do driver GPU](./media/n-series-amd-driver-setup/dxdiag-output.png)

Se você estiver executando o Windows 10 Build 1903 ou superior, o DxDiag não mostrará nenhuma informação na guia ' display '. Use a opção ' salvar todas as informações ' na parte inferior e o arquivo de saída mostrará as informações relacionadas ao AMD MI25 GPU.

![Propriedades do driver GPU](./media/n-series-amd-driver-setup/dxdiag-details.png)


