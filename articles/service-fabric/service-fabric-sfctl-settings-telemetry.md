---
title: CLI do Azure Service Fabric-telemetria de configurações de sfctl
description: Saiba mais sobre o sfctl, a interface de linha de comando Service Fabric do Azure. Inclui uma lista de comandos para configurar a telemetria do sfctl.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 6af5fa944ef399756f9e890ddd77a7f5f32e2bfb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "76903019"
---
# <a name="sfctl-settings-telemetry"></a>sfctl settings telemetry
Defina as configurações de telemetria locais para esta instância do sfctl.

A telemetria do sfctl coleta o nome do comando sem parâmetros fornecidos ou seus valores, versão do sfctl, tipo de sistema operacional, versão do Python, êxito ou falha do comando e a mensagem de erro retornada.

## <a name="commands"></a>Comandos

|Comando|Descrição|
| --- | --- |
| set-telemetry | Ativa ou desliga a telemetria. |

## <a name="sfctl-settings-telemetry-set-telemetry"></a>set-telemetry da telemetria de configurações do sfctl
Ativa ou desliga a telemetria.

### <a name="arguments"></a>Argumentos

|Argumento|Descrição|
| --- | --- |
| --off | Desliga a telemetria. |
| --on | Ativa a telemetria. Este é o valor padrão. |

### <a name="global-arguments"></a>Argumentos globais

|Argumento|Descrição|
| --- | --- |
| --debug | Aumente o detalhamento do log para mostrar todos os logs de depuração. |
| --help -h | Mostrar esta mensagem de ajuda e sair. |
| --output -o | Formato de saída.  Valores permitidos\: json, jsonc, tabela, tsv.  Padrão\: json. |
| --query | Cadeia de caracteres de consulta JMESPath. Veja http\://jmespath.org/ para saber mais e obter exemplos. |
| --verbose | Aumentar o detalhamento do log. Use --debug para logs de depuração completos. |

### <a name="examples"></a>Exemplos

Desliga a telemetria.

```
sfctl settings telemetry set_telemetry --off
```

Ativa a telemetria.

```
sfctl settings telemetry set_telemetry --on
```


## <a name="next-steps"></a>Próximas etapas
- [Configure](service-fabric-cli.md) a CLI do Service Fabric.
- Saiba como usar a CLI do Service Fabric usando os [scripts de exemplo](/azure/service-fabric/scripts/sfctl-upgrade-application).