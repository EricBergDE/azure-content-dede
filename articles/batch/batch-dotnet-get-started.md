<properties
	pageTitle="Tutorial – Erste Schritte mit der Azure Batch-Bibliothek für .NET | Microsoft Azure"
	description="Erfahren Sie mehr über grundlegende Konzepte zu Azure Batch, und informieren Sie sich anhand eines Beispielszenarios darüber, wie Sie den Batch-Dienst einsetzen können."
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.devlang="dotnet"
	ms.topic="hero-article"
	ms.tgt_pltfrm="na"
	ms.workload="big-compute"
	ms.date="08/15/2016"
	ms.author="marsma"/>

# Erste Schritte mit der Azure Batch-Bibliothek für .NET

> [AZURE.SELECTOR]
- [.NET](batch-dotnet-get-started.md)
- [Python](batch-python-tutorial.md)

Informieren Sie sich in diesem Artikel, in dem eine C#-Beispielanwendung Schritt für Schritt beschrieben wird, über die Grundlagen von [Azure Batch][azure_batch] und die [Batch-Bibliothek für .NET][net_api]. Wir sehen uns an, wie diese Beispielanwendung den Batch-Dienst nutzt, um eine parallele Workload in der Cloud zu verarbeiten, und wie sie mit [Azure Storage](../storage/storage-introduction.md) in Bezug auf die Dateibereitstellung und -wiederherstellung interagiert. Sie lernen gängige Verfahren für den Batch-Anwendungsworkflow kennen. Außerdem entwickeln Sie ein grundlegendes Verständnis der Hauptkomponenten von Batch, z. B. Aufträge, Aufgaben, Pools und Computeknoten.

![Batch-Lösungsworkflow (einfach)][11]<br/>

## Voraussetzungen

In diesem Artikel wird davon ausgegangen, dass Sie über C#- und Visual Studio-Grundkenntnisse verfügen. Außerdem wird vorausgesetzt, dass Sie die Anforderungen an die Kontoerstellung erfüllen können, die unten für Azure und die Batch- und Storage-Dienste angegeben sind.

### Konten

- **Azure-Konto**: Wenn Sie nicht bereits über ein Azure-Abonnement verfügen, können Sie ein [kostenloses Azure-Konto erstellen][azure_free_account].
- **Batch-Konto**: Wenn Sie über ein Azure-Abonnement verfügen, können Sie ein [Azure Batch-Konto erstellen](batch-account-create-portal.md).
- **Storage-Konto**: Weitere Informationen finden Sie unter [Informationen zu Azure-Speicherkonten](../storage/storage-create-storage-account.md) im Abschnitt [Erstellen eines Speicherkontos](../storage/storage-create-storage-account.md#create-a-storage-account).

> [AZURE.IMPORTANT] Von Batch wird derzeit *ausschließlich* der Speicherkontotyp **Allgemein** unterstützt, wie in [Informationen zu Azure-Speicherkonten](../storage/storage-create-storage-account.md) unter Schritt 5 ([Erstellen Sie ein Speicherkonto](../storage/storage-create-storage-account.md#create-a-storage-account)) beschrieben.

### Visual Studio

Zur Erstellung des Beispielprojekts benötigen Sie **Visual Studio 2015**. Kostenlose Versionen und Testversionen von Visual Studio finden Sie in der [Übersicht der Visual Studio 2015-Produkte][visual_studio].

### Codebeispiel *DotNetTutorial*

Das Beispiel [DotNetTutorial][github_dotnettutorial] ist eines der vielen Codebeispiele im Repository [azure-batch-samples][github_samples] auf GitHub. Klicken Sie zum Herunterladen des Beispiels entweder auf der Seite des Repositorys auf die Schaltfläche **Download ZIP**, oder klicken Sie zum direkten Herunterladen auf den Link [azure-batch-samples-master.zip][github_samples_zip]. Nachdem Sie den Inhalt der ZIP-Datei extrahiert haben, befindet sich die Projektmappe im folgenden Ordner:

`\azure-batch-samples\CSharp\ArticleProjects\DotNetTutorial`

### Azure Batch-Explorer (optional)

Der [Azure Batch-Explorer][github_batchexplorer] ist ein kostenloses Hilfsprogramm, das auf GitHub im Repository [azure-batch-samples][github_samples] zur Verfügung steht. Zum Durchführen dieses Tutorials ist es zwar nicht erforderlich, es kann aber beim Entwickeln und Debuggen Ihrer Batch-Lösungen hilfreich sein.

## Übersicht über das Beispielprojekt DotNetTutorial

Bei dem Codebeispiel *DotNetTutorial* handelt es sich um eine Visual Studio 2015-Projektmappe mit zwei Projekten: **DotNetTutorial** und **TaskApplication**.

- **DotNetTutorial** ist die Clientanwendung, die mit den Diensten Batch und Storage interagiert, um auf Computeknoten (virtuellen Computern) eine parallele Workload auszuführen. DotNetTutorial wird auf Ihrer lokalen Arbeitsstation ausgeführt.

- **TaskApplication** ist das Programm, das auf Computeknoten in Azure ausgeführt wird, um die eigentliche Arbeit zu erledigen. Im Beispiel analysiert `TaskApplication.exe` den Text in einer Datei, die aus Azure Storage heruntergeladen wird (Eingabedatei). Anschließend wird eine Textdatei produziert (Ausgabedatei), die eine Liste mit den drei wichtigsten Wörtern aus der Eingabedatei enthält. Nach dem Erstellen der Ausgabedatei lädt TaskApplication die Datei nach Azure Storage hoch. Sie steht für die Clientanwendung dann zum Herunterladen zur Verfügung. TaskApplication wird im Batch-Dienst parallel auf mehreren Computeknoten ausgeführt.

Das folgende Diagramm veranschaulicht die wichtigsten Vorgänge, die von der Clientanwendung *DotNetTutorial* und von der durch die Aufgaben ausgeführten Anwendung *TaskApplication* durchgeführt werden. Dieser grundlegende Workflow ist typisch für viele Computelösungen, die mit Batch erstellt werden. Er veranschaulicht zwar nicht alle verfügbaren Features des Batch-Diensts, aber fast jedes Batch-Szenario enthält ähnliche Prozesse.

![Batch-Beispielworkflow][8]<br/>

[**Schritt 1:**](#step-1-create-storage-containers) Erstellen von **Containern** in Azure Blob Storage<br/> [**Schritt 2:**](#step-2-upload-task-application-and-data-files) Hochladen von Aufgabenanwendungs- und Eingabedateien in Container<br/> [**Schritt 3:**](#step-3-create-batch-pool) Erstellen eines Batch-**Pools**<br/> &nbsp;&nbsp;&nbsp;&nbsp;**3a.** Die Aufgabe **StartTask** des Pools lädt die Binärdateien (TaskApplication) auf Knoten herunter, wenn diese dem Pool beitreten.<br/> [**Schritt 4:**](#step-4-create-batch-job) Erstellen eines Batch-**Auftrags**<br/> [**Schritt 5:**](#step-5-add-tasks-to-job) Hinzufügen von **Aufgaben** zum Auftrag<br/> &nbsp;&nbsp;&nbsp;&nbsp;**5a.** Die Aufgaben werden für die Ausführung auf Knoten geplant.<br/> &nbsp;&nbsp;&nbsp;&nbsp;**5b.** Jede Aufgabe lädt ihre Eingabedaten aus Azure Storage und beginnt dann mit der Ausführung.<br/> [**Schritt 6:**](#step-6-monitor-tasks) Überwachen von Aufgaben<br/> &nbsp;&nbsp;&nbsp;&nbsp;**6a.** Nach Abschluss der Aufgaben werden die zugehörigen Ausgabedaten in Azure Storage hochgeladen.<br/> [**Schritt 7:**](#step-7-download-task-output) Herunterladen der Aufgabenausgabe aus Storage

Wie bereits erwähnt, werden nicht von jeder Batch-Lösung genau diese Schritte ausgeführt, und es können auch erheblich mehr Schritte enthalten sein. In der Beispielanwendung *DotNetTutorial* werden die Prozesse veranschaulicht, die in einer Batch-Lösung häufig vorkommen.

## Erstellen des Beispielprojekts *DotNetTutorial*

Zur erfolgreichen Ausführung des Beispiels müssen Sie zunächst die Batch- und Storage-Kontoanmeldeinformationen in der Datei `Program.cs` des Projekts *DotNetTutorial* angeben. Falls die Projektmappe noch nicht geöffnet ist, öffnen Sie sie in Visual Studio, indem Sie auf die Projektmappendatei `DotNetTutorial.sln` doppelklicken. Alternativ können Sie die Projektmappe auch in Visual Studio über **Datei > Öffnen > Projekt/Projektmappe** öffnen.

Öffnen Sie `Program.cs` im Projekt *DotNetTutorial*. Fügen Sie Ihre Anmeldeinformationen dann so hinzu, wie sie im oberen Bereich der Datei angegeben sind:

```
// Update the Batch and Storage account credential strings below with the values
// unique to your accounts. These are used when constructing connection strings
// for the Batch and Storage client objects.

// Batch account credentials
private const string BatchAccountName = "";
private const string BatchAccountKey  = "";
private const string BatchAccountUrl  = "";

// Storage account credentials
private const string StorageAccountName = "";
private const string StorageAccountKey  = "";
```

> [AZURE.IMPORTANT] Wie bereits erwähnt, müssen momentan Anmeldeinformationen für ein Speicherkonto vom Typ **Allgemein** in Azure Storage angegeben werden. Ihre Batch-Anwendungen verwenden Blob Storage innerhalb des Speicherkontos vom Typ **Allgemein**. Geben Sie keine Anmeldeinformationen für ein Speicherkonto an, das mit dem Kontotyp *Blob Storage* erstellt wurde.

Die Kontoanmeldeinformationen für Batch und Storage finden Sie im [Azure-Portal][azure_portal] auf dem Kontoblatt des jeweiligen Diensts:

![Batch-Anmeldeinformationen im Portal][9] ![Storage-Anmeldeinformationen im Portal][10]<br/>

Nachdem Sie das Projekt mit Ihren Anmeldeinformationen aktualisiert haben, klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf die Projektmappe und klicken anschließend auf **Projektmappe erstellen**. Bestätigen Sie die Wiederherstellung von NuGet-Paketen, wenn Sie hierzu aufgefordert werden.

> [AZURE.TIP] Falls die NuGet-Pakete nicht automatisch wiederhergestellt werden oder ein Wiederherstellungsfehler auftritt, sollten Sie sich vergewissern, dass der [NuGet-Paket-Manager][nuget_packagemgr] installiert ist. Aktivieren Sie anschließend das Herunterladen fehlender Pakete. Informationen zum Aktivieren des Paketdownloads finden Sie unter [Enabling Package Restore During Build][nuget_restore] \(Aktivieren der Paketwiederherstellung während des Buildvorgangs).

In den folgenden Abschnitten unterteilen wir die Beispielanwendung in die Schritte, die ausgeführt werden, um eine Workload im Batch-Dienst zu verarbeiten. Diese werden dann ausführlich beschrieben. Es ist hilfreich, die geöffnete Projektmappe in Visual Studio zu verfolgen, während Sie den restlichen Text dieses Artikels durcharbeiten. Es wird nämlich nicht einzeln auf jede Codezeile des Beispiels eingegangen.

Navigieren Sie zum Anfang der `MainAsync`-Methode in der Datei `Program.cs` des Projekts *DotNetTutorial*, um mit Schritt 1 zu beginnen. Jeder nachfolgende Schritt entspricht dann im Wesentlichen dem Ablauf der Methodenaufrufe in `MainAsync`.

## Schritt 1: Erstellen von Storage-Containern

![Container in Azure Storage erstellen][1] <br/>

Batch enthält integrierte Unterstützung für die Interaktion mit Azure Storage. Über Container in Ihrem Storage-Konto werden die Dateien angegeben, die für die in Ihrem Batch-Konto ausgeführten Aufgaben erforderlich sind. Die Container stellen auch einen Ort zum Speichern von Ausgabedaten dar, die von den Aufgaben produziert werden. Als Erstes erstellt die Clientanwendung *DotNetTutorial* drei Container in [Azure Blob Storage](../storage/storage-introduction.md):

- **application:** Dieser Container dient zum Speichern der Anwendung, die von den Aufgaben ausgeführt wird, sowie aller Abhängigkeiten (etwa DLLs).
- **input:** Die zu verarbeitenden Datendateien werden von den Aufgaben aus dem Container *input* heruntergeladen.
- **output:** Nach Abschluss der Verarbeitung der Eingabedateien werden die Ergebnisse in den Container *output* hochgeladen.

Für die Interaktion mit einem Speicherkonto und die Containererstellung verwenden wir die [Azure Storage-Clientbibliothek für .NET][net_api_storage]. Wir erstellen einen Verweis auf das Konto mit [CloudStorageAccount][net_cloudstorageaccount] und erstellen auf dieser Grundlage ein [CloudBlobClient][net_cloudblobclient]-Element:

```csharp
// Construct the Storage account connection string
string storageConnectionString = String.Format(
    "DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}",
    StorageAccountName,
    StorageAccountKey);

// Retrieve the storage account
CloudStorageAccount storageAccount =
    CloudStorageAccount.Parse(storageConnectionString);

// Create the blob client, for use in obtaining references to
// blob storage containers
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
```

Der Verweis `blobClient` wird in der gesamten Anwendung verwendet und als Parameter an mehrere Methoden übergeben. Ein Beispiel hierfür finden Sie in dem Codeabschnitt, der direkt auf den obigen Code folgt. Darin wird `CreateContainerIfNotExistAsync` aufgerufen, um die eigentliche Erstellung der Container durchzuführen.

```csharp
// Use the blob client to create the containers in Azure Storage if they don't
// yet exist
const string appContainerName    = "application";
const string inputContainerName  = "input";
const string outputContainerName = "output";
await CreateContainerIfNotExistAsync(blobClient, appContainerName);
await CreateContainerIfNotExistAsync(blobClient, inputContainerName);
await CreateContainerIfNotExistAsync(blobClient, outputContainerName);
```

```csharp
private static async Task CreateContainerIfNotExistAsync(
    CloudBlobClient blobClient,
    string containerName)
{
		CloudBlobContainer container =
            blobClient.GetContainerReference(containerName);

		if (await container.CreateIfNotExistsAsync())
		{
				Console.WriteLine("Container [{0}] created.", containerName);
		}
		else
		{
				Console.WriteLine("Container [{0}] exists, skipping creation.",
                    containerName);
		}
}
```

Nachdem die Container erstellt wurden, kann die Anwendung die Dateien hochladen, die von den Aufgaben verwendet werden.

> [AZURE.TIP] [How to use Blob Storage from .NET](../storage/storage-dotnet-how-to-use-blobs.md) ermöglicht eine gute Übersicht über die Verwendung von Azure Storage-Containern und -Blobs. Es ist ratsam, sich diese Informationen zu Beginn der Nutzung von Batch durchzulesen.

## Schritt 2: Hochladen der Aufgabenanwendungs- und Datendateien

![Aufgabenanwendungs- und Eingabe(daten)dateien in Container hochladen][2] <br/>

Beim Vorgang für den Dateiupload definiert *DotNetTutorial* zuerst die Sammlungen mit den Dateipfaden für **application** und **input** auf Grundlage der auf dem lokalen Computer vorhandenen Dateipfade. Diese Dateien werden dann in die Container hochgeladen, die Sie im vorherigen Schritt erstellt haben.

```
// Paths to the executable and its dependencies that will be executed by the tasks
List<string> applicationFilePaths = new List<string>
{
    // The DotNetTutorial project includes a project reference to TaskApplication,
    // allowing us to determine the path of the task application binary dynamically
    typeof(TaskApplication.Program).Assembly.Location,
    "Microsoft.WindowsAzure.Storage.dll"
};

// The collection of data files that are to be processed by the tasks
List<string> inputFilePaths = new List<string>
{
    @"..\..\taskdata1.txt",
    @"..\..\taskdata2.txt",
    @"..\..\taskdata3.txt"
};

// Upload the application and its dependencies to Azure Storage. This is the
// application that will process the data files, and will be executed by each
// of the tasks on the compute nodes.
List<ResourceFile> applicationFiles = await UploadFilesToContainerAsync(
    blobClient,
    appContainerName,
    applicationFilePaths);

// Upload the data files. This is the data that will be processed by each of
// the tasks that are executed on the compute nodes within the pool.
List<ResourceFile> inputFiles = await UploadFilesToContainerAsync(
    blobClient,
    inputContainerName,
    inputFilePaths);
```

`Program.cs` enthält zwei am Uploadvorgang beteiligte Methoden:

- `UploadFilesToContainerAsync`: Diese Methode gibt eine Sammlung mit (weiter unten beschriebenen) [ResourceFile][net_resourcefile]-Objekten zurück und ruft intern `UploadFileToContainerAsync` auf, um die einzelnen Dateien hochzuladen, die im Parameter *filePaths* übergeben werden.
- `UploadFileToContainerAsync`: Dies ist die Methode, mit der die Dateien tatsächlich hochgeladen und die [ResourceFile][net_resourcefile]-Objekte erstellt werden. Nach dem Hochladen der Datei wird eine Shared Access Signature (SAS) für die Datei abgerufen, und es wird ein ResourceFile-Objekt zurückgegeben, das die Datei repräsentiert. Shared Access Signatures werden auch weiter unten beschrieben.

```
private static async Task<ResourceFile> UploadFileToContainerAsync(
    CloudBlobClient blobClient,
    string containerName,
    string filePath)
{
		Console.WriteLine(
            "Uploading file {0} to container [{1}]...", filePath, containerName);

		string blobName = Path.GetFileName(filePath);

		CloudBlobContainer container = blobClient.GetContainerReference(containerName);
		CloudBlockBlob blobData = container.GetBlockBlobReference(blobName);
		await blobData.UploadFromFileAsync(filePath, FileMode.Open);

		// Set the expiry time and permissions for the blob shared access signature.
        // In this case, no start time is specified, so the shared access signature
        // becomes valid immediately
		SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
		{
				SharedAccessExpiryTime = DateTime.UtcNow.AddHours(2),
				Permissions = SharedAccessBlobPermissions.Read
		};

		// Construct the SAS URL for blob
		string sasBlobToken = blobData.GetSharedAccessSignature(sasConstraints);
		string blobSasUri = String.Format("{0}{1}", blobData.Uri, sasBlobToken);

		return new ResourceFile(blobSasUri, blobName);
}
```

### ResourceFiles

Ein [ResourceFile][net_resourcefile]-Objekt stellt für Aufgaben in Batch die URL zu einer Datei in Azure Storage bereit, die vor dem Ausführen der Aufgabe auf einen Computeknoten heruntergeladen wird. Mit der [ResourceFile.BlobSource][net_resourcefile_blobsource]-Eigenschaft wird die vollständige URL der Datei so angegeben, wie sie in Azure Storage vorhanden ist. Die URL kann auch eine Shared Access Signature (SAS) enthalten, die sicheren Zugriff auf die Datei gewährt. Die meisten Aufgabentypen in Batch .NET enthalten eine *ResourceFiles*-Eigenschaft. Hierzu zählen:

- [CloudTask][net_task]
- [StartTask][net_pool_starttask]
- [JobPreparationTask][net_jobpreptask]
- [JobReleaseTask][net_jobreltask]

In der DotNetTutorial-Beispielanwendung werden die Aufgabentypen JobPreparationTask und JobReleaseTask nicht verwendet. Weitere Informationen hierzu finden Sie bei Bedarf unter [Ausführen von Auftragsvorbereitungs- und Auftragsabschlussaufgaben auf Azure Batch-Computeknoten](batch-job-prep-release.md).

### Shared Access Signature (SAS)

Bei Shared Access Signatures handelt es sich um Zeichenfolgen, die – bei Einbindung in eine URL – den sicheren Zugriff auf Container und Blobs in Azure Storage ermöglichen. Die Anwendung DotNetTutorial nutzt Shared Access Signature-URLs sowohl von Blobs als auch von Containern und veranschaulicht, wie diese Shared Access Signature-Zeichenfolgen aus dem Storage-Dienst abgerufen werden.

- **Blob-Shared Access Signatures:** In der StartTask-Aufgabe des Pools in DotNetTutorial werden Blob-Shared Access Signatures verwendet, wenn die Anwendungsbinärdateien und Eingabedatendateien aus Storage heruntergeladen werden (siehe Schritt 3 weiter unten). Die `UploadFileToContainerAsync`-Methode in der Datei `Program.cs` von DotNetTutorial enthält den Code, mit dem die SAS der einzelnen Blobs abgerufen wird. Hierzu wird [CloudBlob.GetSharedAccessSignature][net_sas_blob] aufgerufen.

- **Container-Shared Access Signatures:** Wenn die Arbeit einer Aufgabe auf dem Computeknoten abgeschlossen ist, wird die Ausgabedatei in Azure Storage in den Container *output* hochgeladen. Hierfür nutzt TaskApplication eine Container-Shared Access Signature, die im Rahmen des Pfads beim Hochladen der Datei Lese-/Schreibzugriff auf den Container ermöglicht. Das Abrufen der Container-Shared Access Signature ähnelt dem Abrufen der Blob-Shared Access Signature. In DotNetTutorial sehen Sie, dass die `GetContainerSasUrl`-Hilfsmethode für diesen Vorgang [CloudBlobContainer.GetSharedAccessSignature][net_sas_container] aufruft. „Schritt 6: Überwachen von Aufgaben“ enthält weitere Informationen dazu, wie TaskApplication die Container-Shared Access Signature verwendet.

> [AZURE.TIP] Sehen Sie sich die zweiteilige Reihe zu Shared Access Signatures an: [Teil 1: Grundlagen zum Shared Access Signature-Modell ](../storage/storage-dotnet-shared-access-signature-part-1.md) und [Teil 2: Erstellen und Verwenden einer SAS mit Blob Storage](../storage/storage-dotnet-shared-access-signature-part-2.md). Darin erhalten Sie weitere Informationen zur Bereitstellung des sicheren Zugriffs auf Daten in Ihrem Speicherkonto.

## Schritt 3: Erstellen eines Batch-Pools

![Batch-Pool erstellen][3] <br/>

Ein Batch-**Pool** ist eine Sammlung von Computeknoten (virtuellen Computern), auf denen Batch die Aufgaben eines Auftrags ausführt.

Nach dem Hochladen der Anwendung und der Datendateien in das Speicherkonto beginnt *DotNetTutorial* unter Verwendung der Batch .NET-Bibliothek die Interaktion mit dem Batch-Dienst. Hierfür wird zuerst ein [BatchClient][net_batchclient] erstellt:

```csharp
BatchSharedKeyCredentials cred = new BatchSharedKeyCredentials(
    BatchAccountUrl,
    BatchAccountName,
    BatchAccountKey);

using (BatchClient batchClient = BatchClient.Open(cred))
{
	...
```

Als Nächstes wird im Batch-Konto durch Aufrufen von `CreatePoolAsync` ein Pool mit Computeknoten erstellt. Von `CreatePoolAsync` wird die [BatchClient.PoolOperations.CreatePool][net_pool_create]-Methode verwendet, um tatsächlich einen Pool im Batch-Dienst zu erstellen.

```csharp
private static async Task CreatePoolAsync(
    BatchClient batchClient,
    string poolId,
    IList<ResourceFile> resourceFiles)
{
    Console.WriteLine("Creating pool [{0}]...", poolId);

    // Create the unbound pool. Until we call CloudPool.Commit() or CommitAsync(),
    // no pool is actually created in the Batch service. This CloudPool instance is
    // therefore considered "unbound," and we can modify its properties.
    CloudPool pool = batchClient.PoolOperations.CreatePool(
			poolId: poolId,
			targetDedicated: 3,           // 3 compute nodes
			virtualMachineSize: "small",  // single-core, 1.75 GB memory, 224 GB disk
			cloudServiceConfiguration:
			    new CloudServiceConfiguration(osFamily: "4")); // Win Server 2012 R2

    // Create and assign the StartTask that will be executed when compute nodes join
    // the pool. In this case, we copy the StartTask's resource files (that will be
    // automatically downloaded to the node by the StartTask) into the shared
    // directory that all tasks will have access to.
    pool.StartTask = new StartTask
    {
        // Specify a command line for the StartTask that copies the task application
        // files to the node's shared directory. Every compute node in a Batch pool
        // is configured with several pre-defined environment variables that you can
        // reference by using commands or applications run by tasks.

        // Since a successful execution of robocopy can return a non-zero exit code
        // (e.g. 1 when one or more files were successfully copied) we need to
        // manually exit with a 0 for Batch to recognize StartTask execution success.
        CommandLine = "cmd /c (robocopy %AZ_BATCH_TASK_WORKING_DIR% %AZ_BATCH_NODE_SHARED_DIR%) ^& IF %ERRORLEVEL% LEQ 1 exit 0",
        ResourceFiles = resourceFiles,
        WaitForSuccess = true
    };

    await pool.CommitAsync();
}
```

Beim Erstellen eines Pools mit [CreatePool][net_pool_create] müssen mehrere Parameter angegeben werden – etwa die Anzahl von Computeknoten, die [Größe der Knoten](../cloud-services/cloud-services-sizes-specs.md) und das Betriebssystem der Knoten. In *DotNetTutorial* verwenden wir [CloudServiceConfiguration][net_cloudserviceconfiguration], um Windows Server 2012 R2 über [Cloud Services](../cloud-services/cloud-services-guestos-update-matrix.md) anzugeben. Wenn Sie stattdessen ein [VirtualMachineConfiguration][net_virtualmachineconfiguration]-Element angeben, können Sie Pools mit Knoten erstellen, die mit Marketplace-Images (für Windows und Linux) erstellt wurden. Weitere Informationen finden Sie unter [Bereitstellen von Linux-Computeknoten in Azure Batch-Pools](batch-linux-nodes.md).

> [AZURE.IMPORTANT] Die Nutzung von Computeressourcen in Batch wird Ihnen berechnet. Zur Kostenminimierung können Sie `targetDedicated` auf 1 verringern, bevor Sie das Beispiel ausführen.

Zusammen mit diesen Eigenschaften des physischen Knotens können Sie auch eine [StartTask][net_pool_starttask]-Aufgabe für den Pool angeben. Die StartTask wird auf jedem Knoten ausgeführt, wenn dieser dem Pool hinzugefügt wird, sowie bei jedem Neustart eines Knotens. Besonders nützlich ist die StartTask zum Installieren von Anwendungen auf Computeknoten vor der Ausführung von Aufgaben. Wenn Daten für Ihre Aufgaben mit Python-Skripts verarbeitet werden, können Sie eine StartTask verwenden, um Python auf den Computeknoten zu installieren.

In dieser Beispielanwendung kopiert die StartTask-Aufgabe die Dateien, die aus Storage heruntergeladen (und mit der [ResourceFiles][net_starttask_resourcefiles]-Eigenschaft von [StartTask][net_starttask] angegeben) werden, aus dem StartTask-Arbeitsverzeichnis in das freigegebene Verzeichnis. Auf dieses Verzeichnis können *alle* auf dem Knoten ausgeführten Aufgaben zugreifen. Im Wesentlichen werden hiermit `TaskApplication.exe` und die dazugehörigen Abhängigkeiten in das freigegebene Verzeichnis jedes Knotens kopiert, wenn der Knoten dem Pool beitritt. So können alle Aufgaben, die auf dem Knoten ausgeführt werden, darauf zugreifen.

> [AZURE.TIP] Das Feature **Anwendungspakete** von Azure Batch ist eine andere Möglichkeit, um Ihre Anwendung auf die Computeknoten in einem Pool zu verteilen. Ausführliche Informationen finden Sie unter [Anwendungsbereitstellung mit Azure Batch-Anwendungspaketen](batch-application-packages.md).

Beachten Sie im obigen Codeausschnitt auch die Verwendung von zwei Umgebungsvariablen in der *CommandLine*-Eigenschaft von StartTask: `%AZ_BATCH_TASK_WORKING_DIR%` und `%AZ_BATCH_NODE_SHARED_DIR%`. Jeder Computeknoten in einem Batch-Pool wird automatisch mit verschiedenen Umgebungsvariablen konfiguriert, die speziell für Batch gelten. Für alle Prozesse, die von einer Aufgabe ausgeführt werden, besteht Zugriff auf diese Umgebungsvariablen.

> [AZURE.TIP] Weitere Informationen zu den Umgebungsvariablen, die auf Computeknoten in einem Batch-Pool verfügbar sind, und Informationen zu Arbeitsverzeichnissen von Aufgaben finden Sie in den Abschnitten [Umgebungseinstellungen für Tasks](batch-api-basics.md#environment-settings-for-tasks) und [Dateien und Verzeichnisse](batch-api-basics.md#files-and-directories) unter [Übersicht über Batch-Features für Entwickler](batch-api-basics.md).

## Schritt 4: Erstellen eines Batch-Auftrags

![Batch-Auftrag erstellen][4]<br/>

Ein Batch-**Auftrag** ist im Wesentlichen eine Sammlung von Aufgaben, die einem Pool mit Computeknoten zugeordnet sind. Die Aufgaben in einem Auftrag werden auf den Computeknoten des zugeordneten Pools ausgeführt.

Mit einem Auftrag wird nicht nur das Organisieren und Nachverfolgen von Aufgaben in zusammengehörenden Workloads ermöglicht, sondern es können auch bestimmte Beschränkungen festgelegt werden. Beispiele hierfür sind die maximale Laufzeit für den Auftrag (und somit auch seiner Aufgaben) sowie die Auftragspriorität in Bezug auf andere Aufträge im Batch-Konto. In diesem Beispiel ist die Aufgabe aber nur dem Pool zugeordnet, der in Schritt 3 erstellt wurde. Es werden keine zusätzlichen Eigenschaften konfiguriert.

Alle Batch-Aufträge sind einem bestimmten Pool zugeordnet. Diese Zuordnung gibt an, auf welchen Knoten die Aufgaben des Auftrags ausgeführt werden. Dies wird mit der [CloudJob.PoolInformation][net_job_poolinfo]-Eigenschaft angegeben (siehe Codeausschnitt weiter unten).

```csharp
private static async Task CreateJobAsync(
    BatchClient batchClient,
    string jobId,
    string poolId)
{
    Console.WriteLine("Creating job [{0}]...", jobId);

    CloudJob job = batchClient.JobOperations.CreateJob();
    job.Id = jobId;
    job.PoolInformation = new PoolInformation { PoolId = poolId };

    await job.CommitAsync();
}
```

Nachdem ein Auftrag erstellt wurde, werden Aufgaben zum Durchführen der Arbeitsschritte hinzugefügt.

## Schritt 5: Hinzufügen von Aufgaben zum Auftrag

![Aufgaben zu Auftrag hinzufügen][5]<br/> *(1) Die Aufgaben werden dem Auftrag hinzugefügt, (2) die Aufgaben werden für die Ausführung auf Knoten eingeplant, und (3) für die Aufgaben werden die zu verarbeitenden Datendateien heruntergeladen.*

Batch-**Aufgaben** sind die einzelnen Arbeitseinheiten, die auf den Computeknoten ausgeführt werden. Eine Aufgabe verfügt über eine Befehlszeile und führt die Skripts oder ausführbaren Dateien aus, die Sie in der Befehlszeile festlegen.

Zum eigentlichen Ausführen der Arbeitsschritte müssen die Aufgaben einem Auftrag hinzugefügt werden. Jede [CloudTask][net_task]-Aufgabe wird mit einer Befehlszeileneigenschaft und einem [ResourceFiles][net_task_resourcefiles]-Objekt (wie bei der StartTask-Aufgabe des Pools) konfiguriert, die vor dem automatischen Ausführen der Befehlszeile von der Aufgabe auf den Knoten heruntergeladen wird. Im Beispielprojekt *DotNetTutorial* verarbeitet jede Aufgabe nur eine Datei. Daher enthält die ResourceFiles-Sammlung in diesem Fall ein einzelnes Element.

```csharp
private static async Task<List<CloudTask>> AddTasksAsync(
    BatchClient batchClient,
    string jobId,
    List<ResourceFile> inputFiles,
    string outputContainerSasUrl)
{
    Console.WriteLine("Adding {0} tasks to job [{1}]...", inputFiles.Count, jobId);

    // Create a collection to hold the tasks that we'll be adding to the job
    List<CloudTask> tasks = new List<CloudTask>();

    // Create each of the tasks. Because we copied the task application to the
    // node's shared directory with the pool's StartTask, we can access it via
    // the shared directory on the node that the task runs on.
    foreach (ResourceFile inputFile in inputFiles)
    {
        string taskId = "topNtask" + inputFiles.IndexOf(inputFile);
        string taskCommandLine = String.Format(
            "cmd /c %AZ_BATCH_NODE_SHARED_DIR%\\TaskApplication.exe {0} 3 "{1}"",
            inputFile.FilePath,
            outputContainerSasUrl);

        CloudTask task = new CloudTask(taskId, taskCommandLine);
        task.ResourceFiles = new List<ResourceFile> { inputFile };
        tasks.Add(task);
    }

    // Add the tasks as a collection, as opposed to issuing a separate AddTask call
    // for each. Bulk task submission helps to ensure efficient underlying API calls
    // to the Batch service.
    await batchClient.JobOperations.AddTaskAsync(jobId, tasks);

    return tasks;
}
```

> [AZURE.IMPORTANT] Beim Zugreifen auf Umgebungsvariablen (beispielsweise `%AZ_BATCH_NODE_SHARED_DIR%`) oder bei der Ausführung einer Anwendung, die nicht im `PATH` des Knotens enthalten ist, müssen Aufgabenbefehlszeilen mit dem Präfix `cmd /c` versehen werden. Hiermit wird der Befehlsinterpreter explizit ausgeführt und angewiesen, den Vorgang nach dem Ausführen Ihres Befehls zu beenden. Diese Anforderung ist nicht relevant, falls von Ihren Aufgaben eine Anwendung im `PATH` des Knotens (etwa *robocopy.exe* oder *powershell.exe*) ausgeführt wird und keine Umgebungsvariablen verwendet werden.

In der `foreach`-Schleife im obigen Codeausschnitt sehen Sie, dass die Befehlszeile für die Aufgabe so erstellt wurde, dass drei Befehlszeilenargumente an *TaskApplication.exe* übergeben werden:

1. Das **erste Argument** ist der Pfad zu der Datei, die verarbeitet werden soll. Dies ist der lokale Pfad zu der Datei, wie sie auf dem Knoten vorhanden ist. Bei der obigen anfänglichen Erstellung des ResourceFile-Objekts in `UploadFileToContainerAsync` wurde der Dateiname für diese Eigenschaft verwendet (als Parameter für den ResourceFile-Konstruktor). Dies deutet darauf hin, dass sich die Datei im gleichen Verzeichnis wie *TaskApplication.exe* befindet.

2. Mit dem **zweiten Argument** wird angegeben, dass die obersten *N* Wörter in die Ausgabedatei geschrieben werden sollen. Im Beispiel ist dies hartcodiert, sodass die obersten drei Wörter in die Ausgabedatei geschrieben werden.

3. Das **dritte Argument** ist die Shared Access Signature (SAS), die Schreibzugriff auf den Container **output** in Azure Storage gewährt. Für *TaskApplication.exe* wird diese SAS-URL beim Hochladen der Ausgabedatei an Azure Storage verwendet. Den Code hierfür finden Sie im TaskApplication-Projekt in der `UploadFileToContainer`-Methode der Datei `Program.cs`:

```csharp
// NOTE: From project TaskApplication Program.cs

private static void UploadFileToContainer(string filePath, string containerSas)
{
		string blobName = Path.GetFileName(filePath);

		// Obtain a reference to the container using the SAS URI.
		CloudBlobContainer container = new CloudBlobContainer(new Uri(containerSas));

		// Upload the file (as a new blob) to the container
		try
		{
				CloudBlockBlob blob = container.GetBlockBlobReference(blobName);
				blob.UploadFromFile(filePath, FileMode.Open);

				Console.WriteLine("Write operation succeeded for SAS URL " + containerSas);
				Console.WriteLine();
		}
		catch (StorageException e)
		{

				Console.WriteLine("Write operation failed for SAS URL " + containerSas);
				Console.WriteLine("Additional error information: " + e.Message);
				Console.WriteLine();

				// Indicate that a failure has occurred so that when the Batch service
                // sets the CloudTask.ExecutionInformation.ExitCode for the task that
                // executed this application, it properly indicates that there was a
                // problem with the task.
				Environment.ExitCode = -1;
		}
}
```

## Schritt 6: Überwachen von Aufgaben

![Aufgaben überwachen][6]<br/> *Die Clientanwendung (1) überwacht für die Aufgaben den Status „Abschluss“ und „Erfolg“, und (2) Aufgaben laden Ergebnisdaten in Azure Storage hoch.*

Wenn einem Auftrag Aufgaben hinzugefügt werden, werden sie automatisch in die Warteschlange eingereiht und für die Ausführung auf Computeknoten in dem Pool eingeplant, der dem Auftrag zugeordnet ist. Basierend auf den Einstellungen, die Sie angeben, führt Batch das Einreihen, Planen und erneute Ausführen sowie andere Schritte der Aufgabenverwaltung für Sie durch. Es gibt viele Ansätze für die Überwachung der Aufgabenausführung. DotNetTutorial enthält ein einfaches Beispiel, bei dem nur der Abschluss und das Fehlschlagen oder der Erfolg von Aufgaben gemeldet wird.

Die `MonitorTasks`-Methode in der Datei `Program.cs` von DotNetTutorial enthält drei Batch .NET-Konzepte, auf die wir hier näher eingehen möchten. Sie sind unten in der Reihenfolge ihrer Darstellung aufgeführt:

1. **ODATADetailLevel:** [ODATADetailLevel][net_odatadetaillevel] muss bei Auflistungsvorgängen (etwa beim Abrufen einer Liste mit Aufgaben eines Auftrags) angegeben werden, um die Leistung der Batch-Anwendung zu optimieren. Machen Sie sich mit den Informationen unter [Effizientes Abfragen des Azure Batch-Diensts](batch-efficient-list-queries.md) vertraut, wenn Sie in Ihren Batch-Anwendungen eine Statusüberwachung verwenden möchten.

2. **TaskStateMonitor:** [TaskStateMonitor][net_taskstatemonitor] stellt Hilfsprogramme zum Überwachen des Aufgabenstatus für Batch .NET-Anwendungen bereit. Bei `MonitorTasks` wartet *DotNetTutorial*, bis alle Aufgaben innerhalb eines bestimmten Zeitraums den Status [TaskState.Completed][net_taskstate] erreicht haben. Anschließend wird der Auftrag beendet.

3. **TerminateJobAsync:** Wenn ein Auftrag mit [JobOperations.TerminateJobAsync][net_joboperations_terminatejob] \(oder durch das Blockieren von JobOperations.TerminateJob) beendet wird, wird der Auftrag als abgeschlossen gekennzeichnet. Dies ist wichtig, wenn für Ihre Batch-Lösung eine [JobReleaseTask][net_jobreltask]-Aufgabe verwendet wird. Dieser besondere Aufgabentyp wird unter [Vorbereitung und Abschluss von Aufträgen](batch-job-prep-release.md) beschrieben.

Hier ist die `MonitorTasks`-Methode aus der Datei `Program.cs` von *DotNetTutorial* angegeben:

```csharp
private static async Task<bool> MonitorTasks(
    BatchClient batchClient,
    string jobId,
    TimeSpan timeout)
{
    bool allTasksSuccessful = true;
    const string successMessage = "All tasks reached state Completed.";
    const string failureMessage = "One or more tasks failed to reach the Completed state within the timeout period.";

    // Obtain the collection of tasks currently managed by the job. Note that we use
    // a detail level to  specify that only the "id" property of each task should be
    // populated. Using a detail level for all list operations helps to lower
    // response time from the Batch service.
    ODATADetailLevel detail = new ODATADetailLevel(selectClause: "id");
    List<CloudTask> tasks =
        await batchClient.JobOperations.ListTasks(JobId, detail).ToListAsync();

    Console.WriteLine("Awaiting task completion, timeout in {0}...",
        timeout.ToString());

    // We use a TaskStateMonitor to monitor the state of our tasks. In this case, we
    // will wait for all tasks to reach the Completed state.
    TaskStateMonitor taskStateMonitor
        = batchClient.Utilities.CreateTaskStateMonitor();

    try
    {
        await taskStateMonitor.WhenAll(tasks, TaskState.Completed, timeout);
    }
    catch (TimeoutException)
    {
        await batchClient.JobOperations.TerminateJobAsync(jobId, failureMessage);
        Console.WriteLine(failureMessage);
        return false;
    }

    await batchClient.JobOperations.TerminateJobAsync(jobId, successMessage);

    // All tasks have reached the "Completed" state, however, this does not
    // guarantee all tasks completed successfully. Here we further check each task's
    // ExecutionInfo property to ensure that it did not encounter a scheduling error
    // or return a non-zero exit code.

    // Update the detail level to populate only the task id and executionInfo
    // properties. We refresh the tasks below, and need only this information for
    // each task.
    detail.SelectClause = "id, executionInfo";

    foreach (CloudTask task in tasks)
    {
        // Populate the task's properties with the latest info from the
        // Batch service
        await task.RefreshAsync(detail);

        if (task.ExecutionInformation.SchedulingError != null)
        {
            // A scheduling error indicates a problem starting the task on the node.
            // It is important to note that the task's state can be "Completed," yet
            // still have encountered a scheduling error.

            allTasksSuccessful = false;

            Console.WriteLine("WARNING: Task [{0}] encountered a scheduling error: {1}",
                task.Id,
                task.ExecutionInformation.SchedulingError.Message);
        }
        else if (task.ExecutionInformation.ExitCode != 0)
        {
            // A non-zero exit code may indicate that the application executed by
            // the task encountered an error during execution. As not every
            // application returns non-zero on failure by default (e.g. robocopy),
            // your implementation of error checking may differ from this example.

            allTasksSuccessful = false;

            Console.WriteLine("WARNING: Task [{0}] returned a non-zero exit code - this may indicate task execution or completion failure.", task.Id);
        }
    }

    if (allTasksSuccessful)
    {
        Console.WriteLine("Success! All tasks completed successfully within the specified timeout period.");
    }

    return allTasksSuccessful;
}
```

## Schritt 7: Herunterladen der Aufgabenausgabe

![Aufgabenausgabe aus Storage herunterladen][7]<br/>

Nachdem der Auftrag abgeschlossen wurde, kann die Ausgabe der Aufgaben aus Azure Storage heruntergeladen werden. Hierzu wird `DownloadBlobsFromContainerAsync` in der Datei `Program.cs` von *DotNetTutorial* aufgerufen:

```csharp
private static async Task DownloadBlobsFromContainerAsync(
    CloudBlobClient blobClient,
    string containerName,
    string directoryPath)
{
		Console.WriteLine("Downloading all files from container [{0}]...", containerName);

		// Retrieve a reference to a previously created container
		CloudBlobContainer container = blobClient.GetContainerReference(containerName);

		// Get a flat listing of all the block blobs in the specified container
		foreach (IListBlobItem item in container.ListBlobs(
                    prefix: null,
                    useFlatBlobListing: true))
		{
				// Retrieve reference to the current blob
				CloudBlob blob = (CloudBlob)item;

				// Save blob contents to a file in the specified folder
				string localOutputFile = Path.Combine(directoryPath, blob.Name);
				await blob.DownloadToFileAsync(localOutputFile, FileMode.Create);
		}

		Console.WriteLine("All files downloaded to {0}", directoryPath);
}
```

> [AZURE.NOTE] Durch den Aufruf von `DownloadBlobsFromContainerAsync` in der Anwendung *DotNetTutorial* wird angegeben, dass die Dateien in den Ordner `%TEMP%` heruntergeladen werden sollen. Sie können diesen Ausgabespeicherort ändern.

## Schritt 8: Löschen von Containern

Da Ihnen Daten berechnet werden, die sich in Azure Storage befinden, ist Folgendes ratsam: Entfernen Sie alle Blobs, die für Ihre Batch-Aufträge nicht mehr benötigt werden. In der DotNetTutorial-Datei `Program.cs` wird dies mit drei Aufrufen der `DeleteContainerAsync`-Hilfsmethode erreicht:

```csharp
// Clean up Storage resources
await DeleteContainerAsync(blobClient, appContainerName);
await DeleteContainerAsync(blobClient, inputContainerName);
await DeleteContainerAsync(blobClient, outputContainerName);
```

Die Methode selbst ruft nur einen Verweis auf den Container ab und anschließend [CloudBlobContainer.DeleteIfExistsAsync][net_container_delete] auf:

```csharp
private static async Task DeleteContainerAsync(
    CloudBlobClient blobClient,
    string containerName)
{
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);

    if (await container.DeleteIfExistsAsync())
    {
        Console.WriteLine("Container [{0}] deleted.", containerName);
    }
    else
    {
        Console.WriteLine("Container [{0}] does not exist, skipping deletion.",
            containerName);
    }
}
```

## Schritt 9: Löschen des Auftrags und des Pools

Im letzten Schritt wird der Benutzer aufgefordert, den Auftrag und den Pool zu löschen, die von der Anwendung DotNetTutorial erstellt wurden. Für die Aufträge und Aufgaben fallen zwar keine Kosten an, *für Computeknoten dagegen schon*. Daher empfehlen wir Ihnen, Knoten nur bei Bedarf zuzuordnen. Sie können das Löschen von nicht verwendeten Pools beispielsweise im Rahmen Ihres Wartungsprozesses durchführen.

[JobOperations][net_joboperations] und [PoolOperations][net_pooloperations] von BatchClient verfügen jeweils über entsprechende Löschmethoden, die aufgerufen werden, wenn der Benutzer das Löschen bestätigt:

```csharp
// Clean up the resources we've created in the Batch account if the user so chooses
Console.WriteLine();
Console.WriteLine("Delete job? [yes] no");
string response = Console.ReadLine().ToLower();
if (response != "n" && response != "no")
{
    await batchClient.JobOperations.DeleteJobAsync(JobId);
}

Console.WriteLine("Delete pool? [yes] no");
response = Console.ReadLine();
if (response != "n" && response != "no")
{
    await batchClient.PoolOperations.DeletePoolAsync(PoolId);
}
```

> [AZURE.IMPORTANT] Denken Sie daran, dass für Computeressourcen Kosten anfallen – und dass Sie durch das Löschen nicht verwendeter Pools Kosten sparen können. Beachten Sie außerdem, dass beim Löschen eines Pools alle Computeknoten dieses Pools gelöscht werden und dass alle Daten auf den Knoten nicht mehr wiederhergestellt werden können, nachdem der Pool gelöscht wurde.

## Ausführen des Beispiels *DotNetTutorial*

Beim Ausführen der Beispielanwendung ähnelt die Konsolenausgabe folgender Ausgabe: Bei der Ausführung kommt es bei `Awaiting task completion, timeout in 00:30:00...` zu einer Pause, während die Computeknoten des Pools gestartet werden. Verwenden Sie das [Azure-Portal][azure_portal], um den Pool, die Computeknoten, den Auftrag und die Aufgaben während und nach der Ausführung zu überwachen. Verwenden Sie das [Azure-Portal][azure_portal] oder den [Azure-Speicher-Explorer][storage_explorers], um die von der Anwendung erstellten Speicherressourcen (Container und Blobs) anzuzeigen.

Die normale Ausführungsdauer beträgt **ca. fünf Minuten**, wenn die Anwendung in der Standardkonfiguration ausgeführt wird.

```
Sample start: 1/8/2016 09:42:58 AM

Container [application] created.
Container [input] created.
Container [output] created.
Uploading file C:\repos\azure-batch-samples\CSharp\ArticleProjects\DotNetTutorial\bin\Debug\TaskApplication.exe to container [application]...
Uploading file Microsoft.WindowsAzure.Storage.dll to container [application]...
Uploading file ..\..\taskdata1.txt to container [input]...
Uploading file ..\..\taskdata2.txt to container [input]...
Uploading file ..\..\taskdata3.txt to container [input]...
Creating pool [DotNetTutorialPool]...
Creating job [DotNetTutorialJob]...
Adding 3 tasks to job [DotNetTutorialJob]...
Awaiting task completion, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Downloading all files from container [output]...
All files downloaded to C:\Users\USERNAME\AppData\Local\Temp
Container [application] deleted.
Container [input] deleted.
Container [output] deleted.

Sample end: 1/8/2016 09:47:47 AM
Elapsed time: 00:04:48.5358142

Delete job? [yes] no: yes
Delete pool? [yes] no: yes

Sample complete, hit ENTER to exit...
```

## Nächste Schritte

Nehmen Sie Änderungen an *DotNetTutorial* und *TaskApplication* vor, um mit unterschiedlichen Computeszenarien zu experimentieren. Versuchen Sie beispielsweise, *TaskApplication* eine Ausführungsverzögerung (etwa mit [Thread.Sleep][net_thread_sleep]) hinzuzufügen, um Aufgaben mit langer Ausführungsdauer zu simulieren und im Portal zu überwachen. Versuchen Sie, weitere Aufgaben hinzuzufügen oder die Anzahl von Computeknoten anzupassen. Fügen Sie Logik hinzu, um das Vorhandensein eines Pools zu prüfen und die Verwendung dieses Pools zu ermöglichen. Das Ziel hierbei ist die Verringerung der Ausführungsdauer. (*Hinweis*: Sehen Sie sich die Datei `ArticleHelpers.cs` im Projekt [Microsoft.Azure.Batch.Samples.Common][github_samples_common] unter [azure-batch-samples][github_samples] an.)

Nachdem Sie sich jetzt mit dem grundlegenden Workflow einer Batch-Lösung vertraut gemacht haben, können wir uns mit den zusätzlichen Features des Batch-Diensts beschäftigen.

- Lesen Sie sich [Übersicht über Batch-Features für Entwickler](batch-api-basics.md) durch. Dies ist für alle neuen Batch-Benutzer zu empfehlen.
- Beginnen Sie mit den anderen Artikeln zur Batch-Entwicklung unter **Entwicklung im Detail** im [Lernpfad für Batch][batch_learning_path].
- Sehen Sie sich eine andere Implementierung einer Verarbeitung der Workload „Oberste N Wörter“ mit Batch im Beispiel [TopNWords][github_topnwords] an.

[azure_batch]: https://azure.microsoft.com/services/batch/
[azure_free_account]: https://azure.microsoft.com/free/
[azure_portal]: https://portal.azure.com
[batch_learning_path]: https://azure.microsoft.com/documentation/learning-paths/batch/
[github_batchexplorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[github_dotnettutorial]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/DotNetTutorial
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_common]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/Common
[github_samples_zip]: https://github.com/Azure/azure-batch-samples/archive/master.zip
[github_topnwords]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/TopNWords
[net_api]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[net_api_storage]: https://msdn.microsoft.com/library/azure/mt347887.aspx
[net_batchclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudblobclient]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobclient.aspx
[net_cloudblobcontainer]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobcontainer.aspx
[net_cloudstorageaccount]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.cloudstorageaccount.aspx
[net_cloudserviceconfiguration]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudserviceconfiguration.aspx
[net_container_delete]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobcontainer.deleteifexistsasync.aspx
[net_job]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_job_poolinfo]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.protocol.models.cloudjob.poolinformation.aspx
[net_joboperations]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.joboperations
[net_joboperations_terminatejob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.terminatejobasync.aspx
[net_jobpreptask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx
[net_jobreltask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobreleasetask.aspx
[net_node]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.aspx
[net_odatadetaillevel]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_pool_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[net_pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_pooloperations]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.pooloperations
[net_resourcefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.aspx
[net_resourcefile_blobsource]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.blobsource.aspx
[net_sas_blob]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.blob.cloudblob.getsharedaccesssignature.aspx
[net_sas_container]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.blob.cloudblobcontainer.getsharedaccesssignature.aspx
[net_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.aspx
[net_starttask_resourcefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.resourcefiles.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_task_resourcefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.resourcefiles.aspx
[net_taskstate]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.common.taskstate.aspx
[net_taskstatemonitor]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskstatemonitor.aspx
[net_thread_sleep]: https://msdn.microsoft.com/library/274eh01d(v=vs.110).aspx
[net_virtualmachineconfiguration]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.virtualmachineconfiguration.aspx
[nuget_packagemgr]: https://docs.nuget.org/consume/installing-nuget
[nuget_restore]: https://docs.nuget.org/consume/package-restore/msbuild-integrated#enabling-package-restore-during-build
[storage_explorers]: http://storageexplorer.com/
[visual_studio]: https://www.visualstudio.com/products/vs-2015-product-editions

[1]: ./media/batch-dotnet-get-started/batch_workflow_01_sm.png "Container in Azure Storage erstellen"
[2]: ./media/batch-dotnet-get-started/batch_workflow_02_sm.png "Aufgabenanwendungs- und Eingabe(daten)dateien in Container hochladen"
[3]: ./media/batch-dotnet-get-started/batch_workflow_03_sm.png "Batch-Pool erstellen"
[4]: ./media/batch-dotnet-get-started/batch_workflow_04_sm.png "Batch-Auftrag erstellen"
[5]: ./media/batch-dotnet-get-started/batch_workflow_05_sm.png "Aufgaben zu Auftrag hinzufügen"
[6]: ./media/batch-dotnet-get-started/batch_workflow_06_sm.png "Aufgaben überwachen"
[7]: ./media/batch-dotnet-get-started/batch_workflow_07_sm.png "Aufgabenausgabe aus Storage herunterladen"
[8]: ./media/batch-dotnet-get-started/batch_workflow_sm.png "Batch-Lösungsworkflow (vollständiges Diagramm)"
[9]: ./media/batch-dotnet-get-started/credentials_batch_sm.png "Batch-Anmeldeinformationen im Portal"
[10]: ./media/batch-dotnet-get-started/credentials_storage_sm.png "Storage-Anmeldeinformationen im Portal"
[11]: ./media/batch-dotnet-get-started/batch_workflow_minimal_sm.png "Batch-Lösungsworkflow (reduziertes Diagramm)"

<!---HONumber=AcomDC_0817_2016-->