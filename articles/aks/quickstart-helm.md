---
title: Desenvolver no AKS (serviço kubernetes do Azure) com o Helm
description: Use Helm com AKS e registro de contêiner do Azure para empacotar e executar contêineres de aplicativo em um cluster.
services: container-service
author: zr-msft
ms.topic: article
ms.date: 04/20/2020
ms.author: zarhoads
ms.openlocfilehash: 1f67605918e093e9ab28aa88be777d27acd831ef
ms.sourcegitcommit: b1e25a8a442656e98343463aca706f4fde629867
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/27/2020
ms.locfileid: "82169561"
---
# <a name="quickstart-develop-on-azure-kubernetes-service-aks-with-helm"></a>Início rápido: desenvolver no serviço de kubernetes do Azure (AKS) com Helm

[Helm][helm] é uma ferramenta de empacotamento de software livre que ajuda a instalar e gerenciar o ciclo de vida de aplicativos Kubernetes. Semelhante a gerenciadores de pacotes do Linux, como *APT* e *Yum*, o Helm é usado para gerenciar gráficos Kubernetes, que são pacotes de recursos de Kubernetes pré-configurados.

Este artigo mostra como usar o Helm para empacotar e executar um aplicativo no AKS. Para obter mais detalhes sobre como instalar um aplicativo existente usando o Helm, consulte [instalar aplicativos existentes com o Helm no AKs][helm-existing].

## <a name="prerequisites"></a>Pré-requisitos

* Uma assinatura do Azure. Se você não tiver uma assinatura do Azure, poderá criar uma [conta gratuita](https://azure.microsoft.com/free).
* A [CLI do Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) instalada.
* Docker instalado e configurado. O Docker fornece pacotes que o configuram facilmente em qualquer sistema [Mac][docker-for-mac], [Windows][docker-for-windows] ou [Linux][docker-for-linux].
* [Helm v3 instalado][helm-install].

## <a name="create-an-azure-container-registry"></a>Criar um Registro de Contêiner do Azure
Para usar o Helm para executar seu aplicativo no cluster do AKS, você precisa de um registro de contêiner do Azure para armazenar suas imagens de contêiner. O exemplo abaixo usa [AZ ACR Create][az-acr-create] para criar um ACR denominado *MyHelmACR* no grupo de recursos *MyResource* Group com a SKU *básica* . Você deve fornecer seu próprio nome de registro exclusivo. O nome do registro deve ser exclusivo no Azure e conter de 5 a 50 caracteres alfanuméricos. A SKU *Basic* é um ponto de entrada de otimização de custo para fins de desenvolvimento que fornece um equilíbrio entre o armazenamento e taxa de transferência.

```azurecli
az group create --name MyResourceGroup --location eastus
az acr create --resource-group MyResourceGroup --name MyHelmACR --sku Basic
```

A saída deverá ser semelhante ao seguinte exemplo: Anote o valor de *loginServer* para seu ACR, pois ele será usado em uma etapa posterior. No exemplo abaixo, *myhelmacr.azurecr.Io* é o *loginServer* para *myhelmacr*.

```console
{
  "adminUserEnabled": false,
  "creationDate": "2019-06-11T13:35:17.998425+00:00",
  "id": "/subscriptions/<ID>/resourceGroups/MyResourceGroup/providers/Microsoft.ContainerRegistry/registries/MyHelmACR",
  "location": "eastus",
  "loginServer": "myhelmacr.azurecr.io",
  "name": "MyHelmACR",
  "networkRuleSet": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "MyResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

Para usar a instância do ACR, primeiro você deve entrar. Use o comando [AZ ACR login][az-acr-login] para entrar. O exemplo abaixo entrará em um ACR chamado *MyHelmACR*.

```azurecli
az acr login --name MyHelmACR
```

O comando retorna uma mensagem de *logon bem-sucedido* após a conclusão.

## <a name="create-an-azure-kubernetes-service-cluster"></a>Criar um cluster do Serviço de Kubernetes do Azure

Criar um cluster do AKS. O comando abaixo cria um cluster AKS chamado MyAKS e anexa MyHelmACR.

```azurecli
az aks create -g MyResourceGroup -n MyAKS --location eastus  --attach-acr MyHelmACR --generate-ssh-keys
```

O cluster AKS precisa de acesso ao ACR para efetuar pull das imagens de contêiner e executá-las. O comando acima também concede o acesso ao cluster *MyAKS* ao seu ACR *MyHelmACR* .

## <a name="connect-to-your-aks-cluster"></a>Conectar-se ao cluster AKS

Para se conectar ao cluster Kubernetes no computador local, use o [kubectl][kubectl], o cliente de linha de comando do Kubernetes.

Se você usa o Azure Cloud Shell, o `kubectl` já estará instalado. Se você quiser instalá-lo localmente, use o comando [az aks install-cli][]:

```azurecli
az aks install-cli
```

Para configurar o `kubectl` para se conectar ao cluster do Kubernetes, use o comando [az aks get-credentials][]. O exemplo a seguir obtém as credenciais para o cluster AKS chamado *MyAKS* no grupo *MyResource*:

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKS
```

## <a name="download-the-sample-application"></a>Baixar o aplicativo de exemplo

Este guia de início rápido usa [um aplicativo node. js de exemplo do repositório de exemplo Azure dev Spaces][example-nodejs]. Clone o aplicativo do GitHub e navegue até o `dev-spaces/samples/nodejs/getting-started/webfrontend` diretório.

```console
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/nodejs/getting-started/webfrontend
```

## <a name="create-a-dockerfile"></a>Criar um Dockerfile

Crie um novo arquivo *Dockerfile* usando o seguinte:

```dockerfile
FROM node:latest

WORKDIR /webfrontend

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 80
CMD ["node","server.js"]
```

## <a name="build-and-push-the-sample-application-to-the-acr"></a>Criar e enviar por push o aplicativo de exemplo para o ACR

Obtenha o endereço do servidor de logon usando o comando [AZ ACR List][az-acr-list] e consultando o *loginServer*:

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

Use o Docker para compilar, marcar e enviar por push o contêiner de aplicativo de exemplo para o ACR:

```console
docker build -t webfrontend:latest .
docker tag webfrontend <acrLoginServer>/webfrontend:v1
docker push <acrLoginServer>/webfrontend:v1
```

## <a name="create-your-helm-chart"></a>Criar seu gráfico do Helm

Gere o gráfico do Helm usando `helm create` o comando.

```console
helm create webfrontend
```

Faça as seguintes atualizações para *WebFrontEnd/Values. YAML*:

* Altere `image.repository` para `<acrLoginServer>/webfrontend`
* Altere `service.type` para `LoadBalancer`

Por exemplo:

```yml
# Default values for webfrontend.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: <acrLoginServer>/webfrontend
  pullPolicy: IfNotPresent
...
service:
  type: LoadBalancer
  port: 80
...
```

Atualize `appVersion` para `v1` no *WebFrontEnd/Char. YAML*. Por exemplo

```yml
apiVersion: v2
name: webfrontend
...
# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application.
appVersion: v1
```

## <a name="run-your-helm-chart"></a>Executar o gráfico do Helm

Use o `helm install` comando para instalar seu aplicativo usando seu gráfico do Helm.

```console
helm install webfrontend webfrontend/
```

Demora alguns minutos para o serviço retornar um endereço IP público. Para monitorar o andamento, use o comando `kubectl get service` com o parâmetro *watch*:

```console
$ kubectl get service --watch

NAME                TYPE          CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
webfrontend         LoadBalancer  10.0.141.72   <pending>     80:32150/TCP   2m
...
webfrontend         LoadBalancer  10.0.141.72   <EXTERNAL-IP> 80:32150/TCP   7m
```

Navegue até o balanceador de carga do seu aplicativo em um navegador usando `<EXTERNAL-IP>` o para ver o aplicativo de exemplo.

## <a name="delete-the-cluster"></a>Excluir o cluster

Quando o cluster não for mais necessário, use o comando [AZ Group Delete][az-group-delete] para remover o grupo de recursos, o cluster AKs, o registro de contêiner, as imagens de contêiner armazenadas lá e todos os recursos relacionados.

```azurecli-interactive
az group delete --name MyResourceGroup --yes --no-wait
```

> [!NOTE]
> Quando você excluir o cluster, a entidade de serviço do Azure Active Directory usada pelo cluster do AKS não será removida. Para obter as etapas para remover a entidade de serviço, confira [Considerações sobre a entidade de serviço do AKS e sua exclusão][sp-delete]. Se você tiver usado uma identidade gerenciada, ela será gerenciada pela plataforma e não exigirá remoção.

## <a name="next-steps"></a>Próximas etapas

Para obter mais informações sobre como usar o Helm, consulte a documentação do Helm.

> [!div class="nextstepaction"]
> [Documentação do Helm][helm-documentation]

[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-list]: /cli/azure/acr#az-acr-list
[az-group-delete]: /cli/azure/group#az-group-delete
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli

[docker-for-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-for-mac]: https://docs.docker.com/docker-for-mac/
[docker-for-windows]: https://docs.docker.com/docker-for-windows/
[example-nodejs]: https://github.com/Azure/dev-spaces/tree/master/samples/nodejs/getting-started/webfrontend
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[helm]: https://helm.sh/
[helm-documentation]: https://helm.sh/docs/
[helm-existing]: kubernetes-helm.md
[helm-install]: https://helm.sh/docs/intro/install/
[sp-delete]: kubernetes-service-principal.md#additional-considerations
