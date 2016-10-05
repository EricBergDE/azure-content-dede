<properties 
   pageTitle="StorSimple-Failover und -Notfallwiederherstellung | Microsoft Azure"
   description="Erfahren Sie, wie Sie ein Failover des StorSimple-Geräts auf sich selbst, auf ein anderes physisches Gerät oder auf ein virtuelles Gerät durchführen."
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
   ms.date="09/16/2016"
   ms.author="alkohli" />

# Ausführen eines Failovers und einer Notfallwiederherstellung für das StorSimple-Gerät

## Übersicht

In diesem Tutorial werden die erforderlichen Schritte für das Ausführen eines Failovers auf einem StorSimple-Gerät bei einem Notfall beschrieben. Ein Failover ermöglicht das Migrieren Ihrer Daten von einem Quellgerät im Datencenter auf ein anderes physisches oder sogar ein virtuelles Gerät an dem gleichen oder einem anderen geografischen Standort.

Die Notfallwiederherstellung wird über das Feature „Gerätefailover“ koordiniert und auf der Seite **Geräte** ausgelöst. Auf dieser Seite werden alle StorSimple-Geräte aufgeführt, die mit dem StorSimple-Manager-Dienst verbunden sind. Für jedes Gerät werden Anzeigename, Status, bereitgestellte und maximale Kapazität, Typ und Modell angezeigt.

![Seite "Geräte"](./media/storsimple-device-failover-disaster-recovery/IC740972.png)

Die Anweisungen in diesem Tutorial gelten für physische und virtuelle StorSimple-Geräte sowie für alle Softwareversionen.



## Notfallwiederherstellung und Gerätefailover

Bei einer Notfallwiederherstellung funktioniert das primäre Gerät nicht mehr. In diesem Fall können Sie die dem ausgefallenen Gerät zugeordneten Clouddaten auf ein anderes Gerät verschieben und dabei das primäre Gerät als *Quelle* und ein anderes Gerät als *Ziel* angeben. Sie können einen oder mehrere Volumecontainer für das Migrieren auf das Zielgerät auswählen. Dieser Vorgang wird als *Failover* bezeichnet.

Während des Failovers wechseln die Volumecontainer vom Quellgerät den Eigentümer und werden auf das Zielgerät übertragen. Sobald der Besitzer der Volumecontainer geändert wurde, werden diese vom Quellgerät gelöscht. Nach dem Löschvorgang kann für das Zielgerät ein Failback erfolgen.

Nach einer Notfallwiederherstellung werden die Daten in der Regel mithilfe der letzten Sicherung auf dem Zielgerät wiederhergestellt. Falls allerdings für das gleiche Volume mehrere Sicherungsrichtlinien vorhanden sind, wird die Sicherungsrichtlinie mit der größten Anzahl von Volumes ausgewählt, und die Daten werden auf dem Zielgerät auf der Grundlage der neuesten Sicherung für diese Richtlinie wiederhergestellt.

Ein Beispiel: Angenommen, es sind zwei Sicherungsrichtlinien – *defaultPol* (Standardrichtlinie) und *customPol* (benutzerdefinierte Richtlinie) – mit folgenden Details vorhanden:

- *defaultPol* : ein Volume (*vol1*). Wird täglich ab 22:30 Uhr ausgeführt.
- *customPol* : vier Volumes (*vol1*, *vol2*, *vol3*, *vol4*). Wird täglich ab 22:00 Uhr ausgeführt.

In diesem Fall wird *customPol* verwendet, da die Richtlinie mehr Volumes umfasst und Absturzkonsistenz Priorität hat. Die Daten werden auf der Grundlage der neuesten Sicherung für diese Richtlinie wiederhergestellt.


## Überlegungen zum Gerätefailover

Bei einem Notfall können Sie als Failoverziel für Ihr StorSimple-Gerät Folgendes auswählen:

- Ein physisches Gerät
- Das Gerät selbst
- Ein virtuelles Gerät

Bedenken Sie bei jedem Gerätefailover Folgendes:

- Als Voraussetzung für die Notfallwiederherstellung müssen alle Volumes innerhalb der Volumecontainer offline sein, und den Volumecontainern muss eine Cloudmomentaufnahme zugeordnet sein.
- Die verfügbaren Zielgeräte für die Notfallwiederherstellung sind Geräte, die über ausreichenden Speicherplatz zur Aufnahme der ausgewählten Volumecontainer verfügen.
- Geräte, die mit dem Dienst verbunden sind, aber das Speicherplatzkriterium nicht erfüllen, stehen als Zielgeräte nicht zur Verfügung.
- Nach einer Notfallwiederherstellung kann für eine bestimmte Dauer die Datenzugriffsleistung erheblich beeinträchtigt sein, da das Gerät auf die Daten aus der Cloud zugreifen und sie lokal speichern muss.

#### Gerätefailover in allen Softwareversionen

Der StorSimple Manager-Dienst verfügt in einer Bereitstellung möglicherweise über mehrere physische und virtuelle Geräte, auf denen unterschiedliche Softwareversionen ausgeführt werden. Je nach Version der Software können sich die Volumetypen auf den Geräten auch unterscheiden. So kann beispielsweise ein Gerät mit Update 2 oder höher über lokal fixierte und mehrstufige Volumes verfügen (mit Archivierung als Teilmenge der mehrstufigen Konfiguration). Ein Gerät ohne Update 2 kann hingegen über mehrstufige und Archivierungsvolumes verfügen.

Bestimmen Sie anhand der folgenden Tabelle, ob ein Failover zu einem anderen Gerät mit anderer Softwareversion möglich ist und wie sich die Volumetypen während der Notfallwiederherstellung verhalten.

| Failoverquelle | Für physische Geräte zulässig | Für virtuelle Geräte zulässig |
|----------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------|
| Update 2 zu Versionen vor Update 1 (Release, 0.1, 0.2, 0.3) | Nein | Nein |
| Update 2 zu Update 1 (1, 1.1, 1.2) | Ja. <br></br>Bei lokal fixierten oder mehrstufigen Volumes oder einer Kombination aus beiden wird immer ein Failover der Volumes als mehrstufig ausgeführt. | Ja.<br></br>Bei lokal fixierten Volumes wird ein Failover als mehrstufiges Volume ausgeführt. |
| Update 2 zu Update 2 (spätere Version) | Ja.<br></br>Bei lokal fixierten oder mehrstufigen Volumes oder einer Kombination der beiden wird ein Failover für die Volumes immer zum ursprünglichen Volumetyp ausgeführt – mehrstufig als mehrstufig und lokal fixiert als lokal fixiert. | Ja.<br></br>Bei lokal fixierten Volumes wird ein Failover als mehrstufiges Volume ausgeführt. |


#### Teilfailover in allen Softwareversionen

Orientieren Sie sich an den folgenden Anweisungen, wenn Sie ein Teilfailover mit einem StorSimple-Quellgerät mit einer Version vor Update 1 auf ein Ziel mit Update 1 oder einer späteren Version durchführen möchten.


| Teilfailover von | Für physische Geräte zulässig | Für virtuelle Geräte zulässig |
|----------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------|
|Version vor Update 1 (Version 0.1, 0.2, 0.3) zu Update 1 oder später | Ja, siehe unten stehenden Tipp für bewährte Methode. | Ja, siehe unten stehenden Tipp für bewährte Methode. |


>[AZURE.TIP] In Update 1 und späteren Versionen wurde eine Änderung an den Cloudmetadaten und am Datenformat vorgenommen. Daher wird von einem Teilfailover von einer Version vor Update 1 auf Update 1 oder spätere Versionen abgeraten. Wenn Sie ein Teilfailover durchführen möchten, empfehlen wir, dass Sie zunächst Update 1 oder eine spätere Version auf beiden Geräten (Quellgerät und Zielgerät) anwenden und anschließend das Failover durchführen.

## Failover auf ein anderes physisches Gerät

Führen Sie die folgenden Schritte aus, um Ihr Gerät auf einem physischen Zielgerät wiederherzustellen.

1. Stellen Sie sicher, dass der Volumecontainer für das Failover über zugeordnete Cloudmomentaufnahmen verfügt.

1. Klicken Sie auf der Seite **Geräte** auf die Registerkarte **Volumecontainer**.

1. Wählen Sie einen Volumecontainer für das Failover auf einem anderen Gerät aus. Klicken Sie auf den Volumecontainer, um die Liste der Volumes innerhalb dieses Containers anzuzeigen. Wählen Sie ein Volume aus, und klicken Sie auf **Offline schalten**, um das Volume offline zu schalten. Wiederholen Sie diesen Vorgang für alle Volumes im Volumecontainer.

1. Wiederholen Sie den vorherigen Schritt für alle Volumecontainer, für die auf dem anderen Gerät ein Failover durchgeführt werden soll.

1. Klicken Sie auf der Seite **Geräte** auf **Failover**.

1. Führen Sie im angezeigten Assistenten unter **Choose volume container to fail over** (Volumecontainer für Failover auswählen) folgende Schritte durch:

	1. Wählen Sie in der Liste der Volumecontainer die Volumecontainer aus, für die ein Failover durchgeführt werden soll. **Nur die Volumecontainer mit zugeordneten Cloudmomentaufnahmen und Offlinevolumes werden angezeigt.**

	1. Wählen Sie unter **Choose a target device for the volumes in the selected containers** ein Zielgerät aus der Dropdownliste der verfügbaren Geräte aus. Nur die Geräte, die über die verfügbare Kapazität verfügen, werden in der Dropdownliste angezeigt.

	1. Überprüfen Sie abschließend die Failover-Einstellungen unter **Failover bestätigen**. Klicken Sie auf das Häkchensymbol ![Häkchensymbol](./media/storsimple-device-failover-disaster-recovery/IC740895.png).

1. Ein Failoverauftrag wird erstellt. Dieser kann über die Seite **Aufträge** überwacht werden. Enthält der Volumecontainer, für den Sie ein Failover durchgeführt haben, lokale Volumes, werden einzelne Wiederherstellungsaufträge für jedes lokale Volume (nicht für mehrstufige Volumes) im Container angezeigt. Diese Wiederherstellungsaufträge können einige Zeit in Anspruch nehmen. Der Failoverauftrag wird wahrscheinlich vorher abgeschlossen. Beachten Sie, dass diese Volumes erst nach Abschluss der Wiederherstellungsaufträge über lokale Garantien verfügen. Wechseln Sie nach Abschluss des Failovers zur Seite **Geräte**.

	1. Wählen Sie das Gerät aus, das Sie als Zielgerät für den Failovervorgang verwendet haben.

	1. Wechseln Sie zur Seite **Volumecontainer**. Es sollten alle Volumecontainer und die Volumes des alten Geräts aufgeführt werden.

## Failover mit einem einzelnen Gerät

Führen Sie die folgenden Schritte aus, wenn Sie nur über ein einzelnes Gerät verfügen und ein Failover durchführen müssen.

1. Erstellen Sie Cloudmomentaufnahmen aller Volumes auf Ihrem Gerät.

1. Setzen Sie das Gerät auf die Werkseinstellungen zurück. Befolgen Sie die ausführlichen Anleitungen unter [Zurücksetzen des Geräts auf die Standardwerkseinstellungen](storsimple-manage-device-controller.md#reset-the-device-to-factory-default-settings).

1. Konfigurieren Sie das Gerät, und registrieren Sie es erneut im StorSimple-Manager-Dienst.

1. Auf der Seite **Geräte** sollte das alte Gerät als **Offline** angezeigt werden. Das neu registrierte Gerät sollte als **Online** angezeigt werden.

1. Führen Sie für das neue Gerät zunächst die Mindestkonfiguration durch.
												
	>[AZURE.IMPORTANT] **Wenn nicht zuerst die Mindestkonfiguration abgeschlossen wird, schlägt die Notfallwiederherstellung aufgrund eines Fehlers in der aktuellen Implementierung fehl. Dieses Verhalten wird in einer späteren Version behoben werden.**

1. Wählen Sie das alte Gerät (Status "Offline") aus, und klicken Sie auf **Failover**. Führen Sie im daraufhin angezeigten Assistenten ein Failover für dieses Gerät durch, und geben Sie das Zielgerät als neu registriertes Gerät an. Ausführliche Anweisungen finden Sie unter [Failover auf ein anderes physisches Gerät](#fail-over-to-another-physical-device).

1. Es wird ein Gerätewiederherstellungsauftrag erstellt, den Sie auf der Seite **Aufträge** überwachen können.

1. Greifen Sie, nachdem der Auftrag erfolgreich abgeschlossen wurde, auf das neue Gerät zu, und navigieren Sie zur Seite **Volumecontainer**. Alle Volumecontainer vom alten Gerät sollten auf das neue Gerät migriert worden sein.

## Failover auf ein virtuelles StorSimple-Gerät

Sie müssen zunächst ein virtuelles StorSimple-Gerät erstellen und konfigurieren, bevor Sie dieses Verfahren ausführen können. Bei der Ausführung von Update 2 sollten Sie für die Notfallwiederherstellung ein virtuelles 8020-Gerät mit 64 TB und Storage Premium verwenden.
 
Führen Sie die folgenden Schritte aus, um Ihr Gerät auf einem virtuellen StorSimple-Zielgerät wiederherzustellen.

1. Stellen Sie sicher, dass der Volumecontainer für das Failover über zugeordnete Cloudmomentaufnahmen verfügt.

1. Klicken Sie auf der Seite **Geräte** auf die Registerkarte **Volumecontainer**.

1. Wählen Sie einen Volumecontainer für das Failover auf einem anderen Gerät aus. Klicken Sie auf den Volumecontainer, um die Liste der Volumes innerhalb dieses Containers anzuzeigen. Wählen Sie ein Volume aus, und klicken Sie auf **Offline schalten**, um das Volume offline zu schalten. Wiederholen Sie diesen Vorgang für alle Volumes im Volumecontainer.

1. Wiederholen Sie den vorherigen Schritt für alle Volumecontainer, für die auf dem anderen Gerät ein Failover durchgeführt werden soll.

1. Klicken Sie auf der Seite **Geräte** auf **Failover**.

1. Führen Sie im angezeigten Assistenten unter **Choose volume container to failover** folgende Schritte durch:
													
	a. Wählen Sie in der Liste der Volumecontainer die Volumecontainer aus, für die ein Failover durchgeführt werden soll.

	**Nur die Volumecontainer mit zugeordneten Cloudmomentaufnahmen und Offlinevolumes werden angezeigt.**

	b. Wählen Sie unter **Choose a target device for the volumes in the selected containers** das virtuelle StorSimple-Gerät aus der Dropdownliste der verfügbaren Geräte aus. **Nur die Geräte, die über ausreichende Kapazität verfügen, werden in der Dropdownliste angezeigt.**
	

1. Überprüfen Sie abschließend die Failover-Einstellungen unter **Failover bestätigen**. Klicken Sie auf das Häkchensymbol ![Häkchensymbol](./media/storsimple-device-failover-disaster-recovery/IC740895.png).

1. Wechseln Sie nach Abschluss des Failovers zur Seite **Geräte**.
													
	a. Wählen Sie das virtuelle StorSimple-Gerät aus, das Sie als Zielgerät für das Failover verwendet haben.
	
	b. Wechseln Sie zur Seite **Volumecontainer**. Es sollten jetzt alle Volumecontainer und die Volumes des alten Geräts aufgeführt werden.

![Video verfügbar](./media/storsimple-device-failover-disaster-recovery/Video_icon.png) **Video verfügbar**

Sie können sich [hier](https://azure.microsoft.com/documentation/videos/storsimple-and-disaster-recovery/) ein Video anschauen, in dem das Wiederherstellen eines fehlgeschlagenen physischen Geräts auf ein virtuelles Gerät in der Cloud demonstriert wird.


## Failback

Für Update 3 oder höher unterstützt StorSimple auch das Failback. Nach dem Failover erfolgen die folgenden Aktionen:

- Die Volumecontainer, für die ein Failover durchgeführt wurde, werden vom Quellgerät bereinigt.

- Auf dem Quellgerät wird ein Hintergrundauftrag pro Volumecontainer (mit Failover) initiiert. Wenn Sie versuchen, ein Failback auszuführen, während der Auftrag ausgeführt wird, erhalten Sie eine entsprechende Benachrichtigung. Sie müssen warten, bis der Auftrag abgeschlossen ist, bevor Sie das Failback starten können.

	Wie lange das Löschen der Volumecontainer dauert, richtet sich nach verschiedenen Faktoren, wie z.B. der Datenmenge, dem Alter der Daten, der Anzahl von Sicherungen und der für den Vorgang verfügbaren Netzwerkbandbreite. Wenn Sie Testfailover/-failbacks planen, wird empfohlen, Volumecontainer mit weniger Daten (GBs) zu testen. In den meisten Fällen können Sie das Failback 24 Stunden nach Abschluss des Failovers starten.




## Häufig gestellte Fragen

F: **Was geschieht, wenn die Notfallwiederherstellung misslingt oder nur teilweise erfolgreich war?**

A: Wenn die Notfallwiederherstellung misslingt, sollten Sie sie wiederholen. Beim zweiten Mal weiß die Notfallwiederherstellung, was beim ersten Mal alles erreicht und wo der Prozess angehalten wurde. Die Notfallwiederherstellung wird an dieser Stelle fortgesetzt.

F: **Kann ich ein Gerät löschen, während das Gerätefailover erfolgt?**

A: Ein Gerät kann nicht gelöscht werden, während eine Notfallwiederherstellung ausgeführt wird. Sie können Ihr Gerät erst nach erfolgter Notfallwiederherstellung löschen.

F: **Wann wird die Garbage Collection auf dem Quellgerät gestartet, damit die lokalen Daten auf dem Quellgerät gelöscht werden?**

A: Die Garbage Collection wird auf dem Quellgerät erst aktiviert, nachdem das Gerät vollständig bereinigt wurde. Die Bereinigung enthält bereinigte Objekte, für die ein Failover vom Quellgerät erfolgt ist, z.B. Volumes, Sicherungsobjekte (keine Daten), Volumecontainer und Richtlinien.

F: **Was geschieht, wenn der den Volumecontainern zugeordnete Löschauftrag auf dem Quellgerät misslingt?**

A: Wenn der Löschauftrag misslingt, müssen Sie das Löschen der Volumecontainer manuell auslösen. Wählen Sie auf der Seite **Geräte** Ihr Quellgerät aus, und klicken Sie auf **Volumecontainer**. Wählen Sie die Volumecontainer aus, für die Sie ein Failover ausgeführt haben, und klicken Sie unten auf der Seite auf **Löschen**. Sobald Sie alle Volumecontainer mit erfolgtem Failover vom Quellgerät gelöscht haben, können Sie das Failback starten.

## Business Continuity Disaster Recovery (BCDR)

Ein Business Continuity Disaster Recovery (BCDR)-Szenario liegt vor, wenn das gesamte Azure-Rechenzentrum nicht mehr funktioniert. Dies kann sich auf den StorSimple Manager-Dienst und die zugehörigen StorSimple-Geräte auswirken.

Wenn StorSimple-Geräte direkt vor einem Notfall registriert wurden, müssen diese StorSimple-Geräte möglicherweise auf die Werkseinstellungen zurückgesetzt werden. Nach dem Notfall wird das StorSimple-Gerät als offline angezeigt. Das StorSimple-Gerät muss aus dem Portal gelöscht und auf die Werkseinstellungen zurückgesetzt werden. Anschließend muss es neu registriert werden.


## Nächste Schritte

- Nach einem Failover müssen Sie unter Umständen Ihr [StorSimple-Gerät deaktivieren oder löschen](storsimple-deactivate-and-delete-device.md).

- Weitere Informationen zum Verwenden des StorSimple Manager-Diensts finden Sie unter [Verwalten Ihres StorSimple-Geräts mithilfe des StorSimple Manager-Diensts](storsimple-manager-service-administration.md).
 

<!---HONumber=AcomDC_0921_2016-->