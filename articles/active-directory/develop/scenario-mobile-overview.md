---
title: Compilar um aplicativo móvel que chama APIs da Web | Azure
titleSuffix: Microsoft identity platform | Azure
description: Saiba como criar um aplicativo móvel que chama APIs da Web (visão geral)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: brandwe
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 1f90f7f23fbdf10b91d8dfc7cd00cca83cd32fbc
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "80882566"
---
# <a name="scenario-mobile-application-that-calls-web-apis"></a>Cenário: aplicativo móvel que chama APIs da Web

Saiba como criar um aplicativo móvel que chama APIs da Web.

## <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [Prerequisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="getting-started"></a>Introdução

Crie seu primeiro aplicativo móvel e experimente um início rápido.

> [!div class="nextstepaction"]
> [Início rápido: adquirir um token e chamar Microsoft Graph API de um aplicativo Android](./quickstart-v2-android.md)
>
> [Início rápido: adquirir um token e chamar Microsoft Graph API de um aplicativo iOS](./quickstart-v2-ios.md)
>
> [Início rápido: adquirir um token e chamar Microsoft Graph API de um aplicativo Xamarin iOS e Android](https://github.com/Azure-Samples/active-directory-xamarin-native-v2)

## <a name="overview"></a>Visão geral

Uma experiência de usuário personalizada e direta é essencial para aplicativos móveis.  A plataforma de identidade da Microsoft permite que os desenvolvedores móveis criem essa experiência para usuários de iOS e Android. Seu aplicativo pode entrar Azure Active Directory (Azure AD) usuários, usuários pessoais conta Microsoft e Azure AD B2C usuários. Ele também pode adquirir tokens para chamar uma API da Web em seu nome. Para implementar esses fluxos, usaremos a MSAL (biblioteca de autenticação da Microsoft). O MSAL implementa o fluxo de código de autorização padrão do setor [OAuth 2.0](v2-oauth2-auth-code-flow.md).

![Aplicativos Daemon](./media/scenarios/mobile-app.svg)

Considerações para aplicativos móveis:

- A **experiência do usuário é a chave**: permite que os usuários vejam o valor do seu aplicativo antes de solicitar a entrada. Solicite apenas as permissões necessárias.
- **Suporte a todas as configurações de usuário**: muitos usuários comerciais móveis devem aderir às políticas de acesso condicional e às políticas de conformidade do dispositivo. Certifique-se de dar suporte a esses cenários principais.
- **Implementar SSO (logon único)**: usando o MSAL e a plataforma de identidade da Microsoft, você pode habilitar o logon único por meio do navegador do dispositivo ou Microsoft Authenticator (e portal da empresa do Intune no Android).
- **Implementar o modo de dispositivo compartilhado**: habilite seu aplicativo a ser usado em cenários de dispositivo compartilhado, por exemplo, hospitais, manufatura, varejo e finanças. [Leia mais sobre como dar suporte ao modo de dispositivo compartilhado](msal-shared-devices.md).

## <a name="specifics"></a>Especificações

Tenha em mente as seguintes considerações ao criar um aplicativo móvel na plataforma Microsoft Identity:

- Dependendo da plataforma, parte da interação do usuário pode ser necessária na primeira vez que os usuários entrarem. Por exemplo, o iOS exige que os aplicativos mostrem a interação do usuário quando usam o SSO pela primeira vez Microsoft Authenticator (e Portal da Empresa do Intune no Android).
- No iOS e no Android, o MSAL pode usar um navegador externo para conectar usuários. O navegador externo pode aparecer na parte superior do seu aplicativo.
- Nunca use um segredo em um aplicativo móvel. Nesses aplicativos, os segredos podem ser acessados por todos os usuários.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Registro do aplicativo](scenario-mobile-app-registration.md)
