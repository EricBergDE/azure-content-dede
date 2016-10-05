<properties 
pageTitle="Indexer-Vorgänge (REST-API für Azure Search-Dienst: 2015-02-28-Preview) | Azure Search-Vorschau-API" 
description="Indexer-Vorgänge (REST-API für Azure Search-Dienst: 2015-02-28-Preview)" 
services="search" 
documentationCenter="" 
authors="chaosrealm" 
manager="pablocas"
editor="" />

<tags 
ms.service="search" 
ms.devlang="rest-api" 
ms.workload="search" 
ms.topic="article"  
ms.tgt_pltfrm="na" 
ms.date="09/07/2016" 
ms.author="eugenesh" />

#Indexer-Vorgänge (REST-API für Azure Search-Dienst: 2015-02-28-Preview)#

> [AZURE.NOTE] Dieser Artikel beschreibt die Indexer in der REST-API [2015-02-28-Preview](search-api-2015-02-28-preview.md). Diese API-Version fügt Vorschauversionen eines Azure Blob Storage-Indexers mit Dokumentextraktion und Azure Table Storage-Indexers sowie weitere Verbesserungen hinzu. Die API unterstützt auch allgemein verfügbare Indexer, einschließlich Indexer für Azure SQL-Datenbank, SQL Server auf virtuellen Azure-Computern und Azure DocumentDB.

## Übersicht ##

Azure Search kann direkt mit einigen allgemeinen Datenquellen integrieren. Das Schreiben von Code zum Indizieren Ihrer Daten ist somit überflüssig. Um diese Funktion einzurichten, rufen Sie die Azure Search-API zum Erstellen und Verwalten mehrerer**Indexer** und **Datenquellen** auf.

Ein **Indexer** ist die Ressource, die Datenquellen mit Zielsuchindizes verbindet. Ein Indexer kann folgendermaßen verwendet werden:

- Eine einmalige Kopie der Daten zum Auffüllen eines Indexes ausführen.
- Einen Index mit Änderungen an der Datenquelle nach einem Zeitplan synchronisieren. Der Zeitplan ist Teil der Indexer-Definition.
- Ausführung bei Bedarf, um den Index je nach Notwendigkeit zu aktualisieren.

Ein **Indexer** ist nützlich, wenn regelmäßige Aktualisierungen eines Indexes durchgeführt werden sollen. Sie können einen eingebetteten Zeitplan als Teil der Definition eines Indexers einrichten oder den Indexer bei Bedarf mit [Indexer ausführen](#RunIndexer) ausführen.

Eine **Datenquelle** gibt an, welche Daten indiziert werden müssen. Sie legt außerdem die Anmeldeinformationen für den Zugriff auf die Daten sowie die Richtlinien zur Aktivierung von Azure Search fest, um Änderungen an den Daten effizient identifizieren zu können (wie z. B. geänderte oder gelöschte Zeilen in einer Datenbanktabelle). Die Datenquelle wird als unabhängige Ressource definiert, sodass sie von mehreren Indexern verwendet werden kann.

Die folgenden Datenquellen werden derzeit unterstützt:

- **Azure SQL-Datenbank** und **SQL Server in Azure VMs**. Eine gezielte exemplarische Vorgehensweise finden Sie [in diesem Artikel](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers-2015-02-28.md).
- **Azure DocumentDB**. Eine gezielte exemplarische Vorgehensweise finden Sie [in diesem Artikel](../documentdb/documentdb-search-indexer.md).
- **Azure-Blobspeicher**, einschließlich der folgenden Dokumentformate: PDF, Microsoft Office (DOCX/DOC, XSLX/XLS, PPTX/PPT, MSG), HTML, XML, ZIP und Nur-Text-Dateien (einschließlich JSON). Eine gezielte exemplarische Vorgehensweise finden Sie [in diesem Artikel](search-howto-indexing-azure-blob-storage.md).
- **Azure Table Storage**. Eine gezielte exemplarische Vorgehensweise finden Sie [in diesem Artikel](search-howto-indexing-azure-tables.md).
	 
Unterstützung für zusätzliche Datenquellen ist für einen späteren Zeitpunkt geplant. Damit wir unsere Entscheidungen besser priorisieren können, lassen Sie uns im [Feedback-Forum für Azure Search](http://feedback.azure.com/forums/263029-azure-search) Ihr Feedback zukommen.

Informationen zu maximalen Grenzwerten mit Bezug auf Indexer- und Datenquellenressourcen finden Sie unter [Grenzwerte](search-limits-quotas-capacity.md).

## Typischer Verwendungsablauf ##

Das Erstellen und Verwalten der Indexer und Datenquellen erfolgt über einfache HTTP-Anforderungen (POST, GET, PUT, DELETE) für die angegebene `data source`- oder `indexer`-Ressource.

Das Einrichten der automatischen Indizierung erfolgt in der Regel in vier Schritten:

1. Identifizieren Sie die Datenquelle, die die Daten enthält, die indiziert werden sollen. Beachten Sie, dass Azure Search möglicherweise nicht alle Datentypen in Ihrer Datenquelle unterstützt. Eine Liste der unterstützten Daten finden Sie unter [Unterstützte Datentypen](https://msdn.microsoft.com/library/azure/dn798938.aspx).

2. Erstellen Sie einen Azure Search-Index, dessen Schema mit Ihrer Datenquelle kompatibel ist.
  
3. Erstellen Sie eine Azure Search-Datenquelle wie in [Datenquelle erstellen](#CreateDataSource) beschrieben.
  
4. Erstellen Sie einen Azure Search-Indexer wie in [Indexer erstellen](#CreateIndexer) beschrieben.

Sie sollten jeweils einen Indexer pro Kombination aus Zielindex und Datenquelle erstellen. Sie können mehrere Indexer erstellen, die in denselben Index schreiben, und dieselbe Datenquelle für mehrere Indexer verwenden. Ein Indexer kann jedoch jeweils nur eine Datenquelle auf einmal nutzen und nur in einen einzelnen Index schreiben.

Nach dem Erstellen eines Indexers können Sie seinen Ausführungsstatus mit dem Vorgang [Indexer-Status abrufen](#GetIndexerStatus) ermitteln. Sie können einen Indexer jederzeit ausführen (anstelle von oder zusätzlich zur regelmäßigen Ausführung nach einem Zeitplan), indem Sie den Vorgang [Indexer ausführen](#RunIndexer) verwenden.

<!-- MSDN has 2 art files plus a API topic link list -->


## Datenquelle erstellen ##

In Azure Search wird eine Datenquelle mit Indexern verwendet, sodass die Verbindungsinformationen für die Aktualisierung von Ad-Hoc- oder geplanten Daten eines Zielindexes bereitgestellt werden. Das Erstellen einer neuen Datenquelle in einem Azure Search-Dienst erfolgt mithilfe einer HTTP POST-Anforderung.
	
    POST https://[service name].search.windows.net/datasources?api-version=[api-version]
    Content-Type: application/json
    api-key: [admin key]

Alternativ können Sie PUT verwenden und den Namen der Datenquelle für den URI angeben. Wenn die Datenquelle nicht vorhanden ist, wird sie erstellt.

    PUT https://[service name].search.windows.net/datasources/[datasource name]?api-version=[api-version]

**Hinweis**: Die maximal zulässige Anzahl von Datenquellen variiert je nach Preisstufe. Der kostenlose Dienst kann bis zu 3 Datenquellen enthalten. Der Standard-Dienst kann 50 Datenquellen enthalten. In den [Einschränkungen für Dienste](search-limits-quotas-capacity.md) finden Sie weitere Informationen.

**Anforderung**

HTTPS ist für alle Dienstanforderungen erforderlich. Die Anforderung **Datenquelle erstellen** kann mit der POST- oder PUT-Methode konstruiert werden. Wenn POST verwendet wird, müssen Sie einen Namen und eine Definition der Datenquelle im Anforderungstext bereitstellen. Bei Verwendung von PUT ist der Name Teil der URL. Wenn die Datenquelle nicht vorhanden ist, wird sie erstellt. Wenn sie bereits vorhanden ist, wird sie mit der neuen Definition aktualisiert.

Der Name der Datenquelle muss in Kleinbuchstaben angegeben werden, mit einem Buchstaben oder einer Zahl beginnen, darf keine Schrägstriche oder Punkte enthalten und muss weniger als 128 Zeichen lang sein. Der Rest des Namens (nach dem Buchstaben bzw. der Zahl zu Beginn des Namens) kann beliebige Buchstaben, Zahlen und Bindestriche enthalten, solange die Striche nicht aufeinander folgen. Weitere Informationen finden Sie unter [Benennungsregeln](https://msdn.microsoft.com/library/azure/dn857353.aspx).

`api-version` ist erforderlich. Die aktuelle Version ist `2015-02-28`.

**Anforderungsheader**

In der folgenden Liste werden die erforderlichen und optionalen Anforderungsheader beschrieben.

- `Content-Type`: Erforderlich. Auf `application/json` festlegen.
- `api-key`: Erforderlich. `api-key` wird zum Authentifizieren der Anforderung beim Search-Dienst verwendet. Es handelt sich um einen für Ihren Dienst eindeutigen Zeichenfolgenwert. Die Anforderung **Datenquelle erstellen** muss einen `api-key`-Header enthalten, der auf Ihren Admin-Schlüssel (im Gegensatz zum Abfrageschlüssel) festgelegt ist.
 
Sie benötigen außerdem den Dienstnamen, um die URL der Anforderung zu erstellen. Sie können den Dienstnamen und den `api-key` in Ihrem Dienstdashboard im [Azure-Portal](https://portal.azure.com/) abrufen. Hilfe bei der Seitennavigation finden Sie unter [Search-Dienst im Portal erstellen](search-create-service-portal.md).

<a name="CreateDataSourceRequestSyntax"></a> **Syntax des Anforderungstextes**

Der Anforderungstext enthält eine Datenquellendefinition, einschließlich Datenquellentyp, Anmeldeinformationen zum Lesen der Daten sowie optionale Richtlinien zur Erkennung von Datenänderungen und Datenlöschungen. Diese Richtlinien werden verwendet, um geänderte und gelöschte Daten in der Datenquelle effizient zu identifizieren, wenn ein regelmäßig geplanter Indexer verwendet wird.

Die Syntax für die Strukturierung der Anforderungsnutzlast ist wie folgt. Eine Beispielanforderung wird weiter unten in diesem Thema bereitgestellt.

    { 
		"name" : "Required for POST, optional for PUT. The name of the data source",
    	"description" : "Optional. Anything you want, or nothing at all",
    	"type" : "Required. Must be one of 'azuresql', 'documentdb', 'azureblob', or 'azuretable'",
    	"credentials" : { "connectionString" : "Required. Connection string for your data source" },
    	"container" : { "name" : "Required. The name of the table, collection, or blob container you wish to index" },
    	"dataChangeDetectionPolicy" : { Optional. See below for details }, 
    	"dataDeletionDetectionPolicy" : { Optional. See below for details }
	}

Die Anforderung enthält die folgenden Eigenschaften:

- `name`: Erforderlich. Der Name der Datenquelle. Der Name der Datenquelle darf nur Kleinbuchstaben, Ziffern oder Bindestriche enthalten, darf nicht mit einem Bindestrich beginnen oder auf einen Bindestrich enden und ist auf 128 Zeichen beschränkt.
- `description`: Eine optionale Beschreibung.
- `type`: Erforderlich. Dabei muss es sich um einen der unterstützten Datenquellentypen handeln:
	- `azuresql`: Azure SQL-Datenbank und SQL Server in Azure VMs
	- `documentdb`: Azure DocumentDB
	- `azureblob`: Azure Blob Storage
	- `azuretable`: Azure Table Storage
- `credentials`:
	- Die erforderlichen Eigenschaft `connectionString` gibt die Verbindungszeichenfolge für die Datenquelle an. Das Format der Verbindungszeichenfolge hängt vom Typ der Datenquelle ab:
		- Für SQL Azure ist dies die übliche SQL Server-Verbindungszeichenfolge. Wenn Sie die Verbindungszeichenfolge über das Azure-Portal abrufen, verwenden Sie die Option `ADO.NET connection string`.
		- Für DocumentDB muss die Verbindungszeichenfolge folgenden Format aufweisen: `"AccountEndpoint=https://[your account name].documents.azure.com;AccountKey=[your account key];Database=[your database id]"`. Alle Werte sind erforderlich. Informationen zu den Werten finden Sie im [Azure-Portal](https://portal.azure.com/).
		- Für Azure Blob Storage und Azure Table Storage ist dies die Verbindungszeichenfolge für das Speicherkonto. Das Format wird [hier](https://azure.microsoft.com/documentation/articles/storage-configure-connection-string/) beschrieben. Ein HTTPS-Endpunktprotokoll ist erforderlich.
- `container`, erforderlich: Gibt die zu indizierenden Daten mithilfe der Eigenschaften `name` und `query` an:
	- `name`, erforderlich:
		- Azure SQL: Gibt die Tabelle oder Sicht an. Sie können schemaqualifizierte Namen verwenden, z.B. `[dbo].[mytable]`.
		- DocumentDB: Gibt die Auflistung an.
		- Azure Blob Storage: Gibt den Speichercontainer an.
		- Azure Table Storage: Gibt den Namen der Tabelle an.
	- `query`, optional:
		- DocumentDB: zum Angeben einer Abfrage, die ein beliebiges JSON-Dokumentlayout in ein Flatfile-Schema reduziert, welches von Azure Search indiziert werden kann.
		- Azure Blob Storage: Erlaubt das Angeben eines virtuellen Ordners im Blobcontainer. Für den Blobpfad `mycontainer/documents/blob.pdf` kann beispielsweise `documents` als virtueller Ordner verwendet werden.
		- Azure Table Storage: Erlaubt das Angeben einer Abfrage, die die Menge der zu importierenden Zeilen filtert.
		- Azure SQL: Abfrage wird nicht unterstützt. Wenn Sie diese Funktion benötigen, stimmen Sie für [diesen Vorschlag](https://feedback.azure.com/forums/263029-azure-search/suggestions/9893490-support-user-provided-query-in-sql-indexer).
- Die optionalen Eigenschaften `dataChangeDetectionPolicy` und `dataDeletionDetectionPolicy` werden nachstehend beschrieben.

<a name="DataChangeDetectionPolicies"></a>**Richtlinien zur Erkennung von Datenänderungen**

Die Richtlinie zum Erkennen von Datenänderungen dient einer effizienten Identifizierung geänderter Datenelemente. Unterstützte Richtlinien hängen vom Typ der Datenquelle ab. Die Richtlinien werden in den folgenden Abschnitten beschrieben.

***Richtlinie zum Erkennen von Änderungen mit oberem Grenzwert***

Verwenden Sie diese Richtlinie, wenn Ihre Datenquelle eine Spalte oder Eigenschaft enthält, die die folgenden Kriterien erfüllt:
 
- Alle Einfügungen geben einen Wert für die Spalte an.
- Alle Updates für ein Element ändern auch den Wert der Spalte.
- Der Wert dieser Spalte wird bei jeder Änderung erhöht.
- Abfragen, die eine Filter-Klausel wie die folgende verwenden `WHERE [High Water Mark Column] > [Current High Water Mark Value]`, können effizient ausgeführt werden.

Wenn Sie beispielsweise Azure-SQL-Datenquellen verwenden, eignet sich eine indizierte `rowversion`-Spalte ideal für die Verwendung mit der Richtlinie mit oberem Grenzwert.

Diese Richtlinie kann wie folgt angegeben werden:

	{ 
		"@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
		"highWaterMarkColumnName" : "[a row version or last_updated column name]" 
	} 

> [AZURE.NOTE] Wenn Sie DocumentDB-Datenquellen verwenden, müssen Sie die von DocumentDB bereitgestellte Eigenschaft `_ts` verwenden.

> [AZURE.NOTE] Bei der Nutzung von Azure-Blobdatenquellen verwendet Azure Search automatisch eine Änderungserkennungsrichtlinie für hohe Grenzwerte auf Basis der letzten Änderung des Blobzeitstempels. Sie müssen eine solche Richtlinie nicht selbst angeben.

***Richtlinie für die integrierte SQL-Erkennung von Änderungen***

Wenn die SQL-Datenbank die [integrierte SQL-Änderungsnachverfolgung](https://msdn.microsoft.com/library/bb933875.aspx) unterstützt, wird empfohlen, die Richtlinie für die integrierte SQL-Änderungsnachverfolgung zu verwenden. Diese Richtlinie ermöglicht die effizienteste Änderungsnachverfolgung und sorgt dafür, dass Azure Search gelöschte Zeilen identifiziert, ohne dass Sie eine explizite "Vorläufig löschen"-Spalte in Ihrem Schema angeben müssen.

Die integrierte Änderungsverfolgung wird , ab den folgenden SQL Server-Datenbankversionen unterstützt:
- SQL Server 2008 R2 bei Verwendung von SQL Server auf virtuellen Azure-Computern.
- Azure SQL-Datenbank V12 bei Verwendung von Azure SQL-Datenbank.

Wenn Sie die Richtlinie für die integrierte SQL-Änderungsnachverfolgung verwenden, geben Sie keine separate Richtlinie für das Erkennen gelöschter Daten an – Unterstützung für die Identifizierung gelöschter Zeilen ist in der Richtlinie bereits integriert.

Diese Richtlinie kann nur mit Tabellen verwendet werden; sie kann nicht mit Ansichten verwendet werden. Sie müssen die Änderungsnachverfolgung für die verwendete Tabelle aktivieren, bevor Sie diese Richtlinie verwenden können. Anweisungen finden Sie unter [Aktivieren und Deaktivieren der Änderungsnachverfolgung](https://msdn.microsoft.com/library/bb964713.aspx).
 
Beim Strukturieren der Anforderung **Datenquelle erstellen** können Sie die Richtlinie für die integrierte SQL-Änderungsnachverfolgung wie folgt angegeben:

	{ 
		"@odata.type" : "#Microsoft.Azure.Search.SqlIntegratedChangeTrackingPolicy" 
	}

<a name="DataDeletionDetectionPolicies"></a>**Richtlinien zur Erkennung von Datenlöschungen**

Die Richtlinie zum Erkennen von Datenlöschungen dient einer effizienten Identifizierung gelöschter Datenelemente. Derzeit wird nur die Richtlinie `Soft Delete` unterstützt. Diese Richtlinie ermöglicht das Identifizieren gelöschter Elemente basierend auf dem Wert einer `soft delete`-Spalte oder -Eigenschaft in der Datenquelle. Diese Richtlinie kann wie folgt angegeben werden:

	{ 
		"@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
		"softDeleteColumnName" : "the column that specifies whether a row was deleted", 
		"softDeleteMarkerValue" : "the value that identifies a row as deleted" 
	}

**HINWEIS:** Es werden nur Spalten mit Werten unterstützt, die einer Zeichenfolge, Ganzzahl oder einem booleschen Werte entsprechen. Der Wert für `softDeleteMarkerValue` muss eine Zeichenfolge sein, selbst wenn die entsprechende Spalte Ganzzahlen oder boolesche Werte enthält. Wenn der in Ihrer Datenquelle angezeigte Wert beispielsweise "1" ist, legen Sie `"1"` für `softDeleteMarkerValue` fest.

<a name="CreateDataSourceRequestExamples"></a>**Beispiel für einen Anforderungstext**

Wenn Sie beabsichtigen, eine Datenquelle mit einem Indexer zu verwenden, der nach einem Zeitplan ausgeführt wird, veranschaulicht dieses Beispiel, wie Sie Richtlinien zu Erkennung von Änderungen und Löschungen angeben:

    { 
		"name" : "asqldatasource",
		"description" : "a description",
    	"type" : "azuresql",
    	"credentials" : { "connectionString" : "Server=tcp:....database.windows.net,1433;Database=...;User ID=...;Password=...;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;" },
    	"container" : { "name" : "sometable" },
    	"dataChangeDetectionPolicy" : { "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy", "highWaterMarkColumnName" : "RowVersion" }, 
    	"dataDeletionDetectionPolicy" : { "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy", "softDeleteColumnName" : "IsDeleted", "softDeleteMarkerValue" : "true" }
	}

Wenn Sie die Datenquelle nur zum einmaligen Kopieren der Daten verwenden möchten, müssen Sie keine Richtlinien angeben:

    { 
		"name" : "asqldatasource",
    	"description" : "anything you want, or nothing at all",
    	"type" : "azuresql",
    	"credentials" : { "connectionString" : "Server=tcp:....database.windows.net,1433;Database=...;User ID=...;Password=...;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;" },
    	"container" : { "name" : "sometable" }
	} 

**Antwort**

Bei erfolgreicher Anforderung: "201 Erstellt".

<a name="UpdateDataSource"></a>
## Datenquelle aktualisieren ##

Sie können eine vorhandene Datenquelle mithilfe einer HTTP-PUT-Anforderung aktualisieren. Geben Sie den Namen der Datenquelle an, die mit der Anforderungs-URI aktualisiert werden soll:

    PUT https://[service name].search.windows.net/datasources/[datasource name]?api-version=[api-version]
    Content-Type: application/json
    api-key: [admin key]

`api-version` ist erforderlich. Die aktuelle Version ist `2015-02-28`. Unter [Versionsverwaltung für Azure Search](https://msdn.microsoft.com/library/azure/dn864560.aspx) finden Sie nähere Angaben und weitere Informationen zu alternativen Versionen.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Anforderung**

Die Syntax des Anforderungstexts ist mit der für [Anforderungen zum Erstellen von Datenquellen](#CreateDataSourceRequestSyntax) identisch.

> [AZURE.NOTE] Manche Eigenschaften vorhandener Datenquellen können nicht aktualisiert werden. Sie können z. B. nicht den Typ einer vorhandenen Datenquelle ändern.

> [AZURE.NOTE] Wenn Sie die Verbindungszeichenfolge für eine vorhandene Datenquelle nicht ändern möchten, können Sie das Literal `<unchanged>` für die Verbindungszeichenfolge angeben. Dies ist hilfreich in Situationen, in denen Sie eine Datenquelle aktualisieren müssen, aber keinen bequemen Zugriff auf die Verbindungszeichenfolge haben, da sie zu den vertraulichen Daten zählt.

**Antwort**

Bei erfolgreicher Anforderung: „201 Erstellt“, wenn eine neue Datenquelle erstellt wurde; „204 Kein Inhalt“, wenn eine vorhandene Datenquelle aktualisiert wurde.

<a name="ListDataSource"></a>
## Datenquellen auflisten ##

Der Vorgang **Datenquellen auflisten** gibt eine Liste der Datenquellen im Azure Search-Dienst zurück.

    GET https://[service name].search.windows.net/datasources?api-version=[api-version]
    api-key: [admin key]

`api-version` ist erforderlich. Die aktuelle Version ist `2015-02-28`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Bei erfolgreicher Anforderung: "200 OK".

Hier ist ein Beispiel-Antworttext:

    {
      "value" : [
        {
          "name": "datasource1",
          "type": "azuresql",
		  ... other data source properties
        }]
    }

Beachten Sie, dass Sie die Antwort filtern können, um nur die Eigenschaften, die Sie interessieren, zu erhalten. Wenn Sie beispielsweise nur eine Liste der Namen der Datenquellen erhalten möchten, verwenden Sie die OData-Abfrageoption `$select`:

    GET /datasources?api-version=205-02-28&$select=name

In diesem Fall würde die Antwort aus dem obigen Beispiel wie folgt aussehen:

    {
      "value" : [ { "name": "datasource1" }, ... ]
    }

Dies ist ein nützliches Verfahren, um Bandbreite zu sparen, wenn Sie über zahlreiche Datenquellen im Search-Dienst verfügen.

<a name="GetDataSource"></a>
## Datenquelle abrufen ##

Der Vorgang **Datenquelle abrufen** ruft die Definition der Datenquelle aus Azure Search ab.

    GET https://[service name].search.windows.net/datasources/[datasource name]?api-version=[api-version]
    api-key: [admin key]

`api-version` ist erforderlich. Die aktuelle Version ist `2015-02-28`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Bei erfolgreicher Antwort wird der Statuscode "200 OK" zurückgegeben.

Die Antwort ähnelt den Beispielen unter [Beispiele für Anforderung "Datenquelle erstellen"](#CreateDataSourceRequestExamples):

	{ 
		"name" : "asqldatasource",
		"description" : "a description",
    	"type" : "azuresql",
    	"credentials" : { "connectionString" : "Server=tcp:....database.windows.net,1433;Database=...;User ID=...;Password=...;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;" },
    	"container" : { "name" : "sometable" },
    	"dataChangeDetectionPolicy" : { 
            "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
			"highWaterMarkColumnName" : "RowVersion" }, 
    	"dataDeletionDetectionPolicy" : { 
            "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
			"softDeleteColumnName" : "IsDeleted", 
			"softDeleteMarkerValue" : "true" }
	}

> [AZURE.NOTE] Legen Sie beim Aufrufen dieser API den `Accept`-Anforderungsheader nicht auf `application/json;odata.metadata=none` fest. Andernfalls wird das `@odata.type`-Attribut nicht in der Antwort berücksichtigt, sodass Sie nicht zwischen den Richtlinien für die Erkennung geänderter und gelöschter Daten verschiedener Typen unterscheiden können.

<a name="DeleteDataSource"></a>
## Datenquelle löschen ##

Der Vorgang **Datenquelle löschen** entfernt eine Datenquelle aus Ihrem Azure Search-Dienst.

    DELETE https://[service name].search.windows.net/datasources/[datasource name]?api-version=[api-version]
    api-key: [admin key]

> [AZURE.NOTE] Wenn die zu löschende Datenquelle von einem Indexer referenziert wird, wird der Löschvorgang dennoch normal fortgesetzt. Der Indexer geht jedoch bei der nächsten Ausführung in den Fehlerstatus über.

`api-version` ist erforderlich. Die aktuelle Version ist `2015-02-28`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Bei erfolgreicher Antwort wird der Statuscode "204 Kein Inhalt" zurückgegeben.

<a name="CreateIndexer"></a>
## Indexer erstellen ##

Sie können einen neuen Indexer innerhalb des Azure Search-Dienstes mithilfe einer HTTP POST-Anforderung erstellen.
	
    POST https://[service name].search.windows.net/indexers?api-version=[api-version]
    Content-Type: application/json
    api-key: [admin key]

Alternativ können Sie PUT verwenden und den Namen der Datenquelle für den URI angeben. Wenn die Datenquelle nicht vorhanden ist, wird sie erstellt.

    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=[api-version]

> [AZURE.NOTE] Die maximal zulässige Anzahl von Indexern variiert je nach Preisstufe. Der kostenlose Dienst kann bis zu 3 Indexer enthalten. Der Standard-Dienst kann 50 Indexer enthalten. In den [Einschränkungen für Dienste](search-limits-quotas-capacity.md) finden Sie weitere Informationen.

`api-version` ist erforderlich. Die aktuelle Version ist `2015-02-28`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.


<a name="CreateIndexerRequestSyntax"></a> **Syntax des Anforderungstextes**

Der Anforderungstext enthält eine Indexer-Definition, in der die Datenquelle und der Zielindex für die Indizierung angegeben sind, sowie einen optionalen Indizierungszeitplan und -parameter.


Die Syntax für die Strukturierung der Anforderungsnutzlast ist wie folgt. Eine Beispielanforderung wird weiter unten in diesem Thema bereitgestellt.

    { 
		"name" : "Required for POST, optional for PUT. The name of the indexer",
    	"description" : "Optional. Anything you want, or null",
    	"dataSourceName" : "Required. The name of an existing data source",
        "targetIndexName" : "Required. The name of an existing index",
        "schedule" : { Optional. See Indexing Schedule below. },
        "parameters" : { Optional. See Indexing Parameters below. },
        "fieldMappings" : { Optional. See Field Mappings below. },
        "disabled" : Optional boolean value indicating whether the indexer is disabled. False by default.  
	}

**Indexer-Zeitplan**

Ein Indexer kann optional einen Zeitplan angeben. Wenn ein Zeitplan vorliegt, wird der Indexer regelmäßig gemäß Zeitplan ausgeführt. Der Zeitplan besitzt die folgenden Attribute:

- `interval`: Erforderlich. Ein Zeitdauerwert, der ein Intervall oder den Zeitraum für Indexer-Ausführungen angibt. Das kleinste zulässige Intervall beträgt 5 Minuten. Das längste ist ein Tag. Es muss als XSD-Wert "dayTimeDuration" formatiert sein (eine eingeschränkte Teilmenge eines [ISO 8601-Zeitdauerwerts](http://www.w3.org/TR/xmlschema11-2/#dayTimeDuration)). Das Muster hierfür lautet wie folgt: `"P[nD][T[nH][nM]]"`. Beispiele: `PT15M` = alle 15 Minuten, `PT2H` = alle 2 Stunden.

- `startTime`: Erforderlich. Ein UTC-DateTime-Wert, der angibt, wann die Ausführung des Indexers beginnen soll.

**Indexer-Parameter**

Optional kann ein Indexer mehrere Parameter angeben, die sein Verhalten beeinflussen. Alle Parameter sind optional.

- `maxFailedItems` : Die maximale Anzahl der Elemente, deren Indizierung fehlschlagen darf. Wird die Anzahl überschritten, gilt die Ausführung des Indexers als fehlgeschlagen. Der Standardwert ist 0. Informationen zu fehlgeschlagenen Elementen werden mithilfe des Vorgangs [Indexer-Status abrufen](#GetIndexerStatus) zurückgegeben.

- `maxFailedItemsPerBatch` : Die maximale Anzahl der Elemente in jedem Batch, deren Indizierung fehlschlagen darf. Wird die Anzahl überschritten, gilt die Ausführung des Indexers als fehlgeschlagen. Der Standardwert ist 0.

- `base64EncodeKeys`: Gibt an, ob Dokumentschlüssel mit base-64 codiert werden. In Azure Search gelten Einschränkungen für Zeichen, die in einem Dokumentschlüssel vorhanden sein können. Allerdings können die Werte in Ihren Quelldaten Zeichen enthalten, die ungültig sind. Wenn solche Werte als Dokumentschlüssel indiziert werden sollen, können Sie dieses Kennzeichen auf "true" festlegen. Der Standardwert ist `false`.

- `batchSize`: Gibt die Anzahl der Elemente an, die aus einer Datenquelle gelesen und als einzelner Batch indiziert werden, um die Leistung zu verbessern. Der Standardwert hängt vom Datenquellentyp ab: 1.000 für Azure SQL und DocumentDB und 10 für Azure-Blobspeicher.

**Feldzuordnungen**

Verwenden Sie Feldzuordnungen, um einen Feldnamen in der Datenquelle einem anderen Feldnamen im Zielindex zuzuordnen. Angenommen, die Quelltabelle enthält ein Feld `_id`. Da Feldnamen in Azure Search nicht einem Unterstrich beginnen dürfen, muss das Feld umbenannt werden. Hierzu können Sie die `fieldMappings`-Eigenschaft des Indexers wie folgt verwenden:
	
	"fieldMappings" : [ { "sourceFieldName" : "_id", "targetFieldName" : "id" } ] 

Sie können mehrere Feldzuordnungen angeben.

	"fieldMappings" : [ 
		{ "sourceFieldName" : "_id", "targetFieldName" : "id" },
        { "sourceFieldName" : "_timestamp", "targetFieldName" : "timestamp" },
	 ]

Beachten Sie bei Quell- und Zielfeldnamen die Groß-und Kleinschreibung.

<a name="FieldMappingFunctions"></a> ***Feldzuordnungsfunktionen***

Feldzuordnungen können auch verwendet werden, um Quellfeldwerte mithilfe von *Zuordnungsfunktionen* umzuwandeln.

Derzeit wird nur eine Zuordnungsfunktion unterstützt: `jsonArrayToStringCollection`. Hierbei wird ein Feld, dass eine als JSON-Array formatierte Zeichenfolge enthält, in ein Collection(Edm.String)-Feld im Zielindex geparst. Die Funktion ist insbesondere zur Verwendung mit dem Azure SQL-Indexer vorgesehen, da SQL über keinen nativen Datentyp "Sammlung" verfügt. Sie können die Funktion wie folgt verwenden:

	"fieldMappings" : [ { "sourceFieldName" : "tags", "mappingFunction" : { "name" : "jsonArrayToStringCollection" } } ] 

Wenn das Quellfeld beispielsweise die Zeichenfolge `["red", "white", "blue"]` enthält, wird das Zielfeld vom Typ `Collection(Edm.String)` mit drei Werten gefüllt: `"red"`, `"white"` und `"blue"`.

Beachten Sie, dass die `targetFieldName`Eigenschaft optional ist. Wenn Sie keinen Wert angeben, wird der `sourceFieldName`-Wert verwendet.

<a name="CreateIndexerRequestExamples"></a>**Beispiel für einen Anforderungstext**

Anhand des folgenden Beispiels wird ein Indexer erstellt, der Daten aus der Tabelle, die von der `ordersds`-Datenquelle referenziert wird, in den `orders`-Index kopiert. Der Vorgang erfolgt nach einem Zeitplan, der am 1. Januar 2015 UTC beginnt und stündlich ausgeführt wird. Die Aufrufe des Indexers sind erfolgreich, wenn nicht mehr als fünf zu indizierende Elemente in jedem Batch fehlschlagen und insgesamt nicht mehr als zehn zu indizierende Elemente fehlschlagen.

	{
        "name" : "myindexer",
        "description" : "a cool indexer",
        "dataSourceName" : "ordersds",
        "targetIndexName" : "orders",
        "schedule" : { "interval" : "PT1H", "startTime" : "2015-01-01T00:00:00Z" },
        "parameters" : { "maxFailedItems" : 10, "maxFailedItemsPerBatch" : 5, "base64EncodeKeys": false }
	}

**Antwort**

Bei erfolgreicher Anforderung: "201 Erstellt".


<a name="UpdateIndexer"></a>
## Indexer aktualisieren ##

Sie können einen vorhandene Indexer mithilfe einer HTTP PUT-Anforderung aktualisieren. Geben Sie den Namen des Indexers an, der mit der Anforderungs-URI aktualisiert werden soll:

    PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=[api-version]
    Content-Type: application/json
    api-key: [admin key]

`api-version` ist erforderlich. Die aktuelle Version ist `2015-02-28`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Anforderung**

Die Syntax des Anforderungstexts ist mit der für [Anforderungen "Indexer erstellen"](#CreateIndexerRequestSyntax) identisch.

**Antwort**

Bei erfolgreicher Anforderung: "201 Erstellt", wenn ein neuer Indexer erstellt wurde; "204 Kein Inhalt", wenn eine vorhandener Indexer aktualisiert wurde.


<a name="ListIndexers"></a>
## Indexer auflisten ##

Der Vorgang **Indexer auflisten** gibt die Liste der Indexer in Ihrem Azure Search-Dienst zurück.

    GET https://[service name].search.windows.net/indexers?api-version=[api-version]
    api-key: [admin key]


`api-version` ist erforderlich. Die Vorschauversion ist `2015-02-28-Preview`. Unter [Versionsverwaltung für Azure Search](https://msdn.microsoft.com/library/azure/dn864560.aspx) finden Sie nähere Angaben und weitere Informationen zu alternativen Versionen.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Bei erfolgreicher Anforderung: "200 OK".

Hier ist ein Beispiel-Antworttext:

    {
      "value" : [
      {
        "name" : "myindexer",
        "description" : "a cool indexer",
        "dataSourceName" : "ordersds",
        "targetIndexName" : "orders",
        ... other indexer properties
	  }]
    }

Beachten Sie, dass Sie die Antwort filtern können, um nur die Eigenschaften, die Sie interessieren, zu erhalten. Wenn Sie beispielsweise nur eine Liste der Namen der Indexer erhalten möchten, verwenden Sie die OData-Abfrageoption `$select`:

    GET /indexers?api-version=2014-10-20-Preview&$select=name

In diesem Fall würde die Antwort aus dem obigen Beispiel wie folgt aussehen:

    {
      "value" : [ { "name": "myindexer" } ]
    }

Dies ist ein nützliches Verfahren, um Bandbreite zu sparen, wenn Sie über zahlreiche Indexer im Search-Dienst verfügen.


<a name="GetIndexer"></a>
## Indexer abrufen ##

Der Vorgang **Indexer abrufen** ruft die Indexer-Definition aus Azure Search ab.

    GET https://[service name].search.windows.net/indexers/[indexer name]?api-version=[api-version]
    api-key: [admin key]

`api-version` ist erforderlich. Die Vorschauversion ist `2015-02-28-Preview`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Bei erfolgreicher Antwort wird der Statuscode "200 OK" zurückgegeben.

Die Antwort ähnelt den Beispielen unter [Beispiele für Anforderung "Datenquelle erstellen"](#CreateIndexerRequestExamples):

	{
        "name" : "myindexer",
        "description" : "a cool indexer",
        "dataSourceName" : "ordersds",
        "targetIndexName" : "orders",
        "schedule" : { "interval" : "PT1H", "startTime" : "2015-01-01T00:00:00Z" },
        "parameters" : { "maxFailedItems" : 10, "maxFailedItemsPerBatch" : 5, "base64EncodeKeys": false }
	}


<a name="DeleteIndexer"></a>
## Indexer löschen ##

Der Vorgang **Indexer löschen** entfernt einen Indexer aus Ihrem Azure Search-Dienst.

    DELETE https://[service name].search.windows.net/indexers/[indexer name]?api-version=[api-version]
    api-key: [admin key]

Beim Löschen eines Indexers, der gerade ausgeführt wird, werden die Ausführungen abgeschlossen, aber keine weiteren Ausführungen mehr geplant. Wenn Sie versuchen, einen nicht mehr vorhandenen Indexer zu verwenden, erhalten Sie den HTTP-Statuscode 404 Nicht gefunden.Versucht, einen nicht vorhandene Indexer verwenden, führt zu HTTP-Statuscode "404 Nicht gefunden".
 
`api-version` ist erforderlich. Die Vorschauversion ist `2015-02-28-Preview`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Bei erfolgreicher Antwort wird der Statuscode "204 Kein Inhalt" zurückgegeben.

<a name="RunIndexer"></a>
## Indexer ausführen ##

Zusätzlich zur Ausführung in regelmäßigen Abständen nach einem Zeitplan kann ein Indexer bei Bedarf auch mit der Anforderung **Indexer ausführen** aufgerufen werden:

	POST https://[service name].search.windows.net/indexers/[indexer name]/run?api-version=[api-version]
    api-key: [admin key]

`api-version` ist erforderlich. Die Vorschauversion ist `2015-02-28-Preview`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Bei erfolgreicher Antwort wird der Statuscode "202 Zulässig" zurückgegeben.

<a name="GetIndexerStatus"></a>
## Indexer-Status abrufen ##

Der Vorgang **Indexer-Status abrufen** ruft den aktuellen Status und den Ausführungsverlauf eines Indexers ab:

	GET https://[service name].search.windows.net/indexers/[indexer name]/status?api-version=[api-version]
    api-key: [admin key]


`api-version` ist erforderlich. Die Vorschauversion ist `2015-02-28-Preview`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Statuscode: "200 OK" bei erfolgreicher Antwort.

Der Antworttext enthält Informationen über den Gesamtintegritätsstatus des Indexers, den letzten Indexer-Aufruf, sowie den Verlauf der jüngsten Indexer-Aufrufe (sofern vorhanden).

Ein Beispiel-Antworttext sieht folgendermaßen aus:

	{
		"status":"running",
		"lastResult": {
			"status":"success",
			"errorMessage":null,
			"startTime":"2014-11-26T03:37:18.853Z",
			"endTime":"2014-11-26T03:37:19.012Z",
			"errors":[],
			"itemsProcessed":11,
			"itemsFailed":0,
			"initialTrackingState":null,
			"finalTrackingState":null
         },
		"executionHistory":[ {
			"status":"success",
         	"errorMessage":null,
			"startTime":"2014-11-26T03:37:18.853Z",
			"endTime":"2014-11-26T03:37:19.012Z",
			"errors":[],
			"itemsProcessed":11,
			"itemsFailed":0,
			"initialTrackingState":null,
			"finalTrackingState":null
		}]
	}

**Indexer-Status**

Der Indexer-Status kann einen der folgenden Werte aufweisen:

- `running` gibt an, dass der Indexer normal ausgeführt wird. Beachten Sie, dass einige der Indexer-Ausführungen nach wie vor fehlerhaft sein können. Überprüfen Sie daher auch die `lastResult`-Eigenschaft.

- `error` gibt an, dass der Indexer einen Fehler festgestellt hat, der ohne das Eingreifen des Benutzers nicht behoben werden kann. Beispiele hierfür sind abgelaufene Anmeldeinformationen für die Datenquelle oder ein geändertes, nun fehlerhaftes Schema der Datenquelle oder des Zielindexes.

**Ergebnis der Indexer-Ausführung**

Das Ergebnis der Indexer-Ausführung enthält Informationen über die Ausführung eines einzelnen Indexers. Das neueste Ergebnis wird als `lastResult`-Eigenschaft des Indexer-Status bereitgestellt. Andere aktuelle Ergebnisse, falls vorhanden, werden als `executionHistory`-Eigenschaft des Indexer-Status zurückgegeben.

Das Indexer-Ausführungsergebnis enthält die folgenden Eigenschaften:

- `status`: Der Status der Ausführung. Weitere Informationen finden Sie weiter unten unter [Status der Indexer-Ausführung](#IndexerExecutionStatus).

- `errorMessage`: Fehlermeldung für eine fehlgeschlagene Ausführung.

- `startTime`: Die Zeit in UTC, zu der die Ausführung gestartet wurde.

- `endTime`: Die Zeit in UTC, zu der die Ausführung gestartet wurde. Dieser Wert wird nicht festgelegt, wenn die Ausführung noch nicht abgeschlossen ist.

- `errors`: Ein Array mit Fehlern auf Elementebene, sofern vorhanden. Jeder Eintrag enthält einen Dokumentschlüssel (`key`-Eigenschaft) und eine Fehlermeldung (`errorMessage`-Eigenschaft).

- `itemsProcessed`: Die Anzahl der vom Indexer in dieser Ausführung zu indizierenden Datenquellen-Elemente (z. B. Zeilen in Tabelle).

- `itemsFailed`: Die Anzahl der Elemente, die während der Ausführung fehlgeschlagen sind.
 
- `initialTrackingState`: Ist immer `null`, wenn der Indexer erstmalig ausgeführt wird oder wenn die Richtlinie für die Änderungsnachverfolgung für Daten für die verwendete Datenquelle nicht aktiviert ist. Wenn eine solche Richtlinie aktiviert ist, gibt dieser Wert in den nachfolgenden Ausführungen den ersten (niedrigste Priorität) Änderungsnachverfolgungswert an, der bei dieser Ausführung verarbeitet wurde.

- `finalTrackingState`: Ist immer `null` wenn die Richtlinie für die Änderungsnachverfolgung für Daten für die verwendete Datenquelle nicht aktiviert ist. Andernfalls wird der neueste (höchste Priorität) Änderungsnachverfolgungswert angegeben, der bei dieser Ausführung erfolgreich verarbeitet wurde.

<a name="IndexerExecutionStatus"></a>**Status der Indexer-Ausführung**

Der Status der Indexer-Ausführung erfasst den Status einer einzelnen Indexer-Ausführung. Die folgenden Werte sind möglich:

- `success` gibt an, dass die Indexer-Ausführung erfolgreich abgeschlossen wurde.

- `inProgress` gibt an, dass der Indexer gerade ausgeführt wird.

- `transientFailure` gibt an, dass eine Indexer-Ausführung fehlgeschlagen ist. Weitere Informationen finden Sie in der Eigenschaft `errorMessage`. Zum Beheben des Fehlers ist möglicherweise ein Eingreifen des Benutzers erforderlich. Die Behebung einer Schema-Inkompatibilität erfordert beispielsweise eine Benutzeraktion; ein vorübergehender Ausfall der Datenquelle hingegen nicht. Die nachfolgenden Indexer-Aufrufe werden wie geplant ausgeführt, falls zutreffend.

- `persistentFailure` gibt an, dass zum Beheben des Indexer-Fehlers eine Benutzeraktion erforderlich ist. Die geplanten Indexer-Ausführungen werden gestoppt. Beheben Sie das Problem und starten Sie die geplanten Ausführungen mithilfe der API zum Zurücksetzen des Indexers neu.

- `reset` gibt an, dass der Indexer durch den Aufruf der API zum Zurücksetzen des Indexers (siehe unten) zurückgesetzt wurde.

<a name="ResetIndexer"></a>
## Indexer zurücksetzen ##

Der Vorgang **Indexer zurücksetzen** setzt den mit dem Indexer verknüpften Status der Änderungsnachverfolgung zurück. Dies ermöglicht Ihnen, eine komplett neue Indizierung durchführen (z. B. wenn Ihr Datenquellenschema geändert wurde) oder die Richtlinie für die Änderungsnachverfolgung für Daten für eine mit dem Indexer verknüpfte Datenquelle zu ändern.

	POST https://[service name].search.windows.net/indexers/[indexer name]/reset?api-version=[api-version]
    api-key: [admin key]

`api-version` ist erforderlich. Die Vorschauversion ist `2015-02-28-Preview`.

`api-key` muss ein Admin-Schlüssel sein (im Gegensatz zu einer Abfrageschlüssel). Weitere Informationen zu Schlüsseln finden Sie unter [REST-API für den Search-Dienst](https://msdn.microsoft.com/library/azure/dn798935.aspx) im Abschnitt "Authentifizierung". In [Erstellen eine Search-Dienstes im Portal](search-create-service-portal.md) wird erläutert, wie Sie die Dienst-URL und Schlüsseleigenschaften, die in der Anforderung verwendet werden, abrufen können.

**Antwort**

Statuscode "204 Kein Inhalt" bei erfolgreicher Antwort.

## Zuordnung zwischen SQL-Datentypen und Azure Search-Datentypen ##

<table style="font-size:12">
<tr>
<td>SQL-Datentyp</td>	
<td>Zulässige Ziel-Index-Feldtypen</td>
<td>Hinweise</td>
</tr>
<tr>
<td>Bit</td>
<td>Edm.Boolean, Edm.String</td>
<td></td>
</tr>
<tr>
<td>int, smallint, tinyint</td>
<td>Edm.Int32, Edm.Int64, Edm.String</td>
<td></td>
</tr>
<tr>
<td>bigint</td>
<td>Edm.Int64, Edm.String</td>
<td></td>
</tr>
<tr>
<td>real, float</td>
<td>Edm.Double, Edm.String</td>
<td></td>
</tr>
<tr>
<td>smallmoney, money<br>decimal<br>numeric
</td>
<td>Edm.String</td>
<td>Azure Search unterstützt nicht die Konvertierung von Dezimaltypen in Edm.Double-Typen, da es hierbei zu Genauigkeitsverlusten kommt.
</td>
</tr>
<tr>
<td>char, nchar, varchar, nvarchar</td>
<td>Edm.String<br/>Collection(Edm.String)</td>
<td>Ausführliche Informationen zum Transformieren einer Zeichenfolgenspalte in Collection(Edm.String) finden Sie unter [Feldzuordnungsfunktionen](#FieldMappingFunctions) in diesem Dokument. </td>
</tr>
<tr>
<td>smalldatetime, datetime, datetime2, date, datetimeoffset</td>
<td>Edm.DateTimeOffset, Edm.String</td>
<td></td>
</tr>
<tr>
<td>uniqueidentifer</td>
<td>Edm.String</td>
<td></td>
</tr>
<tr>
<td>geography</td>
<td>Edm.GeographyPoint</td>
<td>Es werden nur Geography-Instanzen vom Typ POINT mit SRID 4326 (Standard) unterstützt.</td>
</tr>
<tr>
<td>rowversion</td>
<td>N/V</td>
<td>rowversion-Spalten können nicht im Suchindex gespeichert werden, sie können jedoch für die Änderungsnachverfolgung verwendet werden.</td>
</tr>
<tr>
<td>time, timespan<br>binary, varbinary, image,<br>xml, geometry, CLR types</td>
<td>–</td>
<td>Nicht unterstützt</td>
</tr>
</table>

## Zuordnung zwischen JSON-Datentypen und Azure Search-Datentypen ##

<table style="font-size:12">
<tr>
<td>JSON-Datentyp</td>	
<td>Zulässige Ziel-Index-Feldtypen</td>
<td>Hinweise</td>
</tr>
<tr>
<td>bool</td>
<td>Edm.Boolean, Edm.String</td>
<td></td>
</tr>
<tr>
<td>Ganzzahlen</td>
<td>Edm.Int32, Edm.Int64, Edm.String</td>
<td></td>
</tr>
<tr>
<td>Gleitkommazahlen</td>
<td>Edm.Double, Edm.String</td>
<td></td>
</tr>
<tr>
<td>string</td>
<td>Edm.String</td>
<td></td>
</tr>
<tr>
<td>Arrays primitiver Typen, z.&#160;B. ["a", "b", "c"]</td>
<td>Collection(Edm.String)</td>
<td></td>
</tr>
<tr>
<td>Zeichenfolgen, die wie Datumsangaben aussehen</td>
<td>Edm.DateTimeOffset, Edm.String</td>
<td></td>
</tr>
<tr>
<td>GeoJSON-Punktobjekte</td>
<td>Edm.GeographyPoint</td>
<td>GeoJSON-Punkte sind JSON-Objekte im folgenden Format: {"Typ" : "Punkt", "Koordinaten": [Breitengrad, Längengrad]} </td>
</tr>
<tr>
<td>Andere JSON-Objekte</td>
<td>N/V</td>
<td>Nicht unterstützt; Azure Search unterstützt derzeit nur primitive Typen und Zeichenfolgensammlungen</td>
</tr>
</table>

<!---HONumber=AcomDC_0914_2016-->