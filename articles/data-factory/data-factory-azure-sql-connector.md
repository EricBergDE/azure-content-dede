<properties 
	pageTitle="Verschieben von Daten in/aus Azure SQL-Datenbank | Microsoft Azure" 
	description="Erfahren Sie, wie Daten mithilfe von Azure Data Factory in und aus Azure SQL-Datenbank verschoben werden." 
	services="data-factory" 
	documentationCenter="" 
	authors="spelluru" 
	manager="jhubbard" 
	editor="monicar"/>

<tags 
	ms.service="data-factory" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/20/2016" 
	ms.author="spelluru"/>

# Verschieben von Daten in und aus Azure SQL-Datenbank mithilfe von Azure Data Factory

In diesem Artikel wird beschrieben, wie Sie die Kopieraktivität in einer Azure Data Factory verwenden können, um Daten aus Azure SQL-Datenbank in einen anderen Datenspeicher zu verschieben (oder umgekehrt). Dieser Artikel baut auf dem Artikel [Datenverschiebungsaktivitäten](data-factory-data-movement-activities.md) auf, der eine allgemeine Übersicht zur Datenverschiebung mit Kopieraktivität und unterstützten Datenspeicherkombinationen bietet.

## Assistent zum Kopieren von Daten
Die einfachste Möglichkeit zum Erstellen einer Pipeline, die Daten in und aus Azure SQL-Datenbank kopiert, ist die Verwendung des Assistenten zum Kopieren von Daten. Unter [Tutorial: Erstellen einer Pipeline mit dem Assistenten zum Kopieren](data-factory-copy-data-wizard-tutorial.md) finden Sie eine kurze exemplarische Vorgehensweise zum Erstellen einer Pipeline mithilfe des Assistenten zum Kopieren von Daten.

Die folgenden Beispiele zeigen JSON-Beispieldefinitionen, die Sie zum Erstellen einer Pipeline mit dem [Azure-Portal](data-factory-copy-activity-tutorial-using-azure-portal.md), mit [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) oder [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) verwenden können. Sie zeigen Ihnen das Kopieren von Daten in und aus Azure SQL-Datenbank und Azure Blob Storage. Daten können jedoch mithilfe der Kopieraktivität in Azure Data Factory **direkt** aus beliebigen Quellen in die [hier](data-factory-data-movement-activities.md#supported-data-stores) aufgeführten Senken kopiert werden.

## Beispiel: Kopieren von Daten aus Azure SQL-Datenbank in ein Azure-Blob

Das Beispiel enthält die folgenden Data Factory-Entitäten:

1. Einen verknüpften Dienst des Typs [AzureSqlDatabase](#azure-sql-linked-service-properties)
2. Einen verknüpften Dienst des Typs [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties)
3. Ein [Eingabedataset](data-factory-create-datasets.md) des Typs [AzureSqlTable](#azure-sql-dataset-type-properties)
4. Ein [Ausgabedataset](data-factory-create-datasets.md) des Typs [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties)
4. Eine [Pipeline](data-factory-create-pipelines.md) mit Kopieraktivität, die [SqlSource](#azure-sql-copy-activity-type-properties) und [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) verwendet

Im Beispiel werden Zeitreihendaten (stündlich, täglich usw.) aus einer Tabelle in einer Azure SQL-Datenbank stündlich in ein Blob kopiert. Die bei diesen Beispielen verwendeten JSON-Eigenschaften werden in den Abschnitten beschrieben, die auf die Beispiele folgen.

**Mit Azure SQL verknüpfter Dienst**

	{
	  "name": "AzureSqlLinkedService",
	  "properties": {
	    "type": "AzureSqlDatabase",
	    "typeProperties": {
	      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
	    }
	  }
	}

Der Abschnitt [Mit Azure SQL verknüpfter Dienst](#azure-sql-linked-service-properties) enthält eine Liste mit den Eigenschaften, die von diesem verknüpften Dienst unterstützt werden.

**Mit Azure-Blobspeicher verknüpfter Dienst**

	{
	  "name": "StorageLinkedService",
	  "properties": {
	    "type": "AzureStorage",
	    "typeProperties": {
	      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
	    }
	  }
	}

Der Artikel [Azure-Blob](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) enthält eine Liste mit den Eigenschaften, die von diesem verknüpften Dienst unterstützt werden.

**Azure SQL-Eingabedataset**

Im Beispiel wird vorausgesetzt, dass Sie die Tabelle "MyTable" in Azure SQL erstellt haben, die für Zeitreihendaten eine Spalte namens "timestampcolumn" enthält.

Durch das Festlegen von „external“ auf „true“ wird dem Azure Data Factory-Dienst mitgeteilt, dass das Dataset für die Data Factory extern ist und nicht durch eine Aktivität in der Data Factory erzeugt wird.

	{
	  "name": "AzureSqlInput",
	  "properties": {
	    "type": "AzureSqlTable",
	    "linkedServiceName": "AzureSqlLinkedService",
	    "typeProperties": {
	      "tableName": "MyTable"
	    },
	    "external": true,
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    },
	    "policy": {
	      "externalData": {
	        "retryInterval": "00:01:00",
	        "retryTimeout": "00:10:00",
	        "maximumRetry": 3
	      }
	    }
	  }
	}

Eine Liste mit den Eigenschaften, die von diesem Dataset-Typ unterstützt werden, finden Sie im Abschnitt [Eigenschaften des Dataset-Typs „Azure SQL“](#azure-sql-dataset-type-properties).

**Azure-Blob-Ausgabedataset**

Daten werden stündlich in ein neues Blob geschrieben ("frequency": "hour", "interval": 1). Der Ordnerpfad des Blobs wird basierend auf der Startzeit des Slices, der verarbeitet wird, dynamisch ausgewertet. Im Ordnerpfad werden Jahr, Monat, Tag und die Stundenteile der Startzeit verwendet.

	{
	  "name": "AzureBlobOutput",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
	      "partitionedBy": [
	        {
	          "name": "Year",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "yyyy"
	          }
	        },
	        {
	          "name": "Month",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "MM"
	          }
	        },
	        {
	          "name": "Day",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "dd"
	          }
	        },
	        {
	          "name": "Hour",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "HH"
	          }
	        }
	      ],
	      "format": {
	        "type": "TextFormat",
	        "columnDelimiter": "\t",
	        "rowDelimiter": "\n"
	      }
	    },
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    }
	  }
	}

Eine Liste mit den Eigenschaften, die von diesem Dataset-Typ unterstützt werden, finden Sie im Abschnitt [Eigenschaften des Dataset-Typs „Azure-Blob“](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).

**Pipeline mit Kopieraktivität**

Die Pipeline enthält eine Kopieraktivität, die für die Verwendung der Ein- und Ausgabedatasets und für eine stündliche Ausführung konfiguriert ist. In der JSON-Definition der Pipeline ist der Typ **source** auf **SqlSource** und der Typ **sink** auf **BlobSink** festgelegt. Die für die **SqlReaderQuery**-Eigenschaft angegebene SQL-Abfrage wählt die Daten der letzten Stunde zum Kopieren aus.

	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2014-06-01T18:00:00",
	    "end":"2014-06-01T19:00:00",
	    "description":"pipeline for copy activity",
	    "activities":[  
	      {
	        "name": "AzureSQLtoBlob",
	        "description": "copy activity",
	        "type": "Copy",
	        "inputs": [
	          {
	            "name": "AzureSQLInput"
	          }
	        ],
	        "outputs": [
	          {
	            "name": "AzureBlobOutput"
	          }
	        ],
	        "typeProperties": {
	          "source": {
	            "type": "SqlSource",
	            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
	          },
	          "sink": {
	            "type": "BlobSink"
	          }
	        },
	       "scheduler": {
	          "frequency": "Hour",
	          "interval": 1
	        },
	        "policy": {
	          "concurrency": 1,
	          "executionPriorityOrder": "OldestFirst",
	          "retry": 0,
	          "timeout": "01:00:00"
	        }
	      }
	     ]
	   }
	}

Im Beispiel ist **sqlReaderQuery** für SqlSource angegeben. Mit der Kopieraktivität wird diese Abfrage für die Azure SQL-Datenbankquelle ausgeführt, um die Daten abzurufen. Alternativ dazu können Sie eine gespeicherte Prozedur angeben, indem Sie **sqlReaderStoredProcedureName** und **storedProcedureParameters** angeben (sofern die gespeicherten Prozeduren Parameter verwenden).

Ohne Angabe von „sqlReaderQuery“ oder „sqlReaderStoredProcedureName“ werden die im Strukturabschnitt des DataSet-JSON-Bereichs definierten Spalten verwendet, um eine Abfrage für Azure SQL-Datenbank zu erstellen. Beispiel: `select column1, column2 from mytable`. Falls die DataSet-Definition nicht über die Struktur verfügt, werden alle Spalten der Tabelle ausgewählt.


Eine Liste mit den Eigenschaften, die von SqlSource und BlobSink unterstützt werden, finden Sie im Abschnitt [Sql Source](#sqlsource) und unter [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties).


## Beispiel: Kopieren von Daten aus einem Azure-Blob in Azure SQL-Datenbank

Im Beispiel werden folgende Data Factory-Entitäten definiert:

1.	Einen verknüpften Dienst des Typs [AzureSqlDatabase](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties)
2.	Einen verknüpften Dienst des Typs [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties)
3.	Ein [Eingabedataset](data-factory-create-datasets.md) des Typs [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties)
4.	Ein [Ausgabedataset](data-factory-create-datasets.md) des Typs [AzureSqlTable](data-factory-azure-sql-connector.md#azure-sql-dataset-type-properties)
4.	Eine [Pipeline](data-factory-create-pipelines.md) mit Kopieraktivität, die [BlobSource](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) und [SqlSink](data-factory-azure-sql-connector.md#azure-sql-copy-activity-type-properties) verwendet

In dem Beispiel werden jede Stunde Zeitreihendaten (stündlich, täglich usw.) aus einem Azure-Blob in eine Tabelle in einer Azure SQL-Datenbank kopiert. Die bei diesen Beispielen verwendeten JSON-Eigenschaften werden in den Abschnitten beschrieben, die auf die Beispiele folgen.


**Mit Azure SQL verknüpfter Dienst**
	
	{
	  "name": "AzureSqlLinkedService",
	  "properties": {
	    "type": "AzureSqlDatabase",
	    "typeProperties": {
	      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
	    }
	  }
	}

Der Abschnitt [Mit Azure SQL verknüpfter Dienst](#azure-sql-linked-service-properties) enthält eine Liste mit den Eigenschaften, die von diesem verknüpften Dienst unterstützt werden.

**Mit Azure-Blobspeicher verknüpfter Dienst**

	{
	  "name": "StorageLinkedService",
	  "properties": {
	    "type": "AzureStorage",
	    "typeProperties": {
	      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
	    }
	  }
	}

Der Artikel [Azure-Blob](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) enthält eine Liste mit den Eigenschaften, die von diesem verknüpften Dienst unterstützt werden.

**Azure-Blob-Eingabedataset**

Daten werden stündlich aus einem neuen Blob übernommen ("frequency": "hour", "interval": 1). Ordnerpfad und Dateiname des Blobs werden basierend auf der Startzeit des Slices, der verarbeitet wird, dynamisch ausgewertet. Der Ordnerpfad enthält das Jahr, den Monat und den Tag der Startzeit, der Dateiname enthält die Stunde der Startzeit. Die Festlegung von „external“ auf „true“ teilt dem Data Factory-Dienst mit, dass diese Tabelle für die Data Factory extern ist und nicht durch eine Aktivität in der Data Factory erzeugt wird.

	{
	  "name": "AzureBlobInput",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
	      "fileName": "{Hour}.csv",
	      "partitionedBy": [
	        {
	          "name": "Year",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "yyyy"
	          }
	        },
	        {
	          "name": "Month",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "MM"
	          }
	        },
	        {
	          "name": "Day",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "dd"
	          }
	        },
	        {
	          "name": "Hour",
	          "value": {
	            "type": "DateTime",
	            "date": "SliceStart",
	            "format": "HH"
	          }
	        }
	      ],
	      "format": {
	        "type": "TextFormat",
	        "columnDelimiter": ",",
	        "rowDelimiter": "\n"
	      }
	    },
	    "external": true,
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    },
	    "policy": {
	      "externalData": {
	        "retryInterval": "00:01:00",
	        "retryTimeout": "00:10:00",
	        "maximumRetry": 3
	      }
	    }
	  }
	}

Eine Liste mit den Eigenschaften, die von diesem Dataset-Typ unterstützt werden, finden Sie im Abschnitt [Eigenschaften des Dataset-Typs „Azure-Blob“](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).

**Azure SQL-Ausgabedataset**

Das Beispiel kopiert Daten in eine Tabelle namens "MyTable" in Azure SQL. Erstellen Sie die Tabelle in Azure SQL mit der gleichen Anzahl von Spalten, die Sie in der Blob-CSV-Datei erwarten. Neue Zeilen werden der Tabelle stündlich hinzugefügt.

	{
	  "name": "AzureSqlOutput",
	  "properties": {
	    "type": "AzureSqlTable",
	    "linkedServiceName": "AzureSqlLinkedService",
	    "typeProperties": {
	      "tableName": "MyOutputTable"
	    },
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    }
	  }
	}

Eine Liste mit den Eigenschaften, die von diesem Dataset-Typ unterstützt werden, finden Sie im Abschnitt [Eigenschaften des Dataset-Typs „Azure SQL“](#azure-sql-dataset-type-properties).

**Pipeline mit Kopieraktivität**

Die Pipeline enthält eine Kopieraktivität, die für die Verwendung der Ein- und Ausgabedatasets und für eine stündliche Ausführung konfiguriert ist. In der JSON-Definition der Pipeline ist der Typ **source** auf **BlobSource** und der Typ **sink** auf **SqlSink** festgelegt.

	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2014-06-01T18:00:00",
	    "end":"2014-06-01T19:00:00",
	    "description":"pipeline with copy activity",
	    "activities":[  
	      {
	        "name": "AzureBlobtoSQL",
	        "description": "Copy Activity",
	        "type": "Copy",
	        "inputs": [
	          {
	            "name": "AzureBlobInput"
	          }
	        ],
	        "outputs": [
	          {
	            "name": "AzureSqlOutput"
	          }
	        ],
	        "typeProperties": {
	          "source": {
	            "type": "BlobSource",
	            "blobColumnSeparators": ","
	          },
	          "sink": {
	            "type": "SqlSink"
	          }
	        },
	       "scheduler": {
	          "frequency": "Hour",
	          "interval": 1
	        },
	        "policy": {
	          "concurrency": 1,
	          "executionPriorityOrder": "OldestFirst",
	          "retry": 0,
	          "timeout": "01:00:00"
	        }
	      }
	      ]
	   }
	}

Eine Liste mit den Eigenschaften, die von SqlSink und BlobSource unterstützt werden, finden Sie im Abschnitt [Sql Sink](#sqlsink) und unter [BlobSource](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties).


## Eigenschaften des mit Azure SQL verknüpften Diensts

Die folgende Tabelle enthält eine Beschreibung der JSON-Elemente, die für den mit Azure SQL verknüpften Dienst spezifisch sind.

| Eigenschaft | Beschreibung | Erforderlich |
| -------- | ----------- | -------- |
| Typ | Die type-Eigenschaft muss auf "AzureSqlDatabase" festgelegt sein. | Ja |
| connectionString | Geben Sie Informationen, die zur Verbindung mit der Azure SQL-Datenbankinstanz erforderlich sind, für die Eigenschaft "connectionString" ein. | Ja |

> [AZURE.NOTE] Konfigurieren Sie die [Azure SQL-Datenbankfirewall](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure) und den Datenbankserver, um [Azure-Diensten den Zugriff auf den Server zu ermöglichen](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure). Gehen Sie wie folgt vor, wenn Sie nicht aus Azure stammende Daten nach Azure SQL-Datenbank kopieren, inklusive Daten aus lokalen Datenquellen mit Data Factory-Gateway: Konfigurieren Sie den entsprechenden IP-Adressbereich für den Computer, der Daten an Azure SQL-Datenbank sendet.

## Eigenschaften des Dataset-Typs „Azure SQL“

Eine vollständige Liste der Abschnitte und Eigenschaften, die zum Definieren von Datasets zur Verfügung stehen, finden Sie im Artikel [Erstellen von Datasets](data-factory-create-datasets.md). Abschnitte wie „structure“, „availability“ und „policy“ des JSON-Codes eines Datasets sind bei allen Dataset-Typen (Azure SQL, Azure-Blob, Azure-Tabelle usw.) ähnlich.

Der Abschnitt "typeProperties" unterscheidet sich bei jedem Typ von Dataset und bietet Informationen zum Speicherort der Daten im Datenspeicher. Der Abschnitt **typeProperties** für ein Dataset des Typs **AzureSqlTable** hat die folgenden Eigenschaften:

| Eigenschaft | Beschreibung | Erforderlich |
| -------- | ----------- | -------- |
| tableName | Name der Tabelle in der Azure SQL-Datenbankinstanz, auf die der verknüpfte Dienst verweist. | Ja |

## Eigenschaften des Azure SQL-Kopieraktivitätstyps

Eine vollständige Liste der Abschnitte und Eigenschaften zum Definieren von Aktivitäten finden Sie im Artikel [Erstellen von Pipelines](data-factory-create-pipelines.md). Eigenschaften wie Name, Beschreibung, Eingabe- und Ausgabetabellen und Richtlinie sind für alle Arten von Aktivitäten verfügbar.

> [AZURE.NOTE] Die Kopieraktivität verwendet nur eine Eingabe und erzeugt nur eine Ausgabe.

Eigenschaften im Abschnitt „typeProperties“ der Aktivität können dagegen je nach Aktivitätstyp variieren. Für die Kopieraktivität variieren die Eigenschaften je nach Art der Quellen und Senken.

### SqlSource

Wenn bei der Kopieraktivität die Quelle den Typ **SqlSource** hat, sind im Abschnitt **typeProperties** die folgenden Eigenschaften verfügbar:

| Eigenschaft | Beschreibung | Zulässige Werte | Erforderlich |
| -------- | ----------- | -------------- | -------- |
| sqlReaderQuery | Verwendet die benutzerdefinierte Abfrage zum Lesen von Daten. | SQL-Abfragezeichenfolge. Beispiel: `select * from MyTable`. | Nein |
| sqlReaderStoredProcedureName | Der Name der gespeicherten Prozedur, die Daten aus der Quelltabelle liest. | Name der gespeicherten Prozedur. | Nein |
| storedProcedureParameters | Parameter für die gespeicherte Prozedur. | Name-Wert-Paare. Die Namen und die Groß-/Kleinschreibung von Parametern müssen denen der Parameter der gespeicherten Prozedur entsprechen. | Nein |

Wenn **sqlReaderQuery** für SqlSource angegeben ist, führt die Kopieraktivität diese Abfrage für die Azure SQL-Datenbankquelle aus, um die Daten abzurufen. Alternativ dazu können Sie eine gespeicherte Prozedur angeben, indem Sie **sqlReaderStoredProcedureName** und **storedProcedureParameters** angeben (sofern die gespeicherten Prozeduren Parameter verwenden).

Wenn Sie sqlReaderQuery oder sqlReaderStoredProcedureName nicht angeben, werden die im Strukturabschnitt des Dataset-JSON-Bereichs definierten Spalten verwendet, um eine Abfrage (`select column1, column2 from mytable`) zur Ausführung für Azure SQL-Datenbank zu erstellen. Falls die DataSet-Definition nicht über die Struktur verfügt, werden alle Spalten der Tabelle ausgewählt.

> [AZURE.NOTE] Bei Verwendung von **sqlReaderStoredProcedureName** müssen Sie trotzdem einen Wert für die **tableName**-Eigenschaft in der Dataset-JSON angeben. Es finden jedoch keine Überprüfungen dieser Tabelle statt.

### Beispiel für SqlSource

    "source": {
        "type": "SqlSource",
        "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
        "storedProcedureParameters": {
            "stringData": { "value": "str3" },
            "id": { "value": "$$Text.Format('{0:yyyy}', SliceStart)", "type": "Int"}
        }
    }

**Die Definition der gespeicherten Prozedur:**

	CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
	(
		@stringData varchar(20),
		@id int
	)
	AS
	SET NOCOUNT ON;
	BEGIN
	     select *
	     from dbo.UnitTestSrcTable
	     where dbo.UnitTestSrcTable.stringData != stringData
	    and dbo.UnitTestSrcTable.id != id
	END
	GO


### SqlSink 

**SqlSink** unterstützt die folgenden Eigenschaften:

| Eigenschaft | Beschreibung | Zulässige Werte | Erforderlich |
| -------- | ----------- | -------------- | -------- |
| writeBatchTimeout | Die Wartezeit für den Abschluss der Batcheinfügung, bis das Timeout wirksam wird. | Zeitraum<br/><br/> Beispiel: 00:30:00 (30 Minuten) | Nein | 
| writeBatchSize | Fügt Daten in die SQL-Tabelle ein, wenn die Puffergröße "writeBatchSize" erreicht. | Integer (Gesamtanzahl von Zeilen)| Nein (Standard = 10000)
| sqlWriterCleanupScript | Geben Sie eine Abfrage für die Kopieraktivität an, bei deren Ausführung die Daten eines bestimmten Slice bereinigt werden. Weitere Informationen finden Sie im Abschnitt zur [Wiederholbarkeit](#repeatability-during-copy). | Eine Abfrageanweisung. | Nein |
| sliceIdentifierColumnName | Geben Sie einen Spaltennamen an, den die Kopieraktivität mit einem automatisch generierten Slicebezeichner füllen soll, der bei erneuter Ausführung zum Bereinigen der Daten eines bestimmten Slice verwendet wird. Weitere Informationen finden Sie im Abschnitt zur [Wiederholbarkeit](#repeatability-during-copy). | Spaltenname einer Spalte mit binärem Datentyp (32). | Nein |
| sqlWriterStoredProcedureName | Name der gespeicherten Prozedur, die Daten in die Zieltabelle mit dem Upsert-Vorgang einfügt oder aktualisiert. | Name der gespeicherten Prozedur. | Nein |
| storedProcedureParameters | Parameter für die gespeicherte Prozedur. | Name-Wert-Paare. Die Namen und die Groß-/Kleinschreibung von Parametern müssen denen der Parameter der gespeicherten Prozedur entsprechen. | Nein | 
| sqlWriterTableType | Geben Sie einen Tabellentypnamen an, der in der gespeicherten Prozedur verwendet werden soll. Die Kopieraktivität macht die verschobenen Daten in einer temporären Tabelle mit diesem Tabellentyp verfügbar. Der gespeicherte Prozedurcode kann dann die kopierten Daten mit vorhandenen Daten zusammenführen. | Ein Tabellentypname. | Nein |

#### Beispiel für SqlSink

    "sink": {
        "type": "SqlSink",
        "writeBatchSize": 1000000,
        "writeBatchTimeout": "00:05:00",
        "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
        "sqlWriterTableType": "CopyTestTableType",
        "storedProcedureParameters": {
            "id": { "value": "1", "type": "Int" },
            "stringData": { "value": "str1" },
            "decimalData": { "value": "1", "type": "Decimal" }
        }
    }

## Identitätsspalten in der Zieldatenbank
Dieser Abschnitt enthält ein Beispiel zum Kopieren von Daten aus einer Quelltabelle ohne eine Identitätsspalte in eine Zieltabelle mit einer Identitätsspalte.

**Quelltabelle:**

	create table dbo.SourceTbl
	(
	       name varchar(100),
	       age int
	)

**Zieltabelle:**

	create table dbo.TargetTbl
	(
	       id int identity(1,1),
	       name varchar(100),
	       age int
	)


Beachten Sie, dass die Zieltabelle über eine Identitätsspalte verfügt.

**Definition der Quell-Dataset-JSON**

	{
	    "name": "SampleSource",
	    "properties": {
	        "published": false,
	        "type": " SqlServerTable",
	        "linkedServiceName": "TestIdentitySQL",
	        "typeProperties": {
	            "tableName": "SourceTbl"
	        },
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        },
	        "external": true,
	        "policy": {}
	    }
	}

**Definition der Ziel-Dataset-JSON**

	{
	    "name": "SampleTarget",
	    "properties": {
	        "structure": [
	            { "name": "name" },
	            { "name": "age" }
	        ],
	        "published": false,
	        "type": "AzureSqlTable",
	        "linkedServiceName": "TestIdentitySQLSource",
	        "typeProperties": {
	            "tableName": "TargetTbl"
	        },
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        },
	        "external": false,
	        "policy": {}
	    }	
	}


Beachten Sie, dass die Quell- und die Zieltabelle unterschiedliche Schemas besitzen (das Ziel verfügt über eine zusätzliche Spalte mit der Identität). In diesem Szenario müssen Sie eine **structure**-Eigenschaft in der Definition des Zieldatasets angeben, die nicht die Identitätsspalte enthält.

[AZURE.INCLUDE [data-factory-type-repeatability-for-sql-sources](../../includes/data-factory-type-repeatability-for-sql-sources.md)]

[AZURE.INCLUDE [data-factory-sql-invoke-stored-procedure](../../includes/data-factory-sql-invoke-stored-procedure.md)]
 

[AZURE.INCLUDE [data-factory-structure-for-rectangular-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

### Typzuordnung für SQL Server und Azure SQL-Datenbank

Wie im Artikel [Datenverschiebungsaktivitäten](data-factory-data-movement-activities.md) beschrieben, führt die Kopieraktivität automatische Typkonvertierungen von Quelltypen in Senkentypen mithilfe der folgenden beiden Schritte durch:

1. Konvertieren von systemeigenen Quelltypen in den .NET-Typ
2. Konvertieren vom .NET-Typ in systemeigenen Senkentyp

Beim Verschieben von Daten in und aus Azure SQL, SQL Server und Sybase werden die folgenden Zuordnungen zwischen SQL-Typ und .NET-Typ und umgekehrt verwendet.

Die Zuordnung ist mit der SQL Server-Datentypzuordnung für ADO.NET identisch.

| Typ "SQL Server-Datenbankmodul" | Typ ".NET Framework" |
| ------------------------------- | ------------------- |
| bigint | Int64 |
| binary | Byte |
| Bit | Boolean |
| char | String, Char |
| date | DateTime |
| Datetime | DateTime |
| datetime2 | DateTime |
| Datetimeoffset | DateTimeOffset |
| Decimal | Decimal |
| FILESTREAM-Attribut (varbinary(max)) | Byte |
| Float | Doppelt |
| image | Byte | 
| int | Int32 | 
| money | Decimal |
| nchar | String, Char |
| ntext | String, Char |
| numeric | Decimal |
| nvarchar | String, Char |
| real | Single |
| rowversion | Byte |
| smalldatetime | DateTime |
| smallint | Int16 |
| smallmoney | Decimal | 
| sql\_variant | Object * |
| Text | String, Char |
| in | TimeSpan |
| timestamp | Byte |
| tinyint | Byte |
| uniqueidentifier | GUID |
| varbinary | Byte |
| varchar | String, Char |
| xml | Xml |


[AZURE.INCLUDE [data-factory-type-conversion-sample](../../includes/data-factory-type-conversion-sample.md)]

[AZURE.INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

## Leistung und Optimierung  
Im Artikel [Handbuch zur Leistung und Optimierung der Kopieraktivität](data-factory-copy-activity-performance.md) werden wichtige Faktoren beschrieben, die sich auf die Leistung der Datenverschiebung (Kopieraktivität) in Azure Data Factory auswirken, sowie verschiedene Möglichkeiten zur Leistungsoptimierung.

<!---HONumber=AcomDC_0921_2016-->