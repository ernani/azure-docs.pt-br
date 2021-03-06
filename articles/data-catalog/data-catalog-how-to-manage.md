---
title: Gerenciar ativos de dados no Catálogo de Dados do Azure
description: O artigo destaca como controlar a visibilidade e a propriedade de ativos de dados registrados no Catálogo de Dados do Azure.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: conceptual
ms.date: 08/01/2019
ms.openlocfilehash: 9905ed72ec54304bbdb0f7ee607cbb013fc645bb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "68736337"
---
# <a name="manage-data-assets-in-azure-data-catalog"></a>Gerenciar ativos de dados no Catálogo de Dados do Azure
## <a name="introduction"></a>Introdução
O Catálogo de Dados do Azure é desenvolvido para descoberta de fonte de dados para que você possa facilmente descobrir e entender as fontes de dados de que você precisa para executar a análise e tomar decisões. Esses recursos de descoberta têm maior impacto quando você e outros usuários podem localizar e compreender a mais ampla variedade de fontes de dados disponíveis. Com esses elementos em mente, o comportamento padrão do Catálogo de Dados é que todas as fontes de dados registradas fiquem visíveis e detectáveis por todos os usuários do catálogo.

O Catálogo de Dados não dá a você acesso aos dados em si. O acesso aos dados é controlado pelo proprietário da fonte de dados. Com o Catálogo de Dados, você pode descobrir fontes de dados e exibir os metadados relacionados às fontes registradas no catálogo.

Porém, pode haver situações em que as fontes de dados só devam estar visíveis a usuários específicos ou membros de grupos específicos. Nesses cenários, os usuários podem assumir a propriedade de ativos de dados registrados no catálogo e então controlar a visibilidade de ativos de propriedade deles.

> [!NOTE]
> A funcionalidade descrita neste artigo está disponível apenas na Edição Standard do Catálogo de Dados do Azure. A Edição Gratuita não fornece recursos para propriedade e restrição da visibilidade de ativos de dados.
>
>

## <a name="manage-ownership-of-data-assets"></a>Gerenciar a propriedade de ativos de dados
Por padrão, ativos de dados registrados no Catálogo de Dados não têm proprietário. Qualquer usuário com permissão para acessar o catálogo pode descobrir e anotar esses ativos. Os usuários podem se apropriar de ativos de dados sem proprietário e limitar a visibilidade dos ativos de propriedade eles.

Quando um ativo de dados no Catálogo de Dados tem um proprietário, apenas usuários autorizados pelos proprietários podem descobrir o ativo e exibir seus metadados e apenas os proprietários podem excluir o ativo do catálogo.

> [!NOTE]
> A propriedade no Catálogo de Dados afeta apenas os metadados armazenados no catálogo. A propriedade não concede nenhuma permissão para a fonte de dados subjacente.
>
>

### <a name="take-ownership"></a>Apropriar-se
Os usuários podem assumir a propriedade dos ativos de dados selecionando a opção **apropriar-se** no portal do catálogo de dados. Nenhuma permissão especial é necessária para apropriar-se de um ativo de dados sem proprietário. Qualquer usuário pode apropriar-se de um ativo de dados sem proprietário.

### <a name="add-owners-and-co-owners"></a>Adicionar proprietários e coproprietários
Se um ativo de dados já tiver um proprietário, outros usuários não poderão simplesmente assumir a propriedade. Eles deverão ser adicionados como coproprietários por um proprietário existente. Qualquer proprietário pode adicionar outros usuários ou grupos de segurança como coproprietários.

> [!NOTE]
> É uma prática recomendada ter pelo menos dois indivíduos como proprietários para qualquer ativo de dados com proprietário.
>
>

### <a name="remove-owners"></a>Remover proprietários
Assim como qualquer proprietário ativo pode adicionar coproprietários, qualquer proprietário ativo pode remover qualquer coproprietário.

Um proprietário de ativo que se remove como um proprietário não pode mais gerenciar o ativo. Se o proprietário do ativo se remover como um proprietário e não houver outros coproprietários, o ativo será revertido para um estado sem proprietário.

## <a name="control-visibility"></a>Controlar a visibilidade
Os proprietários de ativos de dados podem controlar a visibilidade dos ativos de dados de propriedade eles. Para restringir a visibilidade como o padrão, em que todos os usuários do Catálogo de Dados podem descobrir e exibir o ativo de dados, o proprietário do ativo pode alternar a configuração de visibilidade de **Todos** para **Proprietários e Estes Usuários** nas propriedades do ativo. Em seguida, os proprietários podem adicionar usuários e grupos de segurança específicos.

> [!NOTE]
> Sempre que possível, as permissões de propriedade e visibilidade de ativo devem ser atribuídas a grupos de segurança e não a usuários individuais.
>
>

## <a name="catalog-administrators"></a>Administradores do Catálogo
Os administradores do Catálogo de Dados são, implicitamente, coproprietários de todos os ativos no catálogo. Os proprietários do ativo não podem remover a visibilidade dos administradores e os administradores podem gerenciar a propriedade e a visibilidade de todos os ativos de dados no catálogo.

## <a name="summary"></a>Resumo
O modelo de crowdsourcing do Catálogo de Dados para descoberta de ativos de dados e metadados permite que todos os usuários do catálogo contribuam e descubram. A Edição Standard do Catálogo de Dados é desenvolvida para propriedade e gerenciamento para limitar a visibilidade e o uso de ativos de dados específicos.
