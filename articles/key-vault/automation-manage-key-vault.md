<properties
	pageTitle="Verwalten des Azure-Schlüsseltresors mit Azure Automation | Microsoft Azure"
	description="Erfahren Sie, wie der Azure Automation-Dienst zum Verwalten des Azure-Schlüsseltresors verwendet werden kann."
	services="Key-Vault, automation"
	documentationCenter=""
	authors="csand-msft"
	manager="eamono"
	editor=""/>

<tags
	ms.service="key-vault"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/18/2016"
	ms.author="csand"/>



#Verwalten des Azure-Schlüsseltresors mit Azure Automation

Dieser Leitfaden bietet eine Einführung in den Azure Automation-Dienst und zeigt, wie Azure Automation zur einfacheren Verwaltung Ihrer Schlüssel und geheimen Schlüssel im Azure-Schlüsseltresor verwendet werden kann.

## Was ist Azure Automation?

[Azure Automation](https://azure.microsoft.com/services/automation/) ist ein Azure-Dienst für die Vereinfachung der Cloudverwaltung durch eine Prozessautomatisierung und Konfiguration des gewünschten Zustands. Mithilfe von Azure Automation können manuelle, sich wiederholende, lang andauernde und fehleranfällige Aufgaben automatisiert werden, um die Zuverlässigkeit und Effizienz für Ihre Organisation zu erhöhen und die Amortisierungszeit zu verkürzen.

Azure Automation bietet ein äußerst zuverlässiges, hoch verfügbares Workflow-Ausführungsmodul, das sich durch Skalierung an Ihre Anforderungen anpasst. In Azure Automation können Prozesse manuell, von Systemen von Drittanbietern oder in geplanten Intervallen gestartet werden, damit Aufgaben genau im Bedarfsfall ausgeführt werden.

Indem Sie die Aufgaben im Zusammenhang mit der Cloudverwaltung mit Azure Automation automatisieren, verringern Sie den Verwaltungsaufwand und ermöglichen es Ihren IT- und DevOps-Mitarbeitern, sich Aufgaben zu widmen, die einen Mehrwert für Ihr Unternehmen liefern.


## Wie kann Azure Automation beim Verwalten des Azure-Schlüsseltresors nützlich sein?

Key Vault kann in Azure Automation mithilfe der [AzureRM Key Vault-Cmdlets](https://www.powershellgallery.com/packages/AzureRM.KeyVault/1.1.4) und der [klassischen Azure Key Vault-Cmdlets](https://msdn.microsoft.com/library/azure/dn868052.aspx) verwaltet werden. Das Azure-Modul für die Verwaltung einer klassischen Key Vault-Instanz ist automatisch in Azure Automation verfügbar. Sie können das [AzureRM-KeyVault-Modul](https://www.powershellgallery.com/packages/AzureRM.KeyVault/1.1.4) in Azure Automation importieren, sodass viele Key Vault-Verwaltungsaufgaben im Dienst ausgeführt werden können. Sie können diese Cmdlets in Azure Automation auch mit den Cmdlets für andere Azure-Dienste koppeln, um komplexe Aufgaben in Azure-Diensten und Systemen anderer Hersteller zu automatisieren.

Mit den Azure Key Vault-Cmdlets können Sie u.a. folgende Aufgaben ausführen: Erstellen und Konfigurieren eines Schlüsseltresors, Erstellen oder Importieren eines Schlüssels, Erstellen oder Aktualisieren eines geheimen Schlüssels, Aktualisieren der Attribute eines Schlüssels, Abrufen eines Schlüssels oder eines geheimen Schlüssels und Löschen eines Schlüssels oder eines geheimen Schlüssels.

Hier sind einige Beispiele für die Verwendung von PowerShell zum Verwalten von Key Vault aufgeführt:
* [Azure Key Vault - Step by Step](https://blogs.technet.microsoft.com/kv/2015/06/02/azure-key-vault-step-by-step/) (Azure Key Vault – Schritt für Schritt)
* [Setting Up and Configuring an Azure Key Vault](https://www.simple-talk.com/cloud/platform-as-a-service/setting-up-and-configuring-an-azure-key-vault/) (Einrichten und Konfigurieren eines Azure-Schlüsseltresors)


## Nächste Schritte

Nachdem Sie sich nun mit den Grundlagen von Azure Automation und dessen Einsatz zum Verwalten des Azure-Schlüsseltresors vertraut gemacht haben, folgen Sie diesen Links, um weitere Informationen zu Azure Automation zu erhalten.

* Weitere Informationen erhalten Sie im Lernprogramm [Erste Schritte](../automation/automation-first-runbook-graphical.md) zu Azure Automation.
* Weitere Informationen finden Sie in den [PowerShell-Skripts des Azure-Schlüsseltresors](https://gallery.technet.microsoft.com/scriptcenter/Azure-Key-Vault-Powershell-1349b091).

<!---HONumber=AcomDC_0518_2016-->