<properties
	pageTitle="Vorgehensweise bei einem Ausfall von Azure Storage | Microsoft Azure"
	description="Vorgehensweise bei einem Ausfall von Azure Storage"
	services="storage"
	documentationCenter=".net"
	authors="robinsh"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="05/17/2016"
	ms.author="robinsh"/>


# Vorgehensweise beim Ausfall von Azure Storage

Bei Microsoft arbeiten wir hart, um sicherzustellen, dass unsere Dienste immer verfügbar sind. Aufgrund höherer Gewalt können ungeplante Dienstausfälle in einer Region oder auch mehreren Regionen auftreten. Damit Sie mit diesen seltenen Vorkommen umgehen können, stellen wir Ihnen den folgenden allgemeinen Leitfaden für Azure Storage-Dienste zur Verfügung.

## So bereiten Sie sich vor 

Es ist wichtig für jeden Kunden, den eigenen Notfallwiederherstellungsplan zu erstellen. Die Wiederherstellung nach einem Speicherausfall erfordert üblicherweise sowohl Personal für den Betrieb als auch automatisierte Vorgehensweise, um wieder funktionsfähige Anwendungen zu erhalten. Weitere Informationen zum Erstellen eines Notfallwiederherstellungsplans finden Sie in der Azure-Dokumentation weiter unten:

-   [Notfallwiederherstellung und hohe Verfügbarkeit für Azure-Anwendungen](https://msdn.microsoft.com/library/azure/dn251004.aspx)

-   [Azure-Geschäftskontinuität – Technische Anleitung](https://msdn.microsoft.com/library/azure/hh873027.aspx)

-   [Azure Site Recovery-Dienste](https://azure.microsoft.com/services/site-recovery/)

-   [Azure Storage-Replikation](storage-redundancy.md)

-   [Azure Backup-Dienst](https://azure.microsoft.com/services/backup/)

## Vorgehensweise zum Erkennen eines Ausfalls von Azure Storage 

Die empfohlene Methode zum Bestimmen des Azure-Dienststatus ist die Anmeldung beim [Azure Service Health Dashboard](https://azure.microsoft.com/status/) (Azure Service-Dashboard zum Dienststatus).

## Vorgehensweise beim Ausfall von Storage

Wenn eine oder mehrere Storage-Dienste in einer Region oder in mehreren Regionen vorübergehend nicht verfügbar sind, stehen Ihnen zwei Optionen zur Auswahl. Wenn Sie sofortigen Zugriff auf Ihre Daten wünschen, sollten Sie Option 2 wählen.

### Option 1: Warten auf die Wiederherstellung

In diesem Fall ist keine weitere Aktion erforderlich. Wir arbeiten intensiv daran, die Verfügbarkeit des Azure-Diensts wiederherzustellen. Sie können den Dienststatus auf unserem [Azure Service Health Dashboard](https://azure.microsoft.com/status/) überwachen.

### Option 2: Kopieren von Daten von der sekundären Region

Wenn Sie [Lesezugriff auf georedundanter Speicher (RA-GRS)](storage-redundancy.md#read-access-geo-redundant-storage) (empfohlen) für Ihre Speicherkonten auswählen, verfügen Sie über Lesezugriff auf Ihre Daten in der sekundären Region. Sie können Tools wie [AzCopy](storage-use-azcopy.md), [Azure PowerShell](storage-powershell-guide-full.md) und die [Azure Data Movement library](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/) verwenden, um Daten aus der sekundären Region in ein anderes Speicherkonto oder in eine nicht betroffene Region zu kopieren. Anschließend können Sie Ihre Anwendungen jeweils für Lese- und Schreibverfügbarkeit auf dieses Speicherkonto verweisen.

## Was bei einem Storage-Failover zu erwarten ist

Wenn Sie [georedundanter Speicher (GRS)](storage-redundancy.md#geo-redundant-storage) oder [Georedundanter Speicher mit Lesezugriff (RA-GRS)](storage-redundancy.md#read-access-geo-redundant-storage) (empfohlen) ausgewählt haben, wird Azure Storage Ihre Daten dauerhaft in zwei Regionen (primär und sekundär) beibehalten. In beiden Regionen unterhält Azure Storage ständig mehrere Replikate Ihrer Daten.

Wenn ein regionaler Notfall die primäre Region betrifft, versuchen wir zunächst, den Dienst in dieser Region wiederherzustellen. Abhängig von der Art des Notfalls und seinen Auswirkungen, ist es uns in seltenen Fällen nicht möglich, die primäre Region wiederherzustellen. Zu diesem Zeitpunkt werden wir ein Geofailover durchführen. Die regionsübergreifende Datenreplikation ist ein asynchroner Prozess, der eine Verzögerung mit sich bringen kann. Es ist daher möglich, dass Änderungen, die noch nicht in die sekundäre Region repliziert wurden, möglicherweise verlorengehen. Sie können die [„Last Sync Time“ (letzte Synchronisierungszeit) Ihres Speicherkontos](https://blogs.msdn.microsoft.com/windowsazurestorage/2013/12/11/windows-azure-storage-redundancy-options-and-read-access-geo-redundant-storage/) abfragen, um Informationen zu Ihrem Replikationsstatus zu erhalten.

Ein paar Punkte in Bezug auf das Geofailoverereignis des Speichers:

-   Das Geofailover des Speichers wird nur durch das Azure Storage-Team ausgelöst, es ist keine Kundenaktion erforderlich.

-   Ihre vorhandenen Speicherdienstendpunkte für Blobs, Tabellen, Warteschlangen und Dateien bleiben nach dem Failover unverändert. Der DNS-Eintrag muss aktualisiert werden, um aus der primären Region in die sekundäre zu wechseln.

-   Vor und während des Geofailovers besitzen Sie keinen Schreibzugriff auf das Speicherkonto aufgrund der Auswirkung des Notfalls. Sie können jedoch weiterhin von der sekundären Region auslesen, wenn das Speicherkonto als RA-GRS konfiguriert wurde.

-   Wenn das Geofailover abgeschlossen wurde und die DNS-Änderungen übermittelt wurden, werden der Lese- und Schreibzugriff auf das Speicherkonto fortgesetzt. Sie können mithilfe der Abfrage [„LastGeoFailoverTime“](https://msdn.microsoft.com/library/azure/ee460802.aspx) weitere Details darüber erhalten, wann für Ihr Speicherkonto zuletzt ein Failover zur sekundären Region durchgeführt wurde.

-   Nach dem Failover ist Ihr Speicherkonto voll funktionsfähig, hat aber den Status „heruntergestuft“, da es tatsächlich in einer eigenständigen Region ohne mögliche Georeplikation gehostet wird. Um dieses Risiko zu verringern, werden wir die ursprüngliche primäre Region wiederherstellen und anschließend ein Geofailback ausführen, um den ursprünglichen Status wiederherzustellen. Wenn die ursprüngliche primäre Region nicht verfügbar ist, wird eine andere sekundäre Region zugeordnet. Weitere Informationen über die Infrastruktur der Georeplikation von Azure Storage finden Sie im Artikel auf dem Storage-Teamblog zu [Windows Azure Storage Redundancy Options and Read Access Geo Redundant Storage](https://blogs.msdn.microsoft.com/windowsazurestorage/2013/12/11/windows-azure-storage-redundancy-options-and-read-access-geo-redundant-storage/) (Windows Azure Storage Redundanzoptionen und Georedundanter Speicher mit Lesezugriff (RA-GRS).

##Bewährte Methoden zum Schützen der Daten

Es gibt einige empfohlenen Vorgehensweisen, um die Speicherdaten in regelmäßigen Abständen zu sichern.

-   VM-Datenträger – Verwenden Sie den [Azure Backup-Dienst](https://azure.microsoft.com/services/backup/), um die von Ihrem virtuellen Computer verwendeten VM-Datenträger zu sichern.

-   Blockblobs – Erstellen Sie eine [Momentaufnahme](https://msdn.microsoft.com/library/azure/hh488361.aspx) von jedem Blockblob, oder kopieren Sie die Blobs in ein anderes Speicherkonto in einer andere Region mithilfe von [AzCopy](storage-use-azcopy.md), [Azure PowerShell](storage-powershell-guide-full.md), oder die [Azure Data Movement library](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/).

-   Tabellen – Verwenden Sie [AzCopy](storage-use-azcopy.md), um die Tabellendaten in ein anderes Speicherkonto in einer anderen Region zu exportieren.

-   Dateien – Verwenden Sie [AzCopy](storage-use-azcopy.md) oder [Azure PowerShell](storage-powershell-guide-full.md), um die Dateien in ein anderes Speicherkonto in einer anderen Region zu kopieren.

<!---HONumber=AcomDC_0518_2016-->