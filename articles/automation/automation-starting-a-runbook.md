<properties
   pageTitle="Starten eines Runbooks in Azure Automation | Microsoft Azure"
   description="Bietet eine Übersicht über die verschiedenen Methoden, die zum Starten eines Runbooks in Azure Automation verwendet werden können, und stellt Informationen zur Verwendung des Azure-Portals sowie von Windows PowerShell bereit."
   services="automation"
   documentationCenter=""
   authors="mgoedtel"
   manager="jwhit"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/15/2016"
   ms.author="magoedte;bwren"/>

# Starten eines Runbooks in Azure Automation

Die folgende Tabelle hilft Ihnen dabei herauszufinden, welche Methode zum Starten eines Runbooks in Azure Automation sich am besten für Ihr jeweiliges Szenario eignet. Dieser Artikel enthält Details zum Starten eines Runbooks über das Azure-Portal oder mit Windows PowerShell. Details zu den anderen Methoden werden in anderen Artikeln bereitgestellt, auf die Sie über unten stehende Links zugreifen können.

| **METHODE** | **MERKMALE** |
|-------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Azure-Portal](#starting-a-runbook-with-the-azure-portal) | <li>Einfachste Methode mit interaktiver Benutzeroberfläche.<br> <li>Formular zum Bereitstellen von einfachen Parameterwerten.<br> <li>Einfaches Nachverfolgen des Auftragsstatus.<br> <li>Authentifizierter Zugriff über Azure-Anmeldung. |
| [Windows PowerShell](https://msdn.microsoft.com/library/dn690259.aspx) | <li>Aufruf über Befehlszeile mit Windows PowerShell-Cmdlets.<br> <li>Kann in eine automatisierte Lösung mit mehreren Schritten eingebunden werden.<br> <li>Anforderung wird über Zertifikat oder OAuth-Benutzerprinzipal/-Dienstprinzipal authentifiziert.<br> <li>Bereitstellen von einfachen und komplexen Parameterwerten.<br> <li>Nachverfolgen des Auftragsstatus.<br> <li>Client erforderlich zur Unterstützung der PowerShell-Befehle. |
| [Azure Automation-API](https://msdn.microsoft.com/library/azure/mt662285.aspx) | <li>Flexibelste Methode, jedoch auch sehr komplex.<br> <li>Aufruf aus jedem benutzerdefinierten Code, der HTTP-Anforderungen ausführen kann.<br> <li>Anforderung wird über Zertifikat oder OAuth-Benutzerprinzipal/-Dienstprinzipal authentifiziert.<br> <li>Bereitstellen von einfachen und komplexen Parameterwerten.<br> <li>Nachverfolgen des Auftragsstatus. |
| [Webhooks](automation-webhooks.md) | <li>Starten eines Runbooks über eine einzelne HTTP-Anforderung.<br> <li>Authentifizierung über Sicherheitstoken in der URL.<br> <li>Client kann keine Parameterwerte überschreiben, die beim Erstellen des Webhooks festgelegt wurden. Ein Runbook kann einen einzelnen Parameter definieren, der mit den Details zur HTTP-Anforderung aufgefüllt wird.<br> <li>Keine Möglichkeit, den Auftragsstatus über die Webhook-URL nachzuverfolgen. |
| [Reagieren auf eine Azure-Warnung](../log-analytics/log-analytics-alerts.md) | <li>Starten eines Runbooks als Reaktion auf eine Azure-Warnung.<br> <li>Konfigurieren eines Webhooks für Runbook und Link zum Warnen.<br> <li>Authentifizierung über Sicherheitstoken in der URL.<br> <li>Unterstützt derzeit nur Warnungen für Metriken. |
| [Zeitplan](automation-scheduling-a-runbook.md) | <li>Automatisches Starten eines Runbooks: stündlich, täglich oder wöchentlich.<br> <li>Bearbeiten des Zeitplans über das Azure-Portal, PowerShell-Cmdlets oder die Azure-API.<br> <li>Bereitstellen von Parameterwerten, die mit dem Zeitplan verwendet werden sollen. |
| [Über ein anderes Runbook](automation-child-runbooks.md) | <li>Verwenden eines Runbooks als Aktivität in einem anderen Runbook.<br> <li>Nützlich bei Funktionen, die von mehreren Runbooks verwendet werden.<br> <li>Bereitstellen von Parameterwerten für untergeordnete Runbooks und Verwenden der Ausgabe im übergeordneten Runbook. |

Die folgende Abbildung veranschaulicht ausführlich die Prozessschritte im Lebenszyklus eines Runbooks. Dies umfasst verschiedene Arten, auf die ein Runbook in Azure Automation gestartet werden kann, sowie Komponenten, die erforderlich sind, damit ein Hybrid Runbook Worker Azure Automation-Runbooks ausführen und Interaktionen zwischen verschiedenen Komponenten unterstützen kann. Weitere Informationen zum Ausführen von Automation-Runbooks in Ihrem Rechenzentrum finden Sie unter [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md).

![Runbook-Architektur](media/automation-starting-runbook/runbooks-architecture.png)

## Starten eines Runbooks über das Azure-Portal

1.	Klicken Sie in Ihrem Automation-Konto auf **Runbooks**, um das Blatt **Runbooks** zu öffnen.
2.	Klicken Sie auf ein Runbook, um das zugehörige **Runbook**-Blatt zu öffnen.
3.	Klicken Sie auf **Start**.
4.	Wenn das Runbook keine Parameter enthält, werden Sie dazu aufgefordert, zu bestätigen, dass Sie es starten möchten. Wenn das Runbook Parameter enthält, wird das Blatt **Runbook starten** geöffnet, auf dem Sie Parameterwerte bereitstellen können. Weitere Informationen zu Parametern finden Sie im Abschnitt [Runbookparameter](#Runbook-parameters) weiter unten.
5.	Das Blatt **Auftrag** wird geöffnet, auf dem Sie den Auftragsstatus nachverfolgen können.

## Starten eines Runbooks mit Windows PowerShell

Sie können das Cmdlet [Start-AzureRmAutomationRunbook](https://msdn.microsoft.com/library/mt603661.aspx) verwenden, um ein Runbook mit Windows PowerShell zu starten. Der folgende Beispielcode startet ein Runbook namens "Test-Runbook".

```
Start-AzureRmAutomationRunbook -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook" -ResourceGroupName "ResourceGroup01"
```

Start-AzureRmAutomationRunbook gibt ein Auftragsobjekt zurück, mit dem Sie den Status nachverfolgen können, sobald das Runbook gestartet wurde. Anschließend können Sie dieses Auftragsobjekt zum Ermitteln des Auftragsstatus mit [Get-AzureRmAutomationJob](https://msdn.microsoft.com/library/mt619440.aspx) und zum Abrufen der Ausgabe mit [Get-AzureRmAutomationJobOutput](https://msdn.microsoft.com/library/mt603476.aspx) verwenden. Der folgende Beispielcode startet ein Runbook namens "Test-Runbook", wartet, bis die Ausführung beendet ist, und zeigt anschließend die Runbookausgabe an.

```
$runbookName = "Test-Runbook"
$ResourceGroup = "ResourceGroup01"
$AutomationAcct = "MyAutomationAccount"

$job = Start-AzureRmAutomationRunbook –AutomationAccountName $AutomationAcct -Name $runbookName -ResourceGroupName $ResourceGroup

$doLoop = $true
While ($doLoop) {
   $job = Get-AzureRmAutomationJob –AutomationAccountName $AutomationAcct -Id $job.JobId -ResourceGroupName $ResourceGroup
   $status = $job.Status
   $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped"))
}

Get-AzureRmAutomationJobOutput –AutomationAccountName $AutomationAcct -Id $job.JobId -ResourceGroupName $ResourceGroup –Stream Output
```

Wenn das Runbook Parameter erfordert, müssen Sie diese als [Hashtabelle](http://technet.microsoft.com/library/hh847780.aspx) bereitstellen, wobei der Schlüssel der Hashtabelle dem Parameternamen und der Wert dem Parameterwert entspricht. Das folgende Beispiel zeigt das Starten eines Runbooks mit zwei Zeichenfolgenparametern "FirstName" und "LastName", einem ganzzahligen Wert namens "RepeatCount" und einem booleschen Parameter namens "Show". Weitere Informationen zu Parametern finden Sie im Abschnitt [Runbookparameter](#Runbook-parameters) weiter unten.

```
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Start-AzureRmAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" -ResourceGroupName "ResourceGroup01" –Parameters $params
```

## Runbookparameter

Wenn Sie ein Runbook über das Azure-Portal oder mit Windows PowerShell starten, wird die Anweisung über den Azure Automation-Webdienst gesendet. Dieser Dienst unterstützt keine Parameter mit komplexen Datentypen. Wenn Sie einen Wert für einen komplexen Parameter bereitstellen müssen, müssen Sie diesen inline aus einem anderen Runbook aufrufen, wie unter [Untergeordnete Runbooks in Azure Automation](automation-child-runbooks.md) beschrieben.

Der Azure Automation-Webdienst bietet spezielle Funktionen für Parameter, die bestimmte Datentypen verwenden, wie in den folgenden Abschnitten beschrieben.

### Benannte Werte

Wenn der Parameter den Datentyp „[object]“ enthält, können Sie das folgende JSON-Format verwenden, um eine Liste benannter Werte an den Parameter zu senden: *{Name1:Wert1, Name2:Wert2, Name3:Wert3}*. Bei den Werten muss es sich um einfache Typen handeln. Das Runbook empfängt den Parameter als [PSCustomObject](https://msdn.microsoft.com/library/system.management.automation.pscustomobject%28v=vs.85%29.aspx) mit Eigenschaften, die den einzelnen benannten Werten entsprechen.

Betrachten Sie das folgende Testrunbook, das einen Parameter namens "user" akzeptiert.

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][object]$user
   )
    $userObject = $user | ConvertFrom-JSON
	if ($userObject.Show) {
        foreach ($i in 1..$userObject.RepeatCount) {
            $userObject.FirstName
            $userObject.LastName
        }
    }
}
```

Für den Parameter "user" könnte folgender Text verwendet werden.

```
{FirstName:'Joe',LastName:'Smith',RepeatCount:'2',Show:'True'}
```

Dadurch wird die folgende Ausgabe zurückgegeben.

```
Joe
Smith
Joe
Smith
```

### Arrays

Wenn es sich bei dem Parameter um ein Array wie beispielsweise [array] oder [string] handelt, können Sie das folgende JSON-Format verwenden, um eine Liste von Werten an den Parameter zu senden: *[Wert1,Wert2,Wert3]*. Bei den Werten muss es sich um einfache Typen handeln.

Betrachten Sie das folgende Testrunbook, das einen Parameter namens *user* akzeptiert.

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][array]$user
   )
    if ($user[3]) {
        foreach ($i in 1..$user[2]) {
            $ user[0]
            $ user[1]
        }
    }
}
```

Für den Parameter "user" könnte folgender Text verwendet werden.

```
["Joe","Smith",2,true]
```

Dadurch wird die folgende Ausgabe zurückgegeben.

```
Joe
Smith
Joe
Smith
```

### Anmeldeinformationen

Wenn der Parameter den Datentyp **PSCredential** aufweist, können Sie den Namen eines Azure Automation-[Anmeldeinformationsobjekts](automation-credentials.md) bereitstellen. Das Runbook ruft die Anmeldeinformationen mit dem von Ihnen angegebenen Namen ab.

Betrachten Sie das folgende Testrunbook, das einen Parameter namens "credential" akzeptiert.

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][PSCredential]$credential
   )
   $credential.UserName
}
```

Für den Parameter "user" kann der folgende Text verwendet werden, vorausgesetzt, es ist ein Anmeldeinformationsobjekt namens *My Credential* vorhanden.

```
My Credential
```

Wenn der Benutzernamen in den Anmeldeinformationen *jsmith* lautet, erhalten Sie folgende Ausgabe.

```
jsmith
```

## Nächste Schritte

-	Die Runbook-Architektur im aktuellen Artikel bietet einen allgemeinen Überblick über Runbooks, die zum Verwalten von Ressourcen in Azure und lokal mit dem Hybrid Runbook Worker eingesetzt werden. Weitere Informationen zum Ausführen von Automation-Runbooks in Ihrem Rechenzentrum finden Sie unter [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md).
-	Weitere Informationen zum Erstellen von modularen Runbooks, die von anderen Runbooks für spezifische oder allgemeine Funktionen verwendet werden, finden Sie unter [Untergeordnete Runbooks](automation-child-runbooks.md).

<!---HONumber=AcomDC_0921_2016-->