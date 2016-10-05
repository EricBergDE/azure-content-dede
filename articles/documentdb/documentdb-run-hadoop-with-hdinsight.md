<properties
	pageTitle="Ausführen eines Hadoop-Auftrags mit DocumentDB und HDInsight | Microsoft Azure"
	description="Erfahren Sie, wie Sie einen einfachen Hive-, Pig- und MapReduce-Auftrag mit DocumentDB und Azure HDInsight ausführen können."
	services="documentdb"
	authors="AndrewHoh"
	manager="jhubbard"
	editor="mimig"
	documentationCenter=""/>


<tags
	ms.service="documentdb"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="java"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="anhoh"/>

#<a name="DocumentDB-HDInsight"></a>Ausführen eines Hadoop-Auftrags mit DocumentDB und HDInsight

In diesem Lernprogramm wird gezeigt, wie Sie [Apache Hive][apache-hive]-, [Apache Pig][apache-pig]- und [Apache Hadoop][apache-hadoop]-MapReduce-Aufträge in Azure HDInsight mit dem Hadoop-Connector von DocumentDB ausführen können. Der Hadoop-Connector der DocumentDB ermöglicht es dieser, sowohl als Quelle als auch als Senke für Hive-, Pig- und MapReduce-Jobs zu fungieren. In diesem Lernprogramm wird DocumentDB sowohl als Datenquelle als auch als Ziel für Hadoop-Jobs verwendet.

Nach Abschluss dieses Lernprogramms können Sie die folgenden Fragen beantworten:

- Wie werden Daten von DocumentDB mithilfe eines Hive-, Pig- oder MapReduce-Jobs geladen?
- Wie werden Daten in DocumentDB mit einem Hive-, Pig- oder MapReduce-Job gespeichert?

Es wird empfohlen, dass Sie sich zunächst das folgende Video anschauen, in dem die Bearbeitung eines Hive-Jobs mit DocumentDB und HDInsight beschrieben wird.

> [AZURE.VIDEO use-azure-documentdb-hadoop-connector-with-azure-hdinsight]

Kehren Sie anschließend zu diesem Artikel zurück. Hier erhalten Sie umfassende Informationen darüber, wie Sie Analysejobs für die DocumentDB-Daten ausführen können.

> [AZURE.TIP] In diesem Lernprogramm wird davon ausgegangen, dass Sie bereits über Erfahrungen im Umgang mit Apache Hadoop, Hive, und/oder Pig verfügen. Wenn Sie noch nicht mit Apache Hadoop, Hive und Pig vertraut sind, empfiehlt es sich, die [Apache Hadoop-Dokumentation][apache-hadoop-doc] zu lesen. In diesem Lernprogramm wird ebenfalls davon ausgegangen, dass Sie bereits über Erfahrung mit DocumentDB sowie ein DocumentDB-Konto verfügen. Wenn Sie noch nicht mit DocumentDB vertraut sind oder noch kein DocumentDB-Konto besitzen, erhalten Sie weitergehende Informationen auf der Seite [Erste Schritte][getting-started].

Sie haben nicht die Zeit, das gesamte Lernprogramm zu bearbeiten und möchten lediglich sämtliche PowerShell-Beispielskripts zu Hive, Pig und MapReduce abrufen? Kein Problem. Sie erhalten diese [hier][documentdb-hdinsight-samples]. Der Download umfasst außerdem die Hql-, Pig- und Java-Dateien zu diesen Beispielen.

## <a name="NewestVersion"></a>Neueste Version

<table border='1'>
	<tr><th>Hadoop-Connector-Version</th>
		<td>1.2.0</td></tr>
	<tr><th>Skript-URI</th>
		<td>https://portalcontent.blob.core.windows.net/scriptaction/documentdb-hadoop-installer-v04.ps1</td></tr>
	<tr><th>Änderungsdatum</th>
		<td>26.04.2016</td></tr>
	<tr><th>Unterstützte HDInsight-Versionen</th>
		<td>3.1, 3.2.</td></tr>
	<tr><th>Änderungsprotokoll</th>
		<td>DocumentDB-Java-SDK auf 1.6.0 aktualisiert</br>
			Unterstützung für partitionierte Sammlungen als Quelle und Senke hinzugefügt</br>
		</td></tr>
</table>

## <a name="Prerequisites"></a>Voraussetzungen
Vor dem Ausführen der Anweisungen zu diesem Lernprogramm, müssen Sie sicherstellen, dass Sie über Folgendes verfügen:

- Ein DocumentDB-Konto, eine Datenbank und eine Auflistung mit Dokumenten. Weitere Informationen finden Sie unter [Erste Schritte mit DocumentDB][getting-started]. Importieren von Beispieldaten in Ihr DocumentDB-Konto mit dem [DocumentDB-Importtool][documentdb-import-data].
- Durchsatz. Lese- und Schreibvorgänge aus HDInsight werden gegen die zugeteilten Anforderungseinheiten Ihrer Sammlungen aufgerechnet. Weitere Informationen finden Sie unter [Bereitgestellter Durchsatz, Anforderungseinheiten und Datenbankvorgänge][documentdb-manage-throughput].
- Kapazität für eine zusätzliche gespeicherte Prozedur innerhalb einer jeden Ausgabeauflistung. Die gespeicherten Prozeduren werden für die Übertragung von resultierenden Dokumenten verwendet. Weitere Informationen finden Sie unter [Sammlungen und bereitgestellter Durchsatz][documentdb-manage-document-storage].
- Kapazität für die resultierenden Dokumente aus den Hive-, Pig- oder MapReduce-Jobs. Weitere Informationen finden Sie unter [Verwalten der Kapazität und Leistung von DocumentDB][documentdb-manage-collections].
- [*Optional*] Kapazität für eine zusätzliche Sammlung. Weitere Informationen finden Sie unter [Bereitgestellter Dokumentspeicher und Index-Overhead][documentdb-manage-document-storage].

> [AZURE.WARNING] Um die Erstellung einer neuen Sammlung während eines Auftrags zu vermeiden, können Sie entweder die Ergebnisse über "stdout" drucken, die Ausgabe im WASB-Container speichern oder eine bereits vorhandene Sammlung angeben. Bei Angabe einer vorhandenen Sammlung werden neue Dokumente innerhalb der Sammlung erstellt. Bereits vorhandene Dokumente sind nur dann betroffen, wenn ein Konflikt in *IDs* vorliegt. **Der Connector überschreibt vorhandene Dokumente mit ID-Konflikten automatisch**. Sie können dieses Feature deaktivieren, indem Sie die Upsert-Option auf "false" (falsch) festlegen. Wenn Upsert falsch ist und ein Konflikt auftritt, schlägt der Hadoop-Job fehl. Daraufhin wird ein Bericht über den ID-Konfliktfehler gesendet.


## <a name="ProvisionHDInsight"></a>Schritt 1: Erstellen eines neuen HDInsight-Clusters
Dieses Tutorial verwendet Skriptaktionen aus dem Azure-Portal, um Ihren HDInsight-Cluster anzupassen. In diesem Tutorial wird das Azure-Portal zum Erstellen Ihres HDInsight-Clusters verwendet. Anweisungen bezüglich der Verwendung von PowerShell-Cmdlets oder von HDInsight .NET SDK finden Sie im Artikel[ Anpassen von HDInsight-Clustern mithilfe von Skriptaktionen][hdinsight-custom-provision].

1. Melden Sie sich beim [Azure-Portal][azure-portal] an.

2. Klicken Sie oben im linken Navigationsbereich auf **+ Neu**, und suchen Sie auf dem Blatt „Neu“ in der oberen Suchleiste nach **HDInsight**.

3. Als erstes Ergebnis wird **HDInsight** (veröffentlicht von **Microsoft**) angezeigt. Klicken Sie auf das Ergebnis und anschließend auf **Erstellen**.

4. Geben Sie auf dem Blatt „Neuer HDInsight-Cluster“ den Namen Ihres Clusters ein, und wählen Sie das **Abonnement** aus, unter dem Sie diese Ressource bereitstellen möchten.

	<table border='1'>
		<tr><td>Clustername</td><td>Der Name des Clusters.<br/>
		Der DNS-Name muss mit einem alphanumerischen Zeichen beginnen und enden. Er darf zudem Bindestriche enthalten.<br/>
		Das Feld muss eine Zeichenfolge darstellen und zwischen 3 und 63 Zeichen aufweisen.</td></tr>
		<tr><td>Abonnementname</td>
			<td>Falls Sie über mehrere Azure-Abonnements verfügen, wählen Sie das Abonnement aus, das als Host für Ihren HDInsight-Cluster fungiert. </td></tr>
	</table>

5. Klicken Sie auf **Clustertyp auswählen**, und legen Sie die folgenden Eigenschaften auf die angegebenen Werte fest:

	<table border='1'>
		<tr><td>Clustertyp</td><td><strong>Hadoop</strong></td></tr>
		<tr><td>Cluster tier (Clustertarif)</td><td><strong>Standard</strong></td></tr>
		<tr><td>Betriebssystem</td><td><strong>Windows</strong></td></tr>
		<tr><td>Version</td><td>Aktuelle Version</td></tr>
	</table>

	Klicken Sie nun auf **AUSWÄHLEN**.

	![Bereitstellen von ersten Hadoop HDInsight Clusterdetails][image-customprovision-page1]

6. Klicken Sie auf **Anmeldeinformationen**, um Ihre Anmeldeinformationen für die Anmeldung und den Remotezugriff festzulegen. Wählen Sie Ihren Benutzernamen und Ihr Kennwort für die Clusteranmeldung.

	Wenn Sie remote auf den Cluster zugreifen möchten, wählen Sie im unteren Bereich des Blatts die Option *Ja* aus, und geben Sie einen Benutzernamen und ein Kennwort an.

7. Klicken Sie auf **Datenquelle**, um Ihren primären Standort für den Datenzugriff festzulegen. Wählen Sie die **Auswahlmethode** aus, und geben Sie ein bereits vorhandenes Speicherkonto an, oder erstellen Sie ein neues.

8. Geben Sie auf dem gleichen Blatt einen **Standardcontainer** und einen **Ort** an. Klicken Sie anschließend auf **Auswählen**.

	> [AZURE.NOTE] Wählen Sie aus Leistungsgründen einen Ort aus, der sich in der Nähe der Region Ihres DocumentDB-Konto befindet.

8. Klicken Sie auf **Preise**, und wählen Sie Anzahl und Art der Knoten aus. Sie können die Standardkonfiguration beibehalten und die Anzahl von Workerknoten später skalieren.

9. Klicken Sie auf **Optionale Konfiguration** und anschließend auf dem Blatt mit der optionalen Konfiguration auf **Skriptaktionen**.

	Geben Sie unter „Skriptaktionen“ die folgenden Informationen ein, um Ihren HDInsight-Cluster anzupassen:

	<table border='1'>
		<tr><th>Eigenschaft</th><th>Wert</th></tr>
		<tr><td>Name</td>
			<td>Geben Sie einen Namen für die Skriptaktion an.</td></tr>
		<tr><td>Skript-URI</td>
			<td>Geben Sie den URI für das Skript an, das aufgerufen wird, um den Cluster anzupassen.</br></br>
			Geben Sie Folgendes ein: </br> <strong>https://portalcontent.blob.core.windows.net/scriptaction/documentdb-hadoop-installer-v04.ps1</strong>.</td></tr>
		<tr><td>Hauptknoten</td>
			<td>Klicken Sie auf das Kontrollkästchen, um das PowerShell-Skript auf dem Hauptknoten auszuführen.</br></br>
			<strong>Aktivieren Sie dieses Kontrollkästchen.</strong></td></tr>
		<tr><td>Worker</td>
			<td>Klicken Sie auf das Kontrollkästchen, um das PowerShell-Skript auf dem Workerknoten auszuführen.</br></br>
			<strong>Aktivieren Sie dieses Kontrollkästchen.</strong></td></tr>
		<tr><td>Zookeeper</td>
			<td>Klicken Sie auf das Kontrollkästchen, um das PowerShell-Skript auf dem Zookeeper-Knoten auszuführen.</br></br>
			<strong>Nicht erforderlich.</strong>
			</td></tr>
		<tr><td>Parameter</td>
			<td>Geben Sie die Parameter an, wenn dies für das Skript erforderlich ist.</br></br>
			<strong>Keine Parameter erforderlich</strong>.</td></tr>
	</table>

10. Erstellen Sie entweder eine neue **Ressourcengruppe**, oder verwenden Sie eine vorhandene Ressourcengruppe unter Ihrem Azure-Abonnement.

11. Aktivieren Sie nun das Kontrollkästchen **An Dashboard anheften**, um die Bereitstellung nachzuverfolgen, und klicken Sie auf **Erstellen**.

## <a name="InstallCmdlets"></a>Schritt 2: Installieren und Konfigurieren von Azure PowerShell

1. Installieren Sie Azure PowerShell. Anweisungen dazu finden Sie [hier][powershell-install-configure].

	> [AZURE.NOTE] Alternativ können Sie den Hive-Online-Editor von HDInsight ausschließlich für Hive-Abfragen verwenden. Melden Sie sich dazu beim [Azure-Portal][azure-portal] an, und klicken Sie im linken Bildschirmbereich auf **HDInsight**, um eine Liste mit Ihren HDInsight-Clustern anzuzeigen. Klicken Sie auf den Cluster, auf dem Hive-Abfragen ausgeführt werden sollen, und klicken Sie anschließend auf **Abfragekonsole**.

2. Öffnen Sie die integrierte Azure PowerShell-Skripting-Umgebung:
	- Sie können die integrierte Suche auf einem Computer verwenden, auf dem Windows 8 oder Windows Server 2012 oder höher ausgeführt wird. Geben Sie auf der Startseite **powershell ise** ein, und drücken Sie die **Eingabetaste**.
	- Verwenden Sie auf einem Computer, auf dem eine niedrigere Version als Windows 8 oder Windows Server 2012 ausgeführt wird, das Startmenü. Geben Sie im Startmenü **Eingabeaufforderung** in das Suchfeld ein, und klicken Sie in der Ergebnisliste auf **Eingabeaufforderung**. Geben Sie in der Eingabeaufforderung **powershell\_ise** ein, und drücken Sie dann die **Eingabetaste**.

3. Fügen Sie Ihr Azure-Konto hinzu.
	1. Geben Sie im Konsolenbereich **Add-AzureAccount** (Azure-Konto hinzufügen) ein, und drücken Sie auf die **Eingabetaste**.
	2. Geben Sie die mit Ihrem Azure-Abonnement verknüpfte E-Mail-Adresse ein, und klicken Sie auf **Weiter**.
	3. Geben Sie das Kennwort für Ihr Azure-Abonnement ein.
	4. Klicken Sie auf **Anmelden**.

4. Das folgende Diagramm kennzeichnet die wichtigen Bestandteile Ihrer Azure PowerShell-Skripting-Umgebung.

	![Diagramm für Azure PowerShell][azure-powershell-diagram]

## <a name="RunHive"></a>Schritt 3: Ausführen eines Hive-Auftrags mit DocumentDB und HDInsight

> [AZURE.IMPORTANT] Alle durch < > gekennzeichneten Variablen müssen mithilfe der entsprechenden Konfigurationseinstellungen eingetragen werden.

1. Legen Sie die folgenden Variablen im PowerShell-Skript-Bereich fest.

		# Provide Azure subscription name, the Azure Storage account and container that is used for the default HDInsight file system.
		$subscriptionName = "<SubscriptionName>"
		$storageAccountName = "<AzureStorageAccountName>"
		$containerName = "<AzureStorageContainerName>"

		# Provide the HDInsight cluster name where you want to run the Hive job.
		$clusterName = "<HDInsightClusterName>"

2. <p>Beginnen Sie mit dem Erstellen der Abfragezeichenfolge. Eine Hive-Abfrage wird geschrieben, welche die systemgenerierten Zeitstempel (_ts) aller Dokumente und die eindeutige ids (_rid) aus einer DocumentDB-Auflistung aufnimmt, alle Dokumente minutengenau aufeinander abstimmt und die Ergebnisse dann in einer neuen DocumentDB-Auflistung abspeichert.</p>

    <p>Zunächst soll eine Hive-Tabelle aus der entsprechenden DocumentDB-Auflistung erstellt werden. Fügen Sie den folgenden Codeausschnitt zum PowerShell-Skript-Bereich <strong>nach</strong> dem Codeausschnitt von Nr.&#160;1 hinzu. Fügen Sie unbedingt den optionalen DocumentDB.query-Parameter ein, um die Dokumente auf „_ts“ und „_rid“ zuzuschneiden.</p>

    > [AZURE.NOTE] **Bei der Benennung von DocumentDB.inputCollections handelt es sich nicht um einen Fehler.** Das Hinzufügen mehrerer Auflistungen ist als Eingabe zulässig: </br>

		'*DocumentDB.inputCollections*' = '*\<DocumentDB Input Collection Name 1\>*,*\<DocumentDB Input Collection Name 2\>*' A1A</br> The collection names are separated without spaces, using only a single comma.


		# Create a Hive table using data from DocumentDB. Pass DocumentDB the query to filter transferred data to _rid and _ts.
		$queryStringPart1 = "drop table DocumentDB_timestamps; "  +
                            "create external table DocumentDB_timestamps(id string, ts BIGINT) "  +
                            "stored by 'com.microsoft.azure.documentdb.hive.DocumentDBStorageHandler' "  +
                            "tblproperties ( " +
                                "'DocumentDB.endpoint' = '<DocumentDB Endpoint>', " +
                                "'DocumentDB.key' = '<DocumentDB Primary Key>', " +
                                "'DocumentDB.db' = '<DocumentDB Database Name>', " +
                                "'DocumentDB.inputCollections' = '<DocumentDB Input Collection Name>', " +
                                "'DocumentDB.query' = 'SELECT r._rid AS id, r._ts AS ts FROM root r' ); "

3.  Als Nächstes wird eine Hive-Tabelle für die Ausgabeauflistung erstellt. Bei den Eigenschaften des Ausgabedokuments handelt es sich um Monat, Tag, Stunde, Minute und die Gesamtanzahl an Vorkommen.

	> [AZURE.NOTE] **Auch hier handelt es sich bei der Benennung von DocumentDB.outputCollections nicht um einen Fehler.** Ja, Sie können mehrere Sammlungen als Ausgabe hinzufügen: </br> 
	'*DocumentDB.outputCollections*' = '*\<DocumentDB output Collection Name 1\>*,*\<DocumentDB Output Collection Name 2\>*' </br> Die Sammlungsnamen werden ohne Leerzeichen nur durch ein Komma getrennt. </br></br> 
	Die Dokumente werden in Umlauf gebracht und über mehrere Auflistungen hinweg per Roundrobin verteilt. Ein Batch von Dokumenten wird in einer Auflistung gespeichert. Ein zweiter Batch von Dokumenten wird dann in der nächsten Auflistung gespeichert usw.

		# Create a Hive table for the output data to DocumentDB.
	    $queryStringPart2 = "drop table DocumentDB_analytics; " +
                              "create external table DocumentDB_analytics(Month INT, Day INT, Hour INT, Minute INT, Total INT) " +
                              "stored by 'com.microsoft.azure.documentdb.hive.DocumentDBStorageHandler' " +
                              "tblproperties ( " +
                                  "'DocumentDB.endpoint' = '<DocumentDB Endpoint>', " +
                                  "'DocumentDB.key' = '<DocumentDB Primary Key>', " +  
                                  "'DocumentDB.db' = '<DocumentDB Database Name>', " +
                                  "'DocumentDB.outputCollections' = '<DocumentDB Output Collection Name>' ); "

4. Zum Schluss werden die Dokumente nach Monat, Tag, Stunde und Minute berechnet. Die Ergebnisse werden zurück in die Hive-Ausgabetabelle eingefügt.

		# GROUP BY minute, COUNT entries for each, INSERT INTO output Hive table.
        $queryStringPart3 = "INSERT INTO table DocumentDB_analytics " +
                              "SELECT month(from_unixtime(ts)) as Month, day(from_unixtime(ts)) as Day, " +
                              "hour(from_unixtime(ts)) as Hour, minute(from_unixtime(ts)) as Minute, " +
                              "COUNT(*) AS Total " +
                              "FROM DocumentDB_timestamps " +
                              "GROUP BY month(from_unixtime(ts)), day(from_unixtime(ts)), " +
                              "hour(from_unixtime(ts)) , minute(from_unixtime(ts)); "

5. Fügen Sie den folgenden Skriptausschnitt hinzu, um eine Hive-Jobdefinition aus der vorherigen Abfrage zu erstellen.

		# Create a Hive job definition.
		$queryString = $queryStringPart1 + $queryStringPart2 + $queryStringPart3
		$hiveJobDefinition = New-AzureHDInsightHiveJobDefinition -Query $queryString

	Sie können auch den Schalter "-File" verwenden, um eine HiveQL-Skriptdatei auf HDFS anzugeben.

6. Fügen Sie den folgenden Ausschnitt hinzu, um die Startzeit zu speichern und den Hive-Job zu übermitteln.

		# Save the start time and submit the job to the cluster.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$hiveJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $hiveJobDefinition

7. Fügen Sie Folgendes hinzu, um auf das Abschließen des Hive-Jobs zu warten.

		# Wait for the Hive job to complete.
		Wait-AzureHDInsightJob -Job $hiveJob -WaitTimeoutInSeconds 3600

8. Fügen Sie Folgendes hinzu, um die Standardausgabe sowie die Start- und Endzeiten auszudrucken.

		# Print the standard error, the standard output of the Hive job, and the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $hiveJob.JobId -StandardOutput
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

9. **Führen** Sie das neue Skript aus! **Klicken** Sie zum Ausführen auf die grüne Schaltfläche.

10. Überprüfen Sie die Ergebnisse. Melden Sie sich beim [Azure-Portal][azure-portal] an.
	1. Klicken Sie im linken Bereich auf <strong>Durchsuchen</strong>. </br>
	2. Klicken Sie oben rechts im Suchbereich auf <strong>Alles</strong>. </br>
	3. Suchen Sie nach und klicken Sie auf <strong>DocumentDB-Konten</strong>. </br>
	4. Suchen Sie als Nächstes das <strong>DocumentDB-Konto</strong>, anschließend die <strong>DocumentDB-Datenbank</strong> und die <strong>DocumentDB-Sammlung</strong>, die mit der in der Hive-Abfrage angegebenen Ausgabesammlung verknüpft ist.</br>
	5. Klicken Sie abschließend auf <strong>Dokument-Explorer</strong> unter <strong>Entwicklertools</strong>.</br></p>

	Sie sehen die Ergebnisse der Hive-Abfrage.

	![Ergebnisse der Hive-Abfrage][image-hive-query-results]

## <a name="RunPig"></a>Schritt 4: Ausführen eines Pig-Auftrags mit DocumentDB und HDInsight

> [AZURE.IMPORTANT] Alle durch < > gekennzeichneten Variablen müssen mithilfe der entsprechenden Konfigurationseinstellungen eingetragen werden.

1. Legen Sie die folgenden Variablen im PowerShell-Skript-Bereich fest.

        # Provide Azure subscription name.
        $subscriptionName = "Azure Subscription Name"

        # Provide HDInsight cluster name where you want to run the Pig job.
        $clusterName = "Azure HDInsight Cluster Name"

2. <p>Beginnen Sie mit dem Erstellen der Abfragezeichenfolge. Eine Pig-Abfrage wird geschrieben, welche die systemgenerierten Zeitstempel (_ts) aller Dokumente und die eindeutige ids (_rid) aus einer DocumentDB-Auflistung aufnimmt, alle Dokumente minutengenau aufeinander abstimmt und die Ergebnisse dann in einer neuen DocumentDB-Auflistung abspeichert.</p>
    <p>Laden Sie zunächst die Dokumente aus DocumentDB in HDInsight. Fügen Sie den folgenden Codeausschnitt zum PowerShell-Skript-Bereich <strong>nach</strong> dem Codeausschnitt von Nr.&#160;1 hinzu. Achten Sie darauf, dass eine DocumentDB-Abfrage zum optionalen DocumentDB-Abfrageparameter hinzugefügt wird, um die Dokumente gemäß _ts und _rid zu verkürzen.</p>

    > [AZURE.NOTE] Ja, Sie können mehrere Sammlungen als Eingabe hinzufügen: </br> 
    '*\<DocumentDB Input Collection Name 1\>*,*\<DocumentDB Input Collection Name 2\>*'</br> Die Sammlungsnamen werden ohne Leerzeichen nur durch ein Komma getrennt. </b>

	Die Dokumente werden in Umlauf gebracht und über mehrere Auflistungen hinweg verteilt. Ein Batch von Dokumenten wird in einer Auflistung gespeichert. Ein zweiter Batch von Dokumenten wird dann in der nächsten Auflistung gespeichert usw.

		# Load data from DocumentDB. Pass DocumentDB query to filter transferred data to _rid and _ts.
        $queryStringPart1 = "DocumentDB_timestamps = LOAD '<DocumentDB Endpoint>' USING com.microsoft.azure.documentdb.pig.DocumentDBLoader( " +
                                                        "'<DocumentDB Primary Key>', " +
                                                        "'<DocumentDB Database Name>', " +
                                                        "'<DocumentDB Input Collection Name>', " +
                                                        "'SELECT r._rid AS id, r._ts AS ts FROM root r' ); "

3.  Als Nächstes werden die Dokumente nach Monat, Tag, Stunde, Minute und Gesamtzahl der Vorkommen zusammengerechnet.

		# GROUP BY minute and COUNT entries for each.
        $queryStringPart2 = "timestamp_record = FOREACH DocumentDB_timestamps GENERATE `$0#'id' as id:int, ToDate((long)(`$0#'ts') * 1000) as timestamp:datetime; " +
                            "by_minute = GROUP timestamp_record BY (GetYear(timestamp), GetMonth(timestamp), GetDay(timestamp), GetHour(timestamp), GetMinute(timestamp)); " +
                            "by_minute_count = FOREACH by_minute GENERATE FLATTEN(group) as (Year:int, Month:int, Day:int, Hour:int, Minute:int), COUNT(timestamp_record) as Total:int; "

4. Schließlich werden die Ergebnisse in der neuen Ausgabeauflistung abgespeichert.

    > [AZURE.NOTE] Ja, Sie können mehrere Sammlungen als Ausgabe hinzufügen: </br> 
'\<DocumentDB Output Collection Name 1\>,\<DocumentDB Output Collection Name 2\>'</br> 
Die Sammlungsnamen werden ohne Leerzeichen nur durch ein Komma getrennt.</br> Die Dokumente werden in Umlauf gebracht und über die mehrfachen Auflistungen hinweg verteilt. Ein Batch von Dokumenten wird in einer Auflistung gespeichert. Ein zweiter Batch von Dokumenten wird dann in der nächsten Auflistung gespeichert usw.

		# Store output data to DocumentDB.
        $queryStringPart3 = "STORE by_minute_count INTO '<DocumentDB Endpoint>' " +
                            "USING com.microsoft.azure.documentdb.pig.DocumentDBStorage( " +
                                "'<DocumentDB Primary Key>', " +
                                "'<DocumentDB Database Name>', " +
                                "'<DocumentDB Output Collection Name>'); "

5. Fügen Sie den folgenden Skriptausschnitt hinzu, um eine Pig-Jobdefinition aus der vorherigen Abfrage zu erstellen.

        # Create a Pig job definition.
        $queryString = $queryStringPart1 + $queryStringPart2 + $queryStringPart3
        $pigJobDefinition = New-AzureHDInsightPigJobDefinition -Query $queryString -StatusFolder $statusFolder

	Mit dem Parameter -File können Sie eine Pig-Skriptdatei unter HDFS angeben.

6. Fügen Sie den folgenden Ausschnitt hinzu, um die Startzeit zu speichern und den Pig-Job zu übermitteln.

		# Save the start time and submit the job to the cluster.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$pigJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $pigJobDefinition

7. Fügen Sie Folgendes hinzu, um auf das Abschließen des Pig-Jobs zu warten.

		# Wait for the Pig job to complete.
		Wait-AzureHDInsightJob -Job $pigJob -WaitTimeoutInSeconds 3600

8. Fügen Sie Folgendes hinzu, um die Standardausgabe sowie die Start- und Endzeiten auszudrucken.

		# Print the standard error, the standard output of the Hive job, and the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $pigJob.JobId -StandardOutput
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

9. **Führen** Sie das neue Skript aus! **Klicken** Sie zum Ausführen auf die grüne Schaltfläche.

10. Überprüfen Sie die Ergebnisse. Melden Sie sich beim [Azure-Portal][azure-portal] an.
	1. Klicken Sie im linken Bereich auf <strong>Durchsuchen</strong>. </br>
	2. Klicken Sie oben rechts im Suchbereich auf <strong>Alles</strong>. </br>
	3. Suchen Sie nach und klicken Sie auf <strong>DocumentDB-Konten</strong>. </br>
	4. Suchen Sie als Nächstes das <strong>DocumentDB-Konto</strong>, anschließend die <strong>DocumentDB-Datenbank</strong> und die <strong>DocumentDB-Sammlung</strong>, die mit der in der Pig-Abfrage angegebenen Ausgabesammlung verknüpft ist.</br>
	5. Klicken Sie abschließend auf <strong>Dokument-Explorer</strong> unter <strong>Entwicklertools</strong>. </br></p>

	Sie sehen die Ergebnisse der Pig-Abfrage.

	![Ergebnisse der Pig-Abfrage][image-pig-query-results]

## <a name="RunMapReduce"></a>Schritt 5: Ausführen eines MapReduce-Auftrags mit DocumentDB und HDInsight

1. Legen Sie die folgenden Variablen im PowerShell-Skript-Bereich fest.

		$subscriptionName = "<SubscriptionName>"   # Azure subscription name
		$clusterName = "<ClusterName>"             # HDInsight cluster name

2. Ein MapReduce-Job soll ausgeführt werden, der die Anzahl der Vorkommen für jede Dokumenteigenschaft aus der DocumentDB-Auflistung zusammenrechnet. Fügen Sie diesen Skriptausschnitt **nach** dem oberen Ausschnitt hinzu.

		# Define the MapReduce job.
		$TallyPropertiesJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/TallyProperties-v01.jar" -ClassName "TallyProperties" -Arguments "<DocumentDB Endpoint>","<DocumentDB Primary Key>", "<DocumentDB Database Name>","<DocumentDB Input Collection Name>","<DocumentDB Output Collection Name>","<[Optional] DocumentDB Query>"

	> [AZURE.NOTE] TallyProperties-v01.jar geht mit der benutzerdefinierten Installation des DocumentDB-Hadoop-Connectors einher.

3. Fügen Sie den folgenden Befehl hinzu, um den MapReduce-Job zu übermitteln.

		# Save the start time and submit the job.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$TallyPropertiesJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $TallyPropertiesJobDefinition | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600  

	Zusätzlich zur MapReduce-Jobdefinition müssen Sie auch den Namen des HDInsight-Clusters angeben, auf dem Sie den MapReduce-Job ausführen möchten, sowie die Anmeldeinformationen. "Start-AzureHDInsightJob" ist ein asynchronisierter Aufruf. Verwenden Sie das Cmdlet *Wait-AzureHDInsightJob*, um zu überprüfen, ob der Job abgeschlossen ist.

4. Fügen Sie den folgenden Befehl hinzu, um bei der Ausführung des MapReduce-Jobs auf etwaige Fehler hin zu prüfen.

		# Get the job output and print the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $TallyPropertiesJob.JobId -StandardError
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

5. **Führen** Sie das neue Skript aus! **Klicken** Sie zum Ausführen auf die grüne Schaltfläche.

6. Überprüfen Sie die Ergebnisse. Melden Sie sich beim [Azure-Portal][azure-portal] an.
	1. Klicken Sie im linken Bereich auf <strong>Durchsuchen</strong>.
	2. Klicken Sie oben rechts im Suchbereich auf <strong>Alles</strong>.
	3. Suchen Sie nach und klicken Sie auf <strong>DocumentDB-Konten</strong>.
	4. Suchen Sie als Nächstes das <strong>DocumentDB-Konto</strong>, anschließend die <strong>DocumentDB-Datenbank</strong> und die <strong>DocumentDB-Sammlung</strong>, die mit der im MapReduce-Job angegebenen Ausgabesammlung verknüpft ist.
	5. Klicken Sie abschließend auf <strong>Dokument-Explorer</strong> unter <strong>Entwicklertools</strong>.

	Sie sehen die Ergebnisse des MapReduce-Jobs.

	![Ergebnisse der MapReduce-Abfrage][image-mapreduce-query-results]

## <a name="NextSteps"></a>Nächste Schritte

Glückwunsch! Sie haben soeben mithilfe von Azure-DocumentDB und HDInsight Ihre ersten Hive-, Pig- und MapReduce-Jobs ausgeführt.

Der Hadoop-Connector steht im Rahmen von "Open Source" zur Verfügung. Wenn Sie möchten, können Sie im [GitHub][documentdb-github] einen Beitrag leisten.

Weitere Informationen finden Sie in den folgenden Artikeln:

- [Entwickeln einer Java-Anwendung mit DocumentDB][documentdb-java-application]
- [Entwickeln von Java MapReduce-Programmen für Hadoop in HDInsight][hdinsight-develop-deploy-java-mapreduce]
- [Erste Schritte bei der Nutzung von Hadoop mit Hive in HDInsight zur Analyse der Verwendung von Mobiltelefonen][hdinsight-get-started]
- [Verwenden von MapReduce mit HDInsight][hdinsight-use-mapreduce]
- [Verwenden von Hive mit HDInsight][hdinsight-use-hive]
- [Verwenden von Pig mit HDInsight][hdinsight-use-pig]
- [Anpassen von HDInsight-Clustern mithilfe von Skriptaktionen][hdinsight-hadoop-customize-cluster]

[apache-hadoop]: http://hadoop.apache.org/
[apache-hadoop-doc]: http://hadoop.apache.org/docs/current/
[apache-hive]: http://hive.apache.org/
[apache-pig]: http://pig.apache.org/
[getting-started]: documentdb-get-started.md

[azure-portal]: https://portal.azure.com/
[azure-powershell-diagram]: ./media/documentdb-run-hadoop-with-hdinsight/azurepowershell-diagram-med.png

[documentdb-hdinsight-samples]: http://portalcontent.blob.core.windows.net/samples/documentdb-hdinsight-samples.zip
[documentdb-github]: https://github.com/Azure/azure-documentdb-hadoop
[documentdb-java-application]: documentdb-java-application.md
[documentdb-manage-collections]: documentdb-manage.md#Collections
[documentdb-manage-document-storage]: documentdb-manage.md#IndexOverhead
[documentdb-manage-throughput]: documentdb-manage.md#ProvThroughput
[documentdb-import-data]: documentdb-import-data.md

[hdinsight-custom-provision]: ../hdinsight/hdinsight-provision-clusters.md#powershell
[hdinsight-develop-deploy-java-mapreduce]: ../hdinsight/hdinsight-develop-deploy-java-mapreduce-linux.md
[hdinsight-hadoop-customize-cluster]: ../hdinsight/hdinsight-hadoop-customize-cluster.md
[hdinsight-get-started]: ../hdinsight/hdinsight-hadoop-tutorial-get-started-windows.md
[hdinsight-storage]: ../hdinsight/hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]: ../hdinsight/hdinsight-use-hive.md
[hdinsight-use-mapreduce]: ../hdinsight/hdinsight-use-mapreduce.md
[hdinsight-use-pig]: ../hdinsight/hdinsight-use-pig.md

[image-customprovision-page1]: ./media/documentdb-run-hadoop-with-hdinsight/customprovision-page1.png
[image-hive-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/hivequeryresults.PNG
[image-mapreduce-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/mapreducequeryresults.PNG
[image-pig-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/pigqueryresults.PNG

[powershell-install-configure]: ../powershell-install-configure.md

<!---HONumber=AcomDC_0921_2016-->