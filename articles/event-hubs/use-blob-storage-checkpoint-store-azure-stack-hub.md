---
title: Usar armazenamento de BLOBs como armazenamento de ponto de verificação no Hub de Azure Stack (versão prévia)
description: Este artigo descreve como usar o armazenamento de BLOBs como um armazenamento de ponto de verificação em hubs de eventos no Hub de Azure Stack (versão prévia).
services: event-hubs
documentationcenter: na
author: spelluru
ms.service: event-hubs
ms.topic: how-to
ms.date: 03/18/2020
ms.author: spelluru
ms.openlocfilehash: 2938099383c32eac493e4b4bb620f03c76ca5c44
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "82023642"
---
# <a name="use-blob-storage-as-checkpoint-store---event-hubs-on-azure-stack-hub-preview"></a>Usar o armazenamento de BLOBs como ponto de verificação do repositório-hubs de eventos no Hub de Azure Stack (versão prévia)
Se você estiver usando o armazenamento de BLOBs do Azure como o armazenamento de ponto de verificação em um ambiente que dá suporte a uma versão diferente do SDK do Storage blob do que aqueles que estão normalmente disponíveis no Azure, você precisará usar o código para alterar a versão da API do serviço de armazenamento para a versão específica com suporte desse ambiente. Por exemplo, se você estiver executando os [hubs de eventos em um hub de Azure Stack versão 2002](https://docs.microsoft.com/azure-stack/user/event-hubs-overview), a versão mais alta disponível para o serviço de armazenamento é a versão 2017-11-09. Nesse caso, você precisa usar o código para direcionar a versão da API do serviço de armazenamento para 2017-11-09. Para obter um exemplo de como direcionar uma versão de API de armazenamento específica, consulte estes exemplos no GitHub: 

- [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/Sample10_RunningWithDifferentStorageVersion.cs)
- [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorWithCustomStorageVersion.java). 
- [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript/receiveEventsWithApiSpecificStorage.js) ou [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/typescript/src/receiveEventsWithApiSpecificStorage.ts) 
- Python – [síncrono](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob/samples/receive_events_using_checkpoint_store_storage_api_version.py), [assíncrono](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/receive_events_using_checkpoint_store_storage_api_version_async.py)

> [!IMPORTANT]
> Os hubs de eventos no Hub Azure Stack estão atualmente em [Visualização](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) e são gratuitos. 

Se você executar o receptor de hubs de eventos que usa o armazenamento de BLOBs como o armazenamento de ponto de verificação sem direcionar a versão à qual Azure Stack Hub dá suporte, você receberá a seguinte mensagem de erro:

```
The value for one of the HTTP headers is not in the correct format
```


## <a name="sample-error-message-in-python"></a>Mensagem de erro de exemplo no Python
Para Python, um erro de `azure.core.exceptions.HttpResponseError` é passado para o manipulador `on_error(partition_context, error)` de erro `EventHubConsumerClient.receive()`do. No entanto, `receive()` o método não gera uma exceção. `print(error)`o imprimirá as seguintes informações de exceção:

```bash
The value for one of the HTTP headers is not in the correct format.

RequestId:f048aee8-a90c-08ba-4ce1-e69dba759297
Time:2020-03-17T22:04:13.3559296Z
ErrorCode:InvalidHeaderValue
Error:None
HeaderName:x-ms-version
HeaderValue:2019-07-07
```

O agente registrará em log dois avisos, como os seguintes:

```bash
WARNING:azure.eventhub.extensions.checkpointstoreblobaio._blobstoragecsaio: 
An exception occurred during list_ownership for namespace '<namespace-name>.eventhub.<region>.azurestack.corp.microsoft.com' eventhub 'python-eh-test' consumer group '$Default'. 

Exception is HttpResponseError('The value for one of the HTTP headers is not in the correct format.\nRequestId:f048aee8-a90c-08ba-4ce1-e69dba759297\nTime:2020-03-17T22:04:13.3559296Z\nErrorCode:InvalidHeaderValue\nError:None\nHeaderName:x-ms-version\nHeaderValue:2019-07-07')

WARNING:azure.eventhub.aio._eventprocessor.event_processor:EventProcessor instance '26d84102-45b2-48a9-b7f4-da8916f68214' of eventhub 'python-eh-test' consumer group '$Default'. An error occurred while load-balancing and claiming ownership. 

The exception is HttpResponseError('The value for one of the HTTP headers is not in the correct format.\nRequestId:f048aee8-a90c-08ba-4ce1-e69dba759297\nTime:2020-03-17T22:04:13.3559296Z\nErrorCode:InvalidHeaderValue\nError:None\nHeaderName:x-ms-version\nHeaderValue:2019-07-07'). Retrying after 71.45254944090853 seconds
```



## <a name="next-steps"></a>Próximas etapas

Consulte o artigo a seguir saiba mais sobre particionamento e ponto de verificação: [balancear a carga da partição entre várias instâncias do seu aplicativo](event-processor-balance-partition-load.md)
