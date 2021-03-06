---
title: Monitorar e coletar dados de Machine Learning pontos de extremidade de serviço Web
titleSuffix: Azure Machine Learning
description: Monitorar serviços Web implantados com Azure Machine Learning usando o Aplicativo Azure insights
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: larryfr
author: blackmist
ms.date: 03/12/2020
ms.openlocfilehash: 464ec1fcf0986dc04bd92bbe9e31b5675e5822d4
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "79136186"
---
# <a name="monitor-and-collect-data-from-ml-web-service-endpoints"></a>Monitorar e coletar dados de pontos de extremidade de serviço Web do ML
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Neste artigo, você aprende a coletar dados do e a monitorar modelos implantados em pontos de extremidade de serviço Web no AKS (serviço de kubernetes do Azure) ou em ACI (instâncias de contêiner do Azure) habilitando informações de Aplicativo Azure por meio de 
* [Azure Machine Learning SDK do Python](#python)
* [Azure Machine Learning Studio](#studio) emhttps://ml.azure.com

Além de coletar dados de saída e resposta de um ponto de extremidade, você pode monitorar:

* As taxas, tempos de resposta e taxas de falha de solicitação
* Taxas de dependência, tempos de resposta e taxas de falha
* Exceções

[Saiba mais sobre o aplicativo Azure insights](../azure-monitor/app/app-insights-overview.md). 


## <a name="prerequisites"></a>Pré-requisitos

* Caso não tenha uma assinatura do Azure, crie uma conta gratuita antes de começar. Experimente a [versão gratuita ou paga do Azure Machine Learning](https://aka.ms/AMLFree) hoje

* Um workspace do Azure Machine Learning, um diretório local que contém seus scripts e o SDK de Aprendizado de Máquina do Azure para Python instalado. Para saber como obter esses pré-requisitos, consulte [como configurar um ambiente de desenvolvimento](how-to-configure-environment.md)

* Um modelo de aprendizado de máquina treinado para ser implantado para o serviço de Kubernetes do Azure (AKS) ou Instância de Contêiner do Azure (ACI). Se você não tiver um, consulte o tutorial [treinar modelo de classificação de imagem](tutorial-train-models-with-aml.md)

## <a name="web-service-metadata-and-response-data"></a>Metadados do serviço Web e dados de resposta

>[!Important]
> Aplicativo Azure insights registra apenas as cargas de até 64 KB. Se esse limite for atingido, somente as saídas mais recentes do modelo serão registradas em log. 

Os metadados e a resposta para o serviço, que corresponde aos metadados do serviço Web e as previsões do modelo, são registrados nos rastreamentos do Aplicativo Azure insights sob `"model_data_collection"`a mensagem. Você pode consultar Aplicativo Azure informações diretamente para acessar esses dados ou configurar uma [exportação contínua](https://docs.microsoft.com/azure/azure-monitor/app/export-telemetry) para uma conta de armazenamento para maior retenção ou processamento adicional. Os dados de modelo podem ser usados no Azure Machine Learning para configurar o rotulamento, o novo treinamento, a explicação, a análise de dados ou outro uso. 

<a name="python"></a>

## <a name="use-python-sdk-to-configure"></a>Usar o SDK do Python para configurar 

### <a name="update-a-deployed-service"></a>Atualizar um serviço implantado

1. Identificar o serviço no seu workspace. O valor de `ws` é o nome do seu espaço de trabalho

    ```python
    from azureml.core.webservice import Webservice
    aks_service= Webservice(ws, "my-service-name")
    ```
2. Atualize seu serviço e habilite insights Aplicativo Azure

    ```python
    aks_service.update(enable_app_insights=True)
    ```

### <a name="log-custom-traces-in-your-service"></a>Rastreamentos de log personalizados em seu serviço

Se você deseja registrar rastreamentos personalizados, siga o processo de implantação padrão para o AKS ou ACI no documento [Como e onde implantar](how-to-deploy-and-where.md). Em seguida, execute as seguintes etapas:

1. Atualizar o arquivo de Pontuação adicionando instruções PRINT
    
    ```python
    print ("model initialized" + time.strftime("%H:%M:%S"))
    ```

2. Atualizar a configuração do serviço
    
    ```python
    config = Webservice.deploy_configuration(enable_app_insights=True)
    ```

3. Criar uma imagem e implantá-lo no [AKS ou ACI](how-to-deploy-and-where.md).

### <a name="disable-tracking-in-python"></a>Desabilitar o controle em Python

Para desabilitar o Aplicativo Azure insights, use o seguinte código:

```python 
## replace <service_name> with the name of the web service
<service_name>.update(enable_app_insights=False)
```

<a name="studio"></a>

## <a name="use-azure-machine-learning-studio-to-configure"></a>Usar o Azure Machine Learning Studio para configurar

Você também pode habilitar informações de Aplicativo Azure do Azure Machine Learning Studio quando estiver pronto para implantar seu modelo com estas etapas.

1. Entre no seu espaço de trabalho emhttps://ml.azure.com/
1. Vá para **modelos** e selecione qual modelo você deseja implantar
1. Selecionar **+ implantar**
1. Preencher o formulário **implantar modelo**
1. Expandir o menu **avançado**

    ![Implantar formulário](./media/how-to-enable-app-insights/deploy-form.png)
1. Selecione **habilitar diagnóstico de Application insights e coleta de dados**

    ![Habilitar o app insights](./media/how-to-enable-app-insights/enable-app-insights.png)
## <a name="evaluate-data"></a>Avaliar dados
Os dados do serviço são armazenados em sua conta do Aplicativo Azure insights, dentro do mesmo grupo de recursos que Azure Machine Learning.
Para exibi-lo:

1. Vá para o espaço de trabalho Azure Machine Learning na [portal do Azure](https://ms.portal.azure.com/) e clique no link Application insights

    [![AppInsightsLoc](./media/how-to-enable-app-insights/AppInsightsLoc.png)](././media/how-to-enable-app-insights/AppInsightsLoc.png#lightbox)

1. Selecione a guia **visão geral** para ver um conjunto básico de métricas para seu serviço

   [![Visão geral](./media/how-to-enable-app-insights/overview.png)](././media/how-to-enable-app-insights/overview.png#lightbox)

1. Para examinar os metadados e a resposta da solicitação de serviço Web, selecione a tabela **solicitações** na seção **logs (análise)** e selecione **executar** para exibir solicitações

   [![Modelar dados](./media/how-to-enable-app-insights/model-data-trace.png)](././media/how-to-enable-app-insights/model-data-trace.png#lightbox)


3. Para examinar seus rastreamentos personalizados, selecione **análise**
4. Na seção de esquema, selecione **Traces**. Em seguida, selecione **executar** para executar sua consulta. Os dados devem aparecer em um formato de tabela e devem ser mapeados para suas chamadas personalizadas em seu arquivo de Pontuação

   [![Rastreamentos personalizados](./media/how-to-enable-app-insights/logs.png)](././media/how-to-enable-app-insights/logs.png#lightbox)

Para saber mais sobre como usar o Aplicativo Azure insights, confira [o que é Application insights?](../azure-monitor/app/app-insights-overview.md).

## <a name="export-data-for-further-processing-and-longer-retention"></a>Exportar dados para processamento adicional e retenção mais longa

>[!Important]
> Aplicativo Azure insights só dá suporte a exportações para o armazenamento de BLOBs. Os limites adicionais desse recurso de exportação estão listados em [Exportar telemetria do App insights](https://docs.microsoft.com/azure/azure-monitor/app/export-telemetry#continuous-export-advanced-storage-configuration).

Você pode usar a [exportação contínua](https://docs.microsoft.com/azure/azure-monitor/app/export-telemetry) do aplicativo Azure insights para enviar mensagens para uma conta de armazenamento com suporte, em que uma retenção mais longa pode ser definida. As `"model_data_collection"` mensagens são armazenadas no formato JSON e podem ser facilmente analisadas para extrair dados de modelo. 

Azure Data Factory, pipelines de ML do Azure ou outras ferramentas de processamento de dados podem ser usadas para transformar os dados conforme necessário. Depois de ter transformado os dados, você poderá registrá-los com o espaço de trabalho Azure Machine Learning como um DataSet. Para fazer isso, consulte [como criar e registrar conjuntos de registros](how-to-create-register-datasets.md).

   [![Exportação contínua](./media/how-to-enable-app-insights/continuous-export-setup.png)](././media/how-to-enable-app-insights/continuous-export-setup.png)


## <a name="example-notebook"></a>Caderno de exemplo

O bloco de anotações [Enable-app-Revisions-in-Production-Service. ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/enable-app-insights-in-production-service/enable-app-insights-in-production-service.ipynb) demonstra conceitos neste artigo. 
 
[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Próximas etapas

* Consulte [como implantar um modelo em um cluster do serviço kubernetes do Azure](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-azure-kubernetes-service) ou [como implantar um modelo em instâncias de contêiner do Azure](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-azure-container-instance) para implantar seus modelos em pontos de extremidade de serviço Web e habilitar aplicativo Azure insights para aproveitar a coleta de dados e o monitoramento de ponto de extremidades
* Consulte [MLOps: gerenciar, implantar e monitorar modelos com Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-model-management-and-deployment) para saber mais sobre como aproveitar os dados coletados de modelos em produção. Esses dados podem ajudar a melhorar continuamente o processo de aprendizado de máquina
