---
title: Configurar o Microsoft authentication
description: Saiba como configurar a autenticação da conta da Microsoft como um provedor de identidade para seu serviço de aplicativo ou Azure Functions aplicativo.
ms.assetid: ffbc6064-edf6-474d-971c-695598fd08bf
ms.topic: article
ms.date: 08/08/2019
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: 6fe0e39584cbaa001e11f712bb2f60b45dba6914
ms.sourcegitcommit: 67bddb15f90fb7e845ca739d16ad568cbc368c06
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "82203481"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-microsoft-account-login"></a>Configurar seu serviço de aplicativo ou Azure Functions aplicativo para usar o logon da conta da Microsoft

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Este tópico mostra como configurar Azure App serviço ou Azure Functions para usar o AAD para dar suporte a logons de conta Microsoft pessoal.

> [!NOTE]
> As contas pessoais da Microsoft e as contas organizacionais usam o provedor de identidade do AAD. Neste momento, não é possível configurar esse provedor de identidade para dar suporte a ambos os tipos de log-ins.

## <a name="register-your-app-with-microsoft-account"></a><a name="register-microsoft-account"> </a>Registrar seu aplicativo na conta da Microsoft

1. Vá para [**registros de aplicativo**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) na portal do Azure. Se necessário, entre com seu conta Microsoft.
1. Selecione **novo registro**e insira um nome de aplicativo.
1. Em **tipos de conta com suporte**, selecione **contas em qualquer diretório organizacional (qualquer Azure ad Directory-multilocatário) e contas pessoais da Microsoft (por exemplo, Skype, Xbox)**
1. Em **URIs de redirecionamento**, selecione **Web**e `https://<app-domain-name>/.auth/login/aad/callback`, em seguida, insira. Substitua * \<app-Domain-Name>* pelo nome de domínio do seu aplicativo.  Por exemplo, `https://contoso.azurewebsites.net/.auth/login/aad/callback`. Certifique-se de usar o esquema HTTPS na URL.

1. Selecione **Registrar**.
1. Copie a **ID do aplicativo (cliente)**. Você precisará dela mais tarde.
1. No painel esquerdo, selecione **certificados & segredos** > **novo segredo do cliente**. Insira uma descrição, selecione a duração da validade e selecione **Adicionar**.
1. Copie o valor que aparece na página **certificados & segredos** . Depois de sair da página, ela não será exibida novamente.

    > [!IMPORTANT]
    > O valor de segredo do cliente (senha) é uma credencial de segurança importante. Não compartilhe a senha com ninguém nem distribua-a em um aplicativo cliente.

## <a name="add-microsoft-account-information-to-your-app-service-application"></a><a name="secrets"> </a>Adicionar informações da conta da Microsoft ao seu aplicativo de Serviço de Aplicativo

1. Vá para seu aplicativo no [portal do Azure].
1. Selecione **configurações** > **autenticação/autorização**e verifique se a **autenticação do serviço de aplicativo** está **ativada**.
1. Em **provedores de autenticação**, selecione **Azure Active Directory**. Selecione **avançado** em **modo de gerenciamento**. Cole a ID do aplicativo (cliente) e o segredo do cliente obtidos anteriormente. Use **`https://login.microsoftonline.com/9188040d-6c67-4c5b-b112-36a304b66dad/v2.0`** para o campo **URL do emissor** .
1. Selecione **OK**.

   O serviço de aplicativo fornece autenticação, mas não restringe o acesso autorizado ao conteúdo e às APIs do site. Você deve autorizar os usuários no código do aplicativo.

1. Adicional Para restringir o acesso a conta Microsoft usuários, defina a **ação a ser tomada quando a solicitação não for autenticada** para **fazer logon com Azure Active Directory**. Quando você define essa funcionalidade, seu aplicativo requer que todas as solicitações sejam autenticadas. Ele também redireciona todas as solicitações não autenticadas para usar o AAD para autenticação. Observe que, como você configurou a **URL do emissor** para usar o locatário da conta da Microsoft, somente os conta pessoais serão autenticados com êxito.

   > [!CAUTION]
   > Restringir o acesso dessa maneira se aplica a todas as chamadas para seu aplicativo, o que pode não ser desejável para aplicativos que têm um home page publicamente disponível, como em muitos aplicativos de página única. Para tais aplicativos, **Permitir solicitações anônimas (nenhuma ação)** pode ser preferível para que o aplicativo inicie manualmente a autenticação em si. Para obter mais informações, consulte [fluxo de autenticação](overview-authentication-authorization.md#authentication-flow).

1. Selecione **Salvar**.

Agora você está pronto para usar uma conta da Microsoft para autenticação em seu aplicativo.

## <a name="next-steps"></a><a name="related-content"> </a>Próximas etapas

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- URLs. -->

[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Azure portal]: https://portal.azure.com/
