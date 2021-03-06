---
title: Criando um controlador de entrada com um novo gateway de aplicativo
description: Este artigo fornece informações sobre como implantar um controlador de entrada do gateway de aplicativo com um novo gateway de aplicativo.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: article
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: b46c9f8b0cad74f3a4e9be8903270a60993c01f4
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "80585893"
---
# <a name="how-to-install-an-application-gateway-ingress-controller-agic-using-a-new-application-gateway"></a>Como instalar um controlador de entrada do gateway de aplicativo (AGIC) usando um novo gateway de aplicativo

As instruções a seguir pressupõem que o controlador de entrada do gateway de aplicativo (AGIC) será instalado em um ambiente sem componentes pré-existentes.

## <a name="required-command-line-tools"></a>Ferramentas de linha de comando necessárias

Recomendamos o uso de [Azure cloud Shell](https://shell.azure.com/) para todas as operações de linha de comando abaixo. Inicie o Shell a partir de shell.azure.com ou clicando no link:

[![Lançamento de inserção](https://shell.azure.com/images/launchcloudshell.png "Iniciar o Azure Cloud Shell")](https://shell.azure.com)

Como alternativa, inicie Cloud Shell de portal do Azure usando o seguinte ícone:

![Inicialização do portal](./media/application-gateway-ingress-controller-install-new/portal-launch-icon.png)

Seu [Azure cloud Shell](https://shell.azure.com/) já tem todas as ferramentas necessárias. Se você optar por usar outro ambiente, verifique se as seguintes ferramentas de linha de comando estão instaladas:

* `az`-CLI do Azure: [instruções de instalação](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
* `kubectl`-Ferramenta de linha de comando-kubernetes: [instruções de instalação](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* `helm`-Kubernetes Package Manager: [instruções de instalação](https://github.com/helm/helm/releases/latest)
* `jq`-processador JSON de linha de comando: [instruções de instalação](https://stedolan.github.io/jq/download/)


## <a name="create-an-identity"></a>Criar uma identidade

Siga as etapas abaixo para criar um objeto de entidade de [serviço](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals#service-principal-object)Azure Active Directory (AAD). Registre os `appId`valores, `password`e `objectId` -eles serão usados nas etapas a seguir.

1. Criar entidade de serviço do AD ([Leia mais sobre o RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)):
    ```azurecli
    az ad sp create-for-rbac --skip-assignment -o json > auth.json
    appId=$(jq -r ".appId" auth.json)
    password=$(jq -r ".password" auth.json)
    ```
    Os `appId` valores `password` e da saída JSON serão usados nas etapas a seguir


1. Use o `appId` da saída do comando anterior para obter a `objectId` nova entidade de serviço:
    ```azurecli
    objectId=$(az ad sp show --id $appId --query "objectId" -o tsv)
    ```
    A saída desse comando é `objectId`, que será usada no modelo de Azure Resource Manager abaixo

1. Crie o arquivo de parâmetro que será usado na implantação do modelo de Azure Resource Manager mais tarde.
    ```bash
    cat <<EOF > parameters.json
    {
      "aksServicePrincipalAppId": { "value": "$appId" },
      "aksServicePrincipalClientSecret": { "value": "$password" },
      "aksServicePrincipalObjectId": { "value": "$objectId" },
      "aksEnableRBAC": { "value": false }
    }
    EOF
    ```
    Para implantar um cluster habilitado para **RBAC** , defina `aksEnabledRBAC` o campo como`true`

## <a name="deploy-components"></a>Implantar componentes
Esta etapa adicionará os seguintes componentes à sua assinatura:

- [Serviço de Kubernetes do Azure](https://docs.microsoft.com/azure/aks/intro-kubernetes)
- [Gateway de aplicativo](https://docs.microsoft.com/azure/application-gateway/overview) v2
- [Rede virtual](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) com 2 [sub-redes](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)
- [Endereço IP público](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address)
- [Identidade gerenciada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview), que será usada pela [identidade Pod do AAD](https://github.com/Azure/aad-pod-identity/blob/master/README.md)

1. Baixe o modelo de Azure Resource Manager e modifique o modelo conforme necessário.
    ```bash
    wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/deploy/azuredeploy.json -O template.json
    ```

1. Implante o modelo de Azure Resource Manager `az cli`usando. Isso pode levar até 5 minutos.
    ```azurecli
    resourceGroupName="MyResourceGroup"
    location="westus2"
    deploymentName="ingress-appgw"

    # create a resource group
    az group create -n $resourceGroupName -l $location

    # modify the template as needed
    az group deployment create \
            -g $resourceGroupName \
            -n $deploymentName \
            --template-file template.json \
            --parameters parameters.json
    ```

1. Após a conclusão da implantação, baixe a saída da implantação em um `deployment-outputs.json`arquivo chamado.
    ```azurecli
    az group deployment show -g $resourceGroupName -n $deploymentName --query "properties.outputs" -o json > deployment-outputs.json
    ```

## <a name="set-up-application-gateway-ingress-controller"></a>Configurar o controlador de entrada do gateway de aplicativo

Com as instruções na seção anterior, criamos e configuramos um novo cluster AKS e um gateway de aplicativo. Agora estamos prontos para implantar um aplicativo de exemplo e um controlador de entrada para nossa nova infraestrutura kubernetes.

### <a name="setup-kubernetes-credentials"></a>Configurar credenciais do kubernetes
Para as etapas a seguir, precisamos do comando Setup [kubectl](https://kubectl.docs.kubernetes.io/) , que usaremos para se conectar ao nosso novo cluster kubernetes. [Cloud Shell](https://shell.azure.com/) O Cloud Shell `kubectl` já foi instalado. Usaremos a `az` CLI para obter credenciais para kubernetes.

Obtenha credenciais para o AKS implantado recentemente ([Leia mais](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough#connect-to-the-cluster)):
```azurecli
# use the deployment-outputs.json created after deployment to get the cluster name and resource group name
aksClusterName=$(jq -r ".aksClusterName.value" deployment-outputs.json)
resourceGroupName=$(jq -r ".resourceGroupName.value" deployment-outputs.json)

az aks get-credentials --resource-group $resourceGroupName --name $aksClusterName
```

### <a name="install-aad-pod-identity"></a>Instalar a identidade do Pod do AAD
  Azure Active Directory identidade Pod fornece acesso baseado em token ao [Azure Resource Manager (ARM)](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview).

  A [identidade do Pod do AAD](https://github.com/Azure/aad-pod-identity) adicionará os seguintes componentes ao cluster kubernetes:
   * Kubernetes [crds](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/): `AzureIdentity`, `AzureAssignedIdentity`,`AzureIdentityBinding`
   * Componente do [MIC (Controlador de Identidades Gerenciadas)](https://github.com/Azure/aad-pod-identity#managed-identity-controllermic)
   * Componente de [NMI (Identidade Gerenciada do Nó)](https://github.com/Azure/aad-pod-identity#node-managed-identitynmi)


Para instalar a identidade do Pod do AAD em seu cluster:

   - *RBAC habilitado* Cluster AKS

     ```bash
     kubectl create -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment-rbac.yaml
     ```

   - *RBAC desabilitado* Cluster AKS

     ```bash
     kubectl create -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment.yaml
     ```

### <a name="install-helm"></a>Instalar o Helm
[Helm](https://docs.microsoft.com/azure/aks/kubernetes-helm) é um Gerenciador de pacotes para kubernetes. Vamos aproveitá-lo para instalar o `application-gateway-kubernetes-ingress` pacote:

1. Instale o [Helm](https://docs.microsoft.com/azure/aks/kubernetes-helm) e execute o seguinte para `application-gateway-kubernetes-ingress` adicionar o pacote Helm:

    - *RBAC habilitado* Cluster AKS

        ```bash
        kubectl create serviceaccount --namespace kube-system tiller-sa
        kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller-sa
        helm init --tiller-namespace kube-system --service-account tiller-sa
        ```

    - *RBAC desabilitado* Cluster AKS

        ```bash
        helm init
        ```

1. Adicione o repositório AGIC do Helm:
    ```bash
    helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
    helm repo update
    ```

### <a name="install-ingress-controller-helm-chart"></a>Instalar o Gráfico do Controlador de Entrada do Helm

1. Use o `deployment-outputs.json` arquivo criado acima e crie as variáveis a seguir.
    ```bash
    applicationGatewayName=$(jq -r ".applicationGatewayName.value" deployment-outputs.json)
    resourceGroupName=$(jq -r ".resourceGroupName.value" deployment-outputs.json)
    subscriptionId=$(jq -r ".subscriptionId.value" deployment-outputs.json)
    identityClientId=$(jq -r ".identityClientId.value" deployment-outputs.json)
    identityResourceId=$(jq -r ".identityResourceId.value" deployment-outputs.json)
    ```
1. Baixe Helm-config. YAML, que irá configurar o AGIC:
    ```bash
    wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/sample-helm-config.yaml -O helm-config.yaml
    ```
    Ou copie o arquivo YAML abaixo: 
    
    ```yaml
    # This file contains the essential configs for the ingress controller helm chart

    # Verbosity level of the App Gateway Ingress Controller
    verbosityLevel: 3
    
    ################################################################################
    # Specify which application gateway the ingress controller will manage
    #
    appgw:
        subscriptionId: <subscriptionId>
        resourceGroup: <resourceGroupName>
        name: <applicationGatewayName>
    
        # Setting appgw.shared to "true" will create an AzureIngressProhibitedTarget CRD.
        # This prohibits AGIC from applying config for any host/path.
        # Use "kubectl get AzureIngressProhibitedTargets" to view and change this.
        shared: false
    
    ################################################################################
    # Specify which kubernetes namespace the ingress controller will watch
    # Default value is "default"
    # Leaving this variable out or setting it to blank or empty string would
    # result in Ingress Controller observing all acessible namespaces.
    #
    # kubernetes:
    #   watchNamespace: <namespace>
    
    ################################################################################
    # Specify the authentication with Azure Resource Manager
    #
    # Two authentication methods are available:
    # - Option 1: AAD-Pod-Identity (https://github.com/Azure/aad-pod-identity)
    armAuth:
        type: aadPodIdentity
        identityResourceID: <identityResourceId>
        identityClientID:  <identityClientId>
    
    ## Alternatively you can use Service Principal credentials
    # armAuth:
    #    type: servicePrincipal
    #    secretJSON: <<Generate this value with: "az ad sp create-for-rbac --subscription <subscription-uuid> --sdk-auth | base64 -w0" >>
    
    ################################################################################
    # Specify if the cluster is RBAC enabled or not
    rbac:
        enabled: false # true/false
    
    # Specify aks cluster related information. THIS IS BEING DEPRECATED.
    aksClusterConfiguration:
        apiServerAddress: <aks-api-server-address>
    ```

1. Edite o Helm-config. YAML recentemente baixado e preencha as seções `appgw` e `armAuth`.
    ```bash
    sed -i "s|<subscriptionId>|${subscriptionId}|g" helm-config.yaml
    sed -i "s|<resourceGroupName>|${resourceGroupName}|g" helm-config.yaml
    sed -i "s|<applicationGatewayName>|${applicationGatewayName}|g" helm-config.yaml
    sed -i "s|<identityResourceId>|${identityResourceId}|g" helm-config.yaml
    sed -i "s|<identityClientId>|${identityClientId}|g" helm-config.yaml

    # You can further modify the helm config to enable/disable features
    nano helm-config.yaml
    ```

   Valores:
     - `verbosityLevel`: Define o nível de detalhamento da infraestrutura de log AGIC. Consulte [Níveis de registro em log](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/463a87213bbc3106af6fce0f4023477216d2ad78/docs/troubleshooting.md#logging-levels) para obter os valores possíveis.
     - `appgw.subscriptionId`: A ID da assinatura do Azure na qual o gateway de aplicativo reside. Exemplo: `a123b234-a3b4-557d-b2df-a0bc12de1234`
     - `appgw.resourceGroup`: Nome do grupo de recursos do Azure no qual o gateway de aplicativo foi criado. Exemplo: `app-gw-resource-group`
     - `appgw.name`: Nome do gateway de aplicativo. Exemplo: `applicationgatewayd0f0`
     - `appgw.shared`: Esse sinalizador booliano deve ser padronizado `false`. Defina como `true` caso precise de um [Gateway de aplicativo compartilhado](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/072626cb4e37f7b7a1b0c4578c38d1eadc3e8701/docs/setup/install-existing.md#multi-cluster--shared-app-gateway).
     - `kubernetes.watchNamespace`: Especifique o espaço de nome, que AGIC deve observar. Pode ser um único valor de cadeia de caracteres ou uma lista separada por vírgulas de namespaces.
    - `armAuth.type`: pode ser `aadPodIdentity` ou`servicePrincipal`
    - `armAuth.identityResourceID`: ID de recurso da identidade gerenciada do Azure
    - `armAuth.identityClientId`: A ID do cliente da identidade. Consulte abaixo para obter mais informações sobre identidade
    - `armAuth.secretJSON`: Necessário somente quando o tipo de segredo da entidade de serviço `armAuth.type` é escolhido (quando `servicePrincipal`foi definido como) 


   > [!NOTE]
   > Os `identityResourceID` e `identityClientID` são valores que foram criados durante as etapas de [implantar componentes](ingress-controller-install-new.md#deploy-components) e podem ser obtidos novamente usando o seguinte comando:
   > ```azurecli
   > az identity show -g <resource-group> -n <identity-name>
   > ```
   > `<resource-group>`no comando acima, está o grupo de recursos do seu gateway de aplicativo. `<identity-name>`é o nome da identidade criada. Todas as identidades de uma determinada assinatura podem ser listadas usando:`az identity list`


1. Instale o pacote do controlador de entrada do Gateway de Aplicativo:

    ```bash
    helm install -f helm-config.yaml application-gateway-kubernetes-ingress/ingress-azure
    ```

## <a name="install-a-sample-app"></a>Instalar um aplicativo de exemplo
Agora que temos o gateway de aplicativo, o AKS e o AGIC instalados, podemos instalar um aplicativo de exemplo por meio do [Azure cloud Shell](https://shell.azure.com/):

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: aspnetapp
  labels:
    app: aspnetapp
spec:
  containers:
  - image: "mcr.microsoft.com/dotnet/core/samples:aspnetapp"
    name: aspnetapp-image
    ports:
    - containerPort: 80
      protocol: TCP

---

apiVersion: v1
kind: Service
metadata:
  name: aspnetapp
spec:
  selector:
    app: aspnetapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: aspnetapp
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: aspnetapp
          servicePort: 80
EOF
```

Como alternativa, você pode:

* Baixe o arquivo YAML acima:

```bash
curl https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml -o aspnetapp.yaml
```

* Aplique o arquivo YAML:

```bash
kubectl apply -f aspnetapp.yaml
```


## <a name="other-examples"></a>Outros exemplos
Este [Guia de instruções](ingress-controller-expose-service-over-http-https.md) contém mais exemplos de como expor um serviço AKs via http ou HTTPS para a Internet com o gateway de aplicativo.
