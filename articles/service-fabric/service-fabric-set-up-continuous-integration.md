<properties
   pageTitle="Continuous Integration für Service Fabric | Microsoft Azure"
   description="Hier erhalten Sie eine Übersicht über die Einrichtung der Continuous Integration für eine Service Fabric-Anwendung mit Visual Studio Team Services (VSTS)."
   services="service-fabric"
   documentationCenter="na"
   authors="mthalman-msft"
   manager="timlt"
   editor="" />
<tags
   ms.service="multiple"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="multiple"
   ms.date="03/29/2016"
   ms.author="mthalman" />

# Einrichten der Continuous Integration für eine Service Fabric-Anwendung mit Visual Studio Team Services

In diesem Artikel werden die Schritte zum Einrichten der Continuous Integration (CI) für eine Azure Service Fabric-Anwendung mit Visual Studio Team Services (VSTS) beschrieben, um die automatische Erstellung, Verpackung und Bereitstellung Ihrer Anwendung sicherzustellen. Beachten Sie, dass die Anweisungen jedes Mal eine Neuerstellung des Clusters zur Folge haben.

Dieses Dokument wurde für die aktuelle Prozedur erstellt und wird wahrscheinlich im Laufe der Zeit geändert.

## Voraussetzungen

Richten Sie zunächst Ihr Projekt in Visual Studio Team Services ein.

1. Richten Sie mithilfe Ihres [Microsoft-Kontos](http://www.microsoft.com/account) ein Team Services-Konto ein, sofern Sie noch keines erstellt haben.

2. Erstellen Sie mit Ihrem Microsoft-Konto in Team Services ein neues Projekt.

3. Führen Sie für die Quelle Ihrer neuen oder vorhandenen Service Fabric-App einen Pushvorgang an dieses Projekt durch.

Weitere Informationen zur Verwendung von Team Services-Projekten finden Sie unter [Herstellen einer Verbindung mit Visual Studio](https://www.visualstudio.com/get-started/setup/connect-to-visual-studio-online).

## Einrichten Ihres Dienstprinzipals

### Einrichten der Authentifizierung für die Automatisierung

Bevor Sie den Buildcomputer einrichten können, müssen Sie einen [Dienstprinzipal](../resource-group-create-service-principal-portal.md) erstellen. Der Build-Agent verwendet ihn, um sich bei Azure zu authentifizieren. Außerdem müssen Sie ein Zertifikat erstellen und es in Azure Key Vault hochladen, da Key Vault die Dienstprinzipalauthentifizierung nicht unterstützt. Diese Schritte können auf einem beliebigen Computer ausgeführt werden. Der Entwicklungscomputer ist dabei eine gute Wahl.

### Installieren von Azure PowerShell und Anmelden

1.  Installieren Sie „PowerShellGet“.

    a. Falls Sie Windows 10 mit den neusten Aktualisierungen verwenden, können Sie diesen Schritt überspringen (PowerShellGet ist bereits vorinstalliert).

    b. Installieren Sie andernfalls [Windows Management Framework 5.0](https://aka.ms/wmf5download), denn PowerShellGet ist darin enthalten.

2.	Installieren und aktualisieren Sie das AzureRM-Modul. Wenn Sie eine ältere Version von Azure PowerShell installiert haben, entfernen Sie sie:

    a. Klicken Sie mit der rechten Maustaste auf die Schaltfläche „Start“, und wählen Sie **Programme hinzufügen/entfernen**.

    b. Suchen Sie nach „Azure PowerShell“, und deinstallieren Sie es.

    c. Öffnen Sie eine PowerShell-Eingabeaufforderung als Administrator.

    d. Installieren Sie das AzureRM-Modul mit dem Befehl `Install-Module AzureRM`.

3.	Deaktivieren (oder aktivieren) Sie die Azure-Datensammlung.

    Andernfalls werden Sie von Azure-Cmdlets immer wieder gefragt, ob Sie die Datensammlung verwenden möchten. Dabei wird auf eine Benutzereingabe gewartet, was die Automatisierung blockiert. Treffen Sie daher bereits vorab eine Entscheidung, um diese Nachfrage zu unterdrücken. Führen Sie dazu einen der folgenden Befehle aus:

    - Enable-AzureRmDataCollection

    - Disable-AzureRmDataCollection

4.	Melden Sie sich bei Azure PowerShell an:

    a. Führen Sie den Befehl `Login-AzureRmAccount` aus.

    b. Geben Sie im angezeigten Dialogfeld Ihre Azure-Anmeldeinformationen ein.

    c. Führen Sie den Befehl `Get-AzureRmSubscription` aus.

    d. Suchen Sie das Abonnement, das Sie verwenden möchten.

    e. Führen Sie den Befehl `Select-AzureRmSubscription -SubscriptionId <ID for your subscription>` aus.

### Erstellen eines Dienstprinzipals

1. Befolgen Sie [diese Anweisungen](https://blogs.msdn.microsoft.com/visualstudioalm/2015/10/04/automating-azure-resource-group-deployment-using-a-service-principal-in-visual-studio-online-buildrelease-management/), um einen Dienstprinzipal und einen Dienstendpunkt für Ihr Projekt zu erstellen.

2. Beachten Sie die Werte, die am Ende der Ausgabe des Skripts gedruckt sind. Sie benötigen diese, um die Builddefinition einzurichten.

### Erstellen und Hochladen eines Zertifikats an einen neuen Azure-Schlüsseltresor

>[AZURE.NOTE] Dieses Beispielskript generiert ein selbstsigniertes Zertifikat – dies ist keine sichere Vorgehensweise und nur für Experimente zulässig. Befolgen Sie die Richtlinien Ihres Unternehmens, um stattdessen ein legitimes Zertifikat zu erhalten. Diese Anweisungen verwenden ein einziges Zertifikat sowohl für den Server als auch für den Client. In der Produktionsumgebung sollten Sie separate Server- und Clientzertifikate verwenden.

1. Laden Sie die Datei [ServiceFabricContinuousIntegrationScripts.zip](https://gallery.technet.microsoft.com/Set-up-continuous-f8b251f6) herunter, und extrahieren Sie sie auf diesem Computer in einem Ordner.

2. Wechseln Sie über eine PowerShell-Eingabeaufforderung mit Administratorrechten zum Verzeichnis `<extracted zip>/Manual`.

3. Führen Sie das PowerShell-Skript `CreateAndUpload-Certificate.ps1` mit den folgenden Parametern aus:

| Parameter | Wert |
| --- | --- |
| KeyVaultLocation | Ein beliebiger Wert. Dieser Parameter muss dem Ort entsprechen, an dem Sie den Cluster erstellen möchten. |
| CertificateSecretName | Ein beliebiger Wert. |
| CertificateDnsName | Muss dem DNS-Namen Ihres Clusters entsprechen. Beispiel: `mycluster.westus.cloudapp.azure.com` |
| SecureCertificatePassword | Ein beliebiger Wert. Dieser Parameter wird verwendet, wenn Sie das Zertifikat auf Ihrem Buildcomputer importieren. |
| KeyVaultName | Ein beliebiger Wert. |
| KeyVaultResourceGroupName | Ein beliebiger Wert. Verwenden Sie jedoch nicht den Ressourcengruppennamen, den Sie für Ihren Cluster verwenden möchten. |
| PfxFileOutputPath| Ein beliebiger Wert. Diese Datei wird verwendet, um das Zertifikat auf Ihren Buildcomputer zu importieren. |

Nach Abschluss des Skripts werden die folgenden drei Werte ausgegeben. Notieren Sie sich diese Werte, da sie als Buildvariablen verwendet werden.

 - `ServiceFabricCertificateThumbprint`
 - `ServiceFabricKeyVaultId`
 - `ServiceFabricCertificateSecretId`

## Einrichten Ihres Buildcomputers

### Installieren von Visual Studio 2015

Wenn Sie bereits einen Computer bereitgestellt haben (oder Ihren eigenen bereitstellen möchten), installieren Sie auf dem ausgewählten Computer [Visual Studio 2015](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx).

Wenn Sie noch nicht über einen Computer verfügen, können Sie schnell einen virtuellen Azure-Computer (VM) mit vorinstalliertem Visual Studio 2015 bereitstellen. Gehen Sie dazu folgendermaßen vor:

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) an.

2. Wählen Sie in der linken oberen Ecke des Bildschirms den Befehl **Neu** aus.

3. Wählen Sie **Marketplace** aus.

4. Suchen Sie nach **Visual Studio 2015**.

5. Wählen Sie **Compute** > **Virtueller Computer** > **Aus Katalog** aus.

6. Wählen Sie das Image **Visual Studio Enterprise 2015 Update 2 mit den universelle Windows-Tools und dem Azure SDK 2.9 unter Windows Server 2012 R2** aus.

    >[AZURE.NOTE] Azure SDK ist zwar keine erforderliche Komponente, es sind jedoch derzeit keine Images verfügbar, bei denen nur Visual Studio 2015 installiert ist.

7.	Befolgen Sie die Anweisungen im Dialogfeld, um Ihren virtuellen Computer zu erstellen.

### Installieren Sie das Service Fabric-SDK.

Installieren Sie das [Service Fabric SDK](service-fabric-get-started.md#install-the-runtime-sdk-and-tools) auf Ihrem Computer.

### Installieren von Azure PowerShell

Führen Sie zum Installieren von Azure PowerShell die Schritte im vorherigen Abschnitt „Installieren von Azure PowerShell und Anmelden“ aus. Überspringen Sie dabei den Schritt „Melden Sie sich bei Azure PowerShell an“.

### Registrieren der Azure PowerShell-Module beim Netzwerkdienstkonto

>[AZURE.NOTE] Führen Sie dies aus, *bevor* Sie den Build-Agent starten. Andernfalls erkennt er nicht die neue Umgebungsvariable.

1. Drücken Sie WINDOWS+R-TASTE, geben Sie **regedit** ein, und drücken Sie die EINGABETASTE.

2. Klicken Sie mit der rechten Maustaste auf den Knoten `HKEY_Users\.Default\Environment`, und wählen Sie **Neu** > **Wert der erweiterbaren Zeichenfolge** aus.

3. Geben Sie für den Namen `PSModulePath` und für den Wert `%PROGRAMFILES%\WindowsPowerShell\Modules` ein. Ersetzen Sie `%PROGRAMFILES%` durch den Wert der Umgebungsvariable `PROGRAMFILES`.

### Importieren Ihres Automation-Zertifikats

1.	Importieren Sie das Zertifikat in Ihren Buildcomputer. Gehen Sie dazu folgendermaßen vor:

    a. Kopieren Sie die durch das Skript „CreateAndUpload-Certificate.ps1“ erstellte PFX-Datei auf den Buildcomputer.

    b. Öffnen Sie eine PowerShell-Eingabeaufforderung mit Administratorrechten, und führen Sie die folgenden Befehle aus. Verwenden Sie dazu das Kennwort, das Sie zuvor an `CreateAndUpload-Certificate.ps1` übergeben haben.

    ```powershell
    $password = Read-Host -AsSecureString
    Import-PfxCertificate -FilePath <path/to/cert.pfx> -CertStoreLocation Cert:\LocalMachine\My -Password $password -Exportable
    ```

2.	Führen Sie den Zertifikat-Manager aus.

    a. Öffnen Sie unter Windows die Systemsteuerung. Klicken Sie mit der rechten Maustaste auf die Schaltfläche „Start“, und wählen Sie anschließend **Systemsteuerung** aus.

    b. Suchen Sie nach **Zertifikat**.

    c. Wählen Sie **Verwaltung** > **Computerzertifikate verwalten** aus.

3.	Erteilen Sie dem Netzwerkdienstkonto die Berechtigung zur Verwendung Ihres Automation-Zertifikats.

    a. Erweitern Sie unter **Zertifikate – lokaler Computer** den Knoten **Persönlich**, und wählen Sie anschließend **Zertifikate** aus.

    b. Suchen Sie in der Liste nach Ihrem Zertifikat.

    c. Klicken Sie mit der rechten Maustaste auf Ihr Zertifikat, und wählen Sie dann **Alle Aufgaben** > **Private Schlüssel verwalten** aus.

    d. Wählen Sie die Schaltfläche **Hinzufügen** aus, geben Sie **Netzwerkdienst** ein, und wählen Sie anschließend **Namen überprüfen** aus.

    e. Klicken Sie auf **OK**.

    ![Screenshot der Schritte zum Erteilen der Berechtigungen für das lokale Dienstkonto](media/service-fabric-set-up-continuous-integration/windows-certificate-manager.png)

4.  Kopieren Sie das Zertifikat in den Ordner `Trusted People`.

    a. Ihr Zertifikat wurde in **Persönlich/Zertifikate** importiert, aber wir müssen es zu **Vertrauenswürdige Personen** hinzufügen. Klicken Sie mit der rechten Maustaste auf das Zertifikat, und wählen Sie **Kopieren** aus. Klicken Sie anschließend mit der rechten Maustaste auf den Ordner **Vertrauenswürdige Personen**, und wählen Sie **Einfügen** aus.

### Registrieren Ihres Build-Agents

1.	Laden Sie „agent.zip“ herunter. Gehen Sie dazu folgendermaßen vor:

    a. Melden Sie sich bei Ihrem Teamprojekt an, z.B. „**https://[your-VSTS-account-name].visualstudio.com**”.

    b. Wählen Sie das Zahnradsymbol in der rechten oberen Bildschirmecke aus.

    c. Klicken Sie auf die Registerkarte **Agentpools**.

    d. Wählen Sie **Agent herunterladen** aus, um die Datei „agent.zip“ herunterzuladen.

    >[AZURE.NOTE] Falls der Agent nicht heruntergeladen wird, überprüfen Sie Ihren Popupblocker.

    e. Kopieren Sie „agent.zip“ auf den zuvor erstellten Buildcomputer.

    f. Entpacken Sie „agent.zip“ in `C:\agent` (oder an einem anderen Speicherort mit einem kurzen Pfad) auf dem Buildcomputer.

    >[AZURE.NOTE] Wenn Sie ASP.NET 5-Webdienste verwenden möchten, empfiehlt sich die Wahl eines möglichst kurzen Ordnernamens. Andernfalls treten bei der Bereitstellung unter Umständen Fehler vom Typ **PathTooLongExceptions** auf. Wenn ein ASP.NET Core veröffentlicht wird, werden die Auswirkungen dieses Problems minimiert.

2.	Führen Sie über eine Eingabeaufforderung mit Administratorrechten `C:\agent\ConfigureAgent.cmd` aus. Das Skript fordert Sie zur Eingabe folgender Parameter auf:

|Parameter|Wert|
|---|---|
|Agent Name|Übernehmen Sie den Standardwert. (`Agent-[machine name]`)|
|TFS Url|Geben Sie die URL zu Ihrem Teamprojekt ein, z.B. `https://[your-VSTS-account-name].visualstudio.com`.|
|Agent Pool|Geben Sie den Namen Ihres Agentpools ein. (Falls Sie keinen Agentpool erstellt haben, übernehmen Sie den Standardwert.)|
|Work folder|Übernehmen Sie den Standardwert. In diesem Ordner erstellt der Build-Agent Ihre Anwendung. Wenn Sie ASP.NET 5-Webdienste verwenden möchten, empfiehlt sich die Wahl eines möglichst kurzen Ordnernamens. Andernfalls treten bei der Bereitstellung unter Umständen Fehler vom Typ „PathTooLongExceptions“ auf.|
|Install as Windows Service?|Der Standardwert ist „N“. Ändern Sie den Wert in **Y** (Ja).|
|User account to run the service|Der Standardwert ist `NT AUTHORITY\LOCAL SERVICE`. Ändern Sie den Standardwert auf `NT AUTHORITY\NetworkService`.|
|Kennwort für `NT AUTHORITY\Network Service`|Das Netzwerkdienstkonto verfügt nicht über ein Kennwort, akzeptiert jedoch keine leeren Kennwörter. Geben Sie eine beliebige nicht-leere Zeichenfolge als Kennwort ein (egal was Sie eingeben, es wird ignoriert).|
|Un-configure existing agent?|Übernehmen Sie den Standardwert. (**N**)|

3.  Geben Sie bei Aufforderung die Anmeldeinformationen für Ihr Microsoft-Konto ein, das über Rechte für Ihr Teamprojekt verfügt.

4.  Überprüfen Sie, dass Ihr Build-Agent registriert wurde, und konfigurieren Sie seine Funktionen. Gehen Sie dazu folgendermaßen vor:

    a. Wechseln Sie wieder zu Ihrem Webbrowser (`https://[your-VSTS-account-name].visualstudio.com/_admin/_AgentPool`), und aktualisieren Sie die Seite.

    b. Wählen Sie den Agentpool aus, den Sie zuvor beim Ausführen von „ConfigureAgent.ps1“ ausgewählt haben.

    c. Vergewissern Sie sich, dass der Build-Agent in der Liste enthalten und die Statusanzeige grün ist. Wenn die Statusanzeige rot ist, kann der Build-Agent keine Verbindung mit Team Services herstellen.

    ![Screenshot des Status des Build-Agents](media/service-fabric-set-up-continuous-integration/vso-configured-agent.png)

    d. Wählen Sie den Build-Agent aus, wählen Sie anschließend die Registerkarte **Funktionen** aus.

    e. Fügen Sie eine Funktion mit dem Namen **azureps** mit einem beliebigen Wert hinzu. Dadurch wird VSTS angezeigt, dass auf diesem Computer Azure PowerShell installiert ist. Dies wird für einige der von VSTS bereitgestellten Buildaufgaben vorausgesetzt.


## Erstellen Ihrer Builddefinition

>[AZURE.NOTE] Die Builddefinition, die Sie anhand dieser Anweisungen erstellen, unterstützt nicht mehrere parallele Builds, auch nicht auf separaten Computern. Das liegt daran, dass die einzelnen Builds jeweils die gleiche Ressourcengruppe/den gleichen Cluster beanspruchen würden. Wenn Sie mehrere Build-Agents ausführen möchten, müssen Sie die folgenden Anweisungen/Skripts anpassen, um diese Störung zu vermeiden.

### Hinzufügen einer Service Fabric Azure Resource Manager-Vorlage zu Ihrer Anwendung

1. Laden Sie `azuredeploy.json` und `azuredeploy.parameters.json` aus [diesem Beispiel](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype-wad) herunter.

2. Öffnen Sie `azuredeploy.parameters.json`, und bearbeiten Sie die folgenden Parameter:

    |Parameter|Wert|
    |---|---|
    |clusterLocation|Muss dem Standort Ihres Schlüsseltresors entsprechen. Beispiel: `westus`|
    |clusterName|Muss mit dem DNS-Namen Ihres Zertifikats übereinstimmen. Wenn der DNS-Name des Zertifikats z.B. `mycluster.westus.cloudapp.net` ist, dann muss `clusterName` `mycluster` sein.|
    |adminPassword|8-123 Zeichen mit mindestens 3 der folgenden Zeichenarten: Großbuchstaben, Kleinbuchstaben, Zahlen, Sonderzeichen.|
    |certificateThumbprint|Aus der Ausgabe von `CreateAndUpload-Certificate.ps1`|
    |sourceVaultValue|Aus der Ausgabe von `CreateAndUpload-Certificate.ps1`|
    |certificateUrlValue|Aus der Ausgabe von `CreateAndUpload-Certificate.ps1`|

3. Fügen Sie die neuen Dateien der Quellcodeverwaltung hinzu, und schieben Sie sie nach VSTS.

### Erstellen der Builddefinition

1.	Erstellen Sie eine leere Builddefinition. Gehen Sie dazu folgendermaßen vor:

    a. Öffnen Sie Ihr Projekt in Visual Studio Team Services.

    b. Wählen Sie die Registerkarte **Erstellen** aus.

    c. Wählen Sie das grüne Pluszeichen (**+**) aus, um eine neue Builddefinition zu erstellen.

    d. Wählen Sie **Leer** und anschließend **Weiter** aus.

    e. Vergewissern Sie sich, dass das richtige Repository und die richtige Verzweigung ausgewählt sind.

    f. Aktivieren Sie das Kontrollkästchen **Fortlaufende Integration**, um sicherzustellen, dass dieser Build ausgelöst wird, wenn die Verzweigung aktualisiert wird.

    g. Wählen Sie die Agent-Warteschlange, für die Sie Ihren Build-Agent registriert haben.

2.	Erstellen Sie auf der Registerkarte **Variablen** die folgenden Variablen mit den angegebenen Werten:

    |Variable|Wert|Geheimer Schlüssel|Zum Einreihungszeitpunkt erlauben|
    |---|---|---|---|
    |BuildConfiguration|Version||X|
    |BuildPlatform|x64||||

3.  Speichern Sie die Builddefinition, und benennen Sie sie. Der Name kann später bei Bedarf geändert werden.

### Fügen Sie den Schritt „NuGet-Pakete wiederherstellen“ hinzu

1. Wählen Sie auf der Registerkarte **Erstellen** den Befehl **Buildschritt hinzufügen...** aus.

2. Wählen Sie **Paket** > **NuGet-Installer**.

3. Wählen Sie neben dem Namen des Buildschritts das Stiftsymbol aus, und ändern Sie den Namen in **NuGet-Pakete wiederherstellen**.

4. Wählen Sie neben dem Feld **Lösung** die Schaltfläche **...** und anschließend Ihre SLN-Datei aus.

5. Speichern Sie die Builddefinition.

### Hinzufügen eines Buildschritts

1.	Wählen Sie auf der Registerkarte **Erstellen** den Befehl **Buildschritt hinzufügen...** aus.

2.	Wählen Sie **Erstellen** > **MSBuild** aus.

3.	Wählen Sie neben dem Namen des Buildschritts das Stiftsymbol aus, und ändern Sie dann den Namen zu **Build**.

4. Kopieren Sie diese Werte:

    |Name der Einstellung|Wert|
    |---|---|
    |Lösung|Klicken Sie auf die Schaltfläche **...**, und wählen Sie die `.sln`-Datei für Ihre Lösung.|
    |Plattform|`$(BuildPlatform)`|
    |Konfiguration|`$(BuildConfiguration)`|

5.	Speichern Sie die Builddefinition.

### Hinzufügen eines Paketschritts

1.	Wählen Sie auf der Registerkarte **Erstellen** den Befehl **Buildschritt hinzufügen...** aus.

2.	Wählen Sie **Erstellen** > **MSBuild** aus.

3.	Wählen Sie neben dem Namen des Buildschritts das Stiftsymbol aus, und ändern Sie dann den Namen in **Paket**.

4. Kopieren Sie diese Werte:

    |Name der Einstellung|Wert|
    |---|---|
    |Lösung|Klicken Sie auf die **...** Schaltfläche, und wählen Sie die `.sfproj`-Datei Ihres Anwendungsprojekts aus.|
    |Plattform|`$(BuildPlatform)`|
    |Konfiguration|`$(BuildConfiguration)`|
    |MSBuild-Argumente|`/t:Package`|

5.	Speichern Sie die Builddefinition.

### <a name="RemoveClusterResourceGroup"></a> Hinzufügen des Schritts „Clusterressourcengruppe entfernen“

Wenn bei einem vorherigen Buildvorgang keine Bereinigung stattgefunden hat (beispielsweise, weil der Buildvorgang vor der Bereinigung abgebrochen wurde), ist unter Umständen eine Ressourcengruppe vorhanden, die einen Konflikt mit der neuen Ressourcengruppe verursachen könnte. Bereinigen Sie zur Vermeidung von Konflikten alle übrig gebliebenen Ressourcengruppen (und die dazugehörigen Ressourcen), bevor Sie eine neue erstellen.

1.	Wählen Sie auf der Registerkarte **Erstellen** den Befehl **Buildschritt hinzufügen...** aus.

2.	Wählen Sie **Bereitstellen** > **Bereitstellung einer Azure-Ressourcengruppe** aus.

3.	Wählen Sie neben dem Namen des Buildschritts das Stiftsymbol aus, und ändern Sie anschließend den Namen in **Clusterressourcengruppe entfernen**.

4. Kopieren Sie diese Werte:

    |Name der Einstellung|Wert|
    |---|---|
    |AzureConnectionType|**Azure Resource Manager**|
    |Azure RM-Abonnement|Wählen Sie den Verbindungsendpunkt aus, den Sie im Abschnitt **Erstellen eines Dienstprinzipals** erstellt haben.|
    |Aktion|**Ressourcengruppe löschen**|
    |Ressourcengruppe|Geben Sie einen nicht verwendeten Namen ein. Sie müssen den gleichen Namen im nächsten Schritt verwenden.|
    |Bei Fehler fortsetzen|Bei diesem Schritt tritt ein Fehler auf, wenn die Ressourcengruppe nicht vorhanden ist. Aktivieren Sie **Bei Fehler fortsetzen** in dem Abschnitt **Steuerungsoptionen** aus, um dies zu vermeiden.|

5.	Speichern Sie die Builddefinition.

### Hinzufügen eines Schritts „Sicheren Cluster bereitstellen“

1.	Wählen Sie auf der Registerkarte **Erstellen** den Befehl **Buildschritt hinzufügen...** aus.

2.	Wählen Sie **Bereitstellen** > **Bereitstellung einer Azure-Ressourcengruppe** aus.

3.	Wählen Sie neben dem Namen des Buildschritts das Stiftsymbol aus, und ändern Sie anschließend den Namen in **Sicheren Cluster bereitstellen**.

4. Kopieren Sie diese Werte:

    |Name der Einstellung|Wert|
    |---|---|
    |AzureConnectionType|**Azure Resource Manager**|
    |Azure RM-Abonnement|Wählen Sie den Verbindungsendpunkt aus, den Sie im Abschnitt **Erstellen eines Dienstprinzipals** erstellt haben.|
    |Aktion|**Erstellen oder Aktualisieren einer Ressourcengruppe**|
    |Ressourcengruppe|Muss dem Namen entsprechen, den Sie im vorherigen Schritt verwendet haben.|
    |Standort|Muss dem Standort Ihres Schlüsseltresors entsprechen.|
    |Vorlage|Klicken Sie auf die Schaltfläche **…**, und wählen Sie `azuredeploy.json` aus.|
    |Vorlagenparameter|Klicken Sie auf die Schaltfläche **…**, und wählen Sie `azuredeploy.parameters.json` aus.|

5.	Speichern Sie die Builddefinition.

### Hinzufügen eines Schritts „Bereitstellen“

1.	Wählen Sie auf der Registerkarte **Erstellen** den Befehl **Buildschritt hinzufügen...** aus.

2.	Wählen Sie **Hilfsprogramm** > **PowerShell** aus.

3.	Wählen Sie neben dem Namen des Buildschritts das Stiftsymbol aus, und ändern Sie dann den Namen in **Bereitstellen**.

4. Kopieren Sie diese Werte:

    |Name der Einstellung|Wert|
    |---|---|
    |Typ|**Dateipfad**|
    |Skriptdateiname|Klicken Sie auf die Schaltfläche **...**, und navigieren Sie zum Verzeichnis **Skripts** innerhalb Ihres Anwendungsprojekts. Wählen Sie `Deploy-FabricApplication.ps1` aus.|
    |Argumente|`-PublishProfileFile path/to/MySolution/MyApplicationProject/PublishProfiles/MyPublishProfile.xml -ApplicationPackagePath path/to/MySolution/MyApplicationProject/pkg/$(BuildConfiguration)`|

5.	Speichern Sie die Builddefinition.

### Hinzufügen des Schritts „Überprüfen“

1. Dieser Schritt ist optional, wenn Sie diese Builddefinition noch konfigurieren. Sobald Sie aber einen Build erfolgreich ausgeführt und die Richtigkeit der anderen Buildschritte sichergestellt haben, können Sie an dieser Stelle Ihren eigenen Buildschritt zur Überprüfung einfügen. Dieser Schritt wäre anwendungsspezifisch und ist dazu bestimmt, die Richtigkeit der in dem Cluster bereitgestellten Anwendung zu überprüfen.
  
### Hinzufügen des abschließenden Schritts „Bereinigen“

1. Führen Sie die gleichen Schritte aus wie in dem Abschnitt [Hinzufügen des Schritts „Clusterressourcengruppe entfernen“](#RemoveClusterResourceGroup). Damit werden alle bereitgestellten Azure-Ressourcen bereinigt, die während des Builds erstellt wurden.

### Ausprobieren

Wählen Sie **Build zur Warteschlange hinzufügen** aus, um einen Build zu starten. Buildvorgänge werden auch beim Pushvorgang oder Einchecken ausgelöst.

## Alternativen

Mit den oben aufgeführten Anweisungen wird für jeden Buildvorgang ein neuer Cluster erstellt und am Ende des Buildvorgangs wieder entfernt. Wenn Sie stattdessen bei jedem Buildvorgang ein Anwendungsupgrade (für einen vorhandenen Cluster) durchführen möchten, führen Sie die folgenden Schritte aus:

1.	Erstellen Sie manuell einen Testcluster über das Azure-Portal oder über Azure PowerShell gemäß [dieser Anweisungen](service-fabric-cluster-creation-via-portal.md).

2.	Konfigurieren Sie Ihr Veröffentlichungsprofil für die Unterstützung von Anwendungsupgrades. Entsprechende Anweisungen finden Sie [hier](service-fabric-visualstudio-configure-upgrade.md).

4.	Entfernen Sie die Buildschritte **Clusterressourcengruppe entfernen** und **Cluster bereitstellen** aus Ihrer Builddefinition.

## Nächste Schritte

Weitere Informationen zu Continuous Integration für Service Fabric-Anwendungen finden Sie in den folgenden Artikeln:

 - [Startseite der Builddokumentation](https://msdn.microsoft.com/Library/vs/alm/Build/overview)
 - [Bereitstellen eines Build-Agents](https://msdn.microsoft.com/Library/vs/alm/Build/agents/windows)
 - [Erstellen und Konfigurieren einer Builddefinition](https://msdn.microsoft.com/Library/vs/alm/Build/vs/define-build)

<!---HONumber=AcomDC_0518_2016-->