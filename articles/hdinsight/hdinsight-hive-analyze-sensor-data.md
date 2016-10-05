<properties
	pageTitle="Analysieren von Sensordaten mit Hive und Hadoop | Microsoft Azure"
	description="Erfahren Sie, wie Sie Sensordaten mithilfe der Hive-Abfragekonsole in HDInsight (Hadoop) analysieren und die Daten anschließend in Microsoft Excel mit Power View visualisieren können."
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="jhubbard"
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/20/2016" 
	ms.author="larryfr"/>

#Analysieren von Sensordaten mit der Hive-Abfragekonsole für Hadoop in HDInsight

Erfahren Sie, wie Sie Sensordaten mithilfe der Hive-Abfragekonsole in HDInsight (Hadoop) analysieren und die Daten anschließend in Microsoft Excel mithilfe von Power View visualisieren können.

> [AZURE.NOTE] Die Schritte in diesem Dokument funktionieren nur mit einem Windows-basierten HDInsight-Cluster.

In diesem Beispiel verarbeiten Sie Verlaufsdaten von Klimaanlagensystemen mithilfe von HDInsight, um herauszufinden, welche Systeme nicht in der Lage sind, eine eingestellte Temperatur zu halten. Sie lernen Folgendes:

- Erstellen von HIVE-Tabellen zum Abfragen von Daten in CSV-Dateien
- Erstellen von HIVE-Abfragen zum Analysieren der Daten
- Verwenden von Microsoft Excel zum Herstellen einer Verbindung mit HDInsight (über eine offene Datenbankverbindung (ODBC)), um die analysierten Daten abzurufen
- Verwenden von Power View zum Visualisieren der Daten

![Ein Diagramm der Lösungsarchitektur](./media/hdinsight-hive-analyze-sensor-data/hvac-architecture.png)

##Voraussetzungen

* Ein HDInsight-Cluster (Hadoop): Unter [Bereitstellen von Hadoop-Clustern in HDInsight](hdinsight-provision-clusters.md) finden Sie weitere Informationen zur Erstellung von Clustern.

* Microsoft Excel 2013 (PowerPivot für Datenanalysten – Microsoft Excel 2010)

	> [AZURE.NOTE] Microsoft Excel dient zur Visualisierung von Daten mithilfe von [Power View](https://support.office.com/Article/Power-View-Explore-visualize-and-present-your-data-98268d31-97e2-42aa-a52b-a68cf460472e?ui=de-DE&rs=de-DE&ad=US).

* [Microsoft Hive ODBC-Treiber](http://www.microsoft.com/download/details.aspx?id=40886)

##Ausführen des Beispiels

1. Navigieren Sie in einem Browser zur folgenden URL: Ersetzen Sie `<clustername>` durch den Namen Ihres HDInsight-Clusters.

	 	https://<clustername>.azurehdinsight.net

	Geben Sie den Benutzernamen und das Kennwort des Administrators ein, den Sie bei der Clusterbereitstellung verwendet haben, wenn Sie dazu aufgefordert werden.

2. Klicken Sie auf der Webseite, die sich daraufhin öffnet, auf die Registerkarte **Erste Schritte mit dem Katalog** und anschließend in der Kategorie **Lösungen mit Beispieldaten** auf das Beispiel **Analysieren von Sensordaten**.

    ![Katalogimage für erste Schritte](./media/hdinsight-hive-analyze-sensor-data/getting-started-gallery.png)

3. Folgen Sie den Anweisungen auf der Webseite, um das Beispiel abzuschließen.

<!---HONumber=AcomDC_0921_2016-->