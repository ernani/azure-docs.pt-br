---
title: Manipuladores de eventos da Grade de Eventos do Azure
description: Descreve os manipuladores de eventos com suporte para a Grade de Eventos do Azure. Automação do Azure, Functions, Hubs de Eventos, Conexões Híbridas, Aplicativos Lógicos, Barramento de Serviço, Armazenamento de Filas, WebHooks.
services: event-grid
author: spelluru
ms.service: event-grid
ms.topic: conceptual
ms.date: 01/21/2020
ms.author: spelluru
ms.openlocfilehash: c29526bdee5ad70b3acb23117ee003877f52a55e
ms.sourcegitcommit: bb0afd0df5563cc53f76a642fd8fc709e366568b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/19/2020
ms.locfileid: "83592450"
---
# <a name="event-handlers-in-azure-event-grid"></a>Manipuladores de eventos na Grade de Eventos do Azure
Um manipulador de eventos é o local para o qual o evento é enviado. O manipulador usa alguma ação adicional para processar o evento. Vários serviços do Azure são automaticamente configurados para manipular eventos. Você também pode usar qualquer webhook para manipular eventos. O WebHook não precisa estar hospedado no Azure para manipular eventos. A Grade de Eventos dá suporte apenas a pontos de extremidade do webhook HTTPS.

## <a name="supported-event-handlers"></a>Manipuladores de evento com suporte
Estes são os manipuladores de evento com suporte: 

- [Webhooks](handler-webhooks.md). Há suporte aos runbooks e Aplicativos Lógicos da Automação do Azure por meio de webhooks. 
- [Azure Functions](handler-functions.md)
- [Hubs de Eventos](handler-event-hubs.md)
- [Conexões híbridas de retransmissão](handler-relay-hybrid-connections.md)
- [Filas e tópicos do Barramento de Serviço](handler-service-bus.md)
- [Filas de armazenamento](handler-storage-queues.md)

## <a name="next-steps"></a>Próximas etapas
- Para ver uma introdução à Grade de Eventos, confira [About Event Grid](overview.md) (Sobre a Grade de Eventos).
