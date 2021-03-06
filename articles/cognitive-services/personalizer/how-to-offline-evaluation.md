---
title: Como executar a avaliação offline – personalizador
titleSuffix: Azure Cognitive Services
description: Este artigo mostrará como usar a avaliação offline para medir a eficácia de seu aplicativo e analisar seu loop de aprendizado.
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 02/20/2020
ms.author: diberry
ms.openlocfilehash: ce85c2d264b2b4849a4a36ed757150292fdf39f0
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/29/2020
ms.locfileid: "77622770"
---
# <a name="analyze-your-learning-loop-with-an-offline-evaluation"></a>Analise seu loop de aprendizagem com uma avaliação offline

Saiba como concluir uma avaliação offline e entender os resultados.

As avaliações offline permitem que você meça a eficiência do personalizador em comparação com o comportamento padrão do aplicativo, saiba quais recursos estão contribuindo com a personalização e descubra automaticamente novos valores de aprendizado de máquina.

Leia sobre [avaliações offline](concepts-offline-evaluation.md) para saber mais.

## <a name="prerequisites"></a>Pré-requisitos

* Um loop personalizado configurado
* O loop personalizador deve ter uma quantidade representativa de dados – como um aproximada, é recomendável pelo menos 50.000 eventos em seus logs para obter resultados significativos de avaliação. Opcionalmente, talvez você também tenha exportado anteriormente arquivos da _política de aprendizado_, que podem ser comparados e testados na mesma avaliação.

## <a name="run-an-offline-evaluation"></a>Executar uma avaliação offline

1. Na [portal do Azure](https://azure.microsoft.com/free/), localize seu recurso personalizador.
1. Na portal do Azure, vá para a seção **avaliações** e selecione **criar avaliação**.
    ![Na portal do Azure, vá para a seção * * avaliações * * e selecione * * criar avaliação * *.](./media/offline-evaluation/create-new-offline-evaluation.png)
1. Configure os seguintes valores:

    * Um nome de avaliação.
    * Data de início e de término – são datas que especificam o intervalo de dados a ser usado na avaliação. Esses dados devem estar presentes nos logs, conforme especificado no valor de [retenção de dados](how-to-settings.md) .
    * Descoberta de otimização definida como **Sim**.

    > [!div class="mx-imgBorder"]
    > ![Escolher configurações de avaliação offline](./media/offline-evaluation/create-an-evaluation-form.png)

1. Inicie a avaliação selecionando **OK**.

## <a name="review-the-evaluation-results"></a>Examinar os resultados da avaliação

As avaliações podem levar muito tempo para serem executadas, dependendo da quantidade de dados a serem processados, do número de políticas de aprendizado para comparação e se uma otimização foi solicitada.

Depois de concluído, você pode selecionar a avaliação na lista de avaliações e, em seguida, selecionar **comparar a pontuação de seu aplicativo com outras configurações de aprendizado potenciais**. Selecione esse recurso quando desejar ver como sua política de aprendizado atual é executada em comparação com uma nova política.

1. Examine o desempenho das [políticas de aprendizado](concepts-offline-evaluation.md#discovering-the-optimized-learning-policy).

    > [!div class="mx-imgBorder"]
    > [![Examinar resultados da avaliação](./media/offline-evaluation/evaluation-results.png)](./media/offline-evaluation/evaluation-results.png#lightbox)

1. Selecione **aplicar** para aplicar a política que melhora o modelo melhor para seus dados.

## <a name="next-steps"></a>Próximas etapas

* Saiba mais sobre [como funcionam as avaliações offline](concepts-offline-evaluation.md).
