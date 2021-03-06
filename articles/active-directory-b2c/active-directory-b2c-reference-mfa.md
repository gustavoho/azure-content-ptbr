<properties
	pageTitle="Visualização do Active Directory B2C do Azure: Multi-Factor Authentication | Microsoft Azure"
	description="Como habilitar o Multi-Factor Authentication em aplicativos voltados para o consumidor protegidos pelo Active Directory B2C do Azure"
	services="active-directory-b2c"
	documentationCenter=""
	authors="swkrish"
	manager="msmbaldwin"
	editor="bryanla"/>

<tags
	ms.service="active-directory-b2c"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/27/2016"
	ms.author="swkrish"/>

# Visualização do Azure Active Directory B2C: habilitar a Multi-Factor Authentication nos seus aplicativos voltados para o consumidor

O Azure AD (Azure Active Directory) B2C integra-se diretamente ao [Azure Multi-Factor Authentication](../multi-factor-authentication/multi-factor-authentication.md) para facilitar a adição de uma segunda camada de segurança para as experiências de inscrição e entrada nos seus aplicativos voltados para o consumidor. E você pode fazer isso sem escrever uma única linha de código. No momentos damos suporte à verificação por ligação telefônica e mensagem de texto. Se você já tiver criado as políticas de credenciais e de entrada, ainda poderá habilitar a Multi-Factor Authentication.

[AZURE.INCLUDE [active-directory-b2c-preview-note](../../includes/active-directory-b2c-preview-note.md)]

> [AZURE.NOTE]
A Multi-Factor Authentication também pode ser habilitada durante a criação de políticas de credenciais e de entrada, não apenas na edição de políticas existentes.

Esse recurso ajuda os aplicativos a lidarem com cenários como os seguintes:

- Você não exige Multi-Factor Authentication para acessar um aplicativo, mas para acessar outro sim. Por exemplo, o consumidor pode entrar em um aplicativo de seguro automobilístico com uma conta local ou social, porém precisa verificar o número de telefone para poder acessar o aplicativo de seguro residencial registrado no mesmo diretório.
- Você não exige a Multi-Factor Authentication para acessar um aplicativo em geral, mas sim para acessar partes confidenciais dele. Por exemplo, o consumidor pode entrar em um aplicativo bancário com uma conta local ou social e consultar o saldo da conta, porém precisa verificar o número de telefone para poder tentar realizar uma transferência bancária.

## Modificar a política de inscrição para habilitar a Multi-Factor Authentication

1. [Siga estas etapas para navegar até a folha de recursos do B2C no Portal do Azure](active-directory-b2c-app-registration.md#navigate-to-the-b2c-features-blade).
2. Clique em **Políticas de inscrição**.
3. Clique na sua política de inscrição (por exemplo, "B2C\_1\_SiUp") para abri-la.
4. Clique em **Autenticação multifator** e ative o **Estado** colocando em **ON**. Clique em **OK**.
5. Clique em **Salvar** na parte superior da folha.

Você pode usar o recurso "Executar agora" da política para verificar a experiência do consumidor. Confirme o seguinte:

Uma conta de consumidor é criada no diretório antes da etapa de Multi-Factor Authentication. Durante a etapa, o consumidor é solicitado a fornecer seu número de telefone e a verificá-lo. Se a verificação for bem-sucedida, o número de telefone será anexado à conta de consumidor para uso posterior. Mesmo que o consumidor cancele a ação ou saia, pode ser solicitado que ele confirme um número de telefone novamente durante a próxima conexão (com a Multi-Factor Authentication habilitada).

## Modificar a política de entrada para habilitar a Multi-Factor Authentication

1. [Siga estas etapas para navegar até a folha de recursos do B2C no Portal do Azure](active-directory-b2c-app-registration.md#navigate-to-the-b2c-features-blade).
2. Clique em **Políticas de entrada**.
3. Clique na sua política de entrada (por exemplo, "B2C\_1\_SiIn") para abri-la. Clique em **Editar** na parte superior da folha.
4. Clique em **Autenticação multifator** e ative o **Estado** colocando em **ON**. Clique em **OK**.
5. Clique em **Salvar** na parte superior da folha.

Você pode usar o recurso "Executar agora" da política para verificar a experiência do consumidor. Confirme o seguinte:

Quando o consumidor entra (usando uma conta local ou social), se um número de telefone verificado está anexado à conta de consumidor, ele deve verificá-lo. Se nenhum número de telefone estiver anexado, o consumidor é solicitado a fornecer seu número de telefone e a confirmá-lo. Se a verificação for bem-sucedida, o número de telefone será anexado à conta do consumidor para uso posterior.

## Multi-Factor Authentication em outras políticas

Conforme descrito para as políticas de inscrição e entrada acima, também é possível habilitar a autenticação multifator nas políticas de inscrição ou entrada e nas políticas de redefinição de senha. Ela estará disponível em breve nas políticas de edição de perfil.

<!---HONumber=AcomDC_0629_2016-->