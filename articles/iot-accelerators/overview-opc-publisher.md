---
title: O que é o OPC Publisher - Azure | Microsoft Docs
description: Este artigo fornece uma visão geral dos recursos do OPC Publisher. Ele permite que você publique dados de telemetria JSON codificados usando uma carga JSON, para o Hub IoT do Azure.
author: dominicbetts
ms.author: dobett
ms.date: 06/10/2019
ms.topic: overview
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: c77dff4a4f89a78dc0a0d723fbb45fe691246112
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/29/2020
ms.locfileid: "81681928"
---
# <a name="what-is-opc-publisher"></a>O que é o OPC Publisher?

OPC Publisher é uma implementação de referência que demonstra como:

- Conectar-se a servidores UA OPC existentes.
- Publicar dados de telemetria codificados em JSON de servidores de UA OPC no formato Pub/Sub UA OPC, usando uma carga JSON, para o Hub IoT do Azure.

Você pode usar qualquer um dos protocolos de transporte com suporte do SDK do cliente do Hub IoT do Azure: HTTPS, AMQP e MQTT.

A implementação de referência inclui:

- Um *cliente* UA OPC para se conectar a servidores UA OPC existentes em sua rede.
- Um *servidor* UA OPC na porta 62222 que você pode usar para gerenciar o que é publicado e oferece métodos diretos do Hub IoT para fazer o mesmo.

Você pode baixar a [implementação de referência do OPC Publisher](https://github.com/Azure/iot-edge-opc-publisher) do GitHub.

O aplicativo é implementado usando a tecnologia .NET Core e pode ser executado em plataformas com suporte do .NET Core.

O OPC Publisher implementa a lógica de repetição para estabelecer conexões com pontos de extremidade que não respondem a um determinado número de solicitações de keep alive. Por exemplo, se um servidor UA OPC parar de responder devido a uma interrupção de energia.

Para cada intervalo de publicação distinto em um servidor UA OPC, o aplicativo cria uma assinatura separada na qual todos os nós no intervalo de publicação são atualizados.

O OPC Publisher oferece suporte ao envio em lote dos dados ao Hub IoT para reduzir a carga de rede. Este envio em lote envia um pacote ao Hub IoT somente se o tamanho do pacote configurado for alcançado.

Esse aplicativo usa a pilha de referência OPC UA da OPC Foundation como pacotes NuGet. Confira os [https://opcfoundation.org/license/redistributables/1.3/](https://opcfoundation.org/license/redistributables/1.3/) para os termos de licenciamento.

### <a name="next-steps"></a>Próximas etapas

Você aprendeu o que é o OPC Publisher, a próxima etapa sugerida é saber como [Configurar o OPC Publisher](howto-opc-publisher-configure.md).
