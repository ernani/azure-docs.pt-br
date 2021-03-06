---
title: Diretrizes para a implantação responsável da tecnologia de voz sintética
titleSuffix: Azure Cognitive Services
description: Diretrizes de design geral da Microsoft para o uso da tecnologia de voz sintética. Elas foram desenvolvidas em estudos que a Microsoft realizou com talento de voz, consumidores, bem como indivíduos com fala autoimunes para guiar o desenvolvimento responsável da voz sintética.
services: cognitive-services
author: benoah
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/11/2019
ms.author: benoah
ms.openlocfilehash: a529ac8b7ce16d3ee4463f1b4bc2e8007e5b79b5
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/29/2020
ms.locfileid: "73836773"
---
# <a name="guidelines-for-responsible-deployment-of-synthetic-voice-technology"></a>Diretrizes para a implantação responsável da tecnologia de voz sintética
Aqui estão as diretrizes de design geral da Microsoft para o uso da tecnologia de voz sintética. Elas foram desenvolvidas em estudos que a Microsoft realizou com talento de voz, consumidores, bem como indivíduos com autoimunes de fala para guiar o desenvolvimento responsável da voz sintética.

## <a name="general-considerations"></a>Considerações gerais
Para a implantação da tecnologia de fala sintética, as diretrizes a seguir se aplicam na maioria dos cenários.

### <a name="disclose-when-the-voice-is-synthetic"></a>Divulgar quando a voz for sintética
A divulgação de uma voz é gerada pelo computador não apenas minimiza o risco de resultados prejudiciais de deconcepção, mas também aumenta a confiança na organização que fornece a voz. Saiba mais sobre [como divulgar](concepts-disclosure-guidelines.md).

### <a name="select-appropriate-voice-types-for-your-scenario"></a>Selecione os tipos de voz apropriados para seu cenário
Considere cuidadosamente o contexto de uso e os possíveis danos associados ao uso de voz sintética. Por exemplo, vozes sintéticas de alta fidelidade podem não ser apropriadas em cenários de alto risco, como para mensagens pessoais, transações financeiras ou situações complexas que exigem adaptação humana ou empatia. Os usuários também podem ter expectativas diferentes para tipos de voz. Por exemplo, ao escutar notícias confidenciais sendo lidas por uma voz sintética, alguns usuários preferem uma leitura mais empática e humana das notícias, enquanto outros preferiram uma voz mais monotônico e sem tendência. Considere testar seu aplicativo para entender melhor as preferências do usuário.

### <a name="be-transparent-about-capabilities-and-limitations"></a>Seja transparente sobre recursos e limitações
É mais provável que os usuários tenham mais expectativas ao interagir com agentes de voz sintéticos de alta fidelidade. Consequentemente, quando os recursos do sistema não atendem a essas expectativas, a confiança pode sofrer e pode resultar em experiências desagradáveis ou ainda prejudiciais.

### <a name="provide-optional-human-support"></a>Fornecer suporte humano opcional
Em cenários ambíguos e transacionais (por exemplo, um Call Center de suporte), os usuários nem sempre confiam em um agente de computador para responder adequadamente às suas solicitações. O suporte humano pode ser necessário nessas situações, independentemente da qualidade realista da voz ou do recurso do sistema.

## <a name="considerations-for-voice-talent"></a>Considerações sobre talento de voz
Ao trabalhar com talento de voz, como atores de voz, para criar vozes sintéticas, a diretriz abaixo se aplica.

### <a name="obtain-meaningful-consent-from-voice-talent"></a>Obtenha um consentimento significativo do talento de voz
O talento de voz espera ter controle sobre sua fonte de voz (como e onde será usado) e ser compensado sempre que for usado. Portanto, os proprietários do sistema devem obter permissão explícita por escrito do talento de voz e ter especificações contratuais claras sobre casos de uso, duração de uso, compensação e assim por diante. Alguns talentos de voz não sabem dos possíveis usos mal-intencionados da tecnologia e devem ser instruídos por proprietários do sistema sobre os recursos da tecnologia. Para saber mais sobre talento e consentimento de voz, leia nossa [divulgação de talento de voz](https://aka.ms/disclosure-voice-talent).


## <a name="considerations-for-those-with-speech-disorders"></a>Considerações para aqueles com autoimunes de fala
Ao trabalhar com indivíduos com o Speech autoimunes, para criar ou implantar a tecnologia de voz sintética, as diretrizes a seguir se aplicam.

### <a name="provide-guidelines-to-establish-contracts"></a>Fornecer diretrizes para estabelecer contratos
Forneça diretrizes para estabelecer contratos com indivíduos que usam voz sintética para assistência em termos de fala. O contrato deve considerar a especificação das partes que possuem a voz, a duração de uso, os critérios de transferência de propriedade, os procedimentos para excluir a fonte de voz e como impedir o acesso não autorizado. Além disso, habilite a transferência contratuais da propriedade da fonte de voz após a morte para os membros da família se essa pessoa tiver concedido permissão.

### <a name="account-for-inconsistencies-in-speech-patterns"></a>Considerar inconsistências em padrões de fala
Para pessoas com autoimunes de fala que registram suas próprias fontes de voz, inconsistências no seu padrão de fala (slurring ou incapacidade de pronunciar determinadas palavras) podem complicar o processo de gravação. Nesses casos, a tecnologia de voz sintética e as sessões de gravação devem acomodá-las (ou seja, fornecer quebras e número adicional de sessões de gravação).

### <a name="allow-modification-over-time"></a>Permitir modificação ao longo do tempo
Indivíduos com autoimunes de fala para fazer atualizações em sua voz sintética para refletir a expiração (por exemplo, um filho que atinja Puberty). Os indivíduos também podem ter preferências estilísticos que mudam ao longo do tempo e talvez queiram fazer alterações de Tom, acentuação ou outras características de voz.


## <a name="reference-docs"></a>Documentos de Referência

* [Divulgação para talento de voz](https://aka.ms/disclosure-voice-talent)
* [Visão geral da retenção](concepts-gating-overview.md)
* [Como divulgar](concepts-disclosure-guidelines.md)
* [Padrões de design de divulgação](concepts-disclosure-patterns.md)

## <a name="next-steps"></a>Próximas etapas

* [Divulgação para talento de voz](https://aka.ms/disclosure-voice-talent)
* [Como divulgar](concepts-disclosure-guidelines.md)
* [Padrões de design de divulgação](concepts-disclosure-patterns.md)
