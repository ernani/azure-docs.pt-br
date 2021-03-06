---
title: Decodificação lógica-banco de dados do Azure para PostgreSQL-servidor único
description: Descreve a decodificação lógica e o wal2json para a captura de dados de alteração no Azure Database para PostgreSQL-servidor único
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 03/31/2020
ms.openlocfilehash: 1213b38f2b67e8fed179cfda4308943808893e1b
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "80522141"
---
# <a name="logical-decoding"></a>Decodificação lógica
 
A [decodificação lógica no PostgreSQL](https://www.postgresql.org/docs/current/logicaldecoding.html) permite que você transmita alterações de dados para consumidores externos. A decodificação lógica é usada de forma popular para os cenários de streaming de eventos e de captura de dados de alteração.

A decodificação lógica usa um plug-in de saída para converter o WAL (log write ahead) de postgres em um formato legível. O banco de dados do Azure para PostgreSQL fornece dois plug-ins de saída: [test_decoding](https://www.postgresql.org/docs/current/test-decoding.html) e [wal2json](https://github.com/eulerto/wal2json).
 

> [!NOTE]
> A decodificação lógica está em visualização pública no banco de dados do Azure para PostgreSQL-servidor único.


## <a name="set-up-your-server"></a>Configurar seu servidor
Para começar a usar a decodificação lógica, habilite o servidor para salvar e transmitir o WAL. 

1. Defina o Azure. replication_support `logical` para usar o CLI do Azure. 
   ```
   az postgres server configuration set --resource-group mygroup --server-name myserver --name azure.replication_support --value logical
   ```

   > [!NOTE]
   > Se você usar réplicas de leitura, o Azure. replication_support `logical` definido como também permite que réplicas sejam executadas. Se você parar de usar a decodificação lógica, altere a configuração `replica`de volta para. 


2. Reinicie o servidor para aplicar as alterações.
   ```
   az postgres server restart --resource-group mygroup --name myserver
   ```

## <a name="start-logical-decoding"></a>Iniciar decodificação lógica

A decodificação lógica pode ser consumida via protocolo de streaming ou interface do SQL. Ambos os métodos usam [Slots de replicação](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html#LOGICALDECODING-REPLICATION-SLOTS). Um slot representa um fluxo de alterações de um banco de dados individual.

O uso de um slot de replicação requer privilégios de replicação do Postgres. Neste momento, o privilégio de replicação só está disponível para o usuário administrador do servidor. 

### <a name="streaming-protocol"></a>Protocolo de streaming
O consumo de alterações usando o protocolo de streaming geralmente é preferível. Você pode criar seu próprio consumidor/conector ou usar uma ferramenta como [Debezium](https://debezium.io/). 

Visite a documentação do wal2json para obter [um exemplo usando o protocolo de streaming com pg_recvlogical](https://github.com/eulerto/wal2json#pg_recvlogical).


### <a name="sql-interface"></a>Interface do SQL
No exemplo a seguir, usamos a interface do SQL com o plug-in wal2json.
 
1. Crie um slot.
   ```SQL
   SELECT * FROM pg_create_logical_replication_slot('test_slot', 'wal2json');
   ```
 
2. Emita comandos SQL. Por exemplo:
   ```SQL
   CREATE TABLE a_table (
      id varchar(40) NOT NULL,
      item varchar(40),
      PRIMARY KEY (id)
   );
   
   INSERT INTO a_table (id, item) VALUES ('id1', 'item1');
   DELETE FROM a_table WHERE id='id1';
   ```

3. Consuma as alterações.
   ```SQL
   SELECT data FROM pg_logical_slot_get_changes('test_slot', NULL, NULL, 'pretty-print', '1');
   ```

   A saída terá a seguinte aparência:
   ```
   {
         "change": [
         ]
   }
   {
         "change": [
                  {
                           "kind": "insert",
                           "schema": "public",
                           "table": "a_table",
                           "columnnames": ["id", "item"],
                           "columntypes": ["character varying(40)", "character varying(40)"],
                           "columnvalues": ["id1", "item1"]
                  }
         ]
   }
   {
         "change": [
                  {
                           "kind": "delete",
                           "schema": "public",
                           "table": "a_table",
                           "oldkeys": {
                                 "keynames": ["id"],
                                 "keytypes": ["character varying(40)"],
                                 "keyvalues": ["id1"]
                           }
                  }
         ]
   }
   ```

4. Descarte o slot quando terminar de usá-lo.
   ```SQL
   SELECT pg_drop_replication_slot('test_slot'); 
   ```


## <a name="monitoring-slots"></a>Slots de monitoramento

Você deve monitorar a decodificação lógica. Qualquer slot de replicação não utilizado deve ser Descartado. Os slots mantêm os logs do postgres WAL e os catálogos de sistema relevantes até que as alterações tenham sido lidas por um consumidor. Se o consumidor falhar ou não tiver sido configurado corretamente, os logs não consumidos irão compilar e preencher o armazenamento. Além disso, os logs não consumidos aumentam o risco da ID de transação delimitador. Ambas as situações podem fazer com que o servidor fique indisponível. Portanto, é essencial que os slots de replicação lógica sejam consumidos continuamente. Se um slot de replicação lógica não for mais usado, descarte-o imediatamente.

A coluna ' ativa ' na exibição pg_replication_slots indicará se há um consumidor conectado a um slot.
```SQL
SELECT * FROM pg_replication_slots;
```

[Defina alertas](howto-alert-on-metric.md) no *armazenamento usado* e no *atraso máximo entre* as métricas de réplicas para notificá-lo quando os valores aumentarem os limites normais anteriores. 

> [!IMPORTANT]
> Você deve remover slots de replicação não utilizados. A falha ao fazer isso pode levar à indisponibilidade do servidor.

## <a name="how-to-drop-a-slot"></a>Como descartar um slot
Se você não estiver consumindo ativamente um slot de replicação, descarte-o.

Para remover um slot de replicação `test_slot` chamado usando SQL:
```SQL
SELECT pg_drop_replication_slot('test_slot');
```

> [!IMPORTANT]
> Se você parar de usar a decodificação lógica, altere o Azure. `replica` replication_support `off`de volta para ou. Os detalhes de WAL retidos pelo `logical` são mais detalhados e devem ser desabilitados quando a decodificação lógica não estiver em uso. 

 
## <a name="next-steps"></a>Próximas etapas

* Visite a documentação do postgres para [saber mais sobre a decodificação lógica](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html).
* Entre em contato com a [nossa equipe](mailto:AskAzureDBforPostgreSQL@service.microsoft.com) se você tiver dúvidas sobre a decodificação lógica.
* Saiba mais sobre [réplicas de leitura](concepts-read-replicas.md).

