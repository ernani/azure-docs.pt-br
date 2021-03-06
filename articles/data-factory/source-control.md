---
title: Controle do código-fonte
description: Saiba como configurar o controle do código-fonte no Azure Data Factory
services: data-factory
ms.service: data-factory
ms.workload: data-services
author: djpmsft
ms.author: daperlov
manager: anandsub
ms.reviewer: ''
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 04/30/2020
ms.openlocfilehash: f327844be57d7f8e177f3bf72b1e3b56c5147e00
ms.sourcegitcommit: 1895459d1c8a592f03326fcb037007b86e2fd22f
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/01/2020
ms.locfileid: "82629313"
---
# <a name="source-control-in-azure-data-factory"></a>Controle do código-fonte no Azure Data Factory
[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Por padrão, o Azure Data Factory os autores da experiência de interface do usuário (UX) diretamente no serviço data factory. Essa experiência tem as seguintes limitações:

- O serviço de Data Factory não inclui um repositório para armazenar as entidades JSON para as alterações. A única maneira de salvar as alterações é por meio do botão **publicar tudo** e todas as alterações são publicadas diretamente no serviço de data Factory.
- O serviço de Data Factory não é otimizado para colaboração e controle de versão.

Para fornecer uma experiência de criação melhor, Azure Data Factory permite que você configure um repositório Git com o Azure Repos ou o GitHub. O Git é um sistema de controle de versão que permite o controle de alterações e a colaboração mais fáceis. Este tutorial descreverá como configurar e trabalhar em um repositório git juntamente com as práticas recomendadas de realce e um guia de solução de problemas.

> [!NOTE]
> A integração do Azure data factory git não está disponível na nuvem do Azure governamental.

## <a name="advantages-of-git-integration"></a>Vantagens da integração do Git

Abaixo está uma lista de algumas das vantagens que a integração do git fornece à experiência de criação:

-   **Controle do código-fonte:** À medida que suas cargas de trabalho de data factory se tornarem cruciais, você desejaria integrar sua fábrica com o Git para aproveitar vários benefícios de controle do código-fonte, como o seguinte:
    -   Capacidade de controlar/auditar as alterações.
    -   Capacidade de reverter as alterações que introduziram bugs.
-   **Gravações parciais:** Ao criar em relação ao serviço de data factory, você não pode salvar as alterações como um rascunho e todas as publicações devem passar data factory validação. Se seus pipelines não forem concluídos ou se você simplesmente não quiser perder as alterações no caso de uma falha do computador, a integração do git permitirá alterações incrementais de data factory recursos, independentemente do estado em que estão. Configurar um repositório git permite que você salve as alterações, permitindo que você publique apenas quando tiver testado suas alterações para sua satisfação.
-   **Colaboração e controle:** Se você tiver vários membros da equipe que contribuem para a mesma fábrica, convém permitir que seus colegas colaborem entre si por meio de um processo de revisão de código. Você também pode configurar sua fábrica de forma que nem todos os colaboradores tenham permissões iguais. Alguns integrantes da equipe só podem ter permissão para fazer alterações por meio do git, e apenas determinadas pessoas na equipe têm permissão para publicar as alterações na fábrica.
-   **CI/CD melhor:**  Se você estiver implantando em vários ambientes com um [processo de entrega contínua](continuous-integration-deployment.md), a integração com o Git facilitará certas ações. Algumas dessas ações incluem:
    -   Configure seu pipeline de liberação para disparar automaticamente assim que houver qualquer alteração feita na sua fábrica de "desenvolvimento".
    -   Personalize as propriedades em sua fábrica que estão disponíveis como parâmetros no modelo do Resource Manager. Isso pode ser útil manter somente o conjunto necessário de propriedades como parâmetros e ter todos os demais elementos embutidos no código.
-   **Melhor desempenho:** Uma fábrica média com a integração do git é carregada 10 vezes mais rápido do que uma criação no serviço de data factory. Essa melhoria de desempenho ocorre porque os recursos são baixados por meio do git.

> [!NOTE]
> A criação direta com o serviço de Data Factory está desabilitada no Azure Data Factory UX quando um repositório Git é configurado. As alterações podem ser feitas diretamente no serviço por meio do PowerShell ou de um SDK.

## <a name="author-with-azure-repos-git-integration"></a>Criar com a integração do Git ao Azure Repos

A criação visual com a integração do Git ao Azure Repos dá suporte ao controle do código-fonte e à colaboração para trabalhar nos pipelines de data factory. Você pode associar um data factory a um repositório da organização do Git do Azure Repos para obter controle do código-fonte, colaboração, controle de versão e assim por diante. Uma única organização do Git do Azure Repos pode ter vários repositórios, mas um repositório Git do Azure Repos pode ser associado a apenas um data factory. Se você não tiver uma organização ou um repositório do Azure Repos, siga [estas instruções](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization-msa-or-work-student) para criar seus recursos.

> [!NOTE]
> Você pode armazenar arquivos de script e de dados em um repositório Git do Azure Repos. No entanto, você precisa carregar os arquivos manualmente para o Armazenamento do Azure. Um pipeline do Data Factory não faz upload automaticamente dos arquivos de dados ou de script armazenados em um repositório Git do Azure Repos para o Armazenamento do Azure.

### <a name="configure-an-azure-repos-git-repository-with-azure-data-factory"></a>Configurar um repositório Git do Azure Repos com o Azure Data Factory

Você pode configurar um repositório Git do Azure Repos com um data factory usando dois métodos.

#### <a name="configuration-method-1-azure-data-factory-home-page"></a>Método de configuração 1: Azure Data Factory home page

Na home page Azure Data Factory, selecione **configurar repositório de código**.

![Configurar um repositório de código do Azure Repos](media/author-visually/configure-repo.png)

#### <a name="configuration-method-2-ux-authoring-canvas"></a>Método de configuração 2: Tela de criação da UX
Na tela de criação do Azure Data Factory UX, selecione o menu suspenso **Data Factory** e, em seguida, selecione **configurar repositório de código**.

![Configurar as definições do repositório de código para a criação de UX](media/author-visually/configure-repo-2.png)

Ambos os métodos abrem o painel de configuração configurações do repositório.

![Configurar as definições do repositório de código](media/author-visually/repo-settings.png)

O painel de configuração mostra as seguintes configurações de Azure Repos repositório de código:

| Configuração | Descrição | Valor |
|:--- |:--- |:--- |
| **Tipo de repositório** | O tipo do repositório de código do Azure Repos.<br/> | Azure DevOps git ou GitHub |
| **Azure Active Directory** | Seu nome de locatário do Microsoft Azure AD. | `<your tenant name>` |
| **Organização do Azure Repos** | O nome da organização do Azure Repos. Localize o nome de organização do Azure Repos em `https://{organization name}.visualstudio.com`. Você pode [entrar na sua organização do Azure Repos](https://www.visualstudio.com/team-services/git/) para acessar seu perfil do Visual Studio e ver seus repositórios e projetos. | `<your organization name>` |
| **ProjectName** | O nome do projeto do Azure Repos. Localize o nome do projeto do Azure Repos em `https://{organization name}.visualstudio.com/{project name}`. | `<your Azure Repos project name>` |
| **Repositoryname** | O nome do seu repositório de código do Azure Repos. Os projetos do Azure Repos contêm repositórios Git para gerenciar seu código-fonte à medida que o projeto aumenta. Você pode criar um novo repositório ou usar um existente que já esteja no projeto. | `<your Azure Repos code repository name>` |
| **Ramificação de colaboração** | Seu branch de colaboração do Azure Repos que é usado para publicação. Por padrão, seu `master`. Altere essa configuração se você desejar publicar recursos de outra ramificação. | `<your collaboration branch name>` |
| **Pasta raiz** | A pasta raiz em seu branch de colaboração do Azure Repos. | `<your root folder name>` |
| **Importar recursos existentes do Data Factory para o repositório** | Especifica se é necessário importar recursos existentes do data factory da **Tela de criação** da UX em um repositório Git do Azure Repos. Selecione a caixa para importar os recursos do data factory para o repositório do Git associado no formato JSON. Esta ação exporta cada recurso individualmente (ou seja, os serviços vinculados e conjuntos de dados são exportados para JSONs separados). Quando essa caixa não está selecionada, os recursos existentes não são importados. | Selecionada (padrão) |
| **Branch para importar o recurso** | Especifica em qual branch os recursos do data factory (pipelines, conjuntos de dados, serviços vinculados etc.) serão importados. Você pode importar recursos para um dos seguintes branches: a. Colaboração b. Criar novo c. Usar Existente |  |

> [!NOTE]
> Se você estiver usando o Microsoft Edge e não vir nenhum valor em seu menu suspenso da conta do Azure DevOps, adicione https://*. VisualStudio. com à lista de sites confiáveis.

### <a name="use-a-different-azure-active-directory-tenant"></a>Usar um locatário do Azure Active Directory diferente

O repositório git Azure Repos pode estar em um locatário Azure Active Directory diferente. Para especificar outro locatário do Azure AD, você precisa ter permissões de administrador para a assinatura do Azure que está usando.

### <a name="use-your-personal-microsoft-account"></a>Usar sua conta Microsoft pessoal

Para usar uma conta Microsoft pessoal para integração com o Git, você poderá vincular seu Repositório do Azure pessoal ao Active Directory da sua organização.

1. Adicione sua conta Microsoft pessoal ao Active Directory da sua organização como convidado. Para saber mais, veja [Adicionar usuários de colaboração B2B do Azure Active Directory no portal do Azure](../active-directory/b2b/add-users-administrator.md).

2. Faça logon no portal do Azure com sua conta Microsoft pessoal. Em seguida, alterne para o Active Directory da sua organização.

3. Vá para a seção Azure DevOps, onde você agora vê seu repositório pessoal. Selecione o repositório e conecte-se com o Active Directory.

Após essas etapas de configuração, seu repositório pessoal estará disponível quando você configurar a integração de Git na IU do Data Factory.

Para saber mais sobre como se conectar ao Azure Repos para o Active Directory da sua organização, veja [Conectar sua organização do Azure DevOps ao Azure Active Directory](/azure/devops/organizations/accounts/connect-organization-to-azure-ad).

## <a name="author-with-github-integration"></a>Criar com a integração do GitHub

A criação visual com a integração do GitHub oferece suporte ao controle do código-fonte e à colaboração para trabalhar em seus pipelines de data factory. Você pode associar um data factory com um repositório de conta do GitHub para controle do código-fonte, colaboração e controle de versão. Uma única conta do GitHub pode ter vários repositórios, mas um repositório do GitHub pode ser associado somente a um data factory. Se você não tiver uma conta ou repositório do GitHub, siga [estas instruções](https://github.com/join) para criar seus recursos.

A integração do GitHub com o Data Factory dá suporte ao GitHub público ( [https://github.com](https://github.com)ou seja,) e ao GitHub Enterprise. Você pode usar os repositórios GitHub públicos e privados com o Data Factory, desde que você tenha permissão de leitura e de escrita para o repositório no GitHub.

Para configurar um repositório GitHub, você deve ter permissões de administrador para a assinatura do Azure que você está usando.

Para ver uma introdução de nove minutos e uma demonstração desse recurso, assista ao seguinte vídeo:

> [!VIDEO https://channel9.msdn.com/shows/azure-friday/Azure-Data-Factory-visual-tools-now-integrated-with-GitHub/player]

### <a name="configure-a-github-repository-with-azure-data-factory"></a>Configurar um repositório GitHub com Azure Data Factory

Você pode configurar um repositório do GitHub com um data factory por meio de dois métodos.

#### <a name="configuration-method-1-azure-data-factory-home-page"></a>Método de configuração 1: Azure Data Factory home page

Na home page Azure Data Factory, selecione **configurar repositório de código**.

![Configurar um repositório de código do Azure Repos](media/author-visually/configure-repo.png)

#### <a name="configuration-method-2-ux-authoring-canvas"></a>Método de configuração 2: Tela de criação da UX

Na tela de criação do Azure Data Factory UX, selecione o menu suspenso **Data Factory** e, em seguida, selecione **configurar repositório de código**.

![Configurar as definições do repositório de código para a criação de UX](media/author-visually/configure-repo-2.png)

Ambos os métodos abrem o painel de configuração configurações do repositório.

![Configurações do repositório do GitHub](media/author-visually/github-integration-image2.png)

O painel de configuração mostra as seguintes configurações do repositório do GitHub:

| **Configuração** | **Descrição**  | **Valor**  |
|:--- |:--- |:--- |
| **Tipo de repositório** | O tipo do repositório de código do Azure Repos. | GitHub |
| **Usar GitHub Enterprise** | Caixa de seleção para selecionar o GitHub Enterprise | não selecionado (padrão) |
| **URL do GitHub Enterprise** | A URL raiz da empresa do GitHub (deve ser HTTPS para o servidor do GitHub corporativo local). Por exemplo: `https://github.mydomain.com`. Necessário somente se **usar o GitHub Enterprise** estiver selecionado | `<your GitHub enterprise url>` |                                                           
| **Conta do GitHub** | Seu nome de conta do GitHub. Esse nome pode ser encontrado em https:\//github.com/{Account Name}/{Repository Name}. Navegar até essa página solicita que você insira as credenciais do GitHub OAuth para sua conta do GitHub. | `<your GitHub account name>` |
| **Nome do repositório**  | O nome do repositório de código do GitHub. As contas do GitHub contêm repositórios Git para gerenciar seu código-fonte. Você pode criar um novo repositório ou usar um existente que já esteja na conta. | `<your repository name>` |
| **Ramificação de colaboração** | Sua ramificação de colaboração do GitHub usada para publicação. Por padrão, seu mestre. Altere essa configuração se você desejar publicar recursos de outra ramificação. | `<your collaboration branch>` |
| **Pasta raiz** | Sua pasta raiz em sua ramificação de colaboração GitHub. |`<your root folder name>` |
| **Importar recursos existentes do Data Factory para o repositório** | Especifica se os recursos existentes do data factory devem ser importados da tela de criação do UX em um repositório do GitHub. Selecione a caixa para importar os recursos do data factory para o repositório do Git associado no formato JSON. Esta ação exporta cada recurso individualmente (ou seja, os serviços vinculados e conjuntos de dados são exportados para JSONs separados). Quando essa caixa não está selecionada, os recursos existentes não são importados. | Selecionada (padrão) |
| **Branch para importar o recurso** | Especifica em qual branch os recursos do data factory (pipelines, conjuntos de dados, serviços vinculados etc.) serão importados. Você pode importar recursos para um dos seguintes branches: a. Colaboração b. Criar novo c. Usar Existente |  |

### <a name="known-github-limitations"></a>Limitações conhecidas do GitHub

- Você pode armazenar arquivos de dados e scripts em um repositório GitHub. No entanto, você precisa carregar os arquivos manualmente para o Armazenamento do Azure. Um pipeline do Data Factory não carrega automaticamente os arquivos de dados ou scripts armazenados em um repositório GitHub para o Armazenamento do Azure.

- O GitHub Enterprise com uma versão mais antiga que 2.14.0 não funciona no navegador Microsoft Edge.

- A integração do GitHub com o Data Factory ferramentas de criação visual funciona apenas na versão disponível do Data Factory.

- Um máximo de 1.000 entidades por tipo de recurso (como pipelines e conjuntos de valores) pode ser buscado em uma única ramificação do GitHub. Se esse limite for atingido, o será sugerido para dividir seus recursos em fábricas separadas. O Git DevOps do Azure não tem essa limitação.

## <a name="version-control"></a>Controle de versão

Os sistemas de controle de versão (também conhecidos como _controle do código-fonte_) permitem aos desenvolvedores colaborar em código e acompanhar as alterações feitas no código base. O controle do código-fonte é uma ferramenta essencial para projetos de vários desenvolvedores.

### <a name="creating-feature-branches"></a>Criando ramificações de recurso

Cada repositório Git do Azure Repos que está associado a um data factory tem um branch de colaboração. (`master` é a ramificação de colaboração padrão). Os usuários também podem criar ramificações de recursos clicando em **+ nova ramificação** na lista suspensa Branch. Depois que o painel novo Branch for exibido, insira o nome do Branch de recursos.

![Criar uma nova ramificação](media/author-visually/new-branch.png)

Quando você estiver pronto para mesclar as alterações do Branch de recursos para sua ramificação de colaboração, clique na lista suspensa Branch e selecione **criar solicitação pull**. Essa ação o levará para o Git do Azure Repos, em que será possível gerar solicitações de pull, realizar revisões de código e mesclar alterações com o branch de colaboração. (`master` é o padrão). Você só tem permissão para publicar no serviço do Data Factory de sua ramificação de colaboração. 

![Criar uma nova solicitação pull](media/author-visually/create-pull-request.png)

### <a name="configure-publishing-settings"></a>Definir configurações de publicação

Por padrão, data factory gera os modelos do Resource Manager da fábrica publicada e os salva em uma ramificação `adf_public`chamada. Para configurar uma ramificação de publicação personalizada, `publish_config.json` adicione um arquivo à pasta raiz no Branch de colaboração. Ao publicar, o ADF lê esse arquivo, procura o campo `publishBranch`e salva todos os modelos do Resource Manager no local especificado. Se o Branch não existir, data factory irá criá-lo automaticamente. Veja abaixo o exemplo de como esse arquivo está abaixo:

```json
{
    "publishBranch": "factory/adf_publish"
}
```

Azure Data Factory pode ter apenas uma ramificação de publicação por vez. Quando você especifica um novo branch de publicação, o Data Factory não exclui o branch de publicação anterior. Se você quiser remover o Branch de publicação anterior, exclua-o manualmente.

> [!NOTE]
> O Data Factory apenas lê o arquivo `publish_config.json` quando ele carrega o factory. Se o factory já estiver carregado no portal, atualize o navegador para que as alterações entrem em vigor.

### <a name="publish-code-changes"></a>Publicar alterações de código

Depois de mesclar as alterações para a ramificação de`master` colaboração (é o padrão), clique em **publicar** para publicar manualmente as alterações de código no Branch mestre para o serviço de data Factory.

![Publicar as alterações no serviço do Data Factory](media/author-visually/publish-changes.png)

Um painel lateral será aberto onde você confirmar que a ramificação de publicação e as alterações pendentes estão corretas. Depois de verificar as alterações, clique em **OK** para confirmar a publicação.

![Confirmar o branch de publicação correto](media/author-visually/configure-publish-branch.png)

> [!IMPORTANT]
> O branch mestre não é representativo do que é implantado no serviço de Data Factory. O branch mestre *deve* ser publicado manualmente no serviço de Data Factory.

## <a name="best-practices-for-git-integration"></a>Melhores práticas para a integração do Git

### <a name="permissions"></a>Permissões

Normalmente, você não quer que todos os membros da equipe tenham permissões para atualizar a fábrica. As seguintes configurações de permissões são recomendadas:

*   Todos os membros da equipe devem ter permissões de leitura para o data factory.
*   Somente um conjunto selecionado de pessoas deve ter permissão para publicar na fábrica. Para fazer isso, eles devem ter a função de **colaborador de data Factory** no grupo de recursos no qual a fábrica está. Para obter mais informações sobre permissões, consulte [funções e permissões para Azure data Factory](concepts-roles-permissions.md).
   
É recomendável não permitir check-ins diretos para a ramificação de colaboração. Essa restrição pode ajudar a evitar bugs, uma vez que cada check-in passará por um processo de revisão de solicitação de pull descrito em [criando ramificações de recursos](source-control.md#creating-feature-branches).

### <a name="using-passwords-from-azure-key-vault"></a>Usando senhas de Azure Key Vault

É recomendável usar Azure Key Vault para armazenar quaisquer cadeias de conexão ou senhas ou a autenticação de identidade gerenciada para Data Factory serviços vinculados. Por motivos de segurança, data factory não armazena segredos no git. Quaisquer alterações nos serviços vinculados que contenham segredos como senhas são publicadas imediatamente no serviço de Azure Data Factory.

Usar a autenticação Key Vault ou MSI também facilita a integração e a implantação contínua, pois você não precisará fornecer esses segredos durante a implantação do modelo do Resource Manager.

## <a name="troubleshooting-git-integration"></a>Solução de problemas de integração do git

### <a name="stale-publish-branch"></a>Branch de publicação obsoleto

Se a ramificação de publicação estiver fora de sincronia com a ramificação mestre e contiver recursos desatualizados, independentemente de uma publicação recente, tente seguir estas etapas:

1. Remover seu repositório git atual
1. Reconfigure o Git com as mesmas configurações, mas certifique-se de que **importar recursos existentes do data Factory para o repositório** esteja selecionado e escolha **novo Branch**
1. Criar uma solicitação de pull para mesclar as alterações para a ramificação de colaboração 

Abaixo estão alguns exemplos de situações que podem causar uma ramificação de publicação obsoleta:
- Um usuário tem várias ramificações. Em um Branch de recurso, eles excluíram um serviço vinculado que não está associado a AKV (os serviços vinculados não AKV são publicados imediatamente, independentemente de estarem em git ou não) e nunca mesclaram a ramificação de recursos no Branch de colaboração.
- Um usuário modificou o data factory usando o SDK ou o PowerShell
- Um usuário moveu todos os recursos para uma nova ramificação e tentou publicar pela primeira vez. Os serviços vinculados devem ser criados manualmente ao importar recursos.
- Um usuário carrega um serviço vinculado não AKV ou um Integration Runtime JSON manualmente. Eles fazem referência a esse recurso de outro recurso, como um conjunto de um DataSet, serviço vinculado ou pipeline. Um serviço vinculado não AKV criado por meio do UX é publicado imediatamente porque as credenciais precisam ser criptografadas. Se você carregar um conjunto de uma referência a esse serviço vinculado e tentar publicar, a UX permitirá que ele exista no ambiente git. Ele será rejeitado no momento da publicação, pois não existe no serviço de data factory.

## <a name="switch-to-a-different-git-repository"></a>Alternar para um repositório git diferente

Para alternar para um repositório git diferente, clique no ícone **configurações do repositório git** no canto superior direito da página Visão geral do data Factory. Se você não conseguir ver o ícone, limpe o cache do navegador local. Selecione o ícone para remover a associação com o repositório atual.

![Ícone do git](media/author-visually/remove-repo.png)

Depois que o painel configurações do repositório for exibido, selecione **remover git**. Insira seu nome de data factory e clique em **confirmar** para remover o repositório git associado ao seu data Factory.

![Remover a associação com o repositório Git atual](media/author-visually/remove-repo2.png)

Depois de remover a associação com o repositório atual, você pode definir as configurações do git para usar um repositório diferente e, em seguida, importar recursos existentes do Data Factory para o novo repositório.

> [!IMPORTANT]
> Remover a configuração do git de um data factory não exclui nada do repositório. A fábrica conterá todos os recursos publicados. Você pode continuar a editar a fábrica diretamente no serviço.

## <a name="next-steps"></a>Próximas etapas

* Para saber mais sobre o monitoramento e o gerenciamento de pipelines, c [Monitorar e gerenciar os pipelines programaticamente](monitor-programmatically.md).
* Para implementar a integração e a implantação contínuas, consulte [integração e entrega contínuas (CI/CD) em Azure data Factory](continuous-integration-deployment.md).
