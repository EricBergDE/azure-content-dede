
<properties
	pageTitle="Verwenden von Azure AD Connect Health mit AD FS | Microsoft Azure"
	description="Dies ist die Azure AD Connect Health-Seite, auf der Sie erfahren, wie Sie Ihre lokale AD FS-Infrastruktur überwachen."
	services="active-directory"
	documentationCenter=""
	authors="karavar"
	manager="femila"
	editor="karavar"/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="07/14/2016"
	ms.author="vakarand"/>

# Verwenden von Azure AD Connect Health mit AD FS
Die folgende Dokumentation bezieht sich auf die Überwachung Ihrer AD FS-Infrastruktur mit Azure AD Connect Health. Informationen zum Überwachen von Azure AD Connect (Sync) mit Azure AD Connect Health finden Sie unter [Verwenden von Azure AD Connect Health für die Synchronisierung](active-directory-aadconnect-health-sync.md). Informationen zur Überwachung der Active Directory-Domänendienste mit Azure AD Connect Health finden Sie unter [Verwenden von Azure AD Connect Health mit AD DS](active-directory-aadconnect-health-adds.md).

## Warnungen für AD FS
Im Abschnitt "Azure AD Connect Health-Warnungen" wird eine Liste der aktiven Warnungen angezeigt. Jede Warnung umfasst relevante Informationen, Lösungsschritte und Links zur verwandten Dokumentation. Durch Auswahl einer aktiven oder behobenen Warnung wird ein neues Blatt angezeigt. Dieses enthält zusätzliche Informationen, Lösungsschritte und Links zu weiterführender Dokumentation. Sie können außerdem Verlaufsdaten zu bereits behobenen Warnungen anzeigen.

Durch Auswahl einer Warnung werden zusätzliche Informationen, Lösungsschritte sowie Links zu weiterführender Dokumentation angezeigt.

![Azure AD Connect Health-Portal](./media/active-directory-aadconnect-health/alert2.png)



## Nutzungsanalyse für AD FS
In der Azure AD Connect Health-Nutzungsanalyse wird der Authentifizierungsdatenverkehr für Ihre Verbundserver analysiert. Bei Auswahl des Felds "Nutzungsanalyse" wird das Blatt zur Nutzungsanalyse geöffnet, in dem Metriken und Gruppierungen angezeigt werden.

>[AZURE.NOTE] Um die Nutzungsanalyse mit AD FS verwenden zu können, müssen Sie sicherstellen, dass die AD FS-Überwachung aktiviert ist. Weitere Informationen finden Sie unter [Aktivieren der Überwachung für AD FS](active-directory-aadconnect-health-agent-install.md#enable-auditing-for-ad-fs).

![Azure AD Connect Health-Portal](./media/active-directory-aadconnect-health/report1.png)

Um zusätzliche Metriken auszuwählen, einen Zeitraum anzugeben oder die Gruppierung zu ändern, klicken Sie einfach mit der rechten Maustaste auf das Diagramm zur Nutzungsanalyse und wählen "Diagramm bearbeiten". Anschließend können Sie den gewünschten Zeitraum angeben, Metriken ändern oder auswählen und die Gruppierung ändern. Sie können die Verteilung des Authentifizierungsdatenverkehrs basierend auf verschiedenen Metriken anzeigen und jede Metrik über die relevanten Parameter gruppieren. Diese Parameter zur Gruppierung werden nachstehend beschrieben.

| Metrik | Gruppieren nach | Bedeutung und Nutzen der Gruppierung |
| ------ | -------- | -------------------------------------------- |
| Anforderungen insgesamt: Die Gesamtzahl der Anforderungen, die durch den Verbunddienst verarbeitet wurden. | Alle | Zeigt die Gesamtzahl von Anforderungen ohne Gruppierung. |
| | Anwendung | Diese Option gruppiert die Gesamtzahl der Anforderungen basierend auf der Zielanwendung (vertrauende Seite). Diese Gruppierung ist nützlich, wenn Sie die prozentuale Verteilung des gesamten Datenverkehrs auf die verschiedenen Anwendungen anzeigen möchten. |
| | Server | Diese Option gruppiert die Gesamtzahl der Anforderungen basierend auf dem Server, der die Anforderung verarbeitet hat. Diese Gruppierung ist nützlich, um die Lastverteilung für den gesamten Datenverkehr anzuzeigen. |
| | In den Arbeitsplatz eingebunden | Diese Option gruppiert die Gesamtzahl der Anforderungen basierend auf den Anforderungen, die von in den Arbeitsplatz eingebundenen Geräten eingehen (bekannt). Diese Gruppierung ist nützlich, wenn auf Ihre Ressourcen über Geräte zugegriffen wird, die der Identitätsinfrastruktur unbekannt sind. |
| | Authentication Method (Authentifizierungsmethode) | Diese Option gruppiert die Gesamtzahl der Anforderungen basierend auf der Authentifizierungsmethode. Diese Gruppierung ist nützlich, wenn Sie wissen möchten, welche Authentifizierungsmethode in der Regel für die Authentifizierung verwendet wird. Folgende Authentifizierungsmethoden sind möglich: <ol> <li>Integrierte Windows-Authentifizierung (Windows)</li> <li>Formularbasierte Authentifizierung (Formulare)</li> <li>Einmaliges Anmelden (Single Sign On, SSO)</li> <li>X509-Zertifikatauthentifizierung (Zertifikat)</li> <br>Beachten Sie, dass eine Anforderung als SSO-Anforderung eingestuft wird, wenn der Verbundserver die Anforderung mit einem SSO-Cookie empfängt. In diesen Fällen wird, sofern es sich um ein gültiges Cookie handelt, der Benutzer nicht zur Eingabe von Anmeldeinformationen aufgefordert, sondern erhält nahtlos Zugriff auf die Anwendung. Dies ist üblich, wenn mehrere vertrauende Seiten von den Verbundservern geschützt werden. |
| | Netzwerkadresse | Diese Option gruppiert die Gesamtzahl der Anforderungen basierend auf der Netzwerkadresse des Benutzers. Dies kann entweder ein Intranet oder Extranet sein. Diese Gruppierung ist nützlich, wenn Sie die prozentuale Verteilung des eingehenden Datenverkehrs auf Intranet und Extranet anzeigen möchten. |
| Fehlerhafte Anforderungen insgesamt: Die Gesamtzahl der Anforderungen den Verbunddienst, bei denen Fehler aufgetreten sind. <br> (Diese Metrik steht nur in AD FS für Windows Server 2012 R2 zur Verfügung.)| Fehlertyp | Zeigt die Anzahl von Fehlern basierend auf den vordefinierten Fehlertypen. Diese Gruppierung ist nützlich, um die allgemeinen Arten von Fehlern zu verstehen. <ul><li>Falscher Benutzername oder falsches Kennwort: Fehler, die durch einen falschen Benutzernamen oder ein falsches Kennwort ausgelöst werden.</li> <li>"Extranetsperrung": Fehler durch Anforderungen von einem Benutzer, der für das Extranet gesperrt wurde.</li><li> "Kennwort abgelaufen": Fehler durch Benutzer, die sich mit einem abgelaufenen Kennwort anmelden.</li><li>"Konto deaktiviert": Fehler durch Benutzer, die sich mit einem deaktivierten Konto anmelden.</li><li>"Authentifizierung mit Gerät": Fehler durch Benutzer, bei denen Probleme bei der Geräteauthentifizierung auftreten.</li><li>"Authentifizierung mit Benutzerzertifikat": Fehler durch Benutzer, bei denen die Authentifizierung aufgrund eines ungültigen Zertifikats nicht möglich ist.</li><li>"MFA": Fehler durch Benutzer, die sich nicht per Multi Factor Authentication authentifizieren können.</li><li>"Andere Anmeldeinformationen": "Ausstellungsautorisierung": Fehler aufgrund von Autorisierungsproblemen.</li><li>"Ausstellungsdelegierung": Fehler aufgrund von Problemen bei der Ausstellungsdelegierung.</li><li>"Tokenannahme": Fehler durch das Ablehnen von externen Identitätsanbietern durch AD FS.</li><li>"Protokoll": Fehler aufgrund von Protokollproblemen.</li><li>"Unbekannt": Fängt alle weiteren Fehler ab. Alle anderen Fehler, die nicht in die definierten Kategorien fallen.</li> |
| | Server | Gruppiert die Fehler basierend auf dem Server. Diese Gruppierung ist nützlich, um die Fehlerverteilung auf die Server anzuzeigen. Eine ungleichmäßige Verteilung könnte darauf hinweisen, dass sich ein Server in fehlerhaftem Zustand befindet. |
| | Netzwerkadresse | Gruppiert die Fehler basierend auf der Netzwerkadresse der Anforderungen (Intranet oder Extranet). Dies ist nützlich, um die Art der Anforderungen herauszufinden, für die Fehler auftreten. |
| | Anwendung | Gruppiert die Fehler basierend auf der Zielanwendung (vertrauende Seite). Mithilfe dieser Gruppierung können Sie anzeigen, für welche Zielanwendung die meisten Fehler auftreten. |
| Benutzeranzahl: Durchschnittliche Anzahl von aktiven Benutzern im System. | Alle | Stellt die durchschnittliche Anzahl von Benutzern bereit, die den Verbunddienst im ausgewählten Zeitraum verwendet haben. Die Benutzer werden nicht gruppiert. <br>Die durchschnittliche Anzahl richtet sich nach dem ausgewählten Zeitraum. |
| | Anwendung | Gruppiert die durchschnittliche Anzahl von Benutzern basierend auf der Zielanwendung (vertrauende Seite). Über diese Gruppierung können Sie anzeigen, wie viele Benutzer welche Anwendung nutzen. |


## Leistungsüberwachung für AD FS
Die Azure AD Connect Health-Leistungsüberwachung liefert Überwachungsinformationen zu verschiedenen Metriken. Durch Auswahl des Felds "Überwachung" wird ein Blatt mit detaillierten Informationen zu den Metriken geöffnet.


![Azure AD Connect Health-Portal](./media/active-directory-aadconnect-health/perf1.png)


Wenn Sie die Filteroption oben im Blatt auswählen, können Sie eine Filterung nach Server vornehmen, um Metriken zu einzelnen Servern anzuzeigen. Um Metriken zu ändern, klicken Sie einfach mit der rechten Maustaste auf das Überwachungsdiagramm unterhalb des Überwachungsblatts, und klicken Sie dann auf "Diagramm bearbeiten". Im dann geöffneten neuen Blatt können Sie zusätzliche Metriken aus der Dropdownliste auswählen und einen Zeitraum für die Anzeige von Leistungsdaten angeben.

## Berichte für AD FS
Azure AD Connect Health bietet Berichte über die Aktivität und Leistung von AD FS. Mithilfe dieser Berichte können Administratoren Einblicke in Aktivitäten auf ihren AD FS-Servern gewinnen.

### 50 Benutzer mit den meisten fehlgeschlagenen Anmeldungen aufgrund eines falschen Benutzernamens/Kennworts

Einer der häufigsten Gründe für das Fehlschlagen von Authentifizierungsanforderungen bei einem AD FS-Server ist die Verwendung ungültiger Anmeldeinformationen, d. h. eines falschen Benutzernamens oder Kennworts. Solche Fehlern sind meist darauf zurückzuführen, dass Kennwörter zu komplex sind, vergessen wurden oder bei der Eingabe Tippfehler unterlaufen.

Es gibt jedoch weitere Gründe, die zu einer unerwartet hohen Anzahl derartiger Anforderungen bei Ihren AD FS-Servern führen können. Dazu zählen von einer Anwendung zwischengespeicherte abgelaufene Benutzeranmeldeinformationen oder ein böswilliger Benutzer, der versucht, sich mit einer Reihe von bekannten Kennwörtern bei einem Benutzerkonto anzumelden.

Azure AD Connect Health für AD FS bietet einen Bericht über die 50 Benutzer, bei denen Anmeldeversuche am häufigsten aufgrund ungültiger Benutzernamen oder Kennwörter fehlschlagen. Zur Erstellung dieses Berichts werden die von allen AD FS-Servern in den Farmen generierten Überwachungsereignisse verarbeitet.

![Azure AD Connect Health-Portal](./media/active-directory-aadconnect-health-adfs/report1a.png)

In diesem Bericht können Sie ganz einfach auf die folgenden Informationen zugreifen:

- Gesamtanzahl fehlgeschlagener Anforderungen mit falschem Benutzernamen/Kennwort in den letzten 30 Tagen
- Durchschnittliche Anzahl von Benutzern pro Tag, bei denen die Anmeldung wegen eines ungültigen Benutzernamens/Kennworts fehlschlägt

Wenn Sie auf diesen Bereich klicken, gelangen Sie zum Hauptberichtblatt, das zusätzliche Details enthält. Dazu zählen ein Graph mit Trendinformationen zum Einrichten einer Baseline für Anforderungen mit falschem Benutzernamen oder Kennwort und die Liste der 50 Benutzer mit den meisten Anmeldefehlern und der zugehörigen Anzahl fehlgeschlagener Versuche.

Der Graph zeigt die folgenden Informationen:

- Gesamtanzahl fehlgeschlagener Anmeldeversuche aufgrund eines ungültigen Benutzernamens/Kennworts pro Tag
- Gesamtanzahl eindeutiger Benutzer, bei denen die Anmeldung fehlgeschlagen ist, pro Tag.

![Azure AD Connect Health-Portal](./media/active-directory-aadconnect-health-adfs/report2a.png)

Der Bericht enthält die folgenden Informationen:

| Berichtselement | Beschreibung
| ------ | -------- |
|Benutzer-ID| Die verwendete Benutzer-ID. Beachten Sie, dass es sich bei diesem Wert um die vom Benutzer eingegebene ID handelt. Mitunter kommt es vor, dass eine falsche Benutzer-ID verwendet wird.|
|Fehlgeschlagene Anmeldeversuche|Die Gesamtanzahl fehlgeschlagener Anmeldeversuche für die jeweilige Benutzer-ID. Die Tabelle ist nach der höchsten Anzahl fehlgeschlagener Versuche in absteigender Reihenfolge sortiert.|
|Letzter Fehler|Der Zeitstempel des letzten Fehlers.



>[AZURE.NOTE] Der Bericht wird automatisch alle zwei Stunden mit den neuen gesammelten Informationen aktualisiert. Anmeldeversuche innerhalb der letzten zwei Stunden sind daher unter Umständen nicht im Bericht enthalten.



## Verwandte Links

* [Azure AD Connect Health](active-directory-aadconnect-health.md)
* [Installieren des Azure AD Connect Health-Agents](active-directory-aadconnect-health-agent-install.md)
* [Azure AD Connect Health-Vorgänge](active-directory-aadconnect-health-operations.md)
* [Verwenden von Azure AD Connect Health für die Synchronisierung](active-directory-aadconnect-health-sync.md)
* [Verwenden von Azure AD Connect Health mit AD DS](active-directory-aadconnect-health-adds.md)
* [Azure AD Connect Health – FAQ](active-directory-aadconnect-health-faq.md)
* [Azure AD Connect Health: Versionsverlauf](active-directory-aadconnect-health-version-history.md)

<!---HONumber=AcomDC_0928_2016-->