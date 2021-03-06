---
title: Ajuste de desempenho com cache de conjunto de resultados
description: Visão geral do recurso de cache do conjunto de resultados para o pool do SQL Synapse no Azure Synapse Analytics
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 10/10/2019
ms.author: xiaoyul
ms.reviewer: nidejaco;
ms.custom: azure-synapse
ms.openlocfilehash: eadbe13269ce1259b4560af117f5b15b3b294151
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "81730595"
---
# <a name="performance-tuning-with-result-set-caching"></a>Ajuste de desempenho com cache de conjunto de resultados

Quando o cache do conjunto de resultados estiver habilitado, a análise do SQL armazenará em cache automaticamente os resultados da consulta no banco de dados do usuário para uso repetitivo.  Isso permite que as execuções de consulta subsequentes obtenham resultados diretamente do cache persistente, de modo que a recomputação não seja necessária.   O cache do conjunto de resultados melhora o desempenho da consulta e reduz o uso de recursos de computação.  Além disso, as consultas que usam resultados em cache definidos não usam nenhum slot de simultaneidade e, portanto, não contam com os limites de simultaneidade existentes. Por segurança, os usuários só poderão acessar os resultados armazenados em cache se tiverem as mesmas permissões de acesso a dados que os usuários que criam os resultados armazenados em cache.  

## <a name="key-commands"></a>Comandos de chave

[Ativar/desativar o cache do conjunto de resultados para um banco de dados de usuário](/sql/t-sql/statements/alter-database-transact-sql-set-options?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

[Ativar/desativar o cache do conjunto de resultados para um banco de dados de usuário](/sql/t-sql/statements/alter-database-transact-sql-set-options?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

[Ativar/desativar o cache do conjunto de resultados para uma sessão](/sql/t-sql/statements/set-result-set-caching-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

[Verificar o tamanho do conjunto de resultados em cache](/sql/t-sql/database-console-commands/dbcc-showresultcachespaceused-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)  

[Limpar o cache](/sql/t-sql/database-console-commands/dbcc-dropresultsetcache-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

## <a name="whats-not-cached"></a>O que não está armazenado em cache  

Quando o armazenamento em cache do conjunto de resultados estiver ATIVADO para um banco de dados, os resultados serão armazenados em cache para todas as consultas até o cache ficar cheio, exceto para estas consultas:

- Consultas que usam funções não determinísticas, como DateTime.Now()
- Consultas que usam funções definidas pelo usuário
- Consultas que usam tabelas com segurança em nível de linha ou segurança em nível de coluna habilitada
- Consultas que retornam dados com tamanho de linha maior que 64 KB
- Consultas retornando dados grandes em tamanho (>10GB) 

> [!IMPORTANT]
> As operações para criar o cache do conjunto de resultados e recuperar dados do cache acontecem no nó de controle de uma instância do pool SQL do Synapse.
> Quando o cache do conjunto de resultados está ativado, a execução de consultas que retornam um conjunto de resultados grande (por exemplo, >1GB) pode causar alta limitação no nó de controle e reduzir a resposta geral da consulta na instância.  Normalmente, essas consultas são usadas normalmente durante a exploração de dados ou operações de ETL. Para evitar sobrecarregar o nó de controle e causar problemas de desempenho, os usuários devem DESATIVAR o cache do conjunto de resultados no banco de dados antes de executar esses tipos de consultas.  

Execute esta consulta para o tempo gasto pelas operações de cache do conjunto de resultados para uma consulta:

```sql
SELECT step_index, operation_type, location_type, status, total_elapsed_time, command
FROM sys.dm_pdw_request_steps
WHERE request_id  = <'request_id'>;
```

Aqui está um exemplo de saída para uma consulta executada com o cache de conjunto de resultados desabilitado.

![Consulta-etapas-com-RSC-desabilitado](./media/performance-tuning-result-set-caching/query-steps-with-rsc-disabled.png)

Aqui está um exemplo de saída para uma consulta executada com o cache do conjunto de resultados habilitado.

![Consulta-etapas-com-habilitado para RSC](./media/performance-tuning-result-set-caching/query-steps-with-rsc-enabled.png)

## <a name="when-cached-results-are-used"></a>Quando os resultados armazenados em cache são usados

O conjunto de resultados armazenado em cache será reutilizado em uma consulta se todos os requisitos a seguir forem atendidos:

- O usuário que executa a consulta tem acesso a todas as tabelas referenciadas na consulta.
- Há uma correspondência exata entre a nova consulta e a anterior que gerou o armazenamento em cache do conjunto de resultados.
- Não há alterações de dados ou esquemas nas tabelas em que o conjunto de resultados armazenado em cache foi gerado.

Execute esse comando para verificar se uma consulta foi executada com uma perda ou ocorrência no cache de resultado. A coluna result_cache_hit retorna 1 para o acesso ao cache, 0 para perda de cache e valores negativos por motivos pelos quais o cache do conjunto de resultados não foi usado. Confira [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) para obter detalhes.

```sql
SELECT request_id, command, result_cache_hit FROM sys.dm_pdw_exec_requests
WHERE request_id = <'Your_Query_Request_ID'>
```

## <a name="manage-cached-results"></a>Gerenciar resultados em cache

O tamanho máximo do cache de conjunto de resultados é de 1 TB por banco de dados.  Os resultados armazenados em cache são automaticamente invalidados quando os dados de consulta subjacentes são alterados.  

A remoção do cache é gerenciada pelo SQL Analytics automaticamente após este agendamento:

- A cada 48 horas, se o conjunto de resultados não foi usado ou se foi invalidado.
- Quando o cache do conjunto de resultados se aproximar do tamanho máximo.

Os usuários podem esvaziar manualmente todo o cache do conjunto de resultados usando uma destas opções:

- DESATIVAR o recurso de cache do conjunto de resultados do banco de dados
- Executar DBCC DROPRESULTSETCACHE enquanto estiver conectado ao banco de dados

Pausar um banco de dados não esvaziará o conjunto de resultados em cache.  

## <a name="next-steps"></a>Próximas etapas

Para obter mais dicas de desenvolvimento, confira [visão geral de desenvolvimento](sql-data-warehouse-overview-develop.md).
