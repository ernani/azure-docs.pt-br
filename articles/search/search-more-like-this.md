---
title: recurso de consulta moreLikeThis (visualização)
titleSuffix: Azure Cognitive Search
description: Descreve o recurso moreLikeThis (versão prévia), que está disponível em versões prévias da API REST do Azure Pesquisa Cognitiva.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 95b9c76a2ff962cb2fa4bacbb1b1e9a953b7014f
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "74873787"
---
# <a name="morelikethis-preview-in-azure-cognitive-search"></a>moreLikeThis (versão prévia) no Azure Pesquisa Cognitiva

> [!IMPORTANT] 
> Esse recurso está atualmente em visualização pública. A funcionalidade de versão prévia é fornecida sem um Contrato de Nível de Serviço e, portanto, não é recomendada para cargas de trabalho de produção. Para obter mais informações, consulte [Termos de Uso Complementares de Versões Prévias do Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). A [API REST versão 2019-05-06-versão prévia](search-api-preview.md) fornece esse recurso. No momento, não há suporte para Portal ou SDK do .NET.

`moreLikeThis=[key]`é um parâmetro de consulta na [API de documentos de pesquisa](https://docs.microsoft.com/rest/api/searchservice/search-documents) que localiza documentos semelhantes ao documento especificado pela chave do documento. Quando uma solicitação de pesquisa é feita com `moreLikeThis`, uma consulta é gerada com termos de pesquisa extraídos do documento fornecido que descrevem melhor o documento. A consulta gerada é usada para fazer a solicitação de pesquisa. Por padrão, o conteúdo de todos os campos pesquisáveis é considerado, menos os campos restritos que você especificou usando o `searchFields` parâmetro. O parâmetro `moreLikeThis` não pode ser usado com o parâmetro de pesquisa, `search=[string]`.

Por padrão, o conteúdo de todos os campos pesquisáveis de nível superior é considerado. Se você quiser especificar campos específicos em vez disso, poderá usar o `searchFields` parâmetro. 

Não é possível `MoreLikeThis` usar em subcampos pesquisáveis em um [tipo complexo](search-howto-complex-data-types.md).

## <a name="examples"></a>Exemplos

Todos os exemplos a seguir usam o exemplo de hotéis de [início rápido: criar um índice de pesquisa no portal do Azure](search-get-started-portal.md).

### <a name="simple-query"></a>Consulta simples

A consulta a seguir localiza documentos cujos campos de descrição são mais semelhantes ao campo do documento de origem, conforme especificado `moreLikeThis` pelo parâmetro:

```
GET /indexes/hotels-sample-index/docs?moreLikeThis=29&searchFields=Description&api-version=2019-05-06-Preview
```

Neste exemplo, a solicitação procura Hotéis semelhantes a um com `HotelId` 29.
Em vez de usar HTTP GET, você também pode `MoreLikeThis` invocar usando http post:

```
POST /indexes/hotels-sample-index/docs/search?api-version=2019-05-06-Preview
    {
      "moreLikeThis": "29",
      "searchFields": "Description"
    }
```

### <a name="apply-filters"></a>Aplicar filtros

`MoreLikeThis`pode ser combinado com outros parâmetros de consulta comuns `$filter`, como. Por exemplo, a consulta pode ser restrita a apenas Hotéis cuja categoria é ' orçamento ' e onde a classificação é maior que 3,5:

```
GET /indexes/hotels-sample-index/docs?moreLikeThis=20&searchFields=Description&$filter=(Category eq 'Budget' and Rating gt 3.5)&api-version=2019-05-06-Preview
```

### <a name="select-fields-and-limit-results"></a>Selecionar campos e limitar os resultados

O `$top` seletor pode ser usado para limitar quantos resultados devem ser retornados em uma `MoreLikeThis` consulta. Além disso, os campos podem ser `$select`selecionados com. Aqui, os três principais hotéis são selecionados junto com sua ID, nome e classificação: 

```
GET /indexes/hotels-sample-index/docs?moreLikeThis=20&searchFields=Description&$filter=(Category eq 'Budget' and Rating gt 3.5)&$top=3&$select=HotelId,HotelName,Rating&api-version=2019-05-06-Preview
```

## <a name="next-steps"></a>Próximas etapas

Você pode usar qualquer ferramenta de teste na Web para experimentar esse recurso.  É recomendável usar o postmaster para este exercício.

> [!div class="nextstepaction"]
> [Explorar as APIs REST do Azure Pesquisa Cognitiva usando o postmaster](search-get-started-postman.md)
