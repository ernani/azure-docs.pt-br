---
title: 'Sincronização do Azure AD Connect: alteração da configuração padrão | Microsoft Docs'
description: Fornece práticas recomendadas para alterar a configuração padrão da sincronização do Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 7638a031-1635-4942-94c3-fce8f09eed5e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/29/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 940a35d89996b1eb9600fe4214863d2b5304750e
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "60242156"
---
# <a name="azure-ad-connect-sync-best-practices-for-changing-the-default-configuration"></a>Sincronização do Azure AD Connect: práticas recomendadas para alterar a configuração padrão
O objetivo deste tópico é descrever as alterações com e sem suporte para a sincronização do Azure AD Connect.

A configuração criada pelo Azure AD Connect funciona "da forma como é" para a maioria dos ambientes que sincronizam o Active Directory local com o AD do Azure. No entanto, em alguns casos, é necessário aplicar alterações a uma configuração para atender a uma necessidade ou requisito específico.

## <a name="changes-to-the-service-account"></a>Alterações na conta de serviço
A sincronização do Azure AD Connect é executada em uma conta de serviço criada pelo assistente de instalação. Essa conta de serviço contém as chaves de criptografia para o banco de dados usado pela sincronização. Ele é criado com uma senha de 127 caracteres de comprimento e a senha é definida como não expirar.

* **Não há suporte** para alteração ou redefinição da senha da conta de serviço. Isso destruirá as chaves de criptografia e o serviço não poderá acessar o banco de dados e iniciar.

## <a name="changes-to-the-scheduler"></a>Alterações no agendador
A partir da versão 1.1 (fevereiro de 2016), você pode configurar o [agendador](how-to-connect-sync-feature-scheduler.md) para ter um ciclo de sincronização diferente do padrão de 30 minutos.

## <a name="changes-to-synchronization-rules"></a>Alterações nas regras de sincronização
O assistente de instalação fornece uma configuração que deve funcionar nos cenários mais comuns. No caso de precisar fazer alterações na configuração, você deve seguir estas regras para continuar com uma configuração com suporte.

> [!WARNING]
> Se você fizer alterações nas regras de sincronização padrão, essas alterações serão substituídas na próxima vez em que o Azure AD Connect for atualizado, ocasionando resultados inesperados e provável sincronização indesejada.

* Você poderá [alterar fluxos de atributos](how-to-connect-sync-change-the-configuration.md#other-common-attribute-flow-changes) se os fluxos de atributos diretos padrão não forem adequados para sua organização.
* Se você optar por [não fluir um atributo](how-to-connect-sync-change-the-configuration.md#do-not-flow-an-attribute) e remover valores do atributo existentes no Azure AD, você precisará criar uma regra para este cenário.
* [Desabilite uma Regra de sincronização indesejada](#disable-an-unwanted-sync-rule) em vez de excluí-la. Uma regra excluída é recriada durante uma atualização.
* Para [alterar uma regra integrada](#change-an-out-of-box-rule), você deverá fazer uma cópia da regra original e desabilitar a regra integrada. O Editor de Regra de Sincronização solicita e ajuda você.
* Exporte suas regras de sincronização personalizadas usando o Editor de regras de sincronização. O editor fornece um script do PowerShell que você pode usar para recriá-los facilmente em um cenário de recuperação de desastres.

> [!WARNING]
> As regras de sincronização integrada têm uma impressão digital. Se você fizer uma alteração a essas regras, a impressão digital não corresponderá mais. Você pode ter problemas no futuro, quando tentar aplicar uma nova versão do Azure AD Connect. Faça alterações somente como é descrito neste artigo.

### <a name="disable-an-unwanted-sync-rule"></a>Desabilite uma Regra de sincronização indesejada
Não exclua uma regra de sincronização integrada. Ela será recriada durante a próxima atualização.

Em alguns casos, o assistente de instalação produz uma configuração que não funciona para sua topologia. Por exemplo, se houver uma topologia de floresta de recurso de conta, mas você estendeu o esquema na floresta de conta com o esquema do Exchange, as regras do Exchange serão criadas para a floresta de contas e para a floresta de recursos. Nesse caso, você deverá desabilitar a Regra de Sincronização do Exchange.

![Regra de sincronização desabilitada](./media/how-to-connect-sync-best-practices-changing-default-configuration/exchangedisabledrule.png)

Na figura acima, o assistente de instalação encontrou um esquema antigo do Exchange 2003 na floresta de contas. Esta extensão de esquema foi adicionada antes da floresta de recursos ter sido introduzida no ambiente da Fabrikam. Para garantir que nenhum atributo da implementação do Exchange antigo seja sincronizado, a regra de sincronização deve ser desabilitada conforme mostrado.

### <a name="change-an-out-of-box-rule"></a>alterar uma regra integrada
A única vez em que você deve alterar uma regra pronta para uso é quando você precisa alterar a regra de associação. Se você precisar alterar um fluxo de atributos, deverá criar uma regra de sincronização com precedência maior do que as regras prontas para uso. A única regra é praticamente necessário clone é a regra **em do AD – usuário Join**. Você pode substituir todas as outras regras com uma regra de prioridade mais alta.

Se você precisar fazer alterações em uma regra integrada, deverá fazer uma cópia da regra integrada e desabilitar a regra original. Em seguida, faça as alterações desejadas à regra clonada. O Editor de Regra de Sincronização ajuda você com essas etapas. Quando você abre uma regra integrada, vê essa caixa de diálogo:   
![Aviso de regra integrada](./media/how-to-connect-sync-best-practices-changing-default-configuration/warningoutofboxrule.png)

Selecione **Sim** para criar uma cópia da regra. Então, a regra clonada é aberta.  
![Regra clonada](./media/how-to-connect-sync-best-practices-changing-default-configuration/clonedrule.png)

Na regra clonada, faça as alterações necessárias no escopo, na associação e nas transformações.

## <a name="next-steps"></a>Próximas etapas
**Tópicos de visão geral**

* [Sincronização do Azure AD Connect: compreender e personalizar a sincronização](how-to-connect-sync-whatis.md)
* [Integrando suas identidades locais ao Azure Active Directory](whatis-hybrid-identity.md)
