---
title: Cancelar API de operação | Azure Marketplace
description: Cancela operações.
author: dsindona
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
ms.date: 04/08/2020
ms.author: dsindona
ms.openlocfilehash: f9e55ff2c581f9392a125f6dc3ec8d903e9876a4
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "81256426"
---
# <a name="cancel-operation"></a>Cancelar operação

> [!NOTE]
> As APIs de Portal do Cloud Partner são integradas ao Partner Center e continuarão funcionando depois que suas ofertas forem migradas para o Partner Center. A integração apresenta pequenas alterações. Examine as alterações listadas em [portal do Cloud Partner referência de API](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cloud-partner-portal-api-overview) para garantir que seu código continue a funcionar após a migração para o Partner Center.

Essa API cancela uma operação atualmente em andamento na oferta. Use [Recuperar API de operações](./cloud-partner-portal-api-retrieve-operations.md) para obter um `operationId` para passar a essa API. Geralmente, o cancelamento é uma operação síncrona, no entanto, em alguns cenários complexos uma nova operação pode ser necessária para cancelar uma operação existente. Nesse caso, o corpo da resposta HTTP contém o local da operação que deve ser usado para consultar o status.

  `POST https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/cancel?api-version=2017-10-31`

## <a name="uri-parameters"></a>Parâmetros de URI

--------------

|  **Nome**    |      **Descrição**                                  |    **Tipo de dados**  |
| ------------ |     ----------------                                  |     -----------   |
| publisherId  |  Identificador do publicar, por exemplo, `contoso`         |   Cadeia de caracteres          |
| offerId      |  Identificador da oferta                                     |   Cadeia de caracteres          |
| api-version  |  Versão atual da API                               |    Data           |
|  |  |  |

## <a name="header"></a>Cabeçalho
------

|  **Nome**              |  **Valor**         |
|  ---------             |  ----------        |
|  Tipo de conteúdo          |  aplicativo/json  |
|  Autorização         |  TOKEN de seu portador |
|  |  |

## <a name="body-example"></a>Exemplo de corpo
------------

### <a name="request"></a>Solicitação

``` json
{
   "metadata": {
     "notification-emails": "jondoe@contoso.com"
    }
}     
```

### <a name="request-body-properties"></a>Solicitar propriedades do corpo

|  **Nome**                |  **Descrição**                                               |
|  --------                |  ---------------                                               |
|  emails de notificação     | Lista separada por vírgula de IDs de email a serem notificados sobre o andamento da operação de publicação. |
|  |  |

### <a name="response"></a>Resposta

#### <a name="migrated-offers"></a>Ofertas migradas

`Location: /api/publishers/contoso/offers/contoso-offer/operations/56615b67-2185-49fe-80d2-c4ddf77bb2e8?api-version=2017-10-31`

#### <a name="non-migrated-offers"></a>Ofertas não migradas

`Location: /api/operations/contoso$contoso-offer$2$preview?api-version=2017-10-31`

### <a name="response-header"></a>Cabeçalho de Resposta

|  **Nome**             |    **Valor**                       |
|  ---------            |    ----------                      |
| Local    | O caminho relativo para recuperar o status da operação. |
|  |  |

### <a name="response-status-codes"></a>Códigos de status de resposta

| **Auto-completar**  |  **Descrição**                                                                       |
|  ------   |  ------------------------------------------------------------------------               |
|  200      | OK. A solicitação foi processada com êxito e a operação é cancelada de forma síncrona. |
|  202      | Aceita. A solicitação foi processada com êxito e a operação está em processo de cancelamento. A localização da operação de cancelamento é retornada no cabeçalho de resposta. |
|  400      | Solicitação incorreta/malformada. O corpo da resposta ao erro pode fornecer mais informações.  |
|  403      | Acesso proibido. O cliente não tem acesso ao namespace especificado na solicitação. |
|  404      | Não encontrado. A entidade especificada não existe. |
|  |  |
