<properties
   pageTitle="Azure AD Connect: Automatisches Upgrade | Microsoft Azure"
   description="Dieses Thema beschreibt das integrierte automatische Upgradefeature in Azure AD Connect Sync."
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="StevenPo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="05/19/2016"
   ms.author="andkjell"/>

# Azure AD Connect: Automatisches Upgrade
Diese Funktion wurde mit Build 1.1.105.0 eingeführt (veröffentlicht im Februar 2016).

## Übersicht
Noch nie konnten Sie so leicht sicherstellen, dass Ihre Azure AD Connect-Installation immer auf dem neuesten Stand ist – mit dem **automatischen Upgradefeature**. Dieses Feature ist standardmäßig für Express-Installationen und DirSync-Upgrades aktiviert. Wenn eine neue Version veröffentlicht wird, erfolgt ein automatisches Upgrade Ihrer Installation.

Das automatische Upgrade ist in folgenden Fällen standardmäßig aktiviert:

- Installationen mit Express-Einstellungen und DirSync-Upgrades.
- Verwendung von SQL Express LocalDB, die bei Express-Einstellungen immer verwendet wird. DirSync mit SQL Express verwendet ebenfalls LocalDB.
- Das AD-Konto ist das MSOL\_-Standardkonto, das mit den Express-Einstellungen und DirSync erstellt wird
- Es sind weniger als 100.000 Objekte im Metaverse enthalten.

Der aktuelle Zustand des automatischen Upgrades kann über das PowerShell-Cmdlet `Get-ADSyncAutoUpgrade` angezeigt werden. Folgende Zustände sind möglich:

Zustand | Kommentar
---- | ----
Aktiviert | Das automatische Upgrade ist aktiviert.
Ausgesetzt | Nur vom System festgelegt. Das System ist nicht mehr zum Erhalten automatischer Upgrades berechtigt.
Deaktiviert | Das automatische Upgrade ist deaktiviert.

Mit `Set-ADSyncAutoUpgrade` können Sie zwischen **aktiviert** und **deaktiviert** wechseln. Nur das System sollte den Zustand **Ausgesetzt** festlegen.

Das automatische Upgrade verwendet Azure AD Connect Health als Upgrade-Infrastruktur. Damit das automatische Upgrade ordnungsgemäß funktioniert, stellen Sie sicher, dass Sie die URLs im Proxy für **Azure AD Connect Health** gemäß der Beschreibung unter [URLs und IP-Adressbereiche von Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) geöffnet haben.

Wenn auf dem Server die **Synchronization Service Manager**-Benutzeroberfläche geöffnet ist, wird das Upgrade angehalten, bis die Benutzeroberfläche geschlossen wird.

## Problembehandlung
Wenn das automatische Upgrade Ihrer Connect-Installation nicht wie erwartet funktioniert, befolgen Sie diese Schritte, um herauszufinden, wo der Fehler liegen könnte.

Ein automatisches Upgrade wird voraussichtlich nicht direkt am Erscheinungstag einer neuen Version versucht. Upgrades unterliegen einer beabsichtigten Zufälligkeit – Sie müssen sich also keine Sorgen machen, wenn Ihre Installation nicht sofort aktualisiert wird.

Wenn Sie der Meinung sind, dass ein Fehler vorliegt, führen Sie zuerst `Get-ADSyncAutoUpgrade` aus, um sicherzustellen, dass das automatische Upgrade aktiviert ist.

Starten Sie die Ereignisanzeige, und sehen Sie sich das Ereignisprotokoll für die **Anwendung** an. Fügen Sie einen Ereignisprotokollfilter für die Quelle **Azure AD Connect Upgrade** und den Ereignis-ID-Bereich **300-399** hinzu. ![Ereignisprotokollfilter für das automatische Upgrade](./media/active-directory-aadconnect-feature-automatic-upgrade/eventlogfilter.png)

Damit zeigen Sie alle Ereignisprotokolle an, die dem Status für das automatische Upgrade zugeordnet sind. ![Ereignisprotokollfilter für das automatische Upgrade](./media/active-directory-aadconnect-feature-automatic-upgrade/eventlogresult.png)

Der Ergebniscode hat ein Präfix mit einer Übersicht über den Status.

Ergebniscodepräfix | Beschreibung
--- | ---
Erfolgreich | Das Upgrade der Installation wurde erfolgreich durchgeführt.
UpgradeAborted | Das Upgrade wurde durch ein temporäres Problem angehalten. Das Upgrade wird erneut versucht, und es wird erwartet, dass es später erfolgreich durchgeführt werden kann.
UpgradeNotSupported | Die Konfiguration des Systems blockiert das automatische Upgrade. Das Upgrade wird erneut versucht, um zu ermitteln, ob sich der Status ändert. Es ist jedoch damit zu rechnen, dass das Upgrade für das System manuell durchgeführt werden muss.

Im Folgenden finden Sie eine Liste der Meldungen, die am häufigsten angezeigt werden. Es werden nicht alle Meldungen aufgeführt, aber aus den Ergebnismeldungen wird das jeweilige Problem klar.

Ergebnismeldung | Beschreibung
--- | ---
**UpgradeAborted** |
UpgradeAbortedCouldNotSetUpgradeMarker | Es konnte nicht in die Registrierung geschrieben werden.
UpgradeAbortedInsufficientDatabasePermissions | Die integrierte Administratorengruppe verfügt nicht über die erforderlichen Berechtigungen für die Datenbank. Führen Sie ein manuelles Upgrade auf die aktuelle Version von Azure AD Connect aus, um dieses Problem zu beheben.
UpgradeAbortedInsufficientDiskSpace | Es ist nicht genügend Speicherplatz vorhanden, um ein Upgrade zu unterstützen.
UpgradeAbortedSecurityGroupsNotPresent | Es konnten nicht alle Sicherheitsgruppen gefunden und aufgelöst werden, die vom Synchronisierungsmodul verwendet werden.
UpgradeAbortedServiceCanNotBeStarted | Der NT-Dienst **Microsoft Azure AD Sync** konnte nicht gestartet werden.
UpgradeAbortedServiceCanNotBeStopped | Der NT-Dienst **Microsoft Azure AD Sync** konnte nicht angehalten werden.
UpgradeAbortedServiceIsNotRunning | Der NT-Dienst **Microsoft Azure AD Sync** wird nicht ausgeführt.
UpgradeAbortedSyncCycleDisabled | Die SyncCycle-Option im [Scheduler](active-directory-aadconnectsync-feature-scheduler.md) wurde deaktiviert.
UpgradeAbortedSyncExeInUse | Die [Synchronization Service Manager-Benutzeroberfläche](active-directory-aadconnectsync-service-manager-ui.md) ist auf dem Server geöffnet.
UpgradeAbortedSyncOrConfigurationInProgress | Der Installations-Assistent wird ausgeführt, oder es wurde eine Synchronisierung außerhalb des Schedulers geplant.
**UpgradeNotSupported** |
UpgradeNotSupportedCustomizedSyncRules | Sie haben der Konfiguration eigene benutzerdefinierte Regeln hinzugefügt.
UpgradeNotSupportedDeviceWritebackEnabled | Sie haben das Feature [Geräterückschreiben](active-directory-aadconnect-feature-device-writeback.md) aktiviert.
UpgradeNotSupportedGroupWritebackEnabled | Sie haben das Feature [Gruppenrückschreiben](active-directory-aadconnect-feature-preview.md#group-writeback) aktiviert.
UpgradeNotSupportedInvalidPersistedState | Die Installation ist keine Express-Einstellung und kein DirSync-Upgrade.
UpgradeNotSupportedMetaverseSizeExceeeded | Es sind mehr als 100.000 Objekte im Metaverse enthalten.
UpgradeNotSupportedMultiForestSetup | Sie stellen Verbindungen mit mehr als einer Gesamtstruktur her. Beim Express-Setup wird nur eine Verbindung mit einer Gesamtstruktur hergestellt.
UpgradeNotSupportedNonLocalDbInstall | Sie verwenden keine SQL Server Express LocalDB-Datenbank.
UpgradeNotSupportedNonMsolAccount | Das [AD-Connectorkonto](active-directory-aadconnect-accounts-permissions.md#active-directory-account) ist nicht mehr das MSOL\_-Standardkonto.
UpgradeNotSupportedStagingModeEnabled | Der Server ist auf den [Stagingmodus](active-directory-aadconnectsync-operations.md#staging-mode) festgelegt.
UpgradeNotSupportedUserWritebackEnabled | Sie haben das Feature [Benutzerrückschreiben](active-directory-aadconnect-feature-preview.md#user-writeback) aktiviert.

## Nächste Schritte
Weitere Informationen zum [Integrieren lokaler Identitäten in Azure Active Directory](active-directory-aadconnect.md).

<!---HONumber=AcomDC_0525_2016-->