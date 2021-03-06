<properties
	pageTitle="Aplicativo Web .NET do Azure AD v2.0 | Microsoft Azure"
	description="Como criar um aplicativo Web do .NET MVC que conecte usuários com a conta pessoal da Microsoft e as contas corporativas ou de estudante."
	services="active-directory"
	documentationCenter=".net"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="05/31/2016"
	ms.author="dastrock"/>

# Adicionar as credenciais para um aplicativo Web .NET MVC

Com o ponto de extremidade v2.0, você pode adicionar autenticação rapidamente a seus aplicativos Web com suporte a contas pessoais da Microsoft e contas corporativas ou de estudante. Nos aplicativos Web ASP.NET, você pode conseguir isso usando o middleware OWIN da Microsoft, incluso no .NET Framework 4.5.

> [AZURE.NOTE]
	Nem todos os recursos e cenários do Azure Active Directory têm suporte no ponto de extremidade v2.0. Para determinar se você deve usar o ponto de extremidade v2.0, leia sobre as [limitações da v2.0](active-directory-v2-limitations.md).

 Compilaremos aqui um aplicativo Web que usa o OWIN para fazer logon do usuário, exibir algumas informações sobre o usuário e fazer logout do usuário do aplicativo.
 
 ## Download O código deste tutorial é mantido [no GitHub](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet). Para acompanhar, você pode [baixar o esqueleto do aplicativo como um .zip](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet/archive/skeleton.zip) ou clonar o esqueleto:

```git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet.git```

O aplicativo concluído é fornecido também no final desse tutorial.

## Registrar um aplicativo
Crie um novo aplicativo em [apps.dev.microsoft.com](https://apps.dev.microsoft.com) ou siga estas [etapas detalhadas](active-directory-v2-app-registration.md). Não se esqueça de:

- Copiar a **ID do Aplicativo** designada ao seu aplicativo, você precisará dela logo.
- Adicionar a plataforma **Web** para seu aplicativo.
- Inserir o **URI de Redirecionamento** correto. O URI de redirecionamento indica ao Azure AD para onde as respostas de autenticação devem ser direcionadas — o padrão para este tutorial é `https://localhost:44326/`.

## Instalar e configurar a autenticação OWIN
Aqui, configuraremos middleware OWIN para usar o protocolo de autenticação OpenID Connect. OWIN será usado para emitir solicitações de entrada e saída, gerenciar a sessão do usuário e obter informações sobre o usuário, entre outras coisas.

-	Para começar, abra o arquivo `web.config` na raiz do projeto e insira os valores de configuração do aplicativo na seção `<appSettings>`.
    -	`ida:ClientId` é a **ID do Aplicativo** atribuída ao seu aplicativo no portal de registro.
    -	`ida:RedirectUri` é o **URI de Redirecionamento** inserido no portal.

-	Em seguida, adicione o ADAL aos pacotes NuGet de middleware ao projeto usando o Console do Gerenciador de Pacotes.

```
PM> Install-Package Microsoft.Owin.Security.OpenIdConnect
PM> Install-Package Microsoft.Owin.Security.Cookies
PM> Install-Package Microsoft.Owin.Host.SystemWeb
```

-	Adicione uma classe de inicialização do OWIN ao projeto chamado `Startup.cs` Clique com o botão direito do mouse no projeto, selecione --> **Adicionar** --> **Novo Item** --> Pesquise por "OWIN". O middleware OWIN invocará o método `Configuration(...)` quando seu aplicativo for iniciado.
-	Altere a declaração de classe para `public partial class Startup` -já implementamos parte dessa classe para você em outro arquivo. No método `Configuration(...)`, faça uma chamada para ConfigureAuth(...) para configurar a autenticação para seu aplicativo Web  

```C#
[assembly: OwinStartup(typeof(Startup))]

namespace TodoList_WebApp
{
	public partial class Startup
	{
		public void Configuration(IAppBuilder app)
		{
			ConfigureAuth(app);
		}
	}
}```

-	Abra o arquivo `App_Start\Startup.Auth.cs` e implemente o método `ConfigureAuth(...)`. Os parâmetros que você fornece em `OpenIdConnectAuthenticationOptions` servirão como coordenadas para seu aplicativo para se comunicar com o AD do Azure. Você também precisa configurar a autenticação de Cookies - o middleware OpenID Connect usa cookies nos bastidores.

```C#
public void ConfigureAuth(IAppBuilder app)
			 {
					 app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

					 app.UseCookieAuthentication(new CookieAuthenticationOptions());

					 app.UseOpenIdConnectAuthentication(
							 new OpenIdConnectAuthenticationOptions
							 {
									 // The `Authority` represents the v2.0 endpoint - https://login.microsoftonline.com/common/v2.0 
									 // The `Scope` describes the permissions that your app will need.  See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/
									 // In a real application you could use issuer validation for additional checks, like making sure the user's organization has signed up for your app, for instance.

									 ClientId = clientId,
									 Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0 "),
									 RedirectUri = redirectUri,
									 Scope = "openid email profile",
									 ResponseType = "id_token",
									 PostLogoutRedirectUri = redirectUri,
									 TokenValidationParameters = new TokenValidationParameters
									 {
											 ValidateIssuer = false,
									 },
									 Notifications = new OpenIdConnectAuthenticationNotifications
									 {
											 AuthenticationFailed = OnAuthenticationFailed,
									 }
							 });
			 }
```

## Enviar solicitações de autenticação
Seu aplicativo agora está configurado corretamente para se comunicar com o ponto de extremidade v2.0 usando o protocolo de autenticação OpenID Connect. O OWIN cuidou de todos os detalhes difíceis da criação de mensagens de autenticação, validação de tokens do AD do Azure e manutenção da sessão do usuário. Tudo o que falta é oferecer aos usuários uma maneira de entrar e sair.

- Você pode usar autorizar marcas em seus controladores para exigir que o usuário entre antes de acessar uma determinada página. Abra `Controllers\HomeController.cs` e adicione a marca `[Authorize]` ao controlador Sobre.

```C#
[Authorize]
public ActionResult About()
{
  ...
```

-	Você também pode usar o OWIN para emitir diretamente solicitações de autenticação de dentro de seu código. Abra `Controllers\AccountController.cs`. Nas ações SignIn() e SignOut(), emita as solicitações de desafio do OpenID Connect e de saída, respectivamente.

```C#
public void SignIn()
{
    // Send an OpenID Connect sign-in request.
    if (!Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge(new AuthenticationProperties { RedirectUri = "/" }, OpenIdConnectAuthenticationDefaults.AuthenticationType);
    }
}

// BUGBUG: Ending a session with the v2.0 endpoint is not yet supported.  Here, we just end the session with the web app.  
public void SignOut()
{
    // Send an OpenID Connect sign-out request.
    HttpContext.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
    Response.Redirect("/");
}
```

-	Agora, abra `Views\Shared\_LoginPartial.cshtml`. É aqui que você vai mostrar ao usuário links de entrada e saída do seu aplicativo e imprimir o nome do usuário em uma exibição.

```HTML
@if (Request.IsAuthenticated)
{
    <text>
        <ul class="nav navbar-nav navbar-right">
            <li class="navbar-text">

                @*The 'preferred_username' claim can be used for showing the user's primary way of identifying themselves.*@

                Hello, @(System.Security.Claims.ClaimsPrincipal.Current.FindFirst("preferred_username").Value)!
            </li>
            <li>
                @Html.ActionLink("Sign out", "SignOut", "Account")
            </li>
        </ul>
    </text>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li>@Html.ActionLink("Sign in", "SignIn", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
    </ul>
}
```

## Exibir informações do usuário
Ao autenticar usuários com o OpenID Connect, o ponto de extremidade v2.0 retorna um id\_token para o aplicativo que contém [declarações](active-directory-v2-tokens.md#id_tokens) ou afirmações sobre o usuário. Você pode usar essas declarações para personalizar seu aplicativo:

- Abra o arquivo `Controllers\HomeController.cs`. Você pode acessar as declarações do usuário em seus controladores por meio do objeto principal de segurança `ClaimsPrincipal.Current`.

```C#
[Authorize]
public ActionResult About()
{
    ViewBag.Name = ClaimsPrincipal.Current.FindFirst("name").Value;

    // The object ID claim will only be emitted for work or school accounts at this time.
    Claim oid = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier");
    ViewBag.ObjectId = oid == null ? string.Empty : oid.Value;

    // The 'preferred_username' claim can be used for showing the user's primary way of identifying themselves
    ViewBag.Username = ClaimsPrincipal.Current.FindFirst("preferred_username").Value;

    // The subject or nameidentifier claim can be used to uniquely identify the user
    ViewBag.Subject = ClaimsPrincipal.Current.FindFirst("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier").Value;

    return View();
}
```

## Executar

Por fim, compile e execute seu aplicativo! Entre com uma conta pessoal da Microsoft ou uma conta corporativa ou de estudante e observe como a identidade do usuário é exibida na barra de navegação superior. Agora você tem um aplicativo Web protegido por protocolos padrão do setor, que podem autenticar usuários com as respectivas contas pessoais e corporativas ou de estudante.

Para referência, o exemplo concluído (sem seus valores de configuração) [é fornecido como um .zip aqui](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet/archive/complete.zip), ou você pode cloná-lo do GitHub:

```git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet.git```

## Próximas etapas

Agora você pode ir para tópicos mais avançados. Você pode desejar experimentar:

[Proteger uma API Web com o ponto de extremidade v2.0 >>](active-directory-devquickstarts-webapi-dotnet.md)

Para obter recursos adicionais, consulte:
- [Guia do desenvolvedor do v2.0 >>](active-directory-appmodel-v2-overview.md)
- [Marca “azure-active-directory” do StackOverflow >>](http://stackoverflow.com/questions/tagged/azure-active-directory)

## Obter atualizações de segurança para nossos produtos

Recomendamos que você obtenha notificações sobre a ocorrência de incidentes de segurança visitando [esta página](https://technet.microsoft.com/security/dd252948) e assinando os alertas do Security Advisory.

<!---HONumber=AcomDC_0608_2016-->