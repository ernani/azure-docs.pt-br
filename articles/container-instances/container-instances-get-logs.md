---
title: Obter logs de instância de contêiner & eventos
description: Saiba como recuperar logs de contêiner e eventos em instâncias de contêiner do Azure para ajudar a solucionar problemas de contêiner
ms.topic: article
ms.date: 12/30/2019
ms.custom: mvc
ms.openlocfilehash: 0991b9cb1f99606910dbdf2c87b111f67da6da7b
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "78249989"
---
# <a name="retrieve-container-logs-and-events-in-azure-container-instances"></a>Recuperar logs e eventos de contêiner nas Instâncias de Contêiner do Azure

Quando você tiver um contêiner com comportamento inadequado nas instâncias de contêiner do Azure, comece exibindo seus logs com [AZ container logs][az-container-logs]e transmita seu erro padrão e padrão com [AZ container Attach][az-container-attach]. Você também pode exibir logs e eventos para instâncias de contêiner no portal do Azure ou enviar dados de log e de evento para grupos de contêineres para [Azure monitor logs](container-instances-log-analytics.md).

## <a name="view-logs"></a>Exibir logs

Para exibir logs do seu código de aplicativo em um contêiner, você pode usar o comando [az container logs][az-container-logs].

A seguir está a saída de log do contêiner de exemplo baseado em tarefa em [defina a linha de comando em uma instância de contêiner](container-instances-start-command.md#azure-cli-example), depois de ter fornecido uma URL inválida usando uma substituição de linha de comando:

```azurecli
az container logs --resource-group myResourceGroup --name mycontainer
```

```output
Traceback (most recent call last):
  File "wordcount.py", line 11, in <module>
    urllib.request.urlretrieve (sys.argv[1], "foo.txt")
  File "/usr/local/lib/python3.6/urllib/request.py", line 248, in urlretrieve
    with contextlib.closing(urlopen(url, data)) as fp:
  File "/usr/local/lib/python3.6/urllib/request.py", line 223, in urlopen
    return opener.open(url, data, timeout)
  File "/usr/local/lib/python3.6/urllib/request.py", line 532, in open
    response = meth(req, response)
  File "/usr/local/lib/python3.6/urllib/request.py", line 642, in http_response
    'http', request, response, code, msg, hdrs)
  File "/usr/local/lib/python3.6/urllib/request.py", line 570, in error
    return self._call_chain(*args)
  File "/usr/local/lib/python3.6/urllib/request.py", line 504, in _call_chain
    result = func(*args)
  File "/usr/local/lib/python3.6/urllib/request.py", line 650, in http_error_default
    raise HTTPError(req.full_url, code, msg, hdrs, fp)
urllib.error.HTTPError: HTTP Error 404: Not Found
```

## <a name="attach-output-streams"></a>Anexar fluxos de saída

O comando [az container attach][az-container-attach] fornece informações de diagnóstico durante a inicialização do contêiner. Depois que o contêiner for iniciado, ele transmite STDOUT e STDERR para o console local.

Por exemplo, aqui está a saída do contêiner baseado em tarefa em [definir a linha de comando em uma instância de contêiner](container-instances-start-command.md#azure-cli-example), depois de ter fornecido uma URL válida de um arquivo de texto grande para processar:

```azurecli
az container attach --resource-group myResourceGroup --name mycontainer
```

```output
Container 'mycontainer' is in state 'Unknown'...
Container 'mycontainer' is in state 'Waiting'...
Container 'mycontainer' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 19:42:39+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
Container 'mycontainer1' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 19:42:39+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
(count: 1) (last timestamp: 2019-03-21 19:42:52+00:00) Successfully pulled image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
(count: 1) (last timestamp: 2019-03-21 19:42:55+00:00) Created container
(count: 1) (last timestamp: 2019-03-21 19:42:55+00:00) Started container

Start streaming logs:
[('the', 22979),
 ('I', 20003),
 ('and', 18373),
 ('to', 15651),
 ('of', 15558),
 ('a', 12500),
 ('you', 11818),
 ('my', 10651),
 ('in', 9707),
 ('is', 8195)]
```

## <a name="get-diagnostic-events"></a>Obtendo eventos de diagnóstico

Se o contêiner não for implantado com êxito, examine as informações de diagnóstico fornecidas pelo provedor de recursos das instâncias de contêiner do Azure. Para exibir os eventos para o contêiner, execute o comando [az container show][az-container-show]:

```azurecli-interactive
az container show --resource-group myResourceGroup --name mycontainer
```

A saída inclui as propriedades principais do contêiner, juntamente com eventos de implantação (exibidos aqui truncados):

```JSON
{
  "containers": [
    {
      "command": null,
      "environmentVariables": [],
      "image": "mcr.microsoft.com/azuredocs/aci-helloworld",
      ...
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:22+00:00",
            "lastTimestamp": "2019-03-21T19:46:22+00:00",
            "message": "pulling image \"mcr.microsoft.com/azuredocs/aci-helloworld\"",
            "name": "Pulling",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:28+00:00",
            "lastTimestamp": "2019-03-21T19:46:28+00:00",
            "message": "Successfully pulled image \"mcr.microsoft.com/azuredocs/aci-helloworld\"",
            "name": "Pulled",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:31+00:00",
            "lastTimestamp": "2019-03-21T19:46:31+00:00",
            "message": "Created container",
            "name": "Created",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:31+00:00",
            "lastTimestamp": "2019-03-21T19:46:31+00:00",
            "message": "Started container",
            "name": "Started",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "name": "mycontainer",
      "ports": [
        {
          "port": 80,
          "protocol": null
        }
      ],
      ...
    }
  ],
  ...
}
```
## <a name="next-steps"></a>Próximas etapas
Saiba como [solucionar problemas de contêiner e implantação](container-instances-troubleshooting.md) nas Instâncias de Contêiner do Azure.

Saiba como enviar dados de log e de evento para grupos de contêineres para [Azure monitor logs](container-instances-log-analytics.md).

<!-- LINKS - Internal -->
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show