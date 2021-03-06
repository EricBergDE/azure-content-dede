<properties
	pageTitle="Azure Active Directory B2C-Vorschau | Microsoft Azure"
	description="Erstellen einer Webanwendung, die eine Web-API mithilfe von Azure Active Directory B2C aufruft."
	services="active-directory-b2c"
	documentationCenter=".net"
	authors="dstrockis"
	manager="msmbaldwin"
	editor=""/>

<tags
	ms.service="active-directory-b2c"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="05/16/2016"
	ms.author="dastrock"/>

# Azure AD B2C-Vorschau: Aufrufen einer Web-API über eine .NET-Web-App


<!-- TODO [AZURE.INCLUDE [active-directory-b2c-devquickstarts-web-switcher](../../includes/active-directory-b2c-devquickstarts-web-switcher.md)]-->

Mit Azure Active Directory (Azure AD) B2C können Sie Ihren Web-Apps und Web-APIs in wenigen Schritten leistungsstarke Self-Service-Features zur Identitätsverwaltung hinzufügen. In diesem Artikel wird erläutert, wie Sie eine .NET Model-View-Controller-Web-App (MVC) für Aufgabenlisten erstellen, die eine .NET-Web-API mithilfe von OAuth 2.0-Bearertoken aufruft. Sowohl die Web-App als auch die Web-API verwenden Azure AD B2C zum Verwalten von Benutzeridentitäten und zum Authentifizieren von Benutzern.

[AZURE.INCLUDE [active-directory-b2c-preview-note](../../includes/active-directory-b2c-preview-note.md)]

Dieser Artikel behandelt nicht das Implementieren der Anmeldung, Registrierung und Profilverwaltung mit Azure AD B2C. Er konzentriert sich auf das Aufrufen von Web-APIs aufrufen, nachdem der Benutzer bereits authentifiziert wurde. Beginnen Sie, sofern noch nicht geschehen, mit dem [Tutorial zu den ersten Schritten mit .NET-Web-Apps](active-directory-b2c-devquickstarts-web-dotnet.md), um Informationen zu den Grundlagen von Azure AD B2C zu erhalten.

## Erstellen eines Azure AD B2C-Verzeichnisses

Bevor Sie Azure AD B2C verwenden können, müssen Sie ein Verzeichnis oder einen Mandanten erstellen. Ein Verzeichnis ist ein Container für alle Benutzer, Apps, Gruppen usw. Wenn Sie noch keines verwenden, [erstellen Sie ein B2C-Verzeichnis](active-directory-b2c-get-started.md), bevor Sie die weiteren Schritte in diesem Leitfaden ausführen.

## Erstellen einer Anwendung

Als Nächstes müssen Sie in Ihrem B2C-Verzeichnis eine App erstellen. Dadurch werden Azure AD die Informationen bereitgestellt, die für die sichere Kommunikation mit Ihrer App erforderlich sind. In diesem Fall werden die Web-App und die Web-API durch eine einzige **Anwendungs-ID** dargestellt, da sie zusammen eine logische App bilden. Befolgen Sie zum Erstellen einer App [diese Anweisungen](active-directory-b2c-app-registration.md). Führen Sie folgende Schritte aus:

- Fügen Sie der Anwendung eine **Web-App/Web-API** hinzu.
- Geben Sie `https://localhost:44316/` als **Antwort-URL** ein. Dies ist die Standard-URL für dieses Codebeispiel.
- Erstellen Sie einen **geheimen Schlüssel für Ihre Anwendung**, und kopieren Sie ihn. Sie benötigen sie später. Beachten Sie, dass dieser Wert vor der Verwendung in [XML-Escapezeichen](https://www.w3.org/TR/2006/REC-xml11-20060816/#dt-escape) gesetzt werden muss.
- Kopieren Sie die **Anwendungs-ID**, die Ihrer App zugewiesen ist. Diese benötigen sie später ebenfalls.

[AZURE.INCLUDE [active-directory-b2c-devquickstarts-v2-apps](../../includes/active-directory-b2c-devquickstarts-v2-apps.md)]

## Erstellen der Richtlinien

In Azure AD B2C wird jede Benutzeroberfläche durch eine [Richtlinie](active-directory-b2c-reference-policies.md) definiert. Diese Web-App enthält drei Benutzeroberflächen, für die Identitäten relevant sind: Registrierung, Anmeldung und Profilbearbeitung. Sie müssen eine Richtlinie jedes Typs erstellen, wie im [Artikel zu Richtlinienreferenzen](active-directory-b2c-reference-policies.md#how-to-create-a-sign-up-policy) beschrieben. Beachten Sie beim Erstellen der drei Richtlinien Folgendes:

- Wählen Sie den **Anzeigenamen** und andere Registrierungsattribute in der Registrierungsrichtlinie aus.
- Wählen Sie den **Anzeigenamen** und die **Objekt-ID** als Anwendungsansprüche in jeder Richtlinie aus. Sie können auch andere Ansprüche auswählen.
- Notieren Sie sich die **Namen** der einzelnen Richtlinien nach ihrer Erstellung. Sie müssen das Präfix `b2c_1_` aufweisen. Sie benötigen diese Richtliniennamen später.

[AZURE.INCLUDE [active-directory-b2c-devquickstarts-policy](../../includes/active-directory-b2c-devquickstarts-policy.md)]

Nachdem Sie die drei Richtlinien erstellt haben, können Sie Ihre App erstellen.

Beachten Sie, dass in diesem Artikel nicht behandelt wird, wie die gerade erstellten Richtlinien verwendet werden. Grundlegende Informationen zur Funktionsweise von Richtlinien in Azure AD B2C finden Sie im [Tutorial zu den ersten Schritten mit .NET-Web-Apps](active-directory-b2c-devquickstarts-web-dotnet.md).

## Herunterladen des Codes

[AZURE.INCLUDE [active-directory-b2c-preview-note](../../includes/active-directory-b2c-devquickstarts-bug-fix.md)]

Der Code für dieses Tutorial wird [auf GitHub](https://github.com/AzureADQuickStarts/B2C-WebApp-WebAPI-OpenIDConnect-DotNet) verwaltet. Zum Erstellen des Beispiels können Sie [das Projektgerüst als ZIP-Datei herunterladen](https://github.com/AzureADQuickStarts/B2C-WebApp-WebAPI-OpenIDConnect-DotNet/archive/skeleton.zip). Sie können das Gerüst auch klonen:

```
git clone --branch skeleton https://github.com/AzureADQuickStarts/B2C-WebApp-WebAPI-OpenIDConnect-DotNet.git
```

Die fertige App ist ebenfalls [als ZIP-Datei verfügbar](https://github.com/AzureADQuickStarts/B2C-WebApp-WebAPI-OpenIDConnect-DotNet/archive/complete.zip) oder unter der Verzweigung `complete` im gleichen Repository enthalten.

Nachdem Sie den Beispielcode heruntergeladen haben, öffnen Sie zum Einstieg die Visual Studio-SLN-Datei. Beachten Sie, dass in der Projektmappe zwei Projekte enthalten sind: ein `TaskWebApp`-Projekt und ein `TaskService`-Projekt. `TaskWebApp` ist das Front-End der Windows Presentation Foundation-Web-App (WPF), mit der der Benutzer interagiert. `TaskService` ist die Back-End-Web-API der App, in der die Aufgabenlisten der einzelnen Benutzer gespeichert werden.

## Konfigurieren des Aufgabendiensts

Wenn `TaskService` eine Anforderung von `TaskWebApp` erhält, sucht er nach einem gültigen Zugriffstoken zum Authentifizieren der Anforderung. Zum Überprüfen des Zugriffstokens müssen Sie `TaskService` Informationen zur App bereitstellen. Öffnen Sie im `TaskService`-Projekt die Datei `web.config` im Stammverzeichnis des Projekts, und ersetzen Sie die Werte im Abschnitt `<appSettings>`:

```
<appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="ida:AadInstance" value="https://login.microsoftonline.com/{0}/{1}/{2}?p={3}" />
    <add key="ida:Tenant" value="{Enter the name of your B2C tenant - it usually looks like constoso.onmicrosoft.com}" />
    <add key="ida:ClientId" value="{Enter the Application ID assigned to your app by the Azure Portal}" />
    <add key="ida:PolicyId" value="{Enter the name of one of the policies you created, like `b2c_1_my_sign_in_policy`}" />
</appSettings>
```

[AZURE.INCLUDE [active-directory-b2c-devquickstarts-tenant-name](../../includes/active-directory-b2c-devquickstarts-tenant-name.md)]


Dieser Artikel behandelt nicht die Details der Sicherung von `TaskService`. Um zu erfahren, wie eine Web-API sicher Anforderungen mithilfe von Azure AD B2C authentifiziert, sehen Sie sich unseren [Artikel mit den ersten Schritten für die Web-API](active-directory-b2c-devquickstarts-api-dotnet.md) an.

## Konfigurieren der Aufgaben-Web-App

Damit `TaskWebApp` mit Azure AD B2C kommunizieren kann, müssen Sie einige allgemeine Parameter angeben. Öffnen Sie im `TaskWebApp`-Projekt die Datei `web.config` im Stammverzeichnis des Projekts, und ersetzen Sie die Werte im Abschnitt `<appSettings>`. Diese Werte werden in der gesamten Web-App verwendet.

```
<appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="ida:Tenant" value="{Enter the name of your B2C directory, e.g. contoso.onmicrosoft.com}" />
    <add key="ida:ClientId" value="{Enter the Application Id assigned to your app by the Azure portal, e.g.580e250c-8f26-49d0-bee8-1c078add1609}" />
    <add key="ida:ClientSecret" value="{Enter the Application Secret you created in the Azure portal, e.g. yGNYWwypRS4Sj1oYXd0443n}" />
    <add key="ida:AadInstance" value="https://login.microsoftonline.com/{0}{1}{2}" />
    <add key="ida:RedirectUri" value="https://localhost:44316/" />
    <add key="ida:SignUpPolicyId" value="[Enter your sign up policy name, e.g.g b2c_1_sign_up" />
    <add key="ida:SignInPolicyId" value="[Enter your sign in policy name, e.g. b2c_1_sign_in]" />
    <add key="ida:UserProfilePolicyId" value="[Enter your edit profile policy name, e.g. b2c_1_profile_edit" />
    <add key="api:TaskServiceUrl" value="https://localhost:44332/" />
</appSettings>
```

[AZURE.INCLUDE [active-directory-b2c-devquickstarts-tenant-name](../../includes/active-directory-b2c-devquickstarts-tenant-name.md)]

Es gibt auch zwei `[PolicyAuthorize]`-Decorator-Elemente, für die Sie den Namen der Anmelderichtlinie angeben müssen. Das `[PolicyAuthorize]`-Attribut wird verwendet, um eine bestimmte Richtlinie aufzurufen, wenn ein Benutzer versucht, auf eine Seite in der App zuzugreifen, für die eine Authentifizierung erforderlich ist.

```C#
// Controllers\HomeController.cs

[PolicyAuthorize(Policy = "{Enter the name of your sign in policy, e.g. b2c_1_my_sign_in}")]
public ActionResult Claims()
{
```

```C#
// Controllers\TasksController.cs

[PolicyAuthorize(Policy = "{Enter the name of your sign in policy, e.g. b2c_1_my_sign_in}")]
public class TasksController : Controller
{
```

## Abrufen von Zugriffstoken und Aufrufen der Aufgaben-API

In diesem Abschnitt wird erläutert, wie Sie einen OAuth 2.0-Tokenaustausch in einer Web-App mit den Bibliotheken und Frameworks von Microsoft durchführen. Falls Sie mit Autorisierungscodes und Zugriffstoken nicht vertraut sind, finden Sie in der [Referenz zum OpenID Connect-Protokoll](active-directory-b2c-reference-protocols.md) weitere Informationen.

### Abrufen eines Autorisierungscodes

Der erste Schritt beim Aufrufen der `TaskService`-Web-API ist das Authentifizieren des Benutzers und das Empfangen eines Autorisierungscodes von Azure AD. Sie können einen Autorisierungscode von Azure AD erhalten, nachdem alle Richtlinien erfolgreich ausgeführt wurden. Dazu gehören Anmelde-, Registrierungs- und Profilbearbeitungsrichtlinien.

Installieren Sie zuerst die OWIN OpenID Connect-Middleware mithilfe der Paket-Manager-Konsole von Visual Studio. Verwenden Sie OWIN, um Authentifizierungsanforderungen an Azure AD zu senden und die Antworten zu verarbeiten:

```
PM> Install-Package Microsoft.Owin.Security.OpenIdConnect -ProjectName TaskWebApp
PM> Install-Package Microsoft.Owin.Security.Cookies -ProjectName TaskWebApp
PM> Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName TaskWebApp
```

Öffnen Sie die Datei `App_Start\Startup.Auth.cs`. Hier konfigurieren Sie die OWIN-Authentifizierungspipeline und geben Details zu Ihrem B2C-Verzeichnis und zur Anwendung an, die Sie erstellt haben:

```C#
// App_Start\Startup.Auth.cs

public partial class Startup
{
	public const string AcrClaimType = "http://schemas.microsoft.com/claims/authnclassreference";
	public const string PolicyKey = "b2cpolicy";
	public const string OIDCMetadataSuffix = "/.well-known/openid-configuration";

	// App config settings
	public static string clientId = ConfigurationManager.AppSettings["ida:ClientId"];
	public static string clientSecret = ConfigurationManager.AppSettings["ida:ClientSecret"];
	public static string aadInstance = ConfigurationManager.AppSettings["ida:AadInstance"];
	public static string tenant = ConfigurationManager.AppSettings["ida:Tenant"];
	public static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];

	// B2C policy identifiers
	public static string SignUpPolicyId = ConfigurationManager.AppSettings["ida:SignUpPolicyId"];
	public static string SignInPolicyId = ConfigurationManager.AppSettings["ida:SignInPolicyId"];
	public static string ProfilePolicyId = ConfigurationManager.AppSettings["ida:UserProfilePolicyId"];

	public void ConfigureAuth(IAppBuilder app)
	{
		app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

		app.UseCookieAuthentication(new CookieAuthenticationOptions());

		OpenIdConnectAuthenticationOptions options = new OpenIdConnectAuthenticationOptions
		{
			// These are standard OpenID Connect parameters, with values pulled from web.config
			ClientId = clientId,
			RedirectUri = redirectUri,
			PostLogoutRedirectUri = redirectUri,
			Notifications = new OpenIdConnectAuthenticationNotifications
			{
				AuthenticationFailed = OnAuthenticationFailed,
				RedirectToIdentityProvider = OnRedirectToIdentityProvider,
				AuthorizationCodeReceived = OnAuthorizationCodeReceived,
			},
			Scope = "openid offline_access",

			// The PolicyConfigurationManager takes care of getting the correct Azure AD authentication
			// endpoints from the OpenID Connect metadata endpoint. It is included in the PolicyAuthHelpers folder.
			ConfigurationManager = new PolicyConfigurationManager(
				String.Format(CultureInfo.InvariantCulture, aadInstance, tenant, "/v2.0", OIDCMetadataSuffix),
				new string[] { SignUpPolicyId, SignInPolicyId, ProfilePolicyId }),

			// This piece is optional - it is used for displaying the user's name in the navigation bar.
			TokenValidationParameters = new System.IdentityModel.Tokens.TokenValidationParameters
			{
				NameClaimType = "name",
			},
		};

		app.UseOpenIdConnectAuthentication(options);
	}
	...
}
```

### Abrufen eines Zugriffstokens mithilfe des Autorisierungscodes

Die Web-App ist jetzt zum Authentifizieren des Benutzers mit Ihrem B2C-Verzeichnis und zum Empfangen eines Autorisierungscodes von Azure AD konfiguriert. Der nächste Schritt besteht darin, Autorisierungscodes durch Zugriffstoken von Azure AD auszutauschen.

Wenn Ihre .NET-Web-Apps Zugriffstoken von Azure AD abrufen müssen, können Sie die Active Directory Authentication Library (ADAL) verwenden. Sie müssen ADAL nicht für diesen Prozess verwenden, aber ADAL übernimmt viele Details und erleichtert Ihnen so den Prozess. Dazu gehören das Senden von OAuth 2.0-Authentifizierungsnachrichten und das Zwischenspeichern und Aktualisieren von Token.

Installieren Sie zunächst mit der Paket-Manager-Konsole ADAL im `TaskWebApp`-Projekt:

```
PM> Install-Package Microsoft.Experimental.IdentityModel.Clients.ActiveDirectory -ProjectName TaskWebApp -IncludePrerelease
```

Dann müssen Sie den Autorisierungscode an ADAL übergeben, damit Token für Sie abgerufen werden können. Die OWIN OpenID Connect-Middleware bietet eine Benachrichtigung zur Verwendung dieses Autorisierungscodes. Die Benachrichtigung wird jedes Mal gesendet, wenn Ihre App einen Autorisierungscode von Azure AD erhält. Implementieren Sie in `App_Start\Startup.Auth.cs` den Benachrichtigungshandler `OnAuthorizationCodeReceived` mithilfe von ADAL:

```C#
// App_Start\Startup.Auth.cs

private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification notification)
{
	// The user's objectId is extracted from the claims provided in the id_token, and used to cache tokens in ADAL
	// The authority is constructed by appending your B2C directory's name to "https://login.microsoftonline.com/"
	// The client credential is where you provide your application secret, and it is used to authenticate the application to Azure AD
	string userObjectID = notification.AuthenticationTicket.Identity.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
	string authority = String.Format(CultureInfo.InvariantCulture, aadInstance, tenant, string.Empty, string.Empty);
	ClientCredential credential = new ClientCredential(clientId, clientSecret);

	// We don't care which policy is used to access the TaskService, so let's use the most recent policy as indicated in the sign-in token
	string mostRecentPolicy = notification.AuthenticationTicket.Identity.FindFirst(Startup.AcrClaimType).Value;

	// The Authentication Context is ADAL's primary class, which represents your connection to your B2C directory
	// ADAL uses an in-memory token cache by default. In this case, we've extended the default cache to use a simple per-user session cache
	AuthenticationContext authContext = new AuthenticationContext(authority, new NaiveSessionCache(userObjectID));

	// Here you ask for a token by using the web app's clientId as the scope, because the web app and service share the same clientId.
	// The token will be stored in the ADAL token cache for use in our controllers
	AuthenticationResult result = await authContext.AcquireTokenByAuthorizationCodeAsync(notification.Code, new Uri(redirectUri), credential, new string[] { clientId }, mostRecentPolicy);
}
```

### Abrufen eines Zugriffstoken in den Controllern

Nachdem Sie ein Zugriffstoken für das `TaskService`-Back-End erhalten und im ADAL-Tokencache gespeichert haben, müssen Sie es verwenden. `TasksController` ist verantwortlich für die Kommunikation mit der `TaskService`-API und sendet die HTTP-Anforderungen an die API, um Aufgaben zu lesen, zu erstellen und zu löschen. Bevor eine HTTP-Anforderung gesendet wird, rufen Sie ein Token von ADAL ab:

```C#
// Controllers\TasksController.cs

public async Task<ActionResult> Index()
{
	AuthenticationResult result = null;
	try
	{
		string userObjectID = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
		string authority = String.Format(CultureInfo.InvariantCulture, Startup.aadInstance, Startup.tenant, string.Empty, string.Empty);
		ClientCredential credential = new ClientCredential(Startup.clientId, Startup.clientSecret);

		// We don't care which policy is used to access the TaskService, so let's use the most recent policy
		string mostRecentPolicy = ClaimsPrincipal.Current.FindFirst(Startup.AcrClaimType).Value;

		// Here you ask for a token by using the web app's clientId as the scope, because the web app and service share the same clientId.
		// AcquireTokenSilentAsync will return a token from the token cache and throw an exception if it cannot do so.
		AuthenticationContext authContext = new AuthenticationContext(authority, new NaiveSessionCache(userObjectID));
		result = await authContext.AcquireTokenSilentAsync(new string[] { Startup.clientId }, credential, UserIdentifier.AnyUser, mostRecentPolicy);

		...
	}
	catch (AdalException ee)
	{
		// If ADAL could not get a token silently, show the user an error indicating they might need to sign in again.
		return new RedirectResult("/Error?message=An Error Occurred Reading To Do List: " + ee.Message + " You might need to log out and log back in.");
	}
	...
}
```

ADAL übernimmt das Zwischenspeichern von Token, aktualisiert sie, wenn sie ablaufen, und informiert Sie, wenn sich der Benutzer erneut anmelden muss, indem Ausnahmen ausgelöst werden. Sie müssen lediglich `AuthenticationContext.AcquireTokenSilentAsync(...)` jedes Mal aufrufen, wenn Sie in Ihrer App ein Token benötigen.

### Lesen von Aufgaben aus der Web-API

Wenn Sie über ein Token verfügen, können Sie es an die HTTP-`GET`-Anforderung im `Authorization`-Header anfügen, um `TaskService` sicher aufzurufen:

```C#
// Controllers\TasksController.cs

public async Task<ActionResult> Index()
{
	...

	try
	{
		HttpClient client = new HttpClient();
		HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, serviceUrl + "/api/tasks");

		// Add the token acquired from ADAL to the request headers
		request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", result.Token);
		HttpResponseMessage response = await client.SendAsync(request);

		if (response.IsSuccessStatusCode)
		{
			String responseString = await response.Content.ReadAsStringAsync();
			JArray tasks = JArray.Parse(responseString);
			ViewBag.Tasks = tasks;
			return View();
		}
		else
		{
			// If the call failed with access denied, then drop the current access token from the cache,
			// and show the user an error that indicates that they might need to sign in again.
			if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
			{
				var todoTokens = authContext.TokenCache.ReadItems().Where(a => a.Scope.Contains(Startup.clientId));
				foreach (TokenCacheItem tci in todoTokens)
					authContext.TokenCache.DeleteItem(tci);

				return new RedirectResult("/Error?message=Error: " + response.ReasonPhrase + " You might need to sign in again.");
			}
		}

		return new RedirectResult("/Error?message=An Error Occurred Reading To Do List: " + response.StatusCode);
	}
	catch (Exception ex)
	{
		return new RedirectResult("/Error?message=An Error Occurred Reading To Do List: " + ex.Message);
	}
}

```

### Erstellen und Löschen von Aufgaben mit der Web-API

Folgen Sie beim Senden von `POST`- und `DELETE`-Anforderungen an `TaskService` dem gleichen Muster. Rufen Sie `AuthenticationContext.AcquireTokenSilentAsync(...)` auf, und fügen Sie das resultierende Token an die Anforderung im `Authorization`-Header an. Wir haben diese Aktion für Sie erstellt. Sie können versuchen, die Löschaktion in `TasksController.cs` zu beenden.

## Abmelden des Benutzers

Wenn sich ein Benutzer von der Web-App abmeldet, sollten Sie den ADAL-Tokencache löschen, um alle Reste der authentifizierten Sitzung des Benutzers zu entfernen:

```C#
// Controllers/AccountController.cs

public void SignOut()
{
	if (Request.IsAuthenticated)
	{
		// When the user signs out, clear their token cache in the process
		string userObjectID = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
		string authority = String.Format(CultureInfo.InvariantCulture, Startup.aadInstance, Startup.tenant, string.Empty, string.Empty);
		AuthenticationContext authContext = new AuthenticationContext(authority, new NaiveSessionCache(userObjectID));
		authContext.TokenCache.Clear();

		HttpContext.GetOwinContext().Authentication.SignOut(
		new AuthenticationProperties(
			new Dictionary<string, string>
			{
				{Startup.PolicyKey, ClaimsPrincipal.Current.FindFirst(Startup.AcrClaimType).Value}
			}), OpenIdConnectAuthenticationDefaults.AuthenticationType, CookieAuthenticationDefaults.AuthenticationType);
	}
}
```

## Ausführen der Beispiel-App

Zum Schluss erstellen Sie `TaskClient` und `TaskService` und führen sie aus. Registrieren Sie sich bei der App, und melden Sie sich an. Erstellen Sie Aufgaben für den angemeldeten Benutzer. Melden Sie sich ab, und melden Sie sich als ein anderer Benutzer an. Erstellen Sie Aufgaben für diesen Benutzer. Beachten Sie, wie die Aufgaben pro Benutzer in der API gespeichert werden, da die API die Benutzeridentität aus dem empfangenen Zugriffstoken extrahiert.

Als Referenz wird das fertige Beispiel [als ZIP-Datei bereitgestellt](https://github.com/AzureADQuickStarts/B2C-WebApp-WebAPI-OpenIDConnect-DotNet/archive/complete.zip). Sie können sie auch aus GitHub klonen:

```git clone --branch complete https://github.com/AzureADQuickStarts/B2C-WebApp-WebAPI-OpenIDConnect-DotNet.git```

<!--

## Next steps

You can now move on to more advanced B2C topics. You might try:

[Call a web API from a web app]()

[Customize the UX for a B2C app]()

-->

<!---HONumber=AcomDC_0525_2016-->