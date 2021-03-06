---
title: 'Início Rápido: Criar um cluster do Apache Hadoop no Azure HDInsight usando o modelo do Resource Manager'
description: Neste início rápido, você cria um cluster do Apache Hadoop no Azure HDInsight usando o modelo do Resource Manager
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 03/13/2020
ms.openlocfilehash: ae0f29b8085bd9637f527f2a58229dd89ce6933b
ms.sourcegitcommit: c2065e6f0ee0919d36554116432241760de43ec8
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/26/2020
ms.locfileid: "80064668"
---
# <a name="quickstart-create-apache-hadoop-cluster-in-azure-hdinsight-using-resource-manager-template"></a>Início Rápido: Criar um cluster do Apache Hadoop no Azure HDInsight usando o modelo do Resource Manager

Neste início rápido, você usará um modelo do Azure Resource Manager para criar um cluster [Apache Hadoop](./apache-hadoop-introduction.md) no Azure HDInsight. O Hadoop era a estrutura open-source original para processamento distribuído e análise de conjuntos de Big Data em clusters. O ecossistema do Hadoop inclui software e utilitários relacionados, inclusive o Apache Hive, o Apache HBase, o Spark, o Kafka e muitos outros.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]
  
Atualmente, o HDInsight vem com [sete tipos diferentes de cluster](../hdinsight-overview.md#cluster-types-in-hdinsight). Cada tipo de cluster dá suporte a um conjunto diferente de componentes. Todos os tipos de cluster dão suporte ao Hive. Para obter uma lista de componentes com suporte no HDInsight, confira [Novidades nas versões de cluster Hadoop fornecidas pelo HDInsight?](../hdinsight-component-versioning.md)  

Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

## <a name="create-an-apache-hadoop-cluster"></a>Criar um cluster do Apache Hadoop

### <a name="review-the-template"></a>Examinar o modelo

O modelo usado neste início rápido é proveniente dos [modelos de Início Rápido do Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-ssh-password).

:::code language="json" source="~/quickstart-templates/101-hdinsight-linux-ssh-password/azuredeploy.json" range="1-148":::


Há dois recursos do Azure definidos no modelo:

* [Microsoft.Storage/storageAccounts](https://docs.microsoft.com/azure/templates/microsoft.storage/storageaccounts): crie uma Conta de Armazenamento do Azure.
* [Microsoft.HDInsight/cluster](https://docs.microsoft.com/azure/templates/microsoft.hdinsight/clusters): crie um cluster HDInsight.

### <a name="deploy-the-template"></a>Implantar o modelo

1. Selecione o botão **Implantar no Azure** abaixo para entrar no Azure e abrir o modelo do Resource Manager.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-ssh-password%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hadoop-linux-tutorial-get-started/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

1. Digite ou selecione os valores a seguir:

    |Propriedade  |Descrição  |
    |---------|---------|
    |Subscription|Na lista suspensa, selecione a assinatura do Azure usada para o cluster.|
    |Resource group|Na lista suspensa, selecione o grupo de recursos existente ou selecione **Criar**.|
    |Location|O valor será preenchido automaticamente com o local usado para o grupo de recursos.|
    |Nome do cluster|Insira um nome global exclusivo. Para este modelo, use apenas letras minúsculas e números.|
    |Tipo de cluster | Selecione **hadoop**. |
    |Nome de usuário de logon do cluster|Forneça o nome de usuário, o padrão é **admin**.|
    |Senha de logon do cluster|Forneça uma senha. A senha precisa ter no mínimo 10 caracteres e precisa conter pelo menos um dígito, uma letra maiúscula, uma minúscula e um caractere não alfanumérico (exceto pelos caracteres ' " ` ). |
    |Nome de Usuário SSH|Forneça o nome de usuário, o padrão é **sshuser**|
    |Senha SSH|Forneça a senha.|

    Algumas propriedades foram inseridas no código do modelo.  Você pode configurar esses valores do modelo. Para obter mais explicações sobre essas propriedades, consulte [Criar clusters do Apache Hadoop no HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).

    > [!NOTE]  
    > Os valores fornecidos devem ser exclusivos e seguir as diretrizes de nomenclatura. O modelo não executa verificações de validação. Se os valores que você fornecer já estiverem em uso ou não seguirem as diretrizes, você receberá um erro após o envio do modelo.  

    ![Introdução ao HDInsight Linux, modelo do Resource Manager no portal](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-linux-get-started-arm-template-on-portal.png "Implantar o cluster Hadoop no HDInsight usando o portal do Azure e um modelo de gerenciador de grupo de recursos")

1. Examine os **TERMOS E CONDIÇÕES**. Em seguida, selecione **Concordo com os termos e condições declarados acima** e **Comprar**. Você receberá uma notificação de que a implantação está em andamento. Demora cerca de 20 minutos para criar um cluster.

## <a name="review-deployed-resources"></a>Examinar os recursos implantados

Depois que o cluster for criado, você receberá uma notificação de **Implantação bem-sucedida** com um link **Ir para o recurso**. A página do Grupo de recursos listará o novo cluster HDInsight e o armazenamento padrão associado a ele. Cada cluster tem uma dependência na conta do [Armazenamento do Azure](../hdinsight-hadoop-use-blob-storage.md) ou na [conta do Azure Data Lake Storage](../hdinsight-hadoop-use-data-lake-store.md). Ela é conhecida como a conta de armazenamento padrão. O cluster do HDInsight e sua conta de armazenamento padrão devem estar colocados na mesma região do Azure. A exclusão dos clusters não exclui a conta de armazenamento.

> [!NOTE]  
> Para obter outros métodos de criação de cluster e compreender as propriedades usadas neste início rápido, confira [Criar clusters do HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).

## <a name="clean-up-resources"></a>Limpar os recursos

Após concluir o início rápido, poderá ser conveniente excluir o cluster. Com o HDInsight, seus dados são armazenados no Armazenamento do Azure, assim você poderá excluir, com segurança, um cluster quando ele não estiver em uso. Você também é cobrado por um cluster HDInsight, mesmo quando ele não está em uso. Como os encargos para o cluster são muitas vezes maiores do que os encargos para armazenamento, faz sentido, do ponto de vista econômico, excluir os clusters quando não estiverem em uso.

> [!NOTE]  
> Se você for prosseguir *imediatamente* no próximo tutorial para saber como executar operações de ETL usando Hadoop no HDInsight, convém manter o cluster em execução. Isso porque, no tutorial, você precisará criar um cluster Hadoop novamente. No entanto, se você não for conferir o próximo tutorial imediatamente, exclua o cluster agora.

No portal do Azure, navegue até seu cluster e selecione **Excluir**.

![Excluir cluster do HDInsight do portal](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-delete-cluster.png "Excluir cluster do HDInsight do portal")

Também é possível selecionar o nome do grupo de recursos para abrir a página do grupo de recursos, e depois selecionar **Excluir grupo de recursos**. Ao excluir o grupo de recursos, você exclui o cluster HDInsight e a conta de armazenamento padrão.

## <a name="next-steps"></a>Próximas etapas

Neste início rápido, você aprenderá a criar um cluster do Apache Hadoop no HDInsight usando um modelo do Resource Manager. No próximo artigo, saiba como executar uma operação de ETL (extração, transformação e carregamento) usando o Hadoop no HDInsight.

> [!div class="nextstepaction"]
> [Extrair, transformar e carregar dados usando a Consulta Interativa no HDInsight](../interactive-query/interactive-query-tutorial-analyze-flight-data.md)
