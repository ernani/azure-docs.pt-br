---
title: Ingerir no pool do SQL no Azure Synapse Analytics
description: Saiba como incluir dados em um pool do SQL no Azure Synapse Analytics
services: synapse-analytics
author: djpmsft
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/15/2020
ms.author: daperlov
ms.reviewer: jrasnick
ms.openlocfilehash: ab25b2e300f9fe9b0b1d077a42538c69fa543af2
ms.sourcegitcommit: 999ccaf74347605e32505cbcfd6121163560a4ae
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/08/2020
ms.locfileid: "82982693"
---
# <a name="ingesting-data-into-a-sql-pool"></a>Ingerir dados em um pool do SQL

Neste artigo, você aprenderá a incluir dados de uma conta de armazenamento Azure Data Lake Gen 2 em um pool do SQL usando o Azure Synapse Analytics.

## <a name="prerequisites"></a>Pré-requisitos

* **Assinatura do Azure**: se você não tiver uma assinatura do Azure, crie uma [conta gratuita do Azure](https://azure.microsoft.com/free/) antes de começar.
* **Conta de armazenamento do Azure**: Use Azure data Lake Storage Gen 2 como um armazenamento de dados de *origem* . Se você não tiver uma conta de armazenamento, consulte [criar uma conta de armazenamento do Azure](../../storage/blobs/data-lake-storage-quickstart-create-account.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) para obter as etapas para criar uma.
* **Azure Synapse Analytics**: você usa um pool SQL como um armazenamento de dados de *coletor* . Se você não tiver uma instância do Azure Synapse Analytics, consulte [criar um pool SQL](../../sql-database/sql-database-get-started-portal.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) para obter as etapas para criar uma.

## <a name="create-linked-services"></a>Criar serviços vinculados

No Azure Synapse Analytics, um serviço vinculado é onde você define as informações de conexão com outros serviços. Nesta seção, você adicionará uma análise de Synapse do Azure e um serviço vinculado do Azure Data Lake Storage Gen2.

1. Abra o UX do Azure Synapse Analytics e vá para a guia **gerenciar** .
1. Em **conexões externas**, selecione **Serviços vinculados**.
1. Para adicionar um serviço vinculado, clique em **novo**.
1. Selecione o bloco Azure Data Lake Storage Gen2 na lista e clique em **continuar**.
1. Insira suas credenciais de autenticação. A chave de conta, a entidade de serviço e a identidade gerenciada são tipos de autenticação com suporte no momento. Clique em testar conexão para verificar se suas credenciais estão corretas. Clique em **Criar** quando terminar.
1. Repita as etapas de 3-5, mas em vez de Azure Data Lake Storage Gen2, selecione o bloco análise de Synapse do Azure e insira as credenciais de conexão correspondentes. Para o Azure Synapse Analytics, a autenticação do SQL, a identidade gerenciada e a entidade de serviço atualmente têm suporte.

## <a name="create-pipeline"></a>Criar um pipeline

Um pipeline contém o fluxo lógico para uma execução de um conjunto de atividades. Nesta seção, você criará um pipeline que contém uma atividade de cópia que ingere dados de ADLS Gen2 em um pool do SQL.

1. Vá para a guia **orquestrar** . Clique no ícone de adição ao lado do cabeçalho pipelines e selecione **pipeline**.
1. Em **mover e transformar** no painel atividades, arraste **copiar dados** para a tela do pipeline.
1. Clique na atividade de cópia e vá para a guia **origem** . clique em **novo** para criar um novo conjunto de fonte de origem.
1. Selecione Azure Data Lake Storage Gen2 como seu armazenamento de dados e clique em continuar.
1. Selecione DelimitedText como seu formato e clique em continuar.
1. No painel definir propriedades, selecione o serviço vinculado ADLS que você criou. Especifique o caminho do arquivo de dados de origem e especifique se a primeira linha tem um cabeçalho. Você pode importar o esquema do repositório de arquivos ou de um arquivo de exemplo. Clique em OK quando terminar.
1. Vá para a guia **coletor** . clique em **novo** para criar um novo conjunto de coletas.
1. Selecione Azure Synapse Analytics como seu armazenamento de dados e clique em continuar.
1. No painel definir propriedades, selecione o serviço vinculado do Azure Synapse Analytics que você criou. Se você estiver gravando em uma tabela existente, selecione-a na lista suspensa. Caso contrário, marque **Editar** e insira no nome da nova tabela. Clique em OK após a conclusão
1. Se você estiver criando uma tabela, habilite a **criação automática de tabela** no campo opção de tabela.

## <a name="debug-and-publish-pipeline"></a>Depurar e publicar o pipeline

Depois de concluir a configuração do pipeline, você poderá executar uma execução de depuração antes de publicar seus artefatos para verificar se tudo está correto.

1. Para depurar o pipeline, selecione **Depurar** na barra de ferramentas. Você verá o status da execução do pipeline na guia **Saída** na parte inferior da janela. 
1. Depois que o pipeline puder ser executado com êxito, na barra de ferramentas superior, selecione **publicar tudo**. Essa ação publica entidades (conjuntos de valores e pipelines) que você criou para o serviço de análise de Synapse.
1. Aguarde até que você veja a mensagem **Publicado com êxito**. Para ver as mensagens de notificação, clique no botão de sino no canto superior direito. 


## <a name="trigger-and-monitor-the-pipeline"></a>Disparar e monitorar o pipeline

Nesta etapa, você dispara manualmente o pipeline publicado na etapa anterior. 

1. Selecione **Adicionar gatilho** na barra de ferramentas e selecione **Disparar Agora**. Na página **Execução de Pipeline**, selecione **Concluir**.  
1. Vá para a guia **Monitor** localizada na barra lateral esquerda. Você verá uma execução do pipeline que é disparada por um gatilho manual. Você pode usar links na coluna **ações** para exibir detalhes da atividade e executar novamente o pipeline.
1. Selecione o link **Exibir atividades em execução** na coluna **Ações** para ver a atividade em execução associada à execução do pipeline. Neste exemplo, há apenas uma atividade, então você vê apenas uma entrada na lista. Para obter detalhes sobre a operação de cópia, selecione o link **detalhes** (ícone de óculos) na coluna **ações** . Para voltar ao modo de exibição Execuções de Pipeline, selecione **Execuções de Pipeline** na parte superior. Para atualizar a exibição, selecione **Atualizar**.
1. Verifique se os dados estão gravados corretamente no pool do SQL.


## <a name="next-steps"></a>Próximas etapas

Para obter mais informações sobre a integração de dados para o Synapse Analytics, consulte o artigo [ingerir dados em Azure data Lake Storage Gen2](data-integration-data-lake.md) .
