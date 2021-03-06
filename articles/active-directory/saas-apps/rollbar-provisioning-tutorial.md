---
title: 'Tutorial: configurar o rollbar para o provisionamento automático de usuário com o Azure Active Directory | Microsoft Docs'
description: Saiba como configurar Azure Active Directory para provisionar e desprovisionar automaticamente contas de usuário para o rollbar.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: d737aa16-8ab4-4c0c-a68b-2911623b41eb
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/26/2019
ms.author: Zhchia
ms.openlocfilehash: 27a26a0c8378f34794afd87cf11b6bb878f7b53c
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/28/2020
ms.locfileid: "78248417"
---
# <a name="tutorial-configure-rollbar-for-automatic-user-provisioning"></a>Tutorial: configurar o rollbar para o provisionamento automático de usuário

Este tutorial descreve as etapas que você precisa executar tanto no rollbar quanto no Azure Active Directory (Azure AD) para configurar o provisionamento automático de usuário. Quando configurado, o Azure AD provisiona e desprovisiona automaticamente usuários e grupos para [rollbar](https://rollbar.com/pricing/) usando o serviço de provisionamento do Azure AD. Para detalhes importantes sobre o que esse serviço faz, como funciona e as perguntas frequentes, consulte [Automatizar o provisionamento e desprovisionamento de usuários para aplicativos SaaS com o Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades com suporte
> [!div class="checklist"]
> * Criar usuários no rollbar
> * Remover usuários no rollbar quando eles não exigem mais acesso
> * Manter os atributos de usuário sincronizados entre o Azure AD e o rollbar
> * Provisionar grupos e associações de grupo no rollbar
> * [Logon único](https://docs.microsoft.com/azure/active-directory/saas-apps/rollbar-tutorial) no rollbar (recomendado)

## <a name="prerequisites"></a>Pré-requisitos

O cenário descrito neste tutorial pressupõe que você já tem os seguintes pré-requisitos:

* [Um locatário do Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Uma conta de usuário no Azure AD com [permissão](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar o provisionamento (por exemplo, administrador de aplicativos, administrador de aplicativos de nuvem, proprietário do aplicativo ou administrador global). 
* [Um locatário do rollbar](https://rollbar.com/pricing/) que tem um plano Enterprise.
* Uma conta de usuário no rollbar com permissões de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Etapa 1. Planejar sua implantação de provisionamento
1. Saiba mais sobre [como o serviço de provisionamento funciona](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quem estará no [escopo do provisionamento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine quais dados [mapeados entre o Azure AD e o rollbar](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-rollbar-to-support-provisioning-with-azure-ad"></a>Etapa 2. Configurar o rollbar para dar suporte ao provisionamento com o Azure AD

Antes de configurar o rollbar para o provisionamento automático de usuário com o Azure AD, será necessário habilitar o provisionamento do SCIM no rollbar.

1. Entre no console do [administrador do rollbar](https://rollbar.com/login/). Clique em **configurações de conta**.

    ![Console de administração do rollbar](media/rollbar-provisioning-tutorial/image00.png)

2. Navegue até o **nome do locatário do Rollbar > provedor de identidade**.

    ![Provedor de identidade rollbar](media/rollbar-provisioning-tutorial/idp.png)

3. Role para baixo até **Opções de provisionamento**. Copie o token de acesso. Esse valor será inserido no campo **token secreto** na guia provisionamento do seu aplicativo Rollbar no portal do Azure. Marque a caixa de seleção **habilitar o provisionamento de usuário e equipe** e clique em **salvar**.

    ![Token de acesso do rollbar](media/rollbar-provisioning-tutorial/token.png)


## <a name="step-3-add-rollbar-from-the-azure-ad-application-gallery"></a>Etapa 3. Adicionar o rollbar da Galeria de aplicativos do Azure AD

Adicione o rollbar da Galeria de aplicativos do Azure AD para começar a gerenciar o provisionamento no rollbar. Se você tiver configurado anteriormente o rollbar para SSO, poderá usar o mesmo aplicativo. No entanto, é recomendável que você crie um aplicativo separado ao testar a integração inicialmente. Saiba mais sobre como adicionar um aplicativo da Galeria [aqui](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Etapa 4. Definir quem estará no escopo para provisionamento 

O serviço de provisionamento do Azure AD permite o escopo que será provisionado com base na atribuição ao aplicativo e ou com base em atributos do usuário/grupo. Se você optar por definir o escopo que será provisionado em seu aplicativo com base na atribuição, poderá usar as [etapas](../manage-apps/assign-user-or-group-access-portal.md) a seguir para atribuir usuários e grupos ao aplicativo. Se você escolher o escopo que será provisionado com base apenas em atributos do usuário ou grupo, você poderá usar um filtro de escopo conforme descrito [aqui](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Ao atribuir usuários e grupos ao rollbar, você deve selecionar uma função diferente de **acesso padrão**. Os usuários com a função de acesso padrão são excluídos do provisionamento e serão marcados como não habilitados com eficiência nos logs de provisionamento. Se a única função disponível no aplicativo for a função de acesso padrão, você poderá [atualizar o manifesto do aplicativo](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para adicionar outras funções. 

* Comece pequeno. Teste com um pequeno conjunto de usuários e grupos antes de distribuir para todos. Quando o escopo do provisionamento é definido como usuários e grupos atribuídos, você pode controlar isso atribuindo um ou dois usuários ou grupos ao aplicativo. Quando o escopo é definido como todos os usuários e grupos, você pode especificar um [filtro de escopo baseado em atributo](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-rollbar"></a>Etapa 5. Configurar o provisionamento automático de usuário para o rollbar 

Esta seção orienta você pelas etapas para configurar o serviço de provisionamento do Azure AD para criar, atualizar e desabilitar usuários e/ou grupos no TestApp com base em atribuições de usuário e/ou grupo no Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-rollbar-in-azure-ad"></a>Para configurar o provisionamento automático de usuário para rollbar no Azure AD:

1. Entre no [portal do Azure](https://portal.azure.com). Selecione **aplicativos empresariais**e, em seguida, selecione **todos os aplicativos**.

    ![Folha de aplicativos empresariais](common/enterprise-applications.png)

2. Na lista de aplicativos, selecione **Rollbar**.

    ![O link do Rollbar na Lista de aplicativos](common/all-applications.png)

3. Selecione a guia **Provisionamento**.

    ![Guia provisionamento](common/provisioning.png)

4. Defina o **modo de provisionamento** como **automático**.

    ![Guia provisionamento](common/provisioning-automatic.png)

5. Na seção **credenciais de administrador** , insira o valor do token de acesso recuperado anteriormente no **token secreto**. Clique em **testar conexão** para garantir que o Azure ad possa se conectar ao rollbar. Se a conexão falhar, verifique se sua conta do rollbar tem permissões de administrador e tente novamente.

    ![Provisionamento](./media/rollbar-provisioning-tutorial/admin.png)

6. No campo **email de notificação** , insira o endereço de email de uma pessoa ou grupo que deve receber as notificações de erro de provisionamento e marque a caixa de seleção **Enviar uma notificação por email quando ocorrer uma falha** .

    ![Email de notificação](common/provisioning-notification-email.png)

7. Selecione **Salvar**.

8. Na seção **mapeamentos** , selecione **sincronizar Azure Active Directory usuários para rollbar**.

9. Examine os atributos de usuário que são sincronizados do Azure AD para o rollbar na seção de **mapeamento de atributo** . Os atributos selecionados como propriedades **correspondentes** são usados para corresponder as contas de usuário no rollbar para operações de atualização. Se você optar por alterar o [atributo de destino correspondente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), será necessário garantir que a API rollbar dê suporte à filtragem de usuários com base nesse atributo. Selecione o botão **Salvar** para confirmar as alterações.

   |Atributo|Type|
   |---|---|
   |userName|Cadeia de caracteres|
   |externalId|Cadeia de caracteres|
   |ativo|Booliano|
   |name.familyName|Cadeia de caracteres|
   |name.givenName|Cadeia de caracteres|
   |emails [tipo EQ "trabalho"]|Cadeia de caracteres|

10. Na seção **mapeamentos** , selecione **sincronizar grupos de Azure Active Directory para rollbar**.

11. Examine os atributos de grupo que são sincronizados do Azure AD para o rollbar na seção de **mapeamento de atributo** . Os atributos selecionados como propriedades **correspondentes** são usados para corresponder os grupos no rollbar para operações de atualização. Selecione o botão **Salvar** para confirmar as alterações.

      |Atributo|Type|
      |---|---|
      |displayName|Cadeia de caracteres|
      |externalId|Cadeia de caracteres|
      |membros|Referência|

12. Para configurar filtros de escopo, consulte as seguintes instruções fornecidas no [tutorial do Filtro de Escopo](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar o serviço de provisionamento do Azure AD para o rollbar, altere o **status de provisionamento** para **ativado** na seção **configurações** .

    ![Status do provisionamento ativado](common/provisioning-toggle-on.png)

14. Defina os usuários e/ou grupos que você deseja provisionar para o rollbar escolhendo os valores desejados no **escopo** na seção **configurações** .

    ![Escopo de provisionamento](common/provisioning-scope.png)

15. Quando estiver pronto para provisionar, clique em **Salvar**.

    ![Salvando a configuração de provisionamento](common/provisioning-configuration-save.png)

Essa operação inicia o ciclo de sincronização inicial de todos os usuários e grupos definidos no **escopo** na seção **configurações** . O ciclo inicial leva mais tempo para ser executado do que os ciclos subsequentes, que ocorrem aproximadamente a cada 40 minutos, desde que o serviço de provisionamento do Azure AD esteja em execução. 

## <a name="step-6-monitor-your-deployment"></a>Etapa 6. Monitorar a implantação
Depois de configurar o provisionamento, use os seguintes recursos para monitorar sua implantação:

1. Use os [logs de provisionamento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar quais usuários foram provisionados com êxito ou sem êxito
2. Verifique a [barra de progresso](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-when-will-provisioning-finish-specific-user) para ver o status do ciclo de provisionamento e como fechá-lo para conclusão
3. Se a configuração de provisionamento parecer estar em um estado não íntegro, o aplicativo entrará em quarentena. Saiba mais sobre os Estados de quarentena [aqui](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).

## <a name="additional-resources"></a>Recursos adicionais

* [Gerenciando o provisionamento de conta de usuário para aplicativos empresariais](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Próximas etapas

* [Saiba como fazer revisão de logs e obter relatórios sobre atividade de provisionamento](../manage-apps/check-status-user-account-provisioning.md)
