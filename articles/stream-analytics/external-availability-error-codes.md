---
title: Códigos de erro de disponibilidade externa - Azure Stream Analytics
description: Solucionar problemas do Azure Stream Analytics com códigos de erro de disponibilidade externa.
ms.author: mamccrea
author: mamccrea
ms.topic: conceptual
ms.date: 05/07/2020
ms.service: stream-analytics
ms.openlocfilehash: 1ce867d60c53b9befe5b4480693b54d2c9eba018
ms.sourcegitcommit: fdec8e8bdbddcce5b7a0c4ffc6842154220c8b90
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/19/2020
ms.locfileid: "83650056"
---
# <a name="azure-stream-analytics-external-availability-error-codes"></a>Códigos de erro de disponibilidade externa do Azure Stream Analytics

Você pode usar logs de atividade e logs de recursos para ajudar a depurar comportamentos inesperados do seu trabalho do Azure Stream Analytics. Este artigo lista a descrição de cada código de erro de disponibilidade externa. Os erros de disponibilidade externa ocorrem quando um serviço dependente está indisponível.

## <a name="externalserviceunavailable"></a>ExternalServiceUnavailable

* **Causa**: Um serviço está temporariamente indisponível.
* **Recomendação**: O Stream Analytics continuará tentando acessar o serviço.

## <a name="eventhubmessagingerror"></a>EventHubMessagingError

* **Causa**: O Stream Analytics encontrou um erro ao se comunicar com o EventHub. 


## <a name="next-steps"></a>Próximas etapas

* [Solucionar problemas de conexões de entrada](stream-analytics-troubleshoot-input.md)
* [Solucionar problemas de saídas do Azure Stream Analytics](stream-analytics-troubleshoot-output.md)
* [Solucionar problemas de consultas do Azure Stream Analytics](stream-analytics-troubleshoot-query.md)
* [Solucionar problemas do Azure Stream Analytics usando logs de recursos](stream-analytics-job-diagnostic-logs.md)
* [Erros de dados do Azure Stream Analytics](data-errors.md)
