---
title: Solução de problemas comuns
description: Saiba como solucionar problemas com vários SDKs ao consultar recursos do Azure com o grafo de recursos do Azure.
ms.date: 10/18/2019
ms.topic: troubleshooting
ms.openlocfilehash: f881db4f75bcee8c13221717596442ac29a4b1ac
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "74303910"
---
# <a name="troubleshoot-errors-using-azure-resource-graph"></a>Solucionar erros usando o grafo de recursos do Azure

Você pode encontrar erros ao consultar recursos do Azure com o grafo de recursos do Azure. Este artigo descreve os vários erros que podem ocorrer e como resolvê-los.

## <a name="finding-error-details"></a>Encontrar detalhes do erro

A maioria dos erros é o resultado de um problema durante a execução de uma consulta com o grafo de recursos do Azure. Quando uma consulta falha, o SDK fornece detalhes sobre a consulta com falha. Essas informações indicam o problema para que ele possa ser corrigido e uma consulta posterior seja realizada com sucesso.

## <a name="general-errors"></a>Erros gerais

### <a name="scenario-too-many-subscriptions"></a><a name="toomanysubscription"></a>Cenário: muitas assinaturas

#### <a name="issue"></a>Problema

Os clientes com acesso a mais de 1000 assinaturas, incluindo assinaturas entre locatários com o [Azure Lighthouse](../../../lighthouse/overview.md), não podem buscar dados em todas as assinaturas em uma única chamada para o grafo de recursos do Azure.

#### <a name="cause"></a>Causa

CLI do Azure e o PowerShell encaminham apenas as primeiras assinaturas 1000 para o grafo de recursos do Azure. A API REST para o grafo de recursos do Azure aceita um número máximo de assinaturas para executar a consulta.

#### <a name="resolution"></a>Resolução

Solicitações em lote para a consulta com um subconjunto de assinaturas para permanecer sob o limite de assinatura 1000. A solução está usando o parâmetro de **assinatura** no PowerShell.

```azurepowershell-interactive
# Replace this query with your own
$query = 'Resources | project type'

# Fetch the full array of subscription IDs
$subscriptions = Get-AzSubscription
$subscriptionIds = $subscriptions.Id

# Create a counter, set the batch size, and prepare a variable for the results
$counter = [PSCustomObject] @{ Value = 0 }
$batchSize = 1000
$response = @()

# Group the subscriptions into batches
$subscriptionsBatch = $subscriptionIds | Group -Property { [math]::Floor($counter.Value++ / $batchSize) }

# Run the query for each batch
foreach ($batch in $subscriptionsBatch){ $response += Search-AzGraph -Query $query -Subscription $batch.Group }

# View the completed results of the query on all subscriptions
$response
```

### <a name="scenario-unsupported-content-type-rest-header"></a><a name="rest-contenttype"></a>Cenário: cabeçalho REST de tipo de conteúdo sem suporte

#### <a name="issue"></a>Problema

Os clientes que consultam a API REST do grafo de recursos do Azure obtêm uma resposta _500_ (erro interno do servidor) retornada.

#### <a name="cause"></a>Causa

A API REST do grafo de recursos do Azure `Content-Type` dá suporte apenas a um **aplicativo/JSON**. Algumas ferramentas REST ou agentes assumem o padrão de **text/plain**, que não é suportado pela API REST.

#### <a name="resolution"></a>Resolução

Valide se a ferramenta ou o agente que você está usando para consultar o grafo de recursos do Azure `Content-Type` tem o cabeçalho da API REST configurado para **Application/JSON**.

### <a name="scenario-no-read-permission-to-all-subscriptions-in-list"></a><a name="rest-403"></a>Cenário: nenhuma permissão de leitura para todas as assinaturas na lista

#### <a name="issue"></a>Problema

Os clientes que passam explicitamente uma lista de assinaturas com uma consulta do grafo de recursos do Azure obtêm uma resposta de _403_ (proibido).

#### <a name="cause"></a>Causa

Se o cliente não tiver permissão de leitura para todas as assinaturas fornecidas, a solicitação será negada devido à falta de direitos de segurança apropriados.

#### <a name="resolution"></a>Resolução

Inclua pelo menos uma assinatura na lista de assinaturas que o cliente que está executando a consulta tenha pelo menos acesso de leitura ao. Para obter mais informações, consulte [permissões no grafo de recursos do Azure](../overview.md#permissions-in-azure-resource-graph).

## <a name="next-steps"></a>Próximas etapas

Se você não encontrou seu problema ou não conseguiu resolver seu problema, visite um dos seguintes canais para obter mais suporte:

- Obtenha respostas de especialistas do Azure por meio dos [fóruns do Azure](https://azure.microsoft.com/support/forums/).
- Conecte- [@AzureSupport](https://twitter.com/azuresupport) se ao – a conta de Microsoft Azure oficial para melhorar a experiência do cliente conectando a Comunidade do Azure aos recursos certos: respostas, suporte e especialistas.
- Se precisar de mais ajuda, você pode registrar um incidente de suporte do Azure. Vá para o [site de suporte do Azure](https://azure.microsoft.com/support/options/) e selecione **obter suporte**.