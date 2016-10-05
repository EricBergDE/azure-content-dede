<properties
	pageTitle="Azure AD v2.0: Bereiche, Berechtigungen und Zustimmung | Microsoft Azure"
	description="Beschreibung der Autorisierung im Azure AD v2.0-Endpunkt, einschließlich Bereichen, Berechtigungen und Zustimmung."
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>

# Bereiche, Berechtigungen und Zustimmung im v2.0-Endpunkt

Apps, die sich in Azure AD integrieren lassen, folgen einem bestimmten Autorisierungsmodell, mit dem Benutzer festlegen können, wie eine App auf ihre Daten zugreift. Die v2.0-Implementierung dieses Autorisierungsmodells wurde aktualisiert, sodass die Art und Weise, wie eine App mit Azure AD interagieren muss, nun anders ist. In diesem Thema werden die grundlegenden Konzepte dieses Autorisierungsmodells einschließlich der Bereiche, Berechtigungen und Zustimmung behandelt.

> [AZURE.NOTE]
	Nicht alle Szenarien und Funktionen von Azure Active Directory werden vom v2.0-Endpunkt unterstützt. Lesen Sie die Informationen zu den [Einschränkungen des v2.0-Endpunkts](active-directory-v2-limitations.md), um herauszufinden, ob Sie den v2.0-Endpunkt verwenden sollten.

## Bereiche und Berechtigungen

Azure AD implementiert das [OAuth 2.0](active-directory-v2-protocols.md)-Autorisierungsprotokoll – eine Methode, mit der eine Drittanbieter-App im Auftrag eines Benutzers auf im Web gehostete Ressourcen zugreifen kann. Alle im Web gehosteten Ressourcen, die in Azure AD integriert werden können, verfügen über eine Ressourcen-ID oder einen **App-ID-URI**. Zu den von Microsoft im Web gehosteten Ressourcen zählen beispielsweise:

- die Office 365 Unified Mail-API: `https://outlook.office.com`
- die Azure AD Graph-API: `https://graph.windows.net`
- Microsoft Graph: `https://graph.microsoft.com`

Dasselbe gilt für alle Ressourcen von Drittanbietern, die in Azure AD integriert wurden. Diese Ressourcen können auch einen Satz von Berechtigungen definieren, die zum Unterteilen der Funktionalität der Ressource in kleinere Einheiten verwendet werden können. Beispielsweise sind für Microsoft Graph folgende Berechtigungen definiert:

- Lesen des Kalenders eines Benutzers
- Schreiben in den Kalender eines Benutzers
- Senden von E-Mails als Benutzer
- [und mehr](https://graph.microsoft.io)

Durch das Definieren dieser Berechtigungen erhält die Ressource eine präzisere Kontrolle über die Daten und die Art und Weise, wie sie der Außenwelt verfügbar gemacht werden. Eine Drittanbieter-App kann diese Berechtigungen dann von einem Endbenutzer anfordern, und der Endbenutzer muss die Berechtigungen genehmigen, bevor die App in seinem Auftrag fungieren kann. Durch Segmentieren der Ressourcenfunktionalität in kleinere Berechtigungssätze können Drittanbieter-Apps so erstellt werden, dass nur die spezifischen Berechtigungen angefordert werden, die diese Apps zum Durchführen ihrer Aufgabe benötigen. Dadurch können Endbenutzer auch genau erkennen, wie eine App ihre Daten verwendet, sodass sie sicher sein können, dass sich die App ohne bösartige Absichten verhält.

In Azure AD und OAuth sind diese Berechtigungen als **Bereiche** bekannt. Manchmal werden sie auch als **OAuth2-Berechtigungen** bezeichnet. Ein Bereich wird in Azure AD als Zeichenfolgenwert dargestellt. Um am Beispiel von Microsoft Graph festzuhalten, lautet der Bereichswert für jede Berechtigung wie folgt:

- Lesen des Kalenders eines Benutzers: `Calendar.Read`
- Schreiben in den Kalender eines Benutzers: `Mail.ReadWrite`
- Senden von E-Mails als Benutzer: `Mail.Send`

Eine Anwendung kann diese Berechtigungen wie unten beschrieben durch Angabe der Bereiche in den Anforderungen an den v2.0-Endpunkt anfordern.


## Zustimmung

In einer Autorisierungsanforderung von [OpenID Connect oder OAuth 2.0](active-directory-v2-protocols.md) kann eine App die erforderlichen Berechtigungen mithilfe des `scope`-Abfrageparameters anfordern. Wenn sich ein Benutzer beispielsweise in einer App anmeldet, sendet die App eine Anforderung wie die folgende (mit Zeilenumbrüchen zur besseren Lesbarkeit):

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendar.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

Der `scope`-Parameter ist eine durch Leerzeichen getrennte Liste von Bereichen, die von der App angefordert wird. Jeder einzelne Bereich wird durch Anhängen des Bereichswerts an den Ressourcenbezeichner (App-ID-URI) angegeben. Die Anforderung oben gibt an, dass die Anwendung eine Berechtigung zum Lesen des Kalenders des Benutzers und Senden von E-Mails als Benutzer benötigt.

Nachdem der Benutzer seine Anmeldeinformationen eingegeben hat, überprüft der v2.0-Endpunkt, ob es einen übereinstimmenden Datensatz der **Benutzerzustimmung** gibt. Wenn der Benutzer den angeforderten Berechtigungen in der Vergangenheit nicht zugestimmt hat, fragt der v2.0-Endpunkt den Benutzer nach den erforderlichen Berechtigungen.

![Screenshot der Zustimmung im Arbeitskonto](../media/active-directory-v2-flows/work_account_consent.png)

Wenn der Benutzer die Berechtigung genehmigt, wird die Zustimmung aufgezeichnet, damit der Benutzer bei nachfolgenden Anmeldevorgängen nicht erneut seine Zustimmung geben muss.

## Verwenden von Berechtigungen

Nachdem der Benutzer den Berechtigungen für Ihre App zugestimmt hat, kann Ihre App Zugriffstoken abrufen, die für die Berechtigung der App stehen, in irgendeiner Weise auf die Ressource zugreifen zu dürfen. Ein bestimmtes Zugriffstoken kann nur für eine einzelne Ressource verwendet werden, allerdings ist darin jede Berechtigung codiert, die Ihrer App für diese Ressource erteilt wurde. Um ein Zugriffstoken zu erhalten, kann Ihre App eine Anforderung an den v2.0-Tokenendpunkt senden:

```
POST common/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/json

{
	"grant_type": "authorization_code",
	"client_id": "6731de76-14a6-49ae-97bc-6eba6914391e",
	"scope": "https://outlook.office.com/mail.read https://outlook.office.com/mail.send",
	"code": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq..."
	"redirect_uri": "https://localhost/myapp",
	"client_secret": "zc53fwe80980293klaj9823"  // NOTE: Only required for web apps
}
```

Das sich ergebende Zugriffstoken kann dann in HTTP-Anforderungen an die Ressource verwendet werden – es wird der Ressource zuverlässig anzeigen, dass Ihre Anwendung über die erforderliche Berechtigung zum Ausführen einer bestimmten Aufgabe verfügt.

Weitere Informationen zum OAuth 2.0-Protokoll und zur Erfassung von Zugriffstoken finden Sie in der [Protokollreferenz zum v2.0-Endpunkt](active-directory-v2-protocols.md).

## OpenId Connect-Bereiche

Die v2.0-Implementierung von OpenID Connect bietet einige klar definierte Bereiche, die für keine bestimmte Ressource gelten: `openid`, `email`, `profile` und `offline_access`.

#### OpenId

Bei einer App-Anmeldung mit [OpenID Connect](active-directory-v2-protocols.md#openid-connect-sign-in-flow) muss der `openid`-Bereich angefordert werden. Der `openid`-Bereich wird auf dem Zustimmungsbildschirm des Arbeitskontos als Berechtigung "Sie werden angemeldet" angezeigt, und auf dem Zustimmungsbildschirm des persönlichen Microsoft-Kontos als Berechtigung "Ihr Profil anzeigen und mit Ihrem Microsoft-Konto eine Verbindung zu Apps und Diensten herstellen". Mit dieser Berechtigung kann eine App einen eindeutigen Bezeichner für den Benutzer in Form des Anspruchs `sub` empfangen. Sie ermöglicht der App zudem Zugriff auf den Endpunkt mit den Benutzerinformationen. Der `openid`-Bereich kann auch auf dem v2.0-Tokenendpunkt zum Abrufen von ID-Token verwendet werden, die zum Sichern von HTTP-Aufrufen zwischen verschiedenen Komponenten einer App genutzt werden können.

#### E-Mail

Der Bereich `email` kann zusammen mit dem Bereich `openid` und weiteren Bereichen eingefügt werden. Er ermöglicht den App Zugriff auf die primäre E-Mail-Adresse des Benutzers in Form des Anspruchs `email`. Der Anspruch `email` ist nur in Token enthalten, wenn dem Benutzerkonto eine E-Mail-Adresse zugewiesen ist (dies ist nicht immer der Fall). Bei Verwendung des Bereichs `email` sollte die Verarbeitung durch Ihre App auch dann möglich sein, wenn der Anspruch `email` nicht im Token vorhanden ist.

#### Profil

Der Bereich `profile` kann zusammen mit dem Bereich `openid` und weiteren Bereichen eingefügt werden. Er ermöglicht der App Zugriff auf eine Fülle von Benutzerinformationen. Dazu gehören u. a. Vorname, Nachname, bevorzugter Benutzername und Objekt-ID des Benutzers. Eine vollständige Liste der in ID-Token verfügbaren Profilansprüche für einen bestimmten Benutzer finden Sie in der [v2.0-Tokenreferenz](active-directory-v2-tokens.md).

#### Offline\_access

Mit dem [`offline_access`-Bereich](http://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess) kann Ihre App im Auftrag des Benutzers für einen längeren Zeitraum auf Ressourcen zugreifen. Auf dem Zustimmungsbildschirm des Arbeitskontos erscheint dieser Bereich als "Jederzeit und überall auf Ihre Daten zugreifen". Auf dem Zustimmungsbildschirm des persönlichen Microsoft-Kontos erscheint er als "Jederzeit auf Ihre Informationen zugreifen". Wenn ein Benutzer den `offline_access`-Bereich genehmigt, ist Ihre App für den Empfang von Aktualisierungstoken vom v2.0-Tokenendpunkt aktiviert. Aktualisierungstoken sind langlebig und ermöglichen es Ihrer App, neue Zugriffstoken zu erhalten, wenn ältere ablaufen.

Wenn die App den `offline_access`-Bereich nicht anfordert, werden auch keine Aktualisierungstoken empfangen. Dies bedeutet, dass Sie beim Einlösen eines Autorisierungscodes im [OAuth 2.0-Authorisierungscodefluss](active-directory-v2-protocols.md#oauth2-authorization-code-flow) nur ein Zugriffstoken vom `/token`-Endpunkt erhalten. Dieses Zugriffstoken bleibt für einen kurzen Zeitraum (in der Regel eine Stunde) gültig, läuft aber anschließend ab. Zu diesem Zeitpunkt muss Ihre App den Benutzer zurück auf den `/authorize`-Endpunkt leiten, um einen neuen Autorisierungscode abzurufen. Während dieser Umleitung muss der Benutzer möglicherweise seine Anmeldeinformationen erneut eingeben oder den Berechtigungen erneut zustimmen, je nach Apptyp.

Weitere Informationen zum Abrufen und Verwenden von Aktualisierungstoken finden Sie in der [v2.0-Protokollreferenz](active-directory-v2-protocols.md).

<!---HONumber=AcomDC_0921_2016-->