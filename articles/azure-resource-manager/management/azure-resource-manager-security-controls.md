---
title: Controles de segurança
description: Uma lista de verificação de controles de segurança internos para avaliar o serviço de Azure Resource Manager.
ms.topic: conceptual
ms.date: 09/04/2019
ms.openlocfilehash: d0a0625153e428a0d261e52d40b31ef5142eddfd
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "75485618"
---
# <a name="security-controls-for-azure-resource-manager"></a>Controles de segurança para Azure Resource Manager

Este artigo documenta os controles de segurança internos do Azure Resource Manager.

[!INCLUDE [Security controls Header](../../../includes/security-controls-header.md)]

## <a name="data-protection"></a>Proteção de dados

| Controle de segurança | Sim/Não | Observações |
|---|---|--|
| Criptografia no lado do servidor em repouso: chaves gerenciadas pela Microsoft | Sim |  |
| Criptografia em trânsito (como criptografia de ExpressRoute, criptografia de vnet e criptografia vnet)| Sim | HTTPS/TLS. |
| Criptografia no lado do servidor em repouso: chaves gerenciadas pelo cliente (BYOK) | N/D | O Azure Resource Manager não armazena nenhum conteúdo do cliente, só controla os dados. |
| Criptografia em nível de coluna (serviços de dados do Azure)| Sim | |
| Chamadas criptografadas à API| Sim | |

## <a name="network"></a>Rede

| Controle de segurança | Sim/Não | Observações |
|---|---|--|
| Suporte ao ponto de extremidade de serviço| Não | |
| Suporte à injeção de VNet| Sim | |
| Isolamento de rede e suporte de firewall| Não |  |
| Suporte a túnel forçado| Não |  |

## <a name="monitoring--logging"></a>Monitorando & log

| Controle de segurança | Sim/Não | Observações|
|---|---|--|
| Suporte ao monitoramento do Azure (log Analytics, app insights, etc.)| Não | |
| Registro e auditoria do plano de gerenciamento e controle| Sim | Os logs de atividades expõem todas as operações de gravação (PUT, POST e DELETE) executadas em seus recursos; consulte [Exibir logs de atividades para auditar ações em recursos](view-activity-logs.md). |
| Log e auditoria do plano de dados| N/D | |

## <a name="identity"></a>Identidade

| Controle de segurança | Sim/Não | Observações|
|---|---|--|
| Autenticação| Sim | Baseado em [Azure Active Directory](/azure/active-directory) .|
| Autorização| Sim | |

## <a name="configuration-management"></a>Gerenciamento de configuração

| Controle de segurança | Sim/Não | Observações|
|---|---|--|
| Suporte ao gerenciamento de configuração (controle de versão de configuração, etc.)| Sim |  |

## <a name="next-steps"></a>Próximas etapas

- Saiba mais sobre os [controles de segurança internos nos serviços do Azure](../../security/fundamentals/security-controls.md).