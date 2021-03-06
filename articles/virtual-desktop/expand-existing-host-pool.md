---
title: Expandir o pool de hosts existente com novos hosts de sessão-Azure
description: Como expandir um pool de hosts existente com novos hosts de sessão na área de trabalho virtual do Windows.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: d46d5618d7e3dc26775401f4a90d0c98d75ea31a
ms.sourcegitcommit: a6d477eb3cb9faebb15ed1bf7334ed0611c72053
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/08/2020
ms.locfileid: "82929206"
---
# <a name="expand-an-existing-host-pool-with-new-session-hosts"></a>Expandir um pool de hosts existente com novos hosts de sessão

>[!IMPORTANT]
>Este conteúdo se aplica à atualização do Spring 2020 com Azure Resource Manager objetos da área de trabalho virtual do Windows. Se você estiver usando a área de trabalho virtual do Windows, a versão 2019 sem Azure Resource Manager objetos, consulte [Este artigo](./virtual-desktop-fall-2019/expand-existing-host-pool-2019.md).
>
> A atualização 2020 de área de trabalho virtual do Windows está em visualização pública no momento. Esta versão de visualização é fornecida sem um contrato de nível de serviço e não é recomendável usá-la para cargas de trabalho de produção. Alguns recursos podem não ter suporte ou podem ter restrição de recursos. 
> Para obter mais informações, consulte [Termos de Uso Complementares de Versões Prévias do Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Ao aumentar o uso em seu pool de hosts, talvez seja necessário expandir seu pool de hosts existente com novos hosts de sessão para lidar com a nova carga.

Este artigo lhe dirá como você pode expandir um pool de hosts existente com novos hosts de sessão.

## <a name="what-you-need-to-expand-the-host-pool"></a>O que você precisa para expandir o pool de hosts

Antes de começar, verifique se você criou um pool de hosts e VMs (máquinas virtuais) de host de sessão usando um dos seguintes métodos:

- [Azure portal](./create-host-pools-azure-marketplace.md)
- [Criar um pool de host com o PowerShell](./create-host-pools-powershell.md)

Você também precisará das seguintes informações ao criar pela primeira vez o pool de hosts e as VMs de host de sessão:

- Tamanho da VM, imagem e prefixo do nome
- Credenciais de administrador de ingresso no domínio
- Nome da rede virtual e nome da sub-rede

## <a name="add-virtual-machines-with-the-azure-portal"></a>Adicionar máquinas virtuais com o portal do Azure

Para expandir o pool de hosts adicionando máquinas virtuais:

1. Entre no portal do Azure.

2. Procure e selecione **área de trabalho virtual do Windows**.

3. No menu no lado esquerdo da tela, selecione **pools de hosts**e, em seguida, selecione o nome do pool de hosts ao qual você deseja adicionar máquinas virtuais.

4. Selecione **máquinas virtuais** no menu no lado esquerdo da tela.

5. Selecione **+ Adicionar** para começar a criar o pool de hosts.

6. Ignore a guia noções básicas e, em vez disso, selecione a guia **detalhes da VM** . Aqui você pode exibir e editar os detalhes da VM (máquina virtual) que deseja adicionar ao pool de hosts.

7. Selecione o grupo de recursos em que você deseja criar as VMs e, em seguida, selecione a região. Você pode escolher a região atual que está usando ou uma nova região.
   
8. Insira o número de hosts de sessão que você deseja adicionar ao seu pool de hosts em **número de VMs**. Por exemplo, se você estiver expandindo o pool de hosts em cinco hosts, insira **5**.
   
    >[!NOTE]
    >Você não pode editar o tamanho ou a imagem das VMs porque é importante garantir que todas as VMs no pool de hosts tenham o mesmo tamanho.
    
9. Para obter as **informações da rede virtual**, selecione a rede virtual e a sub-rede às quais você deseja que as máquinas virtuais sejam unidas. Você pode selecionar a mesma rede virtual que seus computadores existentes usam atualmente ou escolher outro que seja mais adequado para a região que você selecionou na etapa 7.

10. Para a **conta de administrador**, insira o nome de usuário e a senha de domínio Active Directory associados à rede virtual selecionada. Essas credenciais serão usadas para unir as máquinas virtuais à rede virtual.

      >[!NOTE]
      >Verifique se os nomes de seu administrador estão em conformidade com as informações fornecidas aqui. E que não há MFA habilitada na conta.

11. Selecione a guia **marca** se você tiver marcas nas quais deseja agrupar as máquinas virtuais. Caso contrário, ignore esta guia. 

12. Selecione a guia **revisar + criar** . revise suas escolhas e, se tudo estiver correto, selecione **criar**. 

## <a name="next-steps"></a>Próximas etapas

Agora que você expandiu o pool de hosts existente, você pode entrar em um cliente de área de trabalho virtual do Windows para testá-los como parte de uma sessão de usuário. Você pode se conectar a uma sessão com qualquer um dos seguintes clientes:

- [Conectar-se ao Cliente de Área de Trabalho do Windows](./connect-windows-7-and-10.md)
- [Conectar-se ao cliente Web](./connect-web.md)
- [Conectar-se ao cliente Android](./connect-android.md)
- [Conectar-se ao cliente macOS](./connect-macos.md)
- [Conectar-se ao cliente iOS](./connect-ios.md)
