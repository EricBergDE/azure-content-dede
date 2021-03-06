<properties
   pageTitle="Erstellen und Ändern einer ExpressRoute-Verbindung über das klassische Bereitstellungsmodell und PowerShell | Microsoft Azure"
   description="In diesem Artikel werden Sie durch die Schritte zum Erstellen und Bereitstellen einer ExpressRoute-Verbindung geführt. Außerdem wird veranschaulicht, wie Sie Status überprüfen, Updates durchführen oder die Verbindung löschen oder deren Bereitstellung aufheben."
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/15/2016"
   ms.author="ganesr"/>

# Erstellen und Ändern einer ExpressRoute-Verbindung

> [AZURE.SELECTOR]
[Azure Portal - Resource Manager](expressroute-howto-circuit-portal-resource-manager.md)
[PowerShell - Resource Manager](expressroute-howto-circuit-arm.md)
[PowerShell - Classic](expressroute-howto-circuit-classic.md)

In diesem Artikel werden Sie durch die Schritte zum Erstellen einer Azure ExpressRoute-Verbindung mithilfe von PowerShell-Cmdlets und des klassischen Bereitstellungsmodells geführt. In diesem Artikel wird auch veranschaulicht, wie Sie den Status prüfen, ein Update durchführen oder eine ExpressRoute-Verbindung löschen oder deren Bereitstellung aufheben.

**Informationen zu Azure-Bereitstellungsmodellen**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]


## Voraussetzungen

- Sie benötigen die neueste Version der Azure PowerShell-Module. Sie können die neuesten PowerShell-Module aus dem Abschnitt „PowerShell“ der [Azure-Downloadseite](https://azure.microsoft.com/downloads/) herunterladen. Eine detaillierte Anleitung zum Konfigurieren des Computers für die Verwendung der PowerShell-Module erhalten Sie auf der Seite [Installieren und Konfigurieren von Azure PowerShell](../powershell-install-configure.md).

- Stellen Sie sicher, dass Sie vor Beginn der Konfiguration die [Voraussetzungen](expressroute-prerequisites.md) und [Workflows](expressroute-workflows.md) gelesen haben.

## Erstellen und Bereitstellen einer ExpressRoute-Verbindung

### 1\. Importieren der PowerShell-Module für ExpressRoute

 Sie müssen die Azure- und ExpressRoute-Module in die PowerShell-Sitzung importieren, um die ExpressRoute-Cmdlets verwenden zu können. Führen Sie zu diesem Zweck die folgenden Befehle aus:

	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

### 2\. Rufen Sie die Liste der unterstützten Anbieter, Standorte und Bandbreiten ab.

Vor dem Erstellen einer ExpressRoute-Verbindung benötigen Sie die Liste der unterstützten Konnektivitätsanbieter, Standorte und Bandbreitenoptionen.

Das PowerShell-Cmdlet `Get-AzureDedicatedCircuitServiceProvider` gibt diese Informationen zurück, die Sie in späteren Schritten verwenden werden:

	Get-AzureDedicatedCircuitServiceProvider

Überprüfen Sie, ob Ihr Konnektivitätsanbieter hier aufgeführt ist. Notieren Sie sich die folgenden Informationen, da Sie diese später beim Erstellen der Verbindung benötigen:

- Name

- PeeringLocations

- BandwidthsOffered

Sie können nun eine ExpressRoute-Verbindung erstellen.

### 3\. Erstellen Sie eine ExpressRoute-Verbindung.

Das folgende Beispiel zeigt, wie Sie eine ExpressRoute-Verbindung mit 200 MBit/s über Equinix im Silicon Valley herstellen können. Wenn Sie einen anderen Anbieter und andere Einstellungen verwenden, ersetzen Sie bei Ihrer Anforderung die entsprechenden Informationen.

>[AZURE.IMPORTANT] Ihre ExpressRoute-Verbindung wird von dem Moment an berechnet, in dem ein Dienstschlüssel ausgegeben wird. Stellen Sie sicher, dass Sie diesen Vorgang ausführen, sobald der Konnektivitätsanbieter dazu bereit ist, die Verbindung bereitzustellen.


Nachfolgend sehen Sie eine Beispielanforderung für einen neuen Dienstschlüssel:

	$Bandwidth = 200
	$CircuitName = "MyTestCircuit"
	$ServiceProvider = "Equinix"
	$Location = "Silicon Valley"

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Standard -BillingType MeteredData

Verwenden Sie das nachstehende Beispiel, falls Sie eine ExpressRoute-Verbindung mit dem Premium-Add-On erstellen möchten. Weitere Details zum Premium-Add-On finden Sie unter [ExpressRoute – Häufig gestellte Fragen](expressroute-faqs.md).

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Premium - BillingType MeteredData


Die Antwort enthält den Dienstschlüssel. Ausführliche Beschreibungen aller Parameter erhalten Sie, wenn Sie Folgendes ausführen:

	get-help new-azurededicatedcircuit -detailed

### 4\. Auflisten aller ExpressRoute-Verbindungen

Um eine Liste mit allen von Ihnen erstellten ExpressRoute-Verbindungen abzurufen, können Sie den Befehl `Get-AzureDedicatedCircuit` ausführen:


	Get-AzureDedicatedCircuit

Die Antwort ähnelt dem folgenden Beispiel:

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : NotProvisioned
	Sku                              : Standard
	Status                           : Enabled

Sie können diese Informationen jederzeit mithilfe des Cmdlets `Get-AzureDedicatedCircuit` abrufen. Wenn Sie den Aufruf ohne Parameter durchführen, werden alle Verbindungen aufgelistet. Der Dienstschlüssel wird im Feld *ServiceKey* aufgeführt.

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : NotProvisioned
	Sku                              : Standard
	Status                           : Enabled

Ausführliche Beschreibungen aller Parameter erhalten Sie, wenn Sie Folgendes ausführen:

	get-help get-azurededicatedcircuit -detailed

### 5\. Senden Sie den Dienstschlüssel zur Bereitstellung an Ihren Konnektivitätsanbieter.


*ServiceProviderProvisioningState* enthält Informationen zum aktuellen Zustand der Bereitstellung auf Dienstanbieterseite. Mit *Status* wird der Zustand auf Microsoft-Seite angegeben. Weitere Informationen zu den Bereitstellungszuständen einer Verbindung finden Sie im Artikel [Workflows](expressroute-workflows.md#expressroute-circuit-provisioning-states).

Wenn Sie eine neue ExpressRoute-Verbindung erstellen, weist die Verbindung folgenden Zustand auf:

	ServiceProviderProvisioningState : NotProvisioned
	Status                           : Enabled


Die Verbindung wird in den folgenden Zustand versetzt, wenn sie vom Konnektivitätsanbieter für Sie aktiviert wird:

	ServiceProviderProvisioningState : Provisioning
	Status                           : Enabled

Eine ExpressRoute-Verbindung muss sich im folgenden Zustand befinden, damit Sie sie verwenden können:

	ServiceProviderProvisioningState : Provisioned
	Status                           : Enabled


### 6\. Überprüfen Sie regelmäßig den Status und Zustand des Verbindungsschlüssels.

So sind Sie informiert, wenn Ihr Anbieter die Verbindung aktiviert hat. Sobald die Verbindung konfiguriert wurde, zeigt *ServiceProviderProvisioningState* wie im folgenden Beispiel den Status *Provisioned* an:

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

### 7\. Erstellen Sie die Routingkonfiguration.

Eine detaillierte Anleitung finden Sie auf der Seite [Routingkonfiguration für ExpressRoute-Verbindung (Erstellen und Ändern von Verbindungspeerings)](expressroute-howto-routing-classic.md).

>[AZURE.IMPORTANT] Diese Anweisungen gelten nur für Verbindungen, die über Dienstanbieter erstellt wurden, die Layer 2-Konnektivitätsdienste anbieten. Wenn Sie einen Dienstanbieter nutzen, der verwaltete Layer 3-Dienste anbietet (meist ein IP VPN, z.B. MPLS), übernimmt Ihr Konnektivitätsanbieter die Konfiguration und Verwaltung des Routings für Sie.

### 8\. Verknüpfen eines virtuellen Netzwerks mit einer ExpressRoute-Verbindung

Verknüpfen Sie anschließend ein virtuelles Netzwerk mit Ihrer ExpressRoute-Verbindung. Lesen Sie die Schritt-für-Schritt-Anweisungen unter [Verknüpfen von virtuellen Netzwerken mit ExpressRoute-Verbindungen](expressroute-howto-linkvnet-classic.md). Wenn Sie ein virtuelles Netzwerk für ExpressRoute mithilfe des klassischen Bereitstellungsmodells erstellen möchten, hilft Ihnen die Anleitung unter [Erstellen eines virtuellen Netzwerks für ExpressRoute](expressroute-howto-vnet-portal-classic.md) weiter.

## Abrufen des Status einer ExpressRoute-Verbindung

Sie können diese Informationen jederzeit mithilfe des Cmdlets `Get-AzureCircuit` abrufen. Wenn Sie den Aufruf ohne Parameter durchführen, werden alle Verbindungen aufgelistet.

	Get-AzureDedicatedCircuit

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

	Bandwidth                        : 1000
	CircuitName                      : MyAsiaCircuit
	Location                         : Singapore
	ServiceKey                       : #################################
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

Sie können Informationen zu einer bestimmten ExpressRoute-Verbindung erhalten, indem Sie den Dienstschlüssel als Parameter an den Aufruf übergeben.

	Get-AzureDedicatedCircuit -ServiceKey "*********************************"

	Bandwidth                        : 200
	CircuitName                      : MyTestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled


Ausführliche Beschreibungen aller Parameter erhalten Sie, wenn Sie Folgendes ausführen:

	get-help get-azurededicatedcircuit -detailed

## Ändern einer ExpressRoute-Verbindung

Sie können bestimmte Eigenschaften einer ExpressRoute-Verbindung ändern, ohne die Konnektivität zu beeinträchtigen.

Sie können folgende Aktionen ausführen, ohne Ausfallzeiten zu verursachen:

- Aktivieren oder deaktivieren Sie ein ExpressRoute Premium-Add-On für Ihre ExpressRoute-Verbindung.
- Erhöhen Sie die Bandbreite Ihrer ExpressRoute-Verbindung. Beachten Sie, dass ein Downgrade der Bandbreite einer Verbindung nicht unterstützt wird.
- Sie können den Abrechnungsplan von „Volumentarif“ zu „Datenflatrate“ ändern. Beachten Sie, dass eine Änderung des Abrechnungsplans von „Datenflatrate“ zu „Volumentarif“ nicht unterstützt wird.
- Sie können die Option *Klassische Vorgänge zulassen* aktivieren und deaktivieren.

Weitere Informationen zu Beschränkungen und Grenzwerten finden Sie unter [ExpressRoute – Häufig gestellte Fragen](expressroute-faqs.md).

### Aktivieren des ExpressRoute Premium-Add-Ons

Für Ihre bereits vorhandene Verbindung können Sie das ExpressRoute Premium-Add-On mit dem folgenden PowerShell-Cmdlet aktivieren:

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Premium

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Premium
	Status                           : Enabled

Für Ihre Verbindung sind nun die Features des ExpressRoute Premium-Add-Ons aktiviert. Beachten Sie, dass die Gebühren für das Premium-Add-On ab der erfolgreichen Ausführung des Befehls erhoben werden.

### Deaktivieren des ExpressRoute Premium-Add-Ons

>[AZURE.IMPORTANT] Bei diesem Vorgang kann ein Fehler auftreten, wenn Sie Ressourcen verwenden, die die zulässige Menge für die Standardverbindung überschreiten.

Beachten Sie Folgendes:

- Sie müssen sicherstellen, dass die Anzahl von virtuellen Netzwerken, die mit der Verbindung verknüpft sind, kleiner als zehn ist, bevor Sie ein Downgrade von Premium auf Standard durchführen. Wenn Sie dies nicht beachten, tritt für die Updateanforderung ein Fehler auf, und Ihnen werden die Premium-Gebühren berechnet.

- Sie müssen die Verknüpfung für alle virtuellen Netzwerke in anderen geopolitischen Regionen aufheben. Wenn Sie dies nicht beachten, tritt für die Updateanforderung ein Fehler auf, und Ihnen werden die Premium-Gebühren berechnet.

- Ihre Routentabelle muss für das private Peering weniger als 4.000 Routen aufweisen. Wenn Ihre Routentabelle mehr als 4.000 Routen umfasst, wird die BGP-Sitzung verworfen. Eine erneute Aktivierung ist erst dann wieder möglich, wenn die Anzahl der angekündigten Präfixe unter 4.000 fällt.

Für Ihre bereits vorhandene Verbindung können Sie das ExpressRoute Premium-Add-On mit dem folgenden PowerShell-Cmdlet deaktivieren:

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Standard

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled



### Aktualisieren der Bandbreite für die ExpressRoute-Verbindung

Informationen zu unterstützten Bandbreitenoptionen für Ihren Anbieter finden Sie unter [ExpressRoute – Häufig gestellte Fragen](expressroute-faqs.md). Sie können jede Größe auswählen, die die Größe der vorhandenen Verbindung übersteigt.

>[AZURE.IMPORTANT] Es ist nicht möglich, die Bandbreite einer ExpressRoute-Verbindung ohne Störungen zu reduzieren. Ein Downgrade der Bandbreite erfordert, dass Sie die Bereitstellung der ExpressRoute-Verbindung aufheben und dann eine neue ExpressRoute-Verbindung bereitstellen.

Sobald Sie sich für die benötigte Größe entschieden haben, können Sie den folgenden Befehl verwenden, um die Größe der Verbindung anzupassen.

	Set-AzureDedicatedCircuitProperties -ServiceKey ********************************* -Bandwidth 1000

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Silicon Valley
	ServiceKey                       : *********************************
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled

Die Größe Ihrer Verbindung wurde bereits auf der Microsoft-Seite angepasst. Sie müssen sich an Ihren Konnektivitätsanbieter wenden, um die entsprechende Aktualisierung der Konfigurationen auf Anbieterseite vornehmen zu lassen. Beachten Sie, dass wir Ihnen ab diesem Zeitpunkt die aktualisierte Bandbreitenoption in Rechnung stellen.

## Löschen und Aufheben der Bereitstellung einer ExpressRoute-Verbindung

Beachten Sie Folgendes:

- Sie müssen die Verknüpfung aller virtuellen Netzwerke mit der ExpressRoute-Verbindung aufheben, damit dieser Vorgang erfolgreich durchgeführt werden kann. Überprüfen Sie, ob noch virtuelle Netzwerke mit der Verbindung verknüpft sind, falls dieser Vorgang nicht erfolgreich ist.

- Wenn der Bereitstellungsstatus des Dienstanbieters für die ExpressRoute-Verbindung aktiviert ist, wechselt der Status von „Enabled“ in *Disabling*. Arbeiten Sie mit Ihrem Service Provider zusammen, um die Bereitstellung der Verbindung auf Anbieterseite aufzuheben. Wir reservieren weiterhin Ressourcen für Sie und stellen Ihnen dies in Rechnung, bis der Service Provider die Aufhebung der Verbindungsbereitstellung abgeschlossen hat und uns eine Benachrichtigung sendet.

- Wenn der Service Provider die Bereitstellung der Verbindung aufgehoben hat (Bereitstellungsstatus des Service Providers lautet *Not provisioned*), bevor Sie das obige Cmdlet ausführen, führen wir die Aufhebung der Verbindungsbereitstellung durch und stellen Ihnen keine Gebühren mehr in Rechnung.

Sie können die ExpressRoute-Verbindung löschen, indem Sie den folgenden Befehl ausführen:

	Remove-AzureDedicatedCircuit -ServiceKey "*********************************"



## Nächste Schritte

Führen Sie nach dem Erstellen Ihrer Verbindung folgende Vorgänge aus:

- [Erstellen und Ändern des Routings für Ihre ExpressRoute-Verbindung](expressroute-howto-routing-classic.md)
- [Verknüpfen Ihres virtuelles Netzwerks mit Ihrer ExpressRoute-Verbindung](expressroute-howto-linkvnet-classic.md)

<!---HONumber=AcomDC_0518_2016-->