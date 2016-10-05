<properties
   pageTitle="Azure Active Directory-Codebeispiele | Microsoft Azure"
   description="Ein Index der Azure Active Directory-Codebeispiele, organisiert nach Szenario."
   services="active-directory"
   documentationCenter="dev-center-name"
   authors="msmbaldwin"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/16/2016"
   ms.author="mbaldwin"/>

# Azure Active Directory-Codebeispiele

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

Sie können Microsoft Azure Active Directory (Azure AD) verwenden, um Ihren Webanwendungen und Web-APIs Authentifizierung und Autorisierung hinzuzufügen. Dieser Abschnitt enthält Links zu Codebeispielen, die die Vorgehensweise erläutern, und zu Codeausschnitten, die Sie in Ihren Anwendungen verwenden können. Auf der Codebeispielseite finden Sie ausführliche Infothemen mit hilfreichen Angaben zu Anforderungen, Installation und Einrichtung. Außerdem ist der Code kommentiert, um die wichtigsten Abschnitte besser verständlich zu machen.

Informationen zum grundlegenden Szenario für jeden Beispieltyp finden Sie unter "Authentifizierungsszenarien für Azure AD".

Schreiben Sie Beiträge zu unseren Beispielen auf GitHub: [Microsoft Azure Active Directory – Beispiele und Dokumentation](https://github.com/Azure-Samples?page=3&query=active-directory).

## Webbrowser zu Webanwendung

Diese Beispiele zeigen, wie Sie eine Webanwendung schreiben, die den Browser des Benutzers zur Anmeldung bei Azure AD leitet.



| Sprache/Plattform | Beispiel | Beschreibung
| ----------------- | ------ | -----------
| C#/.NET | [WebApp-OpenIDConnect-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect) | Authentifiziert Benutzer eines Azure AD-Mandanten mithilfe von OpenID Connect (ASP.Net OpenID Connect OWIN-Middleware).
| C#/.NET | [WebApp-MultiTenant-OpenIdConnect-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-multitenant-openidconnect) | Eine mehrinstanzenfähige .NET MVC-Webanwendung zur Authentifizierung von Benutzern mehrerer Azure AD-Mandanten mithilfe von OpenID Connect (ASP.Net OpenID Connect OWIN-Middleware).
| C#/.NET | [WebApp-WSFederation-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation) | Authentifiziert Benutzer eines Azure AD-Mandanten mithilfe von WS-Federation (ASP.Net WS-Federation OWIN-Middleware).






## Single-Page-Anwendung (SPA)

Dieses Beispiel zeigt, wie Sie eine Single-Page-Anwendung schreiben, die mit Azure AD gesichert ist.

| Sprache/Plattform | Beispiel | Beschreibung
| ----------------- | ------ | -----------
| JavaScript, C#/.NET | [SinglePageApp-DotNet](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp) | Sichert eine AngularJS-basierte, mit einem ASP.NET Web-API-Back-End implementierte Single-Page-Anwendung mithilfe von ADAL für JavaScript und Azure AD.


## Systemeigene Anwendung zu Web-API

Diese Codebeispiele zeigen, wie Sie systemeigene Clientanwendungen erstellen, die von Azure AD gesicherte Web-APIs aufrufen. Dabei kommen [Azure AD Authentication Library (ADAL)](active-directory-authentication-libraries.md) und [OAuth 2.0 in Azure AD](https://msdn.microsoft.com/library/azure/dn645545.aspx) zum Einsatz.

| Sprache/Plattform | Beispiel | Beschreibung
| ----------------- | ------ | -----------
| JavaScript | [NativeClient-MultiTarget-Cordova](https://github.com/Azure-Samples/active-directory-cordova-multitarget) | Erstellt mithilfe des ADAL-Plug-Ins für Apache Cordova eine Apache Cordova-Anwendung, die eine Web-API aufruft und Azure AD für die Authentifizierung verwendet.
| C#/.NET | [NativeClient-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-native-desktop) | Eine .NET WPF-Anwendung, die eine mit Azure AD gesicherte Web-API aufruft.
| C#/.NET | [NativeClient-WindowsStore](https://github.com/Azure-Samples/active-directory-dotnet-windows-store) | Eine Windows Store-Anwendung, die eine mit Azure AD gesicherte Web-API aufruft.
| C#/.NET | [NativeClient-WebAPI-MultiTenant-WindowsStore](https://github.com/Azure-Samples/active-directory-dotnet-webapi-multitenant-windows-store) | Eine Windows Store-Anwendung, die eine mit Azure AD gesicherte, mehrinstanzenfähige Web-API aufruft.
| C#/.NET | [WebAPI-OnBehalfOf-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof) | Eine systemeigene Clientanwendung, die eine Web-API zum Abrufen eines Tokens aufruft, um im Namen des ursprünglichen Benutzers zu agieren, und dann mit dem Token eine andere Web-API aufruft.
| C#/.NET | [NativeClient-WindowsPhone8.1](https://github.com/Azure-Samples/active-directory-dotnet-windowsphone-8.1) | Eine Windows Store-Anwendung für Windows Phone 8.1, die eine mit Azure AD gesicherte Web-API aufruft.
| ObjC | [NativeClient-iOS](https://github.com/Azure-Samples/active-directory-ios) | Eine iOS-Anwendung, die eine Web-API aufruft, bei der Azure AD zur Authentifizierung erforderlich ist.
| C#/.NET | [WebAPI-ManuallyValidateJwt-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapi-manual-jwt-validation) | Eine systemeigene Clientanwendung, die Logik zur Verarbeitung eines JWT-Tokens in einer Web-API beinhaltet, anstatt OWIN-Middleware zur verwenden.
| C#/Xamarin | [NativeClient-Xamarin-Android](https://github.com/Azure-Samples/active-directory-xamarin-android) | Eine Xamarin-Bindung an die systemeigene Azure AD Authentication Library (ADAL) für die Android-Bibliothek.
| C#/Xamarin | [NativeClient-Xamarin-iOS](https://github.com/Azure-Samples/active-directory-xamarin-ios) | Eine Xamarin-Bindung an die systemeigene Azure AD Authentication Library (ADAL) für iOS.
| C#/Xamarin | [NativeClient-MultiTarget-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-native-multitarget) | Ein Xamarin-Projekt für fünf Zielplattformen, das eine von Azure AD gesicherte Web-API aufruft.
| C#/.NET | [NativeClient-Headless-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-native-headless) | Eine systemeigene Anwendung, die eine nicht interaktive Authentifizierung durchführt und eine von Azure AD gesicherte Web-API aufruft.



## Webanwendung zu Web-API

Diese Codebeispiele zeigen die Verwendung von [OAuth 2.0 in Azure AD](https://msdn.microsoft.com/library/azure/dn645545.aspx) zum Erstellen von Webanwendungen, die von Azure AD gesicherte Web-APIs aufrufen.

| Sprache/Plattform | Beispiel | Beschreibung
| ----------------- | ------ | -----------
| C#/.NET | [WebApp-WebAPI-OpenIDConnect-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-openidconnect) | Ruft eine Web-API mit den Berechtigungen des angemeldeten Benutzers auf.
| C#/.NET | [WebApp-WebAPI-OAuth2-AppIdentity-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-appidentity) | Ruft eine Web-API mit den Berechtigungen der Anwendung auf.
| C#/.NET | [WebApp-WebAPI-OAuth2-UserIdentity-Dotnet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-useridentity) | Fügt einer vorhandenen Webanwendung die Autorisierung mit [OAuth 2.0 in Azure AD](https://msdn.microsoft.com/library/azure/dn645545.aspx) hinzu, damit die Webanwendung eine Web-API aufrufen kann.
| JavaScript | [WebAPI-Nodejs](https://github.com/Azure-Samples/active-directory-node-webapi) | Richtet einen REST-API-Dienst mit Azure AD-Integration zum Schutz der API ein. Enthält einen Node.js-Server mit einer Web-API.
| C#/.NET | [WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-multitenant-openidconnect) | Eine mehrinstanzenfähige MVC-Webanwendung zur Authentifizierung von Benutzern eines Azure AD-Mandanten mithilfe von OpenID Connect (ASP.Net OpenID Connect OWIN-Middleware). Verwendet einen Autorisierungscode zum Aufrufen der Graph-API.

## Server- oder Daemonanwendung zu Web-API

Diese Codebeispiele zeigen die Erstellung einer Daemon- oder Serveranwendung, die Ressourcen unter Verwendung von [Azure AD Authentication Library (ADAL)](active-directory-authentication-libraries.md) und [OAuth 2.0 in Azure AD](https://msdn.microsoft.com/library/azure/dn645545.aspx) von einer Web-API abruft.

| Sprache/Plattform | Beispiel | Beschreibung
| ----------------- | ------ | -----------
| C#/.NET | [Daemon-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-daemon) | Eine Konsolenanwendung, die eine Web-API aufruft. Die Clientanmeldeinformation ist ein Kennwort.
| C#/.NET | [Daemon-CertificateCredential-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential) | Eine Konsolenanwendung, die eine Web-API aufruft. Die Clientanmeldeinformation ist ein Zertifikat.


## Aufrufen der Azure AD Graph-API

Dieses Codebeispiel zeigt, wie Sie Anwendungen erstellen, die die Azure AD Graph-API zum Lesen und Schreiben von Verzeichnisdaten aufrufen.

| Sprache/Plattform | Beispiel | Beschreibung
| ----------------- | ------ | -----------
| Java | [WebApp-GraphAPI-Java](https://github.com/Azure-Samples/active-directory-java-graphapi-web) | Eine Webanwendung, die mithilfe der Graph-API auf Azure AD-Verzeichnisdaten zugreift.
| PHP | [WebApp-GraphAPI-PHP](https://github.com/Azure-Samples/active-directory-php-graphapi-web) | Eine Webanwendung, die mithilfe der Graph-API auf Azure AD-Verzeichnisdaten zugreift.
| C#/.NET | [WebApp-GraphAPI-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-web) | Eine Webanwendung, die mithilfe der Graph-API auf Azure AD-Verzeichnisdaten zugreift.
| C#/.NET | [ConsoleApp-GraphAPI-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-console) | Diese Konsolen-App veranschaulicht allgemeine Lese- und Schreibaufrufe an die Graph-API, die Ausführung der Benutzerlizenzzuweisung sowie die Aktualisierung des Miniaturfotos und der Links eines Benutzers.
| C#/.NET | [ConsoleApp-GraphAPI-DiffQuery-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-diffquery) | Eine Konsolenanwendung, die die differenzielle Abfrage in der Graph-API verwendet, um periodische Änderungen an Benutzerobjekten eines Azure AD-Mandanten abzurufen.
| C#/.NET | [WebApp-GraphAPI-DirectoryExtensions-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-directoryextensions-web) | Eine MVC-Anwendung, die auf der Grundlage von Graph-API-Abfragen ein einfaches Unternehmensorganigramm zu generiert.
| PHP | [WebApp-GraphAPI-DirectoryExtensions-PHP](https://github.com/Azure-Samples/active-directory-php-graphapi-directoryextensions-web) | Eine PHP-Anwendung, die die Graph-API aufruft, um eine Erweiterung zu registrieren und dann Werte im Erweiterungsattribut zu lesen, zu aktualisieren und zu löschen.


## Autorisierung

Diese Codebeispiele zeigen, wie Azure AD für die Autorisierung verwendet wird.

| Sprache/Plattform | Beispiel | Beschreibung
| ----------------- | ------ | -----------
| C#/.NET | [WebApp-GroupClaims-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-groupclaims) | Führt die rollenbasierte Zugriffssteuerung (Role Based Access Control, RBAC) unter Verwendung von Azure Active Directory-Gruppenansprüchen in einer Anwendung durch, die in Azure AD integriert ist.
| C#/.NET | [WebApp-RoleClaims-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims) | Führt die rollenbasierte Zugriffssteuerung (Role Based Access Control, RBAC) unter Verwendung von Azure Active Directory-Anwendungsrollen in einer Anwendung durch, die in Azure AD integriert ist.


## Ältere exemplarische Vorgehensweisen

In diesen exemplarischen Vorgehensweisen wird zwar eine etwas ältere Technologie verwendet, sie können aber trotzdem von Interesse sein.

| Sprache/Plattform | Beispiel | Beschreibung
| ----------------- | ------ | -----------
| C#/.NET | [Rollen- und ACL-basierte Autorisierung in einer Microsoft Azure AD-Anwendung](http://go.microsoft.com/fwlink/?LinkId=331694) | Führt die rollen- und ACL-basierte Autorisierung in einer Anwendung durch, die in Azure AD integriert ist.
| C#/.NET | [AAL – Windows Store-App zu REST-Dienst – Authentifizierung](http://go.microsoft.com/fwlink/?LinkId=330605) | Verwendet [Azure AD Authentication Library (ADAL)](active-directory-authentication-libraries.md) (ehemals AAL) für Windows Store Beta, um einer Windows Store-App Funktionen für die Benutzerauthentifizierung hinzuzufügen.
| C#/.NET | [ADAL – systemeigene App zu REST-Dienst – Authentifizierung mit AAD mittels Browserdialogfeld](http://go.microsoft.com/fwlink/?LinkId=259814) | Verwendet [Azure AD Authentication Library (ADAL)](active-directory-authentication-libraries.md), um einem WPF-Client Funktionen für die Benutzerauthentifizierung hinzuzufügen.
| C#/.NET | [ADAL – systemeigene App zu REST-Dienst – Authentifizierung mit ACS mittels Browserdialogfeld](http://code.msdn.microsoft.com/AAL-Native-App-to-REST-de57f2cc) | Verwendet [Azure AD Authentication Library (ADAL)](active-directory-authentication-libraries.md) und [Access Control Service 2.0 (ACS)](http://msdn.microsoft.com/library/azure/hh147631.aspx), um einem WPF-Client Funktionen für die Benutzerauthentifizierung hinzuzufügen.
| C#/.NET | [ADAL – Server-zu-Server-Authentifizierung](http://go.microsoft.com/fwlink/?LinkId=259816) | Verwendet [Azure AD Authentication Library (ADAL)](active-directory-authentication-libraries.md) zum Sichern von Dienstaufrufen von einem serverseitigen Prozess an einen MVC4 Web-API-REST-Dienst.
| C#/.NET | [Hinzufügen einer Anmeldung zu einer Webanwendung mithilfe von Azure AD](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect) | Konfiguriert eine .NET-Anwendung so, dass eine einmalige Webanmeldung beim Azure AD-Unternehmensverzeichnis durchgeführt wird.
| C#/.NET | [Entwickeln mehrinstanzenfähiger Webanwendungen mit Azure AD](https://github.com/Azure-Samples/active-directory-dotnet-webapp-multitenant-openidconnect) | Verwendet Azure AD, um die Funktionen für einmaliges Anmelden und Verzeichniszugriff einer .NET-Anwendung auf mehrere Organisationen auszuweiten.
JAVA | [Java-Beispielanwendung für die Graph-API von Azure AD](http://go.microsoft.com/fwlink/?LinkId=263969) | Verwendet die Graph-API für den Zugriff auf Verzeichnisdaten aus Azure AD.
PHP | [PHP-Beispielanwendung für die Graph-API von Azure AD](http://code.msdn.microsoft.com/PHP-Sample-App-For-Windows-228c6ddb) | Verwendet die Graph-API für den Zugriff auf Verzeichnisdaten aus Azure AD.
| C#/.NET | [Beispiel-App für die Graph-API von Azure AD](http://go.microsoft.com/fwlink/?LinkID=262648) | Verwendet die Graph-API für den Zugriff auf Verzeichnisdaten aus Azure AD.
| C#/.NET | [Beispiel-App für differenzielle Abfragen der Graph-API von Azure AD](http://go.microsoft.com/fwlink/?LinkId=275398) | Verwendet die differenzielle Abfrage der Graph-API, um periodische Änderungen an Benutzerobjekten eines Azure AD-Mandanten abzurufen.
| C#/.NET | [Beispiel-App für die Integration einer mehrinstanzenfähigen Cloud-Anwendung für Azure AD](http://go.microsoft.com/fwlink/?LinkId=275397) | Integriert Eine mehrinstanzenfähige Anwendung in Azure AD.
| C#/.NET | [Sichern einer Windows Store-Anwendung und des REST-Webdiensts mithilfe von Azure AD](https://github.com/Azure-Samples/active-directory-dotnet-windows-store) | Erstellt eine einfache Web-API-Ressource und eine Windows Store-Clientanwendung mithilfe von Azure AD und [Azure AD Authentication Library (ADAL)](active-directory-authentication-libraries.md).
| C#/.NET| [Abfragen von Azure AD mithilfe der Graph-API](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-web) | Konfiguriert eine Microsoft .NET-Anwendung zur Verwendung der Graph-API von Azure AD für den Zugriff auf Daten aus einem Azure AD-Mandantenverzeichnis.

## Siehe auch

##### Weitere Ressourcen

[Entwicklerhandbuch zu Azure Active Directory](active-directory-developers-guide.md)

[Azure AD-Graph-API – Konzepte und Referenz](https://msdn.microsoft.com/library/azure/hh974476.aspx)

[Bibliothek für das Azure AD Graph-API-Hilfsprogramm](https://www.nuget.org/packages/Microsoft.Azure.ActiveDirectory.GraphClient)

<!---HONumber=AcomDC_0921_2016-->