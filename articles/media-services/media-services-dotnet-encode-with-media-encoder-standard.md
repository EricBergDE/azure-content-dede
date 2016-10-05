<properties 
	pageTitle="Codieren eines Medienobjekts mit Media Encoder Standard mit .NET | Microsoft Azure" 
	description="In diesem Thema wird veranschaulicht, wie Sie .NET mithilfe von Media Encoder Standard verwenden, um ein Asset zu codieren." 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
 	ms.date="09/19/2016"
	ms.author="juliako;anilmur"/>


# Codieren eines Medienobjekts mit Media Encoder Standard mit .NET

Die Codierung ist einer der häufigsten Verarbeitungsvorgänge in Media Services. Sie erstellen Codierungsaufträge, um Mediendateien von einer Codierung in eine andere zu konvertieren. Zum Codieren können Sie den in Media Services integrierten Media Encoder verwenden. Außerdem können Sie Codierer von Media Services-Partnern verwenden. Externe Codierer finden Sie im Azure Marketplace.

In diesem Thema wird veranschaulicht, wie Sie .NET verwenden, um Medienobjekte mithilfe von Media Encoder Standard (MES) zu codieren. Media Encoder Standard wird mithilfe von Encoder-Voreinstellungen konfiguriert, die [hier](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409) beschrieben sind.

Es wird empfohlen, Zwischendateien immer in einen MP4-Satz mit adaptiver Bitrate zu codieren und anschließend mithilfe der [dynamischen Paketerstellung](media-services-dynamic-packaging-overview.md) in das gewünschte Format zu konvertieren. Um dynamische Paketerstellung nutzen zu können, ist mindestens eine bedarfsgesteuerte Streamingeinheit für den Streamingendpunkt erforderlich, aus dem die Inhalte bereitgestellt werden sollen. Weitere Informationen finden Sie unter [Skalieren von Media Services](media-services-portal-manage-streaming-endpoints.md).

Wenn Ihr Ausgabemedienobjekt speicherverschlüsselt ist, müssen Sie die Übermittlungsrichtlinien für Medienobjekte konfigurieren. Weitere Informationen finden Sie unter [Konfigurieren von Übermittlungsrichtlinien für Medienobjekte](media-services-dotnet-configure-asset-delivery-policy.md).

>[AZURE.NOTE]MES erzeugt eine Ausgabedatei mit einem Namen, der die ersten 32 Zeichen des Namens der Eingabedatei enthält. Der Name basiert auf der Angabe in der Voreinstellungsdatei. Beispiel: "FileName": "{Basename}\_{Index}{Extension}". {Basename} wird durch die ersten 32 Zeichen des Namens der Eingabedatei ersetzt.

###MES-Formate

[Formate und Codecs](media-services-media-encoder-standard-formats.md)

###MES-Voreinstellungen

Media Encoder Standard wird mithilfe von Encoder-Voreinstellungen konfiguriert, die [hier](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409) beschrieben sind.

###Eingabe- und Ausgabemetadaten

Wenn Sie Eingabemedienobjekte mithilfe von MES codieren, erhalten Sie bei erfolgreichem Abschluss dieser Codieraufgabe ein Ausgabemedienobjekt. Das Ausgabeasset enthält Video- und Audioinhalte, Miniaturansichten, das Manifest usw. Dies hängt von den verwendeten Codierungsvoreinstellungen ab.

Das Ausgabeasset enthält auch eine Datei mit Metadaten zum Eingabeasset. Der Name der XML-Metadatendatei weist das folgende Format auf: <Medienobjekt-ID>\_metadata.xml (z.B. „41114ad3-eb5e-4c57-8d92-5354e2b7d4a4\_metadata.xml“), wobei <Medienobjekt-ID> der „AssetId“-Wert des Eingabemedienobjekts ist. Das Schema dieser XML-Eingabemetadaten ist [hier](http://msdn.microsoft.com/library/azure/dn783120.aspx) beschrieben.

Das Ausgabeasset enthält auch eine Datei mit Metadaten zum Ausgabeasset. Der Name der XML-Metadatendatei weist das folgende Format auf: <source_file_name>\_manifest.xml (z.B. „BigBuckBunny\_manifest.xml“). Das Schema dieser XML-Ausgabemetadaten ist [hier](http://msdn.microsoft.com/library/azure/dn783217.aspx) beschrieben.

Wenn Sie eine der beiden Metadatendateien untersuchen möchten, können Sie einen SAS-Locator erstellen und die Datei auf Ihren lokalen Computer herunterladen. Ein Beispiel zum Erstellen eines SAS-Locators mit der Möglichkeit zum Herunterladen der Datei finden Sie in den .NET SDK-Erweiterungen für Media Services.

##Beispiel herunterladen

Laden Sie ein Beispiel von [hier](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/) herunter, und führen Sie es aus.

##Beispiel

Im folgenden Codebeispiel wird das Media Services-.NET-SDK verwendet, um die folgenden Aufgaben auszuführen:

- Erstellen eines Codierungsauftrags.
- Abrufen eines Verweises auf den Media Encoder Standard-Encoder
- Angeben, dass die Voreinstellung „H264 Multiple Bitrate 720p“ verwendet werden soll. Alle Voreinstellungen finden Sie [hier](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409). Sie können sich auch das Schema ansehen, dem diese Voreinstellungen entsprechen müssen. Klicken Sie dazu [hier](https://msdn.microsoft.com/library/mt269962.aspx).
- Fügen Sie eine einzelne Codierungsaufgabe zum Auftrag hinzu.
- Geben Sie das zu codierende Asset an.
- Erstellen Sie ein Ausgabeasset, das das codierte Asset enthalten soll.
- Fügen Sie einen Ereignishandler hinzu, um den Auftragsstatus zu überprüfen.
- Übermitteln des Auftrags.
		
		static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
		{
		    // Declare a new job.
		    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
		    // Get a media processor reference, and pass to it the name of the 
		    // processor to use for the specific task.
		    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");
		

		    // Create a task with the encoding details, using a string preset.
		    // In this case "H264 Multiple Bitrate 720p" preset is used.
		    ITask task = job.Tasks.AddNew("My encoding task",
		        processor,
		        "H264 Multiple Bitrate 720p",
		        TaskOptions.None);
		
		    // Specify the input asset to be encoded.
		    task.InputAssets.Add(asset);
		    // Add an output asset to contain the results of the job. 
		    // This output is specified as AssetCreationOptions.None, which 
		    // means the output asset is not encrypted. 
		    task.OutputAssets.AddNew("Output asset",
		        AssetCreationOptions.None);
		
		    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
		    job.Submit();
		    job.GetExecutionProgressTask(CancellationToken.None).Wait();
		
		    return job.OutputMediaAssets[0];
		}
		
		private static void JobStateChanged(object sender, JobStateChangedEventArgs e)
		{
		    Console.WriteLine("Job state changed event:");
		    Console.WriteLine("  Previous state: " + e.PreviousState);
		    Console.WriteLine("  Current state: " + e.CurrentState);
		    switch (e.CurrentState)
		    {
		        case JobState.Finished:
		            Console.WriteLine();
		            Console.WriteLine("Job is finished. Please wait while local tasks or downloads complete...");
		            break;
		        case JobState.Canceling:
		        case JobState.Queued:
		        case JobState.Scheduled:
		        case JobState.Processing:
		            Console.WriteLine("Please wait...\n");
		            break;
		        case JobState.Canceled:
		        case JobState.Error:
		
		            // Cast sender as a job.
		            IJob job = (IJob)sender;
		
		            // Display or log error details as needed.
		            break;
		        default:
		            break;
		    }
		}
		
		
		private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
		{
		    var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
		    ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();
		
		    if (processor == null)
		        throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));
		
		    return processor;
		}


##Media Services-Lernpfade

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Feedback geben

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

##Weitere Informationen 

[Erstellen von Miniaturansichten über Media Encoder Standard mit .NET](media-services-dotnet-generate-thumbnail-with-mes.md) [Media Services-Codierung (Übersicht)](media-services-encode-asset.md)

<!---HONumber=AcomDC_0921_2016-->