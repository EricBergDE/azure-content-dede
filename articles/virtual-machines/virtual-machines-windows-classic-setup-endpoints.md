<properties
	pageTitle="Einrichten von Endpunkten auf einer klassischen Windows-VM | Microsoft Azure"
	description="Erfahren Sie, wie Endpunkte im klassischen Azure-Portal eingerichtet werden, um die Kommunikation mit einem virtuellen Windows-Computer in Azure zu ermöglichen."
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/19/2016"
	ms.author="cynthn"/>

# Gewusst wie: Einrichten von Endpunkten auf einem klassischen virtuellen Computer in Azure


Alle virtuellen Windows-Computer, die Sie mit dem klassischen Bereitstellungsmodell in Azure erstellen, können automatisch über einen privaten Netzwerkkanal mit anderen virtuellen Computern im gleichen Clouddienst oder virtuellen Netzwerk kommunizieren. Computer im Internet oder anderen virtuellen Netzwerken erfordern jedoch von den Endpunkten, dass sie den eingehenden Netzwerkdatenverkehr auf einen virtuellen Computer leiten. Dieser Artikel ist auch für [virtuelle Linux-Computer](virtual-machines-linux-classic-setup-endpoints.md) verfügbar.

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)] [Resource Manager model](virtual-machines-windows-nsg-quickstart-portal.md). Im **Resource Manager**-Bereitstellungsmodell werden Endpunkte mithilfe von **Netzwerksicherheitsgruppen (NSGs)** konfiguriert. Weitere Informationen finden Sie unter [Ermöglichen des externen Zugriffs auf einen virtuellen Computer über das Azure-Portal](virtual-machines-windows-nsg-quickstart-portal.md).

Beim Erstellen eines virtuellen Windows-Computers im klassischen Azure-Portal werden allgemeine Endpunkte, z.B. für Remotedesktop und Windows PowerShell-Remoting, in der Regel automatisch erstellt. Sie können zusätzliche Endpunkte beim Erstellen des virtuellen Computers oder später nach Bedarf konfigurieren.



[AZURE.INCLUDE [virtual-machines-common-classic-setup-endpoints](../../includes/virtual-machines-common-classic-setup-endpoints.md)]

## Nächste Schritte

* Ein Azure PowerShell-Cmdlet zum Einrichten eines VM-Endpunkts finden Sie unter [Add-AzureEndpoint](https://msdn.microsoft.com/library/azure/dn495300.aspx).

* Informationen dazu, wie Sie ein Azure PowerShell-Cmdlet zum Verwalten einer ACL für einen Endpunkt verwenden, finden Sie unter [Verwalten von Zugriffssteuerungslisten (ACLs) für Endpunkte mithilfe von PowerShell](../virtual-network/virtual-networks-acl-powershell.md).

* Wenn Sie einen virtuellen Computer im Resource Manager-Bereitstellungsmodell erstellt haben, können Sie Azure PowerShell verwenden, um [Netzwerksicherheitsgruppen](../virtual-network/virtual-networks-create-nsg-arm-ps.md) zum Steuern des Datenverkehrs an den virtuellen Computer zu erstellen.

<!---HONumber=AcomDC_0608_2016-->