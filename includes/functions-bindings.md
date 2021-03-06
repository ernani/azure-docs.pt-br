---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/04/2018
ms.author: glenga
ms.custom: fasttrack-edit
ms.openlocfilehash: 819eed1eb4e32469bf0572a6548f5a2a44091ef0
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "77279515"
---
Esta tabela mostra as associações com suporte nas versões principais do tempo de execução de Azure Functions:


| Type | 1.x | 2. x e superior<sup>1</sup> | Gatilho | Entrada | Saída |
| ---- | :-: | :-: | :------: | :---: | :----: |
| [Armazenamento de BLOBs](../articles/azure-functions/functions-bindings-storage-blob.md)          |✔|✔|✔|✔|✔|
| [Cosmos DB](../articles/azure-functions/functions-bindings-documentdb.md)               |✔|✔|✔|✔|✔|
| [Grade de Eventos](../articles/azure-functions/functions-bindings-event-grid.md)              |✔|✔|✔| |✔|
| [Hubs de Evento](../articles/azure-functions/functions-bindings-event-hubs.md)              |✔|✔|✔| |✔|
| [WebHooks de & de HTTP](../articles/azure-functions/functions-bindings-http-webhook.md)             |✔|✔|✔| |✔|
| [Hub IoT](../articles/azure-functions/functions-bindings-event-iot.md)             |✔|✔|✔| |✔|
| [Microsoft Graph<br/>Tabelas do Excel](../articles/azure-functions/functions-bindings-microsoft-graph.md)   ||✔| |✔|✔|
| [Microsoft Graph<br/>Arquivos do OneDrive](../articles/azure-functions/functions-bindings-microsoft-graph.md) ||✔| |✔|✔|
| [Microsoft Graph<br/>Email do Outlook](../articles/azure-functions/functions-bindings-microsoft-graph.md)  ||✔| | |✔|
| [Eventos<br/>de Microsoft Graph](../articles/azure-functions/functions-bindings-microsoft-graph.md)         ||✔|✔|✔|✔|
| [Microsoft Graph<br/>Tokens de autenticação](../articles/azure-functions/functions-bindings-microsoft-graph.md)    ||✔| |✔| |
| [Aplicativos Móveis](../articles/azure-functions/functions-bindings-mobile-apps.md)             |✔| | |✔|✔|
| [Hubs de Notificação](../articles/azure-functions/functions-bindings-notification-hubs.md) |✔|| | |✔|
| [Armazenamento de filas](../articles/azure-functions/functions-bindings-storage-queue.md)         |✔|✔|✔| |✔|
| [SendGrid](../articles/azure-functions/functions-bindings-sendgrid.md)                   |✔|✔| | |✔|
| [Barramento de Serviço](../articles/azure-functions/functions-bindings-service-bus.md)             |✔|✔|✔| |✔|
| [SignalR](../articles/azure-functions/functions-bindings-signalr-service.md)             | |✔| |✔|✔|
| [Armazenamento de tabelas](../articles/azure-functions/functions-bindings-storage-table.md)         |✔|✔| |✔|✔|
| [Timer](../articles/azure-functions/functions-bindings-timer.md)                         |✔|✔|✔| | |
| [Twilio](../articles/azure-functions/functions-bindings-twilio.md)                       |✔|✔| | |✔|

<sup>1</sup> a partir do tempo de execução da versão 2. x, todas as associações, exceto http e Timer, devem ser registradas. Confira [Registrar as extensões de associação](../articles/azure-functions/functions-bindings-register.md).
