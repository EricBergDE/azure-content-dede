<properties 
   pageTitle="StorSimple Manager-Dienstverwaltung | Microsoft Azure"
   description="Erfahren Sie, wie Sie Ihr StorSimple-Gerät mithilfe des StorSimple Manager-Diensts im klassischen Azure-Portal verwalten."
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carmonm"
   editor="" />
<tags 
   ms.service="storsimple"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/21/2016"
   ms.author="alkohli" />

# Verwalten Ihres StorSimple-Geräts mithilfe des StorSimple Manager-Diensts

## Übersicht

Dieser Artikel beschreibt die Benutzeroberfläche des StorSimple Manager-Diensts und erläutert, wie Sie eine Verbindung herstellen und welche Optionen verfügbar sind. Darüber hinaus enthält der Artikel Links zu den Workflows, die über diese Benutzeroberfläche ausgeführt werden können. Die Anleitung gilt sowohl für das physische als auch für das virtuelle StorSimple-Gerät.

In diesem Artikel lernen Sie Folgendes:

- Herstellen einer Verbindung mit dem StorSimple Manager-Dienst
- Navigieren auf der Benutzeroberfläche von StorSimple Manager
- Verwalten Ihres StorSimple-Geräts über den StorSimple Manager-Dienst


## Herstellen einer Verbindung mit dem StorSimple Manager-Dienst

Der StorSimple Manager-Dienst wird in Microsoft Azure ausgeführt. Er stellt Verbindungen mit mehreren StorSimple-Geräten her. Diese Geräte werden über ein zentrales browserbasiertes klassisches Microsoft Azure-Portal verwaltet. Gehen Sie wie folgt vor, um eine Verbindung mit dem StorSimple Manager-Dienst herzustellen:

#### So stellen Sie eine Verbindung mit dem Dienst her

1. Rufen Sie [https://manage.windowsazure.com/](https://manage.windowsazure.com/) auf.

1. Melden Sie sich mit den Anmeldeinformationen Ihres Microsoft-Kontos beim klassischen Microsoft Azure-Portal (rechts oben) an.

1. Führen Sie im linken Navigationsbereich einen Bildlauf nach unten durch, um auf den StorSimple Manager-Dienst zuzugreifen.


## Navigieren auf der Benutzeroberfläche des StorSimple Manager-Diensts

Die folgende Tabelle gibt Aufschluss über die Navigationshierarchie der Benutzeroberfläche des StorSimple Manager-Diensts.

- Über die Zielseite von **StorSimple Manager** gelangen Sie zu den Seiten auf Dienstebene, die alle Geräte in einem Dienst betreffen.

- Über die Seite **Geräte** gelangen zu den Seiten auf Geräteebene, die ein bestimmtes Gerät betreffen.

- Über die Seite **Volumecontainer** gelangen Sie zur Volumeseite mit allen dem Gerät zugeordneten Volumes.


#### Navigationshierarchie des StorSimple Manager-Diensts

|Zielseite|Seiten auf Dienstebene|Seiten auf Geräteebene|Seiten auf Geräteebene|
|---|---|---|---|
|StorSimple Manager-Dienst|Dienstdashboard|Gerätedashboard||
||Geräte →|Monitor|
||Sicherungskatalog|Volumecontainer→|Volumes|
||Konfigurieren (Dienst)|Sicherungsrichtlinien||
||Aufträge|Konfigurieren (Gerät)|
||Warnungen|Wartung|

![Video verfügbar](./media/storsimple-manager-service-administration/Video_icon.png) **Video verfügbar**

Um sich ein Video anzusehen, das Sie durch die Benutzeroberfläche des StorSimple Manager-Diensts führt, klicken Sie [hier](https://azure.microsoft.com/documentation/videos/storsimple-manager-service-overview/).

## Verwalten des StorSimple-Geräts mithilfe des StorSimple Manager-Diensts

Die folgende Tabelle zeigt eine Zusammenfassung der allgemeinen Verwaltungsaufgaben und komplexen Workflows, die auf der Benutzeroberfläche des StorSimple Manager-Diensts ausgeführt werden können. Diese Aufgaben sind nach der Benutzeroberflächenseite strukturiert, auf der sie jeweils initiiert werden.

Klicken Sie auf die Verfahren in der Tabelle, um weitere Informationen zu den einzelnen Workflows zu erhalten.

#### StorSimple Manager-Workflows

|Gewünschte Aktion|Seite auf der Benutzeroberfläche|Verfahren|
|---|---|---|
|Dienst erstellen</br>Dienst löschen</br>Dienstregistrierungsschlüssel abrufen</br>Dienstregistrierungsschlüssel erneut generieren|StorSimple Manager-Dienst|[Bereitstellen eines StorSimple Manager-Diensts](storsimple-manage-service.md)
|Dienstdaten-Verschlüsselungsschlüssel ändern</br>Vorgangsprotokolle anzeigen|StorSimple Manager-Dienst → Dashboard|[Verwenden des Dashboards des StorSimple Manager-Diensts](storsimple-service-dashboard.md)|
|Gerät deaktivieren</br>Gerät löschen|StorSimple Manager-Dienst → Geräte|[Deaktivieren oder Löschen eines Geräts](storsimple-deactivate-and-delete-device.md)|
|Über Notfallwiederherstellung und Gerätefailover informieren</br>Failover auf ein physisches Gerät ausführen</br>Failover auf ein virtuelles Gerät ausführen</br>BCDR (Business Continuity Disaster Recovery) ausführen|StorSimple Manager-Dienst → Geräte|[Ausführen eines Failovers und einer Notfallwiederherstellung für das StorSimple-Gerät](storsimple-device-failover-disaster-recovery.md)|
|Liste mit Sicherungen für ein Volume anzeigen</br>Sicherungssatz auswählen</br>Sicherungssatz löschen|StorSimple Manager-Dienst → Sicherungskatalog|[Verwalten von Sicherungen](storsimple-manage-backup-catalog.md)|
|Volume klonen|StorSimple Manager-Dienst → Sicherungskatalog|[Klonen eines Volumes](storsimple-clone-volume.md)|
|Sicherungssatz wiederherstellen|StorSimple Manager-Dienst → Sicherungskatalog|[Wiederherstellen eines Sicherungssatzes](storsimple-restore-from-backup-set.md)|
|Über Speicherkonten informieren</br>Speicherkonto hinzufügen</br>Speicherkonto bearbeiten</br>Speicherkonto löschen</br>Schlüsselrotation für Speicherkonten ausführen|StorSimple Manager-Dienst → Konfigurieren|[Verwalten von Speicherkonten](storsimple-manage-storage-accounts.md)|
|Über Bandbreitenvorlagen informieren</br>Bandbreitenvorlage hinzufügen</br>Bandbreitenvorlage bearbeiten</br>Bandbreitenvorlage löschen</br>Standard-Bandbreitenvorlage verwenden</br>Ganztägige Bandbreitenvorlage mit definiertem Startzeitpunkt erstellen|StorSimple Manager-Dienst → Konfigurieren|[Verwalten von Bandbreitenvorlagen](storsimple-manage-bandwidth-templates.md)|
|Über Zugriffssteuerungsdatensätze informieren</br>Zugriffssteuerungsdatensatz erstellen</br>Zugriffssteuerungsdatensatz bearbeiten</br>Zugriffssteuerungsdatensatz löschen|StorSimple Manager-Dienst → Konfigurieren|[Verwalten von Zugriffssteuerungsdatensätzen](storsimple-manage-acrs.md)|
|Auftragsdetails anzeigen</br>Auftrag abbrechen|StorSimple Manager-Dienst → Aufträge|[Verwalten von Aufträgen](storsimple-manage-jobs.md)
|Warnungsbenachrichtigungen empfangen</br>Benachrichtigungen verwalten</br>Warnungen prüfen|StorSimple Manager-Dienst → Warnungen|[Anzeigen und Verwalten von StorSimple-Warnungen](storsimple-manage-alerts.md)
|Verbundene Initiatoren anzeigen</br>Seriennummer des Geräts ermitteln</br>Ziel-IQN ermitteln|StorSimple Manager-Dienst → Geräte → Dashboard|[Verwenden des StorSimple-Gerätedashboards](storsimple-device-dashboard.md)|
|Überwachungsdiagramme erstellen|StorSimple Manager-Dienst → Geräte → Überwachen|[Überwachen von StorSimple-Geräten](storsimple-monitor-device.md)|
|Volumecontainer hinzufügen</br>Volumecontainer ändern</br>Volumecontainer löschen|StorSimple Manager-Dienst → Geräte → Volumecontainer|[Verwalten von Volumecontainern](storsimple-manage-volume-containers.md)|
|Volume hinzufügen</br>Volume ändern</br>Volume offline schalten</br>Volume löschen</br>Volume überwachen|StorSimple Manager-Dienst → Geräte → Volumecontainer → Volumes|[Verwalten von Volumes](storsimple-manage-volumes.md)|
|Geräteeinstellungen ändern</br>Zeiteinstellungen ändern</br>Einstellungen für „DNS.md“ ändern</br>Netzwerkschnittstellen konfigurieren|StorSimple Manager-Dienst → Geräte → Konfigurieren|[Ändern der Gerätekonfiguration für Ihr StorSimple-Gerät](storsimple-modify-device-config.md)|
|Webproxyeinstellungen anzeigen|StorSimple Manager-Dienst → Geräte → Konfigurieren|[Konfigurieren des Webproxys für Ihr Gerät](storsimple-configure-web-proxy.md)|
|Geräteadministratorkennwort ändern</br>Kennwort für StorSimple Snapshot Manager ändern|StorSimple Manager-Dienst → Geräte → Konfigurieren|[Ändern von StorSimple-Kennwörtern](storsimple-change-passwords.md)|
|Remoteverwaltung konfigurieren|StorSimple Manager-Dienst → Geräte → Konfigurieren|[Herstellen einer Remoteverbindung mit dem StorSimple-Gerät](storsimple-remote-connect.md)|
|Warnungseinstellungen konfigurieren|StorSimple Manager-Dienst → Geräte → Konfigurieren|[Anzeigen und Verwalten von StorSimple-Warnungen](storsimple-manage-alerts.md)|
|CHAP für Ihr StorSimple-Gerät konfigurieren|StorSimple Manager-Dienst → Geräte → Konfigurieren|[Konfigurieren von CHAP für Ihr StorSimple-Gerät](storsimple-configure-chap.md)|
|Sicherungsrichtlinie hinzufügen</br>Zeitplan hinzufügen oder ändern</br>Sicherungsrichtlinie löschen</br>Manuelle Sicherung ausführen</br>Benutzerdefinierte Sicherungsrichtlinie mit mehreren Volumes und Zeitplänen erstellen|StorSimple Manager-Dienst → Geräte → Sicherungsrichtlinien|[Verwalten von Sicherungsrichtlinien](storsimple-manage-backup-policies.md)|
|Gerätecontroller beenden</br>Gerätecontroller neu starten</br>Gerätecontroller herunterfahren</br>Gerät auf Werkseinstellungen zurücksetzen</br>(nur bei lokalen Geräten)|StorSimple Manager-Dienst → Geräte → Wartung|[Verwalten des StorSimple-Gerätecontrollers](storsimple-manage-device-controller.md)|
|Über StorSimple-Hardwarekomponenten informieren</br>Hardwarestatus überwachen</br>(nur bei lokalen Geräten)|StorSimple Manager-Dienst → Geräte → Wartung|[Überwachen der Hardwarekomponenten](storsimple-monitor-hardware-status.md)|
|Unterstützungspaket erstellen|StorSimple Manager-Dienst → Geräte → Wartung|[Erstellen und Verwalten eines Unterstützungspakets](storsimple-create-manage-support-package.md)|
|Softwareupdates installieren|StorSimple Manager-Dienst → Geräte → Wartung|[Aktualisieren Ihres Geräts](storsimple-update-device.md)|


##Nächste Schritte
Informationen zu Problemen beim alltäglichen Betrieb Ihres StorSimple-Geräts oder zu Problemen mit dessen Hardwarekomponenten finden Sie hier:

- [Problembehandlung bei einem betriebsbereiten Gerät](storsimple-troubleshoot-operational-device.md)
- [StorSimple-Überwachungsindikatoren](storsimple-monitoring-indicators.md)

Falls sich die Probleme nicht beheben lassen und Sie eine Serviceanfrage erstellen müssen, finden Sie unter [Microsoft-Support kontaktieren](storsimple-contact-microsoft-support.md) weitere Informationen.

<!---HONumber=AcomDC_0921_2016-->