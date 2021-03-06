<properties
	pageTitle="Sichern des privilegierten Zugriffs in Azure AD | Microsoft Azure"
	description="In diesem Thema werden Methoden zum Sichern des privilegierten Zugriffs über Azure, Azure Active Directory und Microsoft Online Services hinweg erläutert."
	services="active-directory"
	documentationCenter=""
	authors="kgremban"
	manager="stevenpo"
	editor="mwahl"/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/15/2016"
	ms.author="kgremban"/>


# Sichern des privilegierten Zugriffs in Azure AD

Das Sichern des privilegierten Zugriffs ist ein entscheidender erster Schritt, um die geschäftlichen Ressourcen in einem modernen Unternehmen zu schützen. Die Sicherheit der meisten oder sogar aller geschäftlichen Ressourcen in einem Unternehmen hängt von der Integrität der privilegierten Konten ab, über die IT-Systeme verwaltet und betrieben werden. Cyberkriminelle versuchen diese Konten anzugreifen, um Zugriff auf die Daten und Systeme eines Unternehmens zu erhalten.

Um den administrativen Zugriff vor solchen entschlossenen Angreifern zu schützen, müssen diese administrativen Konten und Systeme von Risiken isoliert werden. Über Clouddienste erhalten immer mehr Benutzer privilegierten Zugriff. Hierzu gehören globale Administratoren von Office 365, Azure-Abonnementadministratoren und Benutzer die administrativen Zugriff zu virtuellen Computern oder SaaS-Apps besitzen.

Microsoft empfiehlt, der Roadmap unter [Securing Privileged Access](https://technet.microsoft.com/library/mt631194.aspx) (Sichern des privilegierten Zugriffs) zu folgen.

Für Kunden, die Azure Active Directory für die Verwaltung des Zugriffs auf Azure, Office 365 oder andere Microsoft-Dienste und -Anwendungen verwenden, gelten diese Prinzipien unabhängig davon, ob die Benutzerkonten über Active Directory oder Azure Active Directory verwaltet und authentifiziert werden. In den folgenden Abschnitten finden Sie weitere Informationen zu den Azure AD-Features, die das Sichern des privilegierten Zugriffs unterstützen.

## Multi-Factor Authentication

Um die Sicherheit der Administratorauthentifizierung zu erhöhen, sollten Sie Multi-Factor Authentication (MFA) erfordern, bevor Berechtigungen gewährt werden. MFA ist eine Methode zum Überprüfen der Identität, für die mehr als nur ein Benutzername und ein Kennwort erforderlich ist. Sie bietet eine zweite Sicherheitsebene für Benutzeranmeldungen und -transaktionen.

Azure Multi-Factor Authentication hilft beim Schutz des Zugriffs auf Daten und Anwendungen und erfüllt gleichzeitig die Anforderungen von Benutzern an ein einfaches Anmeldeverfahren. Sie bietet eine leistungsfähige Authentifizierung mittels verschiedener einfacher Überprüfungsoptionen: Telefonanruf, SMS, Benachrichtigung in der mobilen App, Überprüfungscode und Drittanbieter-OATH-Token.

Eine Übersicht über die Funktionsweise von Azure Multi-Factor Authentication finden Sie im folgenden Video.

>[AZURE.VIDEO windows-azure-multi-factor-authentication]

Weitere Informationen finden Sie unter [MFA for Office 365 and MFA for Azure](https://blogs.technet.microsoft.com/ad/2014/02/11/mfa-for-office-365-and-mfa-for-azure/) (MFA für Office 365 und MFA für Azure).

## Zeitgebundene Berechtigungen

Einige Organisationen stellen möglicherweise fest, dass zu viele Benutzer Rollen mit hohen Berechtigungen innehaben. Möglicherweise wurden Benutzer für eine bestimmte Aktivität zu einer Rolle hinzugefügt, beispielsweise zur Anmeldung bei einem Dienst, haben diese Berechtigung später aber nicht mehr häufig verwendet.

Um Berechtigungen nicht unnötig lange zu gewähren und den Einblick in die Verwendung der Berechtigungen zu erhöhen, sollten Sie Just-In-Time-Berechtigungen für Benutzer einrichten, die die Nutzung der Berechtigung auf die Dauer der Aufgabenausführung beschränken. Für Azure Active Directory und Microsoft Online Services können Sie [Azure AD Privileged Identity Management (PIM)](http://aka.ms/AzurePIM) verwenden.


![PIM-Dashboard][2]


## Angriffserkennung

[Azure Active Directory Identity Protection](active-directory-identityprotection.md) bietet eine umfassende Übersicht über Risikoereignisse und potenzielle Sicherheitsrisiken, die für die Identitäten Ihrer Organisation bestehen. Basierend auf Risikoereignissen berechnet Identity Protection eine Benutzerrisikostufe für jeden Benutzer, sodass Sie risikobasierte Richtlinien konfigurieren können, um die Identitäten Ihrer Organisation automatisch zu schützen. Mit diesen Richtlinien in Verbindung mit anderen Steuerungselementen für den bedingten Zugriff, die von Azure Active Directory und EMS bereitgestellt werden, können Benutzer blockiert oder Vorschläge unterbreitet werden. Hierzu gehören Kennwortzurücksetzungen und die Durchsetzung der Multi-Factor Authentication.

![Azure AD Identity Protection][3]

## Bedingter Zugriff

Mit der bedingten Zugriffssteuerung überprüft Azure Active Directory beim Authentifizieren des Benutzers die von Ihnen ausgewählten besonderen Bedingungen, bevor der Zugriff auf eine Anwendung gewährt wird. Nachdem diese Bedingungen erfüllt sind, wird der Benutzer authentifiziert und erhält Zugriff auf die Anwendung.


![Festlegen bedingter Zugriffsregeln mit MFA][4]


## Rollenmodell

Um Sicherheitsrisiken zu verringern, sollten Sie Ihr Rollen- und Delegierungsmodell überprüfen und Administratorrollen von Clouddiensten sowie die Nutzbarkeit dieser Administratorrollen als wichtigen Grundsatz einbeziehen. Dieses Modell sollte Just-In-Time-Funktionen nutzen.

## Nächste Schritte

- [Azure Multi-Factor Authentication](../multi-factor-authentication/multi-factor-authentication-get-started-cloud.md) aktivieren
- [Azure AD Privileged Identity Management](active-directory-privileged-identity-management-configure.md) aktivieren
- [Azure AD Identity Protection](active-directory-identityprotection.md) aktivieren
- [Steuerung des bedingten Zugriffs](active-directory-conditional-access.md) aktivieren


Weitere Informationen zum Aufbau einer vollständigen Sicherheitsroadmap finden Sie im Abschnitt „Customer responsibilities and roadmap“ (Zuständigkeiten des Kunden und Kundenroadmap) des Dokuments [Microsoft Cloud Security for Enterprise Architects](http://aka.ms/securecustomer) (Microsoft-Cloudsicherheit für Unternehmensarchitekten). Wenn Sie weitere Informationen zum Einsatz von Microsoft-Diensten im Zusammenhang mit einem dieser Themen wünschen, wenden Sie sich an Ihren Microsoft-Vertriebsbeauftragten, oder besuchen Sie unsere Seite zu [Cybersicherheitslösungen](https://www.microsoft.com/microsoftservices/campaigns/cybersecurity-protection.aspx).

<!--Image references-->
[1]: ./media/active-directory-privileged-identity-management-configure/Search_PIM.png
[2]: ./media/active-directory-privileged-identity-management-configure/PIM_Dash.png
[3]: ./media/active-directory-identityprotection/29.png
[4]: ./media/active-directory-conditional-access/conditionalaccess-saas-apps.png

<!---HONumber=AcomDC_0518_2016-->