---
title: Conectar computadores Linux ao Azure Monitor | Microsoft Docs
description: Este artigo descreve como conectar computadores Linux hospedados em outras nuvens ou locais para Azure Monitor com o agente de Log Analytics para Linux.
ms.subservice: logs
ms.topic: conceptual
author: mgoedtel
ms.author: magoedte
ms.date: 01/21/2020
ms.openlocfilehash: 9807d6eeb07b953ab75b328ce64c5166ca52dd2a
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "80637516"
---
# <a name="connect-linux-computers-to-azure-monitor"></a>Conectar computadores Linux ao Azure Monitor

Para monitorar e gerenciar máquinas virtuais ou computadores físicos em seu datacenter local ou em outro ambiente de nuvem com Azure Monitor, você precisa implantar o agente de Log Analytics e configurá-lo para relatar a um espaço de trabalho Log Analytics. Adicionalmente, o agente fornece suporte à função do Hybrid Runbook Worker para a Automação do Azure.

O agente do Log Analytics para Linux pode ser instalado usando um dos métodos a seguir. Detalhes sobre como usar cada método são fornecidos posteriormente neste artigo.

* [Baixe e instale manualmente](#install-the-agent-manually) o agente. Isso é necessário quando o computador Linux não tem acesso à Internet e se comunicará com o Azure Monitor ou a automação do Azure por meio do [Gateway de log Analytics](gateway.md). 
* [Instale o agente para Linux usando um script wrapper](#install-the-agent-using-wrapper-script) hospedado no github. Esse é o método recomendado para instalar e atualizar o agente quando o computador tem conectividade com a Internet, diretamente ou por meio de um servidor proxy.

Para entender a configuração com suporte, revise [suporte para sistemas operacionais Linux](log-analytics-agent.md#supported-linux-operating-systems) e [configuração de firewall de rede](log-analytics-agent.md#network-requirements).

>[!NOTE]
>O agente do Log Analytics para Linux não pode ser configurado para se reportar a mais de um espaço de trabalho do Log Analytics. Ele só pode ser configurado para relatar tanto para um grupo de gerenciamento System Center Operations Manager quanto para Log Analytics espaço de trabalho simultaneamente, ou para individualmente.

## <a name="agent-install-package"></a>Pacote de instalação do agente

O agente do Log Analytics para Linux é composto por vários pacotes. O arquivo de versão contém os seguintes pacotes, que estão disponíveis ao executar o pacote de shell `--extract` com o parâmetro:

**Pacote** | **Versão** | **Descrição**
----------- | ----------- | --------------
omsagent | 1.12.15 | O agente de Log Analytics para Linux
omsconfig | 1.1.1 | Agente de configuração para o agente de Log Analytics
omi | 1.6.3 | OMI (infraestrutura de gerenciamento aberta) – um servidor CIM leve. *Observe que o OMI requer acesso de raiz para executar um trabalho cron necessário para o funcionamento do serviço*
scx | 1.6.3 | Provedores de CIM OMI para métricas de desempenho do sistema operacional
apache-cimprov | 1.0.1 | Provedor de monitoramento de desempenho do Servidor HTTP Apache para OMI. Instalado somente se o Servidor HTTP Apache for detectado.
mysql-cimprov | 1.0.1 | Provedor de monitoramento de desempenho do Servidor MySQL para OMI. Instalado somente se o servidor MySQL/MariaDB for detectado.
docker-cimprov | 1.0.0 | Provedor do Docker para OMI. Instalado somente se o Docker for detectado.

### <a name="agent-installation-details"></a>Detalhes de instalação do agente

Depois de instalar o agente de Log Analytics para pacotes do Linux, as seguintes alterações de configuração adicionais em todo o sistema são aplicadas. Esses artefatos serão removidos quando o pacote omsagent for desinstalado.

* Um usuário sem privilégios chamado: `omsagent` é criado. O daemon é executado nessa credencial. 
* Um arquivo de *inclusão* do sudors é `/etc/sudoers.d/omsagent`criado no. Isso autoriza `omsagent` a reiniciar os daemons syslog e omsagent. Se o sudo *incluir* diretivas não tiver suporte na versão instalada do sudo, essas entradas serão gravadas no `/etc/sudoers`.
* A configuração de syslog é modificada para encaminhar um subconjunto de eventos para o agente. Para obter mais informações, consulte [Configure syslog Data Collection](data-sources-syslog.md).

Em um computador Linux monitorado, o agente é listado `omsagent`como. `omsconfig`é o agente de configuração do Log Analytics Agent para Linux que procura a nova configuração do lado do portal a cada 5 minutos. A configuração nova e atualizada é aplicada aos arquivos de configuração do agente localizados `/etc/opt/microsoft/omsagent/conf/omsagent.conf`em.

## <a name="obtain-workspace-id-and-key"></a>Obter a ID do workspace e a chave

Antes de instalar o agente do Log Analytics para Linux, você precisa da ID do espaço de trabalho e da chave para o espaço de trabalho do Log Analytics. Essas informações são necessárias durante a instalação do agente para configurá-lo corretamente e garantir que ele possa se comunicar com êxito com Azure Monitor.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]  

1. No canto superior esquerdo do portal do Azure, selecione **Todos os serviços**. Na caixa de pesquisa, insira **Log Analytics**. A lista filtra com base em sua entrada, à medida que você digita. Selecione **log Analytics espaços de trabalho**.

2. Na lista de espaços de trabalho do Log Analytics, selecione o espaço de trabalho criado anteriormente. Você deve ter atribuído a ele o nome de **DefaultLAWorkspace**.

3. Selecione **Configurações avançadas**:

    ![Menu Configurações Avançadas do Log Analytics no portal do Azure](../learn/media/quick-collect-azurevm/log-analytics-advanced-settings-azure-portal.png) 
 
4. Selecione **Fontes Conectadas** e, em seguida, selecione **Servidores Linux**.

5. O valor à direita da **ID do Workspace** e **Chave Primária**. Copie e cole os dois em seu editor favorito.

## <a name="install-the-agent-manually"></a>Instalar o agente manualmente

O agente de Log Analytics para Linux é fornecido em um pacote de script de Shell de extração automática e instalável. Este pacote contém pacotes Debian e RPM para cada um dos componentes do agente e pode ser instalado diretamente ou extraído para recuperar os pacotes individuais. Um pacote é fornecido para x64 e outro para arquiteturas x86. 

> [!NOTE]
> Para VMs do Azure, recomendamos que você instale o agente neles usando a [extensão de VM log Analytics do Azure](../../virtual-machines/extensions/oms-linux.md) para Linux. 

1. [Baixe](https://github.com/microsoft/OMS-Agent-for-Linux#azure-install-guide) e transfira o pacote apropriado (x64 ou x86) para sua VM do Linux ou computador físico, usando o SCP/SFTP.

2. Instale o pacote usando o `--install` argumento. Para carregar um espaço de trabalho do Log Analytics durante a instalação, `-w <WorkspaceID>` forneça `-s <workspaceKey>` os parâmetros e copiados anteriormente.

    >[!NOTE]
    >Você precisará usar o `--upgrade` argumento se algum pacote dependente, como OMI, SCX, programa omsconfig ou suas versões mais antigas forem instalados, como seria o caso se o agente do system Center Operations Manager para Linux já estiver instalado. 

    ```
    sudo sh ./omsagent-*.universal.x64.sh --install -w <workspace id> -s <shared key>
    ```

3. Para configurar o agente do Linux para instalar e conectar-se a um Log Analytics espaço de trabalho por meio de um gateway de Log Analytics, execute o comando a seguir fornecendo os parâmetros proxy, ID do espaço de trabalho e chave do espaço de trabalho. Essa configuração pode ser especificada na linha de comando, incluindo `-p [protocol://][user:password@]proxyhost[:port]`. A propriedade *proxyHost* aceita um nome de domínio totalmente qualificado ou endereço IP do servidor de gateway de log Analytics.  

    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -p https://<proxy address>:<proxy port> -w <workspace id> -s <shared key>
    ```

    Se a autenticação for necessária, você precisará especificar o nome de usuário e a senha. Por exemplo: 
    
    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -p https://<proxy user>:<proxy password>@<proxy address>:<proxy port> -w <workspace id> -s <shared key>
    ```

4. Para configurar o computador Linux para se conectar a um Log Analytics espaço de trabalho na nuvem do Azure governamental, execute o seguinte comando fornecendo a ID do espaço de trabalho e a chave primária copiadas anteriormente.

    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -w <workspace id> -s <shared key> -d opinsights.azure.us
    ```

Se você quiser instalar os pacotes de agente e configurá-lo para relatar para um espaço de trabalho específico do Log Analytics em um momento posterior, execute o seguinte comando:

```
sudo sh ./omsagent-*.universal.x64.sh --upgrade
```

Se você quiser extrair os pacotes de agente do pacote sem instalar o agente, execute o seguinte comando:

```
sudo sh ./omsagent-*.universal.x64.sh --extract
```

## <a name="install-the-agent-using-wrapper-script"></a>Instalar o agente usando o script de wrapper

As etapas a seguir configuram a instalação do agente para Log Analytics no Azure e na nuvem do Azure governamental usando o script de wrapper para computadores Linux que podem se comunicar diretamente ou por meio de um servidor proxy para baixar o agente hospedado no GitHub e instalar o agente.  

Se o seu computador Linux precisar se comunicar por meio de um servidor proxy para Log Analytics, essa configuração poderá ser especificada na linha de `-p [protocol://][user:password@]proxyhost[:port]`comando, incluindo. A propriedade *Protocol* aceita `http` ou `https`, e a propriedade *proxyHost* aceita um nome de domínio totalmente qualificado ou um endereço IP do servidor proxy. 

Por exemplo: `https://proxy01.contoso.com:30443`

Se a autenticação for necessária em ambos os casos, você precisará especificar o nome de usuário e a senha. Por exemplo: `https://user01:password@proxy01.contoso.com:30443`

1. Para configurar o computador Linux para se conectar a um Log Analytics espaço de trabalho, execute o seguinte comando fornecendo a ID do espaço de trabalho e a chave primária. O comando a seguir baixa o agente, valida sua soma de verificação e o instala.
    
    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

    O seguinte comando inclui o parâmetro de proxy `-p` e a sintaxe de exemplo quando a autenticação é exigida pelo servidor proxy:

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://]<proxy user>:<proxy password>@<proxyhost>[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

2. Para configurar o computador Linux para se conectar ao workspace do Log Analytics na nuvem do Azure Governamental, execute o seguinte comando fornecendo a ID do workspace e a chave primária copiadas anteriormente. O comando a seguir baixa o agente, valida sua soma de verificação e o instala. 

    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ``` 

    O seguinte comando inclui o parâmetro de proxy `-p` e a sintaxe de exemplo quando a autenticação é exigida pelo servidor proxy:

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://]<proxy user>:<proxy password>@<proxyhost>[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ```
2. Reinicie o agente executando o seguinte comando: 

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ``` 

## <a name="upgrade-from-a-previous-release"></a>Atualizar de uma versão anterior

A atualização de uma versão anterior, começando com a versão 1.0.0-47, tem suporte em cada versão. Execute a instalação com o `--upgrade` parâmetro para atualizar todos os componentes do agente para a versão mais recente.

## <a name="next-steps"></a>Próximas etapas

- Examine [o gerenciamento e a manutenção do agente de log Analytics para Windows e Linux](agent-manage.md) para saber mais sobre como reconfigurar, atualizar ou remover o agente da máquina virtual.

- Consulte [Solução de problemas do agente do Linux](agent-linux-troubleshoot.md) se você encontrar problemas ao instalar ou gerenciar o agente.
