---
title: Diretrizes de Apache Spark no Azure HDInsight
description: Aprenda as diretrizes para usar Apache Spark no Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 01cdc121abded954c2443599c5d69689acd69b62
ms.sourcegitcommit: 856db17a4209927812bcbf30a66b14ee7c1ac777
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/29/2020
ms.locfileid: "82562331"
---
# <a name="apache-spark-guidelines"></a>Diretrizes de Apache Spark

Este artigo fornece várias diretrizes para usar o Apache Spark no Azure HDInsight.

## <a name="how-do-i-run-or-submit-spark-jobs"></a>Como fazer executar ou enviar trabalhos do Spark?

| Opção | Documentos |
|---|---|
| VSCode | [Use as ferramentas do Spark & Hive para Visual Studio Code](../hdinsight-for-vscode.md) |
| Jupyter Notebooks | [Tutorial: Carregar dados e executar consultas em um cluster do Apache Spark no Azure HDInsight](./apache-spark-load-data-run-query.md) |
| IntelliJ | [Tutorial: usar Azure Toolkit for IntelliJ para criar aplicativos Apache Spark para um cluster HDInsight](./apache-spark-intellij-tool-plugin.md) |
| IntelliJ | [Tutorial: Criar um aplicativo Scala Maven para Apache Spark no HDInsight usando IntelliJ](./apache-spark-create-standalone-application.md) |
| Blocos de anotações do Zeppelin | [Usar notebooks do Apache Zeppelin com o cluster do Apache Spark no Azure HDInsight](./apache-spark-zeppelin-notebook.md) |
| Envio de trabalho remoto com Livy | [Use a API REST do Apache Spark para enviar trabalhos remotos para um cluster do HDInsight Spark](./apache-spark-livy-rest-interface.md) |
|[Apache Oozie](../hdinsight-use-oozie-linux-mac.md)|O Oozie é um sistema de fluxo de trabalho e coordenação que gerencia trabalhos do Hadoop.|
|[Apache Livy](./apache-spark-livy-rest-interface.md)|Você pode usar a Livy para executar shells interativos do Spark ou enviar trabalhos em lotes a serem executados no Spark.|
|[Azure Data Factory para Apache Spark](../../data-factory/transform-data-using-spark.md)|A atividade do Spark em um pipeline de Data Factory executa um programa Spark por conta própria ou [cluster HDInsight sob demanda.|
|[Azure Data Factory para Apache Hive](../../data-factory/transform-data-using-hadoop-hive.md)|A atividade do hive do HDInsight em um pipeline Data Factory executa consultas de Hive em seu próprio cluster HDInsight ou sob demanda.|

## <a name="how-do-i-monitor-and-debug-spark-jobs"></a>Como fazer monitorar e depurar trabalhos do Spark?

| Opção | Documentos |
|---|---|
| Kit de Ferramentas do Azure para IntelliJ | [Falha na depuração do trabalho Spark com Azure Toolkit for IntelliJ (versão prévia)](apache-spark-intellij-tool-failure-debug.md) |
| Azure Toolkit for IntelliJ por meio de SSH | [Depurar aplicativos Apache Spark local ou remotamente em um cluster HDInsight com Azure Toolkit for IntelliJ por meio do SSH](apache-spark-intellij-tool-debug-remotely-through-ssh.md) |
| Azure Toolkit for IntelliJ por meio de VPN | [Usar o Azure Toolkit for IntelliJ para depurar aplicativos Apache Spark remotamente no HDInsight por meio de VPN](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md) |
| Grafo de trabalho no servidor de histórico de Apache Spark | [Use o Apache Spark History Server estendido para depurar e diagnosticar aplicativos do Apache Spark](./apache-azure-spark-history-server.md) |

## <a name="how-do-i-make-my-spark-jobs-run-more-efficiently"></a>Como fazer fazer com que meus trabalhos do Spark sejam executados com mais eficiência?

| Opção | Documentos |
|---|---|
| Cache de e/s | [Melhorar o desempenho de cargas de trabalho do Apache Spark usando o Cache de E/S do Azure HDInsight (versão prévia)](./apache-spark-improve-performance-iocache.md) |
| Opções de configuração | [Otimizar trabalhos do Apache Spark](./apache-spark-perf.md) |

## <a name="how-do-i-connect-to-other-azure-services"></a>Como fazer conectar-se a outros serviços do Azure?

| Opção | Documentos |
|---|---|
| Apache Hive no HDInsight | [Integrar o Apache Spark e Apache Hive ao Hive Warehouse Connector](../interactive-query/apache-hive-warehouse-connector.md) |
| Apache HBase no HDInsight | [Usar o Apache Spark para ler e gravar dados do Apache HBase](../hdinsight-using-spark-query-hbase.md) |
| Apache Kafka no HDInsight | [Tutorial: Use o fluxo estruturado do Apache Spark com Apache Kafka no Azure HDInsight](../hdinsight-apache-kafka-spark-structured-streaming.md) |
| Azure Cosmos DB | [Azure Cosmos DB: implementar uma arquitetura lambda na plataforma Azure](../../cosmos-db/lambda-architecture.md) |

## <a name="what-are-my-storage-options"></a>Quais são minhas opções de armazenamento?

| Opção | Documentos |
|---|---|
| Armazenamento do Data Lake Gen2 | [Usar Gen2 de armazenamento do Azure Data Lake com clusters de HDInsight do Azure](../hdinsight-hadoop-use-data-lake-storage-gen2.md) |
| Armazenamento do Data Lake Gen1 | [Usar Data Lake Storage Gen1 com clusters HDInsight do Azure](../hdinsight-hadoop-use-data-lake-store.md) |
| Armazenamento do Blobs do Azure | [Usar o Armazenamento do Azure com clusters do Azure HDInsight](../hdinsight-hadoop-use-blob-storage.md) |

## <a name="next-steps"></a>Próximas etapas

* [Definir configurações do Apache Spark](apache-spark-settings.md)
* [Otimizar Apache Spark trabalhos no HDInsight](apache-spark-perf.md)
