<properties
	pageTitle="Der Azure MFA-Anmeldevorgang mit Azure Multi-Factor Authentication"
	description="Auf dieser Seite finden Sie Anleitung zu den verschiedenen Anmeldemethoden, die mit Azure MFA verfügbar sind."
	keywords="Benutzerauthentifizierung, Anmeldevorgang, Anmelden mit dem Mobiltelefon, Anmelden mit dem Bürotelefon"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="curtland"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/22/2016"
	ms.author="kgremban"/>

# Der Anmeldevorgang mit Azure Multi-Factor Authentication
> [AZURE.NOTE]  Die folgende Dokumentation auf dieser Seite zeigt eine normale Anmeldeumgebung. Hilfe zur Anmeldung finden Sie unter [Beheben von Problemen mit Azure Multi-Factor Authentication](multi-factor-authentication-end-user-manage-settings.md).



## Was sieht Ihr Anmeldevorgang aus?
Der Anmeldevorgang hängt davon ab, wie Sie sich anmelden und Multi-Factor Authentication verwenden. In diesem Abschnitt finden Sie Informationen dazu, was Sie bei der Anmeldung erwartet. Wählen Sie die Option aus, die am besten auf Sie zutrifft.


Vorgehensweise|Beschreibung
:------------- | :------------- |
[Anmeldung mit Mobil- oder Bürotelefon](#signing-in-with-mobile-or-office-phone) | Beschreibt die Anmeldung mit einem Mobil- oder Bürotelefon.
[Anmelden mit der Microsoft Authenticator-App über eine Benachrichtigung](#signing-in-with-the-microsoft-authenticator-app-using-notification) | Beschreibt die benachrichtigungsbasierte Anmeldung mit der Microsoft Authenticator-App.
[Anmelden mit der Microsoft Authenticator-App über einen Überprüfungscode](#signing-in-with-the-microsoft-authenticator-app-using-verification-code)|Beschreibt die prüfcodebasierte Anmeldung mit der Microsoft Authenticator-App.
[Anmelden mit einer alternativen Methode](#signing-in-with-an-alternate-method)|Hier erfahren Sie, was Sie erwartet, wenn Sie eine alternative Methode verwenden möchten.

## Anmeldung mit Mobil- oder Bürotelefon

Im Folgenden wird der Vorgang für Multi-Factor Authentication mit dem Büro- oder Mobiltelefon beschrieben.

### So melden Sie sich über einen Aufruf bei Ihrem Büro- oder Mobiltelefon an

- Melden Sie sich mit Ihrem Benutzernamen und Kennwort bei einer Anwendung oder einem Dienst wie Office 365 an.
- Sie erhalten einen Anruf von Microsoft.

![Microsoft-Anrufe](./media/multi-factor-authentication-end-user-signin-phone/call.png)

- Nehmen Sie den Anruf entgegen, und drücken Sie die #-Taste.

![Antwort](./media/multi-factor-authentication-end-user-signin-phone/phone.png)

- Sie sollten jetzt angemeldet sein.</li>

## Anmelden mit der Microsoft Authenticator-App über eine Benachrichtigung

Die folgenden Informationen beschreiben die Verwendung der Multi-Factor Authentication mit der Microsoft Authenticator-App über eine an Sie gesendete Benachrichtigung.

### So melden Sie sich mit der Microsoft Authenticator-App über eine Benachrichtigung an

- Melden Sie sich mit Ihrem Benutzernamen und Kennwort bei einer Anwendung oder einem Dienst wie Office 365 an.
- Sie erhalten eine Benachrichtigung von Microsoft.

![Microsoft sendet Benachrichtigung](./media/multi-factor-authentication-end-user-signin-app-notify/notify.png)


- Rufen Sie die Benachrichtigung auf Ihrem Mobiltelefon auf, und tippen Sie auf die Schaltfläche "Überprüfen". Wenn Ihr Unternehmen eine PIN anfordert, werden Sie hier zur Eingabe gebeten.

![Überprüfen](./media/multi-factor-authentication-end-user-signin-app-notify/phone.png)

![Einrichtung](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan3.png)

- Sie sollten jetzt angemeldet sein.


## Anmelden mit der Microsoft Authenticator-App über einen Überprüfungscode

Im Folgenden wird der Vorgang für die Multi-Factor Authentication mit der Microsoft Authenticator-App unter Verwendung eines Überprüfungscodes beschrieben.

### So melden Sie sich mit der Microsoft Authenticator-App über einen Überprüfungscode an

- Melden Sie sich mit Ihrem Benutzernamen und Kennwort bei einer Anwendung oder einem Dienst wie Office 365 an.
- Sie werden von Microsoft aufgefordert, einen Bestätigungscode einzugeben.

![Eingeben des Überprüfungscodes](./media/multi-factor-authentication-end-user-signin-app-verify/verify.png)

- Öffnen Sie die Microsoft Authenticator-App auf Ihrem Smartphone, und geben Sie den Code bei der Anmeldung in das entsprechende Feld ein.

![Abrufen des Codes](./media/multi-factor-authentication-end-user-signin-app-verify/phone.png)



- Sie sollten jetzt angemeldet sein.


## Anmelden mit einer alternativen Methode


Im folgenden Abschnitt erfahren Sie, wie Sie sich über eine alternative Methode anmelden, wenn die primäre Methode nicht verfügbar ist.

### So melden Sie sich mit einer alternativen Methode an

- Melden Sie sich mit Ihrem Benutzernamen und Kennwort bei einer Anwendung oder einem Dienst wie Office 365 an.
- Wählen Sie eine andere Überprüfungsoption aus. Es stehen verschiedene Optionen zur Auswahl. Die Anzahl der verfügbaren Optionen hängt davon ab, wie viele Sie eingerichtet haben.

![Anwenden einer alternativen Methode](./media/multi-factor-authentication-end-user-signin-alt/alt.png)

- Wählen Sie eine alternative Methode aus, und melden Sie sich an.

<!---HONumber=AcomDC_0921_2016-->