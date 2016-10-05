<properties
	pageTitle="Azure AD v2.0-Tokenreferenz| Microsoft Azure"
	description="Die vom v2.0-Endpunkt ausgestellten Token- und Anspruchstypen"
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

# v2.0-Tokenreferenz

Der Endpunkt mit der Version 2.0 stellt bei der Verarbeitung der einzelnen [Authentifizierungsflüsse](active-directory-v2-flows.md) verschiedene Arten von Sicherheitstoken aus. In diesem Dokument sind das Format, die Sicherheitsmerkmale und der Inhalt aller Tokentypen beschrieben.

> [AZURE.NOTE]
	Nicht alle Szenarien und Funktionen von Azure Active Directory werden vom v2.0-Endpunkt unterstützt. Lesen Sie die Informationen zu den [Einschränkungen des v2.0-Endpunkts](active-directory-v2-limitations.md), um herauszufinden, ob Sie den v2.0-Endpunkt verwenden sollten.

## Tokentypen

Der Endpunkt mit der Version 2.0 unterstützt das [OAuth 2.0-Autorisierungsprotokoll](active-directory-v2-protocols.md), das Zugriffstoken und Aktualisierungstoken verwendet. Er unterstützt darüber hinaus die Authentifizierung und Anmeldung über [OpenID Connect](active-directory-v2-protocols.md#openid-connect-sign-in-flow). Dabei wird ein dritter Tokentyp, das ID-Token, eingeführt. Alle diese Token werden als Bearertoken dargestellt.

Ein Trägertoken ist ein einfaches Sicherheitstoken, das dem „Träger“ den Zugriff auf eine geschützte Ressource ermöglicht. In diesem Kontext ist der „Träger“ jede beliebige Partei, die das Token vorweisen kann. Um das Trägertoken zu erhalten, muss sich die Partei zwar zunächst bei Azure AD authentifizieren, falls jedoch keine Maßnahmen ergriffen werden, um das Token bei der Übertragung und Speicherung zu schützen, kann das Token von einer fremden Partei abgefangen und verwendet werden. Einige Sicherheitstoken verfügen über einen integrierten Mechanismus, der eine unbefugte Verwendung durch nicht autorisierte Parteien verhindert. Trägertoken besitzen dagegen keinen solchen Mechanismus und müssen über einen sicheren Kanal wie etwa Transport Layer Security (HTTPS) übertragen werden. Wird ein Trägertoken als Klartext gesendet, kann eine böswillige Partei das Token mithilfe eines Man-in-the-Middle-Angriffs abfangen und damit unautorisiert auf eine geschützte Ressource zugreifen. Die gleichen Sicherheitsprinzipien gelten für die (Zwischen-)Speicherung von Trägertoken zur späteren Verwendung. Stellen Sie daher sicher, dass Ihre App Bearertoken stets auf sichere Weise überträgt und speichert. Weitere Sicherheitsüberlegungen zu Trägertoken finden Sie unter [RFC 6750, Abschnitt 5](http://tools.ietf.org/html/rfc6750).

Viele der vom Endpunkt mit der Version 2.0 ausgestellten Token werden als Json-Webtoken (JWTs) implementiert. Ein JWT stellt eine kompakte, URL-sichere Methode zum Übertragen von Informationen zwischen zwei Parteien dar. Die in JWTs enthaltenen Informationen werden als „Ansprüche“ oder Assertionen von Informationen zum Träger und Antragsteller des Tokens bezeichnet. Die Ansprüche in JWTs sind JSON-Objekte, die für die Übertragung codiert und serialisiert wurden. Da die vom Endpunkt mit der Version 2.0 ausgestellten JWTs signiert, aber nicht verschlüsselt sind, können Sie den Inhalt eines JWTs problemlos für Debugzwecke untersuchen. Weitere Informationen zu JWTs finden Sie in der [JWT-Spezifikation](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html).

## ID-Token

ID-Token sind eine Form von Anmeldesicherheitstoken, die Ihre App empfängt, wenn die Authentifizierung mit [OpenID Connect](active-directory-v2-protocols.md#openid-connect-sign-in-flow) ausgeführt wird. Sie werden als [JWTs](#types-of-tokens) dargestellt und enthalten Ansprüche, die Sie für die Anmeldung der Benutzer bei der App verwenden können. Sie können die Ansprüche nach Bedarf in einem ID-Token verwenden. Im Allgemeinen werden Sie in einer App zum Anzeigen von Kontoinformationen oder zum Treffen von Entscheidungen hinsichtlich der Zugriffssteuerung eingesetzt. Der Endpunkt mit der Version 2.0 stellt nur einen Typ von ID-Token aus. Dieser enthält unabhängig vom angemeldeten Benutzertyp einen konsistenten Satz von Ansprüchen. Das bedeutet, dass das Format und der Inhalt der ID-Token für persönliche Microsoft-Kontobenutzer und Geschäfts- oder Schulkonten gleich sind.

ID-Token sind zu diesem Zeitpunkt zwar signiert, aber nicht verschlüsselt. Wenn Ihre App ein ID-Token empfängt, muss sie [die Signatur überprüfen](#validating-tokens), um die Authentizität des Tokens nachzuweisen. Zudem müssen einige Ansprüche im Token überprüft werden, um seine Gültigkeit zu belegen. Die von einer App überprüften Ansprüche variieren je nach Szenarioanforderungen. Es gibt jedoch einige [allgemeine Anspruchsüberprüfungen](#validating-tokens), die Ihre App in jedem Szenario ausführen muss.

Vollständige Informationen zu den Ansprüchen in ID-Token sowie ein Beispiel für ein ID-Token sind nachfolgend aufgeführt. Beachten Sie, dass die Ansprüche in ID-Token nicht in einer bestimmten Reihenfolge zurückgegeben werden. Darüber hinaus können neue Ansprüche in ID-Token zu einem beliebigen Zeitpunkt eingeführt werden. Ihre App darf jedoch nicht unterbrochen werden, wenn neue Ansprüche eingeführt werden. Die folgende Liste enthält die Ansprüche, die Ihre App zum Zeitpunkt der Erstellung dieses Dokuments zuverlässig interpretieren kann. Ausführlichere Informationen finden Sie ggf. in der [OpenID Connect-Spezifikation](http://openid.net/specs/openid-connect-core-1_0.html).

#### ID-Beispieltoken

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSJ9.eyJhdWQiOiI2NzMxZGU3Ni0xNGE2LTQ5YWUtOTdiYy02ZWJhNjkxNDM5MWUiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vYjk0MTk4MTgtMDlhZi00OWMyLWIwYzMtNjUzYWRjMWYzNzZlL3YyLjAiLCJpYXQiOjE0NTIyODUzMzEsIm5iZiI6MTQ1MjI4NTMzMSwiZXhwIjoxNDUyMjg5MjMxLCJuYW1lIjoiQmFiZSBSdXRoIiwibm9uY2UiOiIxMjM0NSIsIm9pZCI6ImExZGJkZGU4LWU0ZjktNDU3MS1hZDkzLTMwNTllMzc1MGQyMyIsInByZWZlcnJlZF91c2VybmFtZSI6InRoZWdyZWF0YmFtYmlub0BueXkub25taWNyb3NvZnQuY29tIiwic3ViIjoiTUY0Zi1nZ1dNRWppMTJLeW5KVU5RWnBoYVVUdkxjUXVnNWpkRjJubDAxUSIsInRpZCI6ImI5NDE5ODE4LTA5YWYtNDljMi1iMGMzLTY1M2FkYzFmMzc2ZSIsInZlciI6IjIuMCJ9.p_rYdrtJ1oCmgDBggNHB9O38KTnLCMGbMDODdirdmZbmJcTHiZDdtTc-hguu3krhbtOsoYM2HJeZM3Wsbp_YcfSKDY--X_NobMNsxbT7bqZHxDnA2jTMyrmt5v2EKUnEeVtSiJXyO3JWUq9R0dO-m4o9_8jGP6zHtR62zLaotTBYHmgeKpZgTFB9WtUq8DVdyMn_HSvQEfz-LWqckbcTwM_9RNKoGRVk38KChVJo4z5LkksYRarDo8QgQ7xEKmYmPvRr_I7gvM2bmlZQds2OeqWLB1NSNbFZqyFOCgYn3bAQ-nEQSKwBaA36jYGPOVG2r2Qv1uKcpSOxzxaQybzYpQ
```

> [AZURE.TIP] Überprüfen Sie zu Übungszwecken die Ansprüche im ID-Beispieltoken, indem Sie sie in [calebb.net](https://calebb.net) einfügen.

#### Ansprüche in ID-Token
| Name | Anspruch | Beispielwert | Beschreibung |
| ----------------------- | ------------------------------- | ------------ | --------------------------------- |
| Zielgruppe | `aud` | `6731de76-14a6-49ae-97bc-6eba6914391e` | Identifiziert den vorgesehenen Empfänger des Tokens. In ID-Token ist die Zielgruppe die Anwendungs-ID der App, die Ihrer App im App-Registrierungsportal zugewiesen wurde. Ihre App sollte diesen Wert überprüfen und das Token ablehnen, wenn er nicht übereinstimmt. |
| Issuer (Aussteller) | `iss` | `https://login.microsoftonline.com/b9419818-09af-49c2-b0c3-653adc1f376e/v2.0 ` | Identifiziert den Sicherheitstokendienst (Security Token Service, STS), der das Token und den Azure AD-Mandanten, in dem der Benutzer authentifiziert wurde, erstellt und zurückgibt. Ihre App sollte den Ausstelleranspruch überprüfen, um sicherzustellen, dass das Token vom Endpunkt mit der Version 2.0 stammt. Sie sollte auch den GUID-Abschnitt des Anspruchs nutzen, um die Gruppe der Mandanten zu beschränken, die sich bei der App anmelden dürfen. Die GUID, die angibt, dass der Benutzer ein Consumer-Benutzer eines Microsoft-Kontos ist, lautet `9188040d-6c67-4c5b-b112-36a304b66dad`. |
| Ausgestellt um | `iat` | `1452285331` | Die Zeit, zu der das Token ausgestellt wurde (dargestellt als Epochenzeit) |
| Ablaufzeit | `exp` | `1452289231` | Die Zeit, zu der das Token ungültig wird (dargestellt als Epochenzeit). Ihre App sollte mit diesem Anspruch die Gültigkeit der Tokenlebensdauer überprüfen. |
| Nicht vor | `nbf` | `1452285331` | Die Zeit, zu der das Token gültig wird (dargestellt als Epochenzeit). Dieser Wert ist normalerweise mit dem Ausstellungszeitpunkt identisch. Ihre App sollte mit diesem Anspruch die Gültigkeit der Tokenlebensdauer überprüfen. |
| Version | `ver` | `2.0` | Die Version des ID-Tokens, wie in Azure AD definiert. Für den v2.0-Endpunkt lautet der Wert `2.0`. |
| Mandanten-ID | `tid` | `b9419818-09af-49c2-b0c3-653adc1f376e` | Eine GUID, die den Azure AD-Mandanten darstellt, aus dem der Benutzer stammt. Bei Geschäfts- und Schulkonten ist die GUID die unveränderliche Mandanten-ID der Organisation, zu der der Benutzer gehört. Für persönliche Konten lautet der Wert `9188040d-6c67-4c5b-b112-36a304b66dad`. Der Bereich `profile` ist erforderlich, um diesen Anspruch zu empfangen. |
| Codehash | `c_hash` | `SGCPtt01wxwfgnYZy2VJtQ` | Der Codehash ist nur in ID-Token enthalten, wenn das ID-Token zusammen mit einem OAuth 2.0-Autorisierungscode ausgestellt wird. Mit seiner Hilfe kann die Authentizität eines Autorisierungscodes überprüft werden. Weitere Informationen zum Ausführen dieser Überprüfung finden Sie in der [OpenID Connect-Spezifikation](http://openid.net/specs/openid-connect-core-1_0.html). |
| Zugriffstokenhash | `at_hash` | `SGCPtt01wxwfgnYZy2VJtQ` | Der Zugriffstokenhash ist nur in ID-Token enthalten, wenn das ID-Token zusammen mit einem OAuth 2.0-Zugriffstoken ausgestellt wird. Mit seiner Hilfe kann die Authentizität eines Zugriffstokens überprüft werden. Weitere Informationen zum Ausführen dieser Überprüfung finden Sie in der [OpenID Connect-Spezifikation](http://openid.net/specs/openid-connect-core-1_0.html). |
| Nonce | `nonce` | `12345` | Mithilfe der Nonce sollen Tokenwiedergabeangriffe abgewehrt werden. Ihre App kann eine Nonce in einer Autorisierungsanforderung mithilfe des `nonce`-Abfrageparameters angeben. Der in der Anforderung angegebene Wert wird (unverändert) im `nonce`-Anspruch des ID-Tokens ausgegeben. Dadurch kann Ihre App den Wert anhand des Werts überprüfen, der in der Anforderung angegeben ist. Dabei wird die Sitzung der App einem bestimmten ID-Token zugeordnet. Ihre App sollte diese Überprüfung während des Überprüfungsvorgangs des ID-Tokens ausführen. |
| Name | `name` | `Babe Ruth` | Der Namensanspruch gibt einen von Menschen lesbaren Wert an, der den Antragsteller des Tokens identifiziert. Der Wert ist nicht zwingend eindeutig, kann geändert werden und dient nur zu Anzeigezwecken. Der Bereich `profile` ist erforderlich, um diesen Anspruch zu empfangen. |
| E-Mail | `email` | `thegreatbambino@nyy.onmicrosoft.com` | Die primäre E-Mail-Adresse, die mit dem Benutzerkonto verknüpft ist (sofern vorhanden). Sein Wert ist änderbar und kann sich für einen bestimmten Benutzer im Laufe der Zeit ändern. Der Bereich `email` ist erforderlich, um diesen Anspruch zu empfangen. |
| Bevorzugter Benutzername | `preferred_username` | `thegreatbambino@nyy.onmicrosoft.com` | Der primäre Benutzername, der zum Darstellen des Benutzers im Endpunkt mit der Version 2.0 verwendet wird. Dabei kann es sich um eine E-Mail-Adresse, eine Telefonnummer oder einen generischen Benutzernamen ohne bestimmtes Format handeln. Sein Wert ist änderbar und kann sich für einen bestimmten Benutzer im Laufe der Zeit ändern. Der Bereich `profile` ist erforderlich, um diesen Anspruch zu empfangen. |
| Subject (Antragsteller) | `sub` | `MF4f-ggWMEji12KynJUNQZphaUTvLcQug5jdF2nl01Q` | Der Prinzipal, für den das Token Informationen zusichert, z. B. der Benutzer einer App. Dieser Wert ist unveränderlich und kann nicht neu zugewiesen oder wiederverwendet werden. Daher kann er für die sichere Durchführung von Autorisierungsüberprüfungen verwendet werden, z. B. wenn das Token verwendet wird, um auf eine Ressource zuzugreifen. Da der Antragsteller immer in den Token vorhanden ist, die Azure AD ausstellt, wird die Nutzung dieses Werts in einem allgemeinen Autorisierungssystem empfohlen. |
| ObjectId | `oid` | `a1dbdde8-e4f9-4571-ad93-3059e3750d23` | Die Objekt-ID des Geschäfts- oder Schulkontos im Azure AD-System. Dieser Anspruch wird nicht für persönliche Microsoft-Konten ausgestellt. Der Bereich `profile` ist erforderlich, um diesen Anspruch zu empfangen. |


## Zugriffstoken

Zugriffstoken, die vom Endpunkt mit der Version 2.0 ausgestellt werden, können nur zu diesem Zeitpunkt von Microsoft-Diensten genutzt werden. Apps sollten keine Validierung oder Überprüfung der Zugriffstoken für alle derzeit unterstützten Szenarien ausführen müssen. Der Inhalt von Zugriffstoken ist nicht zugänglich. Bei ihnen handelt es sich lediglich um Zeichenfolgen, die von Ihrer App in HTTP-Anforderungen an Microsoft gesendet werden können.

In Kürze wird es auf dem Endpunkt mit der Version 2.0 für Ihre App möglich sein, Zugriffstoken von anderen Clients zu empfangen. Zu diesem Zeitpunkt werden die Informationen mit den Daten aktualisiert, die die App für die Überprüfung von Zugriffstoken und andere ähnliche Aufgaben benötigt.

Beim Anfordern eines Zugriffstokens vom Endpunkt mit der Version 2.0 gibt der Endpunkt auch einige Metadaten zum Zugriffstoken für die Nutzung durch Ihre App zurück. Diese Informationen umfassen die Ablaufzeit eines Zugriffstokens und die Bereiche, für die es gilt. Dies ermöglicht Ihrer App das intelligente Zwischenspeichern von Zugriffstoken, ohne dass dabei das Zugriffstoken selbst analysiert werden muss.

## Aktualisierungstoken

Aktualisierungstoken sind Sicherheitstoken, mit denen Ihre App neue Zugriffstoken in einem OAuth 2.0-Fluss abrufen kann. Dadurch erhält Ihre App langfristig Zugriff auf Ressourcen im Auftrag eines Benutzers, ohne dass ein Benutzereingriff erforderlich ist.

Aktualisierungstoken bestehen aus mehreren Ressourcen. Das bedeutet, dass ein Aktualisierungstoken, das bei einer Tokenanforderung für eine Ressource empfangen wurde, für Zugriffstoken bei einer völlig anderen Ressource eingelöst werden kann.

Um eine Aktualisierung bei einer Tokenantwort zu erhalten, muss Ihre App den `offline_acesss`-Bereich anfordern, und dieser Bereich muss gewährt werden. Weitere Informationen zum `offline_access`-Bereich finden Sie [hier im Artikel zu Zustimmung und Bereichen](active-directory-v2-scopes.md).

Der Inhalt von Aktualisierungstoken ist für Ihre App niemals zugänglich. Sie werden vom Azure AD-Endpunkt mit der Version 2.0 ausgestellt und können nur vom Endpunkt mit der Version 2.0 überprüft und interpretiert werden. Sie sind zwar sehr lange gültig, in Ihrer App darf aber nicht von einer unbegrenzten Gültigkeitsdauer ausgegangen werden. Aktualisierungstoken können jederzeit aus unterschiedlichen Gründen ungültig werden. Die einzige Möglichkeit für Ihre App, die Gültigkeit eines Aktualisierungstokens zu überprüfen, besteht in der Einlösung des Tokens. Führen Sie dazu eine Tokenanforderung auf dem Endpunkt mit der Version 2.0 aus.

Wenn Sie ein Aktualisierungstoken für ein neues Zugriffstoken einlösen (und wenn Ihrer App der `offline_access`-Bereich zugeteilt wurde), erhalten Sie ein neues Aktualisierungstoken in der Tokenantwort. Speichern Sie das neu ausgestellte Aktualisierungstoken, und ersetzen Sie das in der Anforderung verwendete Token. Dadurch wird sichergestellt, dass die Aktualisierungstoken möglichst lange gültig bleiben.

## Überprüfen von Token

Zu diesem Zeitpunkt müssen Ihre Apps nur ID-Token überprüfen. Bei der Überprüfung eines ID-Tokens muss Ihre App sowohl die Signatur des ID-Tokens als auch die Ansprüche im ID-Token überprüfen.

<!-- TODO: Link -->
Wir stellen Bibliotheken und Codebeispiele bereit, die die Tokenüberprüfung veranschaulichen. Die folgenden Informationen werden lediglich für Benutzer bereitgestellt, die den zugrunde liegenden Prozess nachvollziehen möchten. Für die JWT-Überprüfung stehen zudem verschiedene Open Source-Bibliotheken von Drittanbietern zur Verfügung. Da ist für nahezu jede Plattform und Sprache mindestens eine Option dabei.

#### Überprüfen der Signatur
Ein JWT enthält drei Segmente, die durch das Zeichen `.` getrennt sind. Das erste Segment wird als **Header**, das zweite als **Text** und das dritte als **Signatur** bezeichnet. Mit dem Signatursegment kann die Authentizität des ID-Tokens überprüft werden, sodass es für Ihre App als vertrauenswürdig eingestuft werden kann.

ID-Token werden mit branchenüblichen asymmetrischen Verschlüsselungsalgorithmen wie etwa RSA 256 signiert. Der Header des ID-Tokens enthält Informationen zum Schlüssel und zur Verschlüsselungsmethode, die zum Signieren des Tokens verwendet wird:

```
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "MnC_VZcATfM5pOYiJHMba9goEKY"
}
```

Der `alg`-Anspruch gibt den Algorithmus an, mit dem das Token signiert wurde. Der `kid`-Anspruch hingegen bezeichnet den bestimmten öffentlichen Schlüssel, mit dem das Token signiert wurde.

Zu einem beliebigen Zeitpunkt signiert der Endpunkt mit der Version 2.0 vielleicht ein ID-Token mithilfe eines bestimmten Satzes von Paaren aus öffentlichen und privaten Schlüsseln. Der Endpunkt mit der Version 2.0 wechselt regelmäßig durch die möglichen Sätze von Schlüsseln. Ihre App muss also auf die automatische Verarbeitung dieser Schlüsseländerungen ausgelegt sein. Die vom Endpunkt mit der Version 2.0 verwendeten öffentlichen Schlüssel sollten alle 24 Stunden auf Änderungen überprüft werden.

Sie können die Signaturschlüsseldaten, die zum Überprüfen der Signatur erforderlich sind, mithilfe des OpenID Connect-Metadatendokuments abrufen. Dieses Dokument befindet sich hier:

```
https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration
```

> [AZURE.TIP] Fügen Sie diese URL in einen Browser ein.

Bei diesem Metadatendokument handelt es sich um ein JSON-Objekt, das zahlreiche nützliche Informationen enthält, beispielsweise den Ort der verschiedenen Endpunkte, die zum Ausführen der OpenID Connect-Authentifizierung erforderlich sind.

Darüber hinaus enthält es einen `jwks_uri`, der den Speicherort des Satzes von öffentlichen Schlüsseln zum Signieren von Token angibt. Das JSON-Dokument unter `jwks_uri` enthält alle Informationen zu den zu diesem Zeitpunkt verwendeten öffentlichen Schlüsseln. Ihre App kann mit dem Anspruch `kid` im JWT-Header auswählen, welcher öffentliche Schlüssel in diesem Dokument zum Signieren eines bestimmten Tokens verwendet wurde. Sie kann anschließend die Signaturüberprüfung mithilfe des korrekten öffentlichen Schlüssels und des angegebenen Algorithmus ausführen.

Die Signaturüberprüfung wird in diesem Dokument nicht erläutert. Es stehen jedoch zahlreiche Open Source-Bibliotheken mit hilfreichen Informationen zur Verfügung.

#### Überprüfen der Ansprüche
Wenn Ihre App bei der Benutzeranmeldung ein ID-Token empfängt, sollte sie auch die Ansprüche im ID-Token überprüfen. Dazu gehören unter anderem folgende Ansprüche:

- Anspruch **Zielgruppe**: Zum Überprüfen, ob das ID-Token an Ihre App übergeben werden sollte.
- Ansprüche **Nicht vor** und **Ablaufzeit**: Zum Sicherstellen, dass das ID-Token nicht abgelaufen ist.
- Anspruch **Aussteller**: Zum Überprüfen, ob das Token tatsächlich vom Endpunkt mit der Version 2.0 für Ihre App ausgestellt wurde.
- **Nonce**: Zur Abwehr von Tokenwiedergabeangriffen.
- und vieles mehr...

Eine vollständige Liste mit Anspruchsüberprüfungen, die von Ihrer App ausgeführt werden sollten, finden Sie in der [OpenID Connect-Spezifikation](http://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation).

Details zu den erwarteten Werten für diese Ansprüche finden Sie oben im Abschnitt [ID-Token](#id_tokens).


## Tokengültigkeitsdauer

Die folgenden Angaben zur Tokengültigkeitsdauer dienen ausschließlich zu Informationszwecken, da sie beim Entwickeln und Debuggen von Apps hilfreich sein können. In Ihren Apps darf nicht davon ausgegangen werden, dass die Gültigkeitsdauer konstant bleibt, da sich diese jederzeit ändern kann.

| Token | Gültigkeitsdauer | Beschreibung |
| ----------------------- | ------------------------------- | ------------ |
| ID-Token (Geschäfts- oder Schulkonten) | 1 Stunde | ID-Token gelten in der Regel eine Stunde. Ihre Web-App kann die gleiche Dauer für die Sitzung mit dem Benutzer verwenden (empfohlen) oder eine andere Sitzungsdauer wählen. Falls Ihre App ein neues ID-Token abrufen muss, muss sie lediglich eine neue Anmeldeanforderung an den Autorisierungsendpunkt mit der Version 2.0 senden. Wenn der Benutzer eine gültige Browsersitzung für den Endpunkt mit der Version 2.0 nutzt, müssen unter Umständen die Anmeldeinformationen nicht erneut eingegeben werden. |
| ID-Token (Persönliche Konten) | 24 Stunden | ID-Token für persönliche Konten gelten in der Regel 24 Stunden. Ihre Web-App kann die gleiche Dauer für die Sitzung mit dem Benutzer verwenden (empfohlen) oder eine andere Sitzungsdauer wählen. Falls Ihre App ein neues ID-Token abrufen muss, muss sie lediglich eine neue Anmeldeanforderung an den Autorisierungsendpunkt mit der Version 2.0 senden. Wenn der Benutzer eine gültige Browsersitzung für den Endpunkt mit der Version 2.0 nutzt, müssen unter Umständen die Anmeldeinformationen nicht erneut eingegeben werden. |
| Zugriffstoken (Geschäfts- oder Schulkonten) | 1 Stunde | Wird in Tokenantworten als Teil der Tokenmetadaten angegeben. |
| Zugriffstoken (Persönliche Konten) | 1 Stunde | Wird in Tokenantworten als Teil der Tokenmetadaten angegeben. Für persönliche Konten ausgestellte Zugriffstoken kann eine andere Gültigkeitsdauer konfiguriert werden. In der Regel sind sie jedoch 1 Stunde lang gültig. |
| Aktualisierungstoken (Geschäfts- oder Schulkonten) | Bis zu 14 Tage | Ein einzelnes Aktualisierungstoken ist maximal 14 Tage gültig. Das Aktualisierungstoken kann jedoch jederzeit aus verschiedenen Gründen ungültig werden. Ihre App muss daher ein Aktualisierungstoken verwenden, bis es fehlschlägt oder bis es von Ihrer App durch ein neues Aktualisierungstoken ersetzt wird. Ein Aktualisierungstoken wird außerdem ungültig, wenn der Benutzer das letzte Mal vor 90 Tagen seine Anmeldeinformationen eingegeben hat. |
| Aktualisierungstoken (Persönliche Konten) | Bis zu 1 Jahr | Ein einzelnes Aktualisierungstoken ist maximal 1 Jahr gültig. Das Aktualisierungstoken kann jedoch jederzeit aus verschiedenen Gründen ungültig werden. Ihre App muss daher ein Aktualisierungstoken verwenden, bis es fehlschlägt. |
| Autorisierungscodes (Geschäfts- oder Schulkonten) | 10 Minuten | Autorisierungscodes haben absichtlich eine kurze Gültigkeitsdauer. Sie müssen umgehend für Zugriffs- und Aktualisierungstoken eingelöst werden, wenn sie empfangen werden. |
| Autorisierungscodes (Persönliche Konten) | 5 Minuten | Autorisierungscodes haben absichtlich eine kurze Gültigkeitsdauer. Sie müssen umgehend für Zugriffs- und Aktualisierungstoken eingelöst werden, wenn sie empfangen werden. Für persönliche Konten ausgestellte Autorisierungscodes können auch nur einmal verwendet werden. |

<!---HONumber=AcomDC_0921_2016-->