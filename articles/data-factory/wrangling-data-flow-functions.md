---
title: Funções de transformação de fluxo de dados do Wrangling no Azure Data Factory
description: Uma visão geral das funções de fluxo de dados do Wrangling disponíveis no Azure Data Factory
author: djpmsft
ms.author: daperlov
ms.reviewer: gamal
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/01/2019
ms.openlocfilehash: 0a0947a5e2b57f728023b0f923428814b3e439ec
ms.sourcegitcommit: 1895459d1c8a592f03326fcb037007b86e2fd22f
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/01/2020
ms.locfileid: "82626676"
---
# <a name="transformation-functions-in-wrangling-data-flow"></a>Funções de transformação no fluxo de dados Wrangling

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

O fluxo de dados do Wrangling no Azure Data Factory permite que você faça a preparação de dados ágil sem código e Wrangling em escala de nuvem. O fluxo de dados Wrangling integra-se com o [Power Query online](https://docs.microsoft.com/powerquery-m/power-query-m-reference) e faz Power Query funções M disponíveis para os dados Wrangling por meio da execução do Spark. 

Atualmente, não há suporte para todas as funções Power Query M para Wrangling de dados, apesar de estarem disponíveis durante a criação. Ao criar os fluxos de dados do Wrangling, a seguinte mensagem de erro será exibida se não houver suporte para uma função:

`The wrangling data flow is invalid. Expression.Error: The transformation logic isn't supported. Please try a simpler expression`

Abaixo está uma lista de funções de Power Query M com suporte.

## <a name="column-management"></a>Gerenciamento de coluna

* Seleção: [Table. SelectColumns](https://docs.microsoft.com/powerquery-m/table-selectcolumns)
* Remoção: [Table. RemoveColumns](https://docs.microsoft.com/powerquery-m/table-removecolumns)
* Renomeando: [Table. RenameColumns](https://docs.microsoft.com/powerquery-m/table-renamecolumns), [tabela. PrefixColumns](https://docs.microsoft.com/powerquery-m/table-prefixcolumns), [Table. TransformColumnNames](https://docs.microsoft.com/powerquery-m/table-transformcolumnnames)
* Reordenando: [Table. ReorderColumns](https://docs.microsoft.com/powerquery-m/table-reordercolumns)

## <a name="row-filtering"></a>Filtragem de linha

Use a função M [Table. SelectRows](https://docs.microsoft.com/powerquery-m/table-selectrows) para filtrar as seguintes condições:

* Igualdade e desigualdade
* Comparações numéricas, de texto e de data (mas não DateTime)
* Informações numéricas, como [número. ismesmo](https://docs.microsoft.com/powerquery-m/number-iseven)/[Odd](https://docs.microsoft.com/powerquery-m/number-iseven)
* Contenção de texto usando [Text. Contains](https://docs.microsoft.com/powerquery-m/text-contains), [Text. StartsWith](https://docs.microsoft.com/powerquery-m/text-startswith)ou [Text. EndsWith](https://docs.microsoft.com/powerquery-m/text-endswith)
* Intervalos de datas incluindo todas as ' [funções de data](https://docs.microsoft.com/powerquery-m/date-functions)de IsIn' ') 
* Combinações dessas usando as condições and, or ou not

## <a name="adding-and-transforming-columns"></a>Adicionando e transformando colunas

As seguintes funções M adicionam ou transformam colunas: [Table. AddColumn](https://docs.microsoft.com/powerquery-m/table-addcolumn), [tabela. TransformColumns](https://docs.microsoft.com/powerquery-m/table-transformcolumns), [Table. replacevalue](https://docs.microsoft.com/powerquery-m/table-replacevalue), [Table. DuplicateColumn](https://docs.microsoft.com/powerquery-m/table-duplicatecolumn). Abaixo estão as funções de transformação com suporte.

* Aritmética numérica
* Concatenação de texto
* AndTime de data aritmética (operadores aritméticos, [Data. Subdias](https://docs.microsoft.com/powerquery-m/date-adddays), [Data. AddMonths](https://docs.microsoft.com/powerquery-m/date-addmonths), [Data. addtrimestres](https://docs.microsoft.com/powerquery-m/date-addquarters), [Data. Addsemanas](https://docs.microsoft.com/powerquery-m/date-addweeks), [Data. AddYears](https://docs.microsoft.com/powerquery-m/date-addyears))
* As durações podem ser usadas para aritmética de data e hora, mas devem ser transformadas em outro tipo antes de serem gravadas em um coletor (operadores aritméticos, [#duration](https://docs.microsoft.com/powerquery-m/sharpduration), [duração. dias](https://docs.microsoft.com/powerquery-m/duration-days), [duração. horas](https://docs.microsoft.com/powerquery-m/duration-hours), duração. [minutos](https://docs.microsoft.com/powerquery-m/duration-minutes), [duração. segundos](https://docs.microsoft.com/powerquery-m/duration-seconds), [duração. TotalDays](https://docs.microsoft.com/powerquery-m/duration-totaldays), duração. [TotalHours](https://docs.microsoft.com/powerquery-m/duration-totalhours), [duração. TotalMinutes](https://docs.microsoft.com/powerquery-m/duration-totalminutes), [duração. TotalSeconds](https://docs.microsoft.com/powerquery-m/duration-totalseconds))    
* A maioria das funções numéricas padrão, científicas e trigonométricas (todas as funções em [operações](https://docs.microsoft.com/powerquery-m/number-functions#operations), [arredondamento](https://docs.microsoft.com/powerquery-m/number-functions#rounding)e [trigonometria](https://docs.microsoft.com/powerquery-m/number-functions#trigonometry) , *exceto* Number. fatorial, Number. permutates e Number. combinações)
* Substituição ([replacer. ReplaceText](https://docs.microsoft.com/powerquery-m/replacer-replacetext), [replaceer. replacevalue](https://docs.microsoft.com/powerquery-m/replacer-replacevalue), [Text. Replace](https://docs.microsoft.com/powerquery-m/text-replace), [Text. Remove](https://docs.microsoft.com/powerquery-m/text-remove))
* Extração de texto posicional ([Text. PositionOf](https://docs.microsoft.com/powerquery-m/text-positionof), [Text. Length](https://docs.microsoft.com/powerquery-m/text-length), [Text. Start](https://docs.microsoft.com/powerquery-m/text-start), [Text. end](https://docs.microsoft.com/powerquery-m/text-end), [Text. middle](https://docs.microsoft.com/powerquery-m/text-middle), [Text. ReplaceRange](https://docs.microsoft.com/powerquery-m/text-replacerange), [Text. RemoveRange](https://docs.microsoft.com/powerquery-m/text-removerange))
* Formatação de texto básica[(Text. Low](https://docs.microsoft.com/powerquery-m/text-lower), [Text. Upper](https://docs.microsoft.com/powerquery-m/text-upper), [Text. arrumar](https://docs.microsoft.com/powerquery-m/text-trim)/[Start](https://docs.microsoft.com/powerquery-m/text-trimstart)/[end](https://docs.microsoft.com/powerquery-m/text-trimend), [Text. PadStart](https://docs.microsoft.com/powerquery-m/text-padstart)/[end](https://docs.microsoft.com/powerquery-m/text-padend), [Text. Reverse](https://docs.microsoft.com/powerquery-m/text-reverse))
* Funções de data/hora ([Data. dia](https://docs.microsoft.com/powerquery-m/date-day), [Data. mês](https://docs.microsoft.com/powerquery-m/date-month), [Data. ano](https://docs.microsoft.com/powerquery-m/date-year) [hora. hora](https://docs.microsoft.com/powerquery-m/time-hour), [hora. minuto](https://docs.microsoft.com/powerquery-m/time-minute), [hora. segundo](https://docs.microsoft.com/powerquery-m/time-second), [Data. DayOfWeek](https://docs.microsoft.com/powerquery-m/date-dayofweek), [Data. DayOfYear](https://docs.microsoft.com/powerquery-m/date-dayofyear), [Date. DaysInMonth](https://docs.microsoft.com/powerquery-m/date-daysinmonth))
* Se as expressões (mas as ramificações devem ter tipos correspondentes)
* Filtros de linha como uma coluna lógica
* Constantes de número, texto, lógica, data e DateTime

<a name="mergingjoining-tables"></a>Mesclando/Unindo tabelas
----------------------
* Power Query irá gerar uma junção aninhada (Table. NestedJoin; os usuários também podem gravar manualmente [Table. AddJoinColumn](https://docs.microsoft.com/powerquery-m/table-addjoincolumn)).
    Os usuários devem expandir a coluna de junção aninhada para uma junção não aninhada (Table. ExpandTableColumn, sem suporte em nenhum outro contexto).
* A tabela de função M [. Join](https://docs.microsoft.com/powerquery-m/table-join) pode ser gravada diretamente para evitar a necessidade de uma etapa de expansão adicional, mas o usuário deve garantir que não haja nenhum nome de coluna duplicado entre as tabelas unidas
* Tipos de junção com suporte: [interno](https://docs.microsoft.com/powerquery-m/joinkind-inner), [LeftOuter](https://docs.microsoft.com/powerquery-m/joinkind-leftouter), [RightOuter](https://docs.microsoft.com/powerquery-m/joinkind-rightouter), [FullOuter](https://docs.microsoft.com/powerquery-m/joinkind-fullouter)
* Ambos os [valores. Equals](https://docs.microsoft.com/powerquery-m/value-equals) e [Value. NullableEquals](https://docs.microsoft.com/powerquery-m/value-nullableequals) têm suporte como comparadores de igualdade de chave

## <a name="group-by"></a>Agrupar por

Use [Table. Group](https://docs.microsoft.com/powerquery-m/table-group) para agregar valores.
* Deve ser usado com uma função de agregação
* Funções de agregação com suporte: [tabela. RowCount](https://docs.microsoft.com/powerquery-m/table-rowcount), [lista. soma](https://docs.microsoft.com/powerquery-m/list-sum), [lista. contagem](https://docs.microsoft.com/powerquery-m/list-count), [lista. média](https://docs.microsoft.com/powerquery-m/list-average), [lista. mín](https://docs.microsoft.com/powerquery-m/list-min)., [lista. máx](https://docs.microsoft.com/powerquery-m/list-max). [lista. i](https://docs.microsoft.com/powerquery-m/list-standarddeviation), lista. [primeiro](https://docs.microsoft.com/powerquery-m/list-first), lista [. último](https://docs.microsoft.com/powerquery-m/list-last)

## <a name="sorting"></a>Classificação

Use [Table. Sort](https://docs.microsoft.com/powerquery-m/table-sort) para classificar valores.

## <a name="reducing-rows"></a>Reduzindo linhas

Manter e remover Top, manter o intervalo (funções de M correspondentes, somente contagens de suporte, não condições: [Table. firstn](https://docs.microsoft.com/powerquery-m/table-firstn), [tabela. Skip](https://docs.microsoft.com/powerquery-m/table-skip), [tabela. removefirstn](https://docs.microsoft.com/powerquery-m/table-removefirstn), [tabela. Range](https://docs.microsoft.com/powerquery-m/table-range), tabela. [Minnesota](https://docs.microsoft.com/powerquery-m/table-minn), [Table. MaxN](https://docs.microsoft.com/powerquery-m/table-maxn))

## <a name="known-unsupported-functions"></a>Funções conhecidas sem suporte

| Função | Status |
| -- | -- |
| Table.PromoteHeaders | Não há suporte. O mesmo resultado pode ser obtido com a configuração "primeira linha como cabeçalho" no conjunto de resultados. |
| Table.CombineColumns | Esse é um cenário comum que não tem suporte direto, mas pode ser obtido com a adição de uma nova coluna que concatena duas colunas especificadas.  Por exemplo, Table. AddColumn (RemoveEmailColumn, "Name", cada [FirstName] & "" & [LastName]) |
| Table.TransformColumnTypes | Isso tem suporte na maioria dos casos. Os cenários a seguir não têm suporte: transformando cadeia de caracteres para tipo de moeda, transformando cadeia de caracteres em tipo de hora, transformando cadeia de caracteres para tipo de porcentagem. |
| Table.NestedJoin | Apenas fazer uma junção resultará em um erro de validação. As colunas devem ser expandidas para que funcionem. |
| Table.Distinct | Não há suporte para a remoção de linhas duplicadas. |
| Table.RemoveLastN | Não há suporte para remover linhas inferiores. |
| Table.RowCount | Sem suporte, mas pode ser obtido com a adição de uma coluna com todas as células vazia (a coluna de condição pode ser usada) e, em seguida, o uso de Group by nessa coluna. Table. Group tem suporte. | 
| Tratamento de erros em nível de linha | No momento, não há suporte para o tratamento de erros em nível de linha. Por exemplo, para filtrar valores não numéricos de uma coluna, uma abordagem seria transformar a coluna de texto em um número. Todas as células que não forem transformadas na transformação estarão em um estado de erro e precisarão ser filtradas. Esse cenário não é possível no fluxo de dados Wrangling. |
| Table.Transpose | Sem suporte |
| Table.Pivot | Sem suporte |

## <a name="next-steps"></a>Próximas etapas

Saiba como [criar um fluxo de dados do Wrangling](wrangling-data-flow-tutorial.md).