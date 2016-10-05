<properties
	pageTitle="Verwenden des Azure-Tabellenspeichers mit Node.js | Microsoft Azure"
	description="Speichern Sie strukturierte Daten mit Azure Table Storage, einem NoSQL-Datenspeicher, in der Cloud."
	services="storage"
	documentationCenter="nodejs"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="nodejs"
	ms.topic="article"
	ms.date="08/11/2016"
	ms.author="micurd"/>


# Verwenden des Azure-Tabellenspeichers mit Node.js

[AZURE.INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
<br/>
[AZURE.INCLUDE [storage-try-azure-tools-queues](../../includes/storage-try-azure-tools-tables.md)]

## Übersicht

In diesem Thema wird erläutert, wie allgemeine Szenarios mit dem Azure-Tabellenspeicherdienst in einer Node.js-Anwendung durchgeführt werden.

Bei den Codebeispielen in diesem Thema wird davon ausgegangen, dass Sie bereits eine Node.js-Anwendung besitzen. Informationen zum Erstellen einer Node.js-Anwendung in Azure finden Sie in folgenden Themen:

- [Erstellen einer Node.js-Web-App in Azure App Service](../app-service-web/web-sites-nodejs-develop-deploy-mac.md)
- [Erstellen und Bereitstellen einer Node.js-Web-App für Azure mithilfe von WebMatrix](../app-service-web/web-sites-nodejs-use-webmatrix.md)
- [Erstellen und Bereitstellen einer Node.js-Anwendung in einem Azure-Clouddienst](../cloud-services/cloud-services-nodejs-develop-deploy-app.md) (mit Windows PowerShell)


[AZURE.INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]


## Konfigurieren der Anwendung für den Zugriff auf Azure Storage

Um Azure Storage verwenden zu können, müssen Sie das Azure Storage-SDK für Node.js herunterladen. Es enthält eine Reihe von Bibliotheken, die mit den REST-Speicherdiensten kommunizieren.

### Verwenden des Node-Paket-Managers (NPM) zum Installieren des Pakets

1.  Verwenden Sie eine Befehlszeilenschnittstelle, z. B. **PowerShell** (Windows), **Terminal** (Mac) oder **Bash** (Unix), und navigieren Sie zu dem Ordner, in dem Sie die Beispielanwendung erstellt haben.

2.  Geben Sie **npm install azure-storage** in das Befehlsfenster ein. Die Ausgabe des Befehls ähnelt dem folgenden Beispiel.

		azure-storage@0.5.0 node_modules\azure-storage
		+-- extend@1.2.1
		+-- xmlbuilder@0.4.3
		+-- mime@1.2.11
		+-- node-uuid@1.4.3
		+-- validator@3.22.2
		+-- underscore@1.4.4
		+-- readable-stream@1.0.33 (string_decoder@0.10.31, isarray@0.0.1, inherits@2.0.1, core-util-is@1.0.1)
		+-- xml2js@0.2.7 (sax@0.5.2)
		+-- request@2.57.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, oauth-sign@0.8.0, tunnel-agent@0.4.1, isstream@0.1.2, json-stringify-safe@5.0.1, bl@0.9.4, combined-stream@1.0.5, qs@3.1.0, mime-types@2.0.14, form-data@0.2.0, http-signature@0.11.0, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)

3.  Sie können den Befehl **ls** manuell aufrufen, um sich davon zu überzeugen, dass der Ordner **node\_modules** erstellt wurde. In diesem Ordner finden Sie das Paket **azure-storage** mit den Bibliotheken, die Sie benötigen, um auf den Speicher zuzugreifen.

### Importieren des Pakets

Fügen Sie den folgenden Code am Anfang der Datei **server.js** in der Anwendung hinzu:

	var azure = require('azure-storage');

## Einrichten einer Azure-Speicherverbindung

Das Azure-Modul entnimmt den Umgebungsvariablen AZURE\_STORAGE\_ACCOUNT und AZURE\_STORAGE\_ACCESS\_KEY oder AZURE\_STORAGE\_CONNECTION\_STRING die Informationen, die zum Herstellen einer Verbindung mit Ihrem Azure-Speicherkonto benötigt werden. Wenn diese Umgebungsvariablen nicht festgelegt wurden, müssen Sie die Kontoinformationen beim Aufruf von **TableService** angeben.

Ein Beispiel zum Festlegen der Umgebungsvariablen für eine Azure-Website im [Azure-Portal](https://portal.azure.com) finden Sie unter [Node.js-Web-App mit Azure-Tabellenspeicherdienst].

## Erstellen einer Tabelle

Mit dem folgenden Code wird ein **TableService**-Objekt erstellt und zum Erstellen einer neuen Tabelle verwendet. Fügen Sie Folgenden am Anfang der Datei **server.js** ein.

	var tableSvc = azure.createTableService();

Durch den Aufruf von **createTableIfNotExists**, wird eine neue Tabelle mit dem angegebenen Namen erstellt, sofern sie nicht bereits vorhanden ist. Im folgenden Beispiel wird eine neue Tabelle namens 'mytable' erstellt, wenn diese noch nicht vorhanden ist:

	tableSvc.createTableIfNotExists('mytable', function(error, result, response){
	  if(!error){
	    // Table exists or created
	  }
	});

`result.created` ist `true`, wenn eine neue Tabelle erstellt wird, und `false`, wenn die Tabelle bereits vorhanden ist. `response` enthält Informationen über die Anforderung.

### Filter

Auf die Vorgänge, die mit **TableService** ausgeführt werden, können optionale Filtervorgänge angewendet werden. Filtervorgänge können Protokollierung, automatische Wiederholung usw. umfassen. Filter sind Objekte, die eine Methode mit der folgenden Signatur implementieren:

	function handle (requestOptions, next)

Nachdem die Vorverarbeitung der Anforderungsoptionen abgeschlossen ist, muss die Methode "next" aufrufen und hierbei eine Rückruffunktion mit der folgenden Signatur übergeben:

	function (returnObject, finalCallback, next)

Nachdem das "returnObject"-Objekt (die Antwort auf die an den Server gesendete Anforderung) verarbeitet wurde, muss in dieser Rückruffunktion entweder "next" aufgerufen werden, wenn die Tabelle vorhanden ist, um weitere Filter zu verarbeiten, oder es muss einfach "finalCallback" aufgerufen werden, um den Dienstaufruf zu beenden.

Zwei Filter, die eine Wiederholungslogik implementieren, sind im Azure SDK für Node.js enthalten: **ExponentialRetryPolicyFilter** und **LinearRetryPolicyFilter**. Mit folgendem Code wird ein **TableService**-Objekt, das **ExponentialRetryPolicyFilter** verwendet:

	var retryOperations = new azure.ExponentialRetryPolicyFilter();
	var tableSvc = azure.createTableService().withFilter(retryOperations);

## Hinzufügen einer Entität zu einer Tabelle

Um eine Entität hinzuzufügen, erstellen Sie zunächst ein Objekt, das die Entitätseigenschaften definiert. Alle Entitäten müssen **PartitionKey** und **RowKey** enthalten, die eindeutige Bezeichner für die Entität darstellen.

* **PartitionKey**: bestimmt die Partition, in der die Entität gespeichert ist.

* **RowKey**: identifiziert die Entität innerhalb der Partition eindeutig.

**PartitionKey** und **RowKey** müssen Zeichenfolgenwerte sein. Weitere Informationen finden Sie unter [Grundlegendes zum Tabellendienst-Datenmodell](http://msdn.microsoft.com/library/azure/dd179338.aspx).

Nachfolgend sehen Sie ein Beispiel für die Definition einer Entität. Beachten Sie, dass **dueDate** vom Typ **Edm.DateTime** definiert ist. Die Angabe des Typs ist optional. Nicht angegebene Typen werden abgeleitet.

	var task = {
	  PartitionKey: {'_':'hometasks'},
	  RowKey: {'_': '1'},
	  description: {'_':'take out the trash'},
	  dueDate: {'_':new Date(2015, 6, 20), '$':'Edm.DateTime'}
	};

> [AZURE.NOTE] Für jeden Datensatz gibt es ein **Timestamp**-Feld, das von Azure festgelegt wird, wenn eine Entität eingefügt oder aktualisiert wird.

Sie können Entitäten auch mit dem **entityGenerator** erstellen. Im folgenden Beispiel wird dieselbe Task-Entität mit dem **entityGenerator** erstellt.

	var entGen = azure.TableUtilities.entityGenerator;
	var task = {
	  PartitionKey: entGen.String('hometasks'),
	  RowKey: entGen.String('1'),
	  description: entGen.String('take out the trash'),
	  dueDate: entGen.DateTime(new Date(Date.UTC(2015, 6, 20))),
	};

Um eine Entität der Tabelle hinzuzufügen, übergeben Sie das Entitätsobjekt der **insertEntity**-Methode.

	tableSvc.insertEntity('mytable',task, function (error, result, response) {
	  if(!error){
	    // Entity inserted
	  }
	});

Ist der Vorgang erfolgreich, enthält `result` das [ETag](http://en.wikipedia.org/wiki/HTTP_ETag) des eingefügten Datensatzes, und `response` enthält Informationen zu dem Vorgang.

Beispielantwort:

	{ '.metadata': { etag: 'W/"datetime\'2015-02-25T01%3A22%3A22.5Z\'"' } }

> [AZURE.NOTE] Standardmäßig gibt **insertEntity** die eingefügte Entität nicht als Teil der `response`-Informationen zurück. Wenn Sie weitere Vorgänge mit der Entität ausführen oder die Informationen zwischenspeichern möchten, können Sie sie als Teil von `result` zurückgeben. Hierzu aktivieren Sie **echoContent** wie folgt:
>
> `tableSvc.insertEntity('mytable', task, {echoContent: true}, function (error, result, response) {...}`

## Aktualisieren einer Entität

Es sind mehrere Methoden zum Aktualisieren einer vorhandenen Entität vorhanden:

* **replaceEntity**: aktualisiert eine vorhandene Entität, indem sie ersetzt wird.

* **mergeEntity**: aktualisiert eine vorhandene Entität durch Zusammenführen neuer Eigenschaftswerte mit der vorhandenen Entität.

* **insertOrReplaceEntity**: aktualisiert eine vorhandene Entität, indem sie ersetzt wird. Wenn keine Entität vorhanden ist, wird eine neue eingefügt.

* **insertOrMergeEntity**: aktualisiert eine vorhandene Entität durch Zusammenführen neuer Eigenschaftswerte mit der vorhandenen Entität. Wenn keine Entität vorhanden ist, wird eine neue eingefügt.

Das folgende Beispiel zeigt, wie eine Entität mit **replaceEntity** aktualisiert wird:

	tableSvc.replaceEntity('mytable', updatedTask, function(error, result, response){
	  if(!error) {
	    // Entity updated
	  }
	});

> [AZURE.NOTE] Standardmäßig wird beim Aktualisieren einer Entität nicht überprüft, ob die aktualisierten Daten zuvor von einem anderen Prozess geändert wurden. Um gleichzeitige Aktualisierungen zu unterstützen, gehen Sie wie folgt vor:
>
> 1. Rufen Sie das Etag des aktualisierten Objekts ab. Es wird im Rahmen der `response` für jeden entitätsbezogenen Vorgang zurückgegeben und kann durch `response['.metadata'].etag` abgerufen werden.
>
> 2. Wenn Sie einen Aktualisierungsvorgang für eine Entität ausführen, sollten Sie der neuen Entität die zuvor abgerufenen ETag-Informationen hinzufügen. Beispiel:
>
>     `entity2['.metadata'].etag = currentEtag;`
>
> 3. Führen Sie den Aktualisierungsvorgang aus. Wurde die Entität seit dem Abruf des ETag-Werts beispielsweise durch eine andere Instanz Ihrer Anwendung geändert, wird ein `error` zurückgegeben, der besagt, dass die in der Anforderung angegebene Aktualisierungsbedingung nicht erfüllt ist.

Bei **replaceEntity** und **mergeEntity** tritt beim Aktualisierungsvorgang ein Fehler auf, wenn die zu aktualisierende Entität nicht vorhanden ist. Daher sollten Sie **insertOrReplaceEntity** oder **insertOrMergeEntity** verwenden, wenn Sie eine Entität unabhängig davon speichern möchten, ob sie bereits vorhanden ist.

Für erfolgreiche Aktualisierungsvorgänge enthält das `result` das **ETag** der aktualisierten Entität.

## Arbeiten mit Gruppen von Entitäten

Gelegentlich ist es sinnvoll, mehrere Vorgänge zusammen in einem Batch zu senden, um die atomische Verarbeitung durch den Server sicherzustellen. Dazu erstellen Sie mit der **TableBatch**-Klasse einen Batch und führen dann mit der **executeBatch**-Methode von **TableService** die Batchvorgänge aus.

 Im folgenden Beispiel wird gezeigt, wie zwei Entitäten in einem Stapel übermittelt werden:

	var task1 = {
	  PartitionKey: {'_':'hometasks'},
	  RowKey: {'_': '1'},
	  description: {'_':'Take out the trash'},
	  dueDate: {'_':new Date(2015, 6, 20)}
	};
	var task2 = {
	  PartitionKey: {'_':'hometasks'},
	  RowKey: {'_': '2'},
	  description: {'_':'Wash the dishes'},
	  dueDate: {'_':new Date(2015, 6, 20)}
	};

	var batch = new azure.TableBatch();

	batch.insertEntity(task1, {echoContent: true});
	batch.insertEntity(task2, {echoContent: true});

	tableSvc.executeBatch('mytable', batch, function (error, result, response) {
	  if(!error) {
	    // Batch completed
	  }
	});

Bei erfolgreichen Batchvorgängen enthält `result` Informationen für jeden Vorgang im Batch.

### Arbeiten mit Batchvorgängen

Vorgänge, die zu einem Batch hinzugefügt wurden, können durch Anzeigen der Eigenschaft `operations` überprüft werden. Sie können auch die folgenden Methoden verwenden, um mit Vorgängen zu arbeiten.

* **clear**: löscht alle Vorgänge aus einem Batch.

* **getOperations**: ruft einen Vorgang aus dem Batch ab.

* **hasOperations**: gibt "true" zurück, wenn der Batch Vorgänge enthält.

* **removeOperations**: entfernt einen Vorgang.

* **size**: gibt die Anzahl von Vorgängen im Batch zurück.

## Abrufen einer Entität nach Schlüssel

Wenn Sie eine bestimmte Entität basierend auf **PartitionKey** und **RowKey** zurückgeben möchten, verwenden Sie die **retrieveEntity**-Methode.

	tableSvc.retrieveEntity('mytable', 'hometasks', '1', function(error, result, response){
	  if(!error){
	    // result contains the entity
	  }
	});

Nach Abschluss des Vorgangs enthält `result` die Entität.

## Abfragen einer Gruppe von Entitäten

Um eine Tabelle abzufragen, erstellen Sie mithilfe des **TableQuery-Objekts** einen Abfrageausdruck mit folgenden Klauseln:

* **select**: die zurückzugebenden Felder aus der Abfrage.

* **where**: die where-Klausel.

	* **and**: eine `and` where-Bedingung.

	* **or**: eine `or` where-Bedingung.

* **top**: die Anzahl der abzurufenden Elemente.


Im folgenden Beispiel wird eine Abfrage erstellt, die die ersten fünf Elemente mit dem PartitionKey "hometasks" zurückgibt.

	var query = new azure.TableQuery()
	  .top(5)
	  .where('PartitionKey eq ?', 'hometasks');

Da **select** nicht verwendet wird, werden alle Felder zurückgegeben. Verwenden Sie **queryEntities**, um die Abfrage für eine Tabelle auszuführen. Im folgenden Beispiel werden mit der Abfrage Entitäten aus 'mytable' zurückgegeben.

	tableSvc.queryEntities('mytable',query, null, function(error, result, response) {
	  if(!error) {
	    // query was successful
	  }
	});

Nach erfolgreicher Ausführung enthält `result.entries` ein Array von Entitäten, die die Abfrage erfüllen. Wenn nicht alle Entitäten von der Abfrage zurückgegeben werden konnten, ist `result.continuationToken` ungleich *Null* und kann als dritter Parameter von **queryEntities** verwendet werden, um weitere Ergebnisse abzurufen. Verwenden Sie in der ersten Abfrage *null* als dritten Parameter.

### Abfragen einer Teilmenge von Entitätseigenschaften

Mit einer Abfrage einer Tabelle können nur einige wenige Felder einer Entität aufgerufen werden. Somit wird die Bandbreite reduziert und die Abfrageleistung gesteigert, vor allem bei großen Entitäten. Verwenden Sie die **select**-Klausel, und übergeben Sie die Namen der zurückzugebenden Felder. Die folgende Abfrage gibt beispielsweise nur die Felder **description** und **dueDate** zurück.

	var query = new azure.TableQuery()
	  .select(['description', 'dueDate'])
	  .top(5)
	  .where('PartitionKey eq ?', 'hometasks');

## Löschen einer Entität

Sie können eine Entität unter Verwendung ihres Partitions- und Zeilenschlüssels löschen. In diesem Beispiel enthält das Objekt **task1** die **RowKey**- und **PartitionKey**-Werte der zu löschenden Entität. Dann wird das Objekt der **deleteEntity**-Methode übergeben.

	var task = {
	  PartitionKey: {'_':'hometasks'},
	  RowKey: {'_': '1'}
	};

	tableSvc.deleteEntity('mytable', task, function(error, response){
	  if(!error) {
	    // Entity deleted
	  }
	});

> [AZURE.NOTE] Es ist ratsam, beim Löschen von Elementen ETags zu verwenden, um sicherzustellen, dass das Element nicht von einem anderen Prozess geändert wurde. Informationen zum Verwenden von ETags finden Sie unter [Aktualisieren einer Entität](#update-an-entity).

## Löschen einer Tabelle

Mit dem folgenden Code wird eine Tabelle aus einem Speicherkonto gelöscht.

	tableSvc.deleteTable('mytable', function(error, response){
		if(!error){
			// Table deleted
		}
	});

Wenn Sie nicht wissen, ob die Tabelle vorhanden ist, verwenden Sie **deleteTableIfExists**.

## Verwenden von Fortsetzungstoken

Wenn Sie in Tabellen umfangreiche Ergebnismengen abfragen, sollten Sie nach Fortsetzungstoken suchen. Möglicherweise sind umfangreiche Datenmengen für die Abfrage verfügbar, die Sie nicht erkennen können, wenn ein Fortsetzungstoken vorhanden ist.

Das beim Abfragen von Entitäten zurückgegebene Ergebnisobjekt legt eine `continuationToken`-Eigenschaft fest, wenn ein derartiges Token vorhanden ist. Diese können Sie dann beim Durchführen einer Abfrage verwenden, um die Partitions- und Tabellenentitäten zu durchlaufen.

Bei der Abfrage wird möglicherweise ein continuationToken-Parameter zwischen der query-Objektinstanz und der callback-Funktion bereitgestellt:

```
var nextContinuationToken = null;
dc.table.queryEntities(tableName,
    query,
    nextContinuationToken,
    function (error, results) {
        if (error) throw error;

        // iterate through results.entries with results

        if (results.continuationToken) {
            nextContinuationToken = results.continuationToken;
        }

    });
```

Das `continuationToken`-Objekt enthält Eigenschaften wie z. B. `nextPartitionKey`, `nextRowKey` und `targetLocation`, die zum Durchlaufen der Ergebnisse verwendet werden können.

Ein fortführendes Beispiel dazu finden Sie im Azure Storage-SDK für Node.js auf GitHub. Suchen Sie nach `examples/samples/continuationsample.js`.

## Arbeiten mit Shared Access Signatures

Shared Access Signatures (SAS) ermöglichen auf sichere Art und Weise differenzierten Zugriff auf Tabellen, ohne Speicherkontonamen oder -schlüssel anzugeben. SAS werden häufig verwendet, um eingeschränkten Zugriff auf Ihre Daten zu bieten, beispielsweise um einer mobilen App die Abfrage von Datensätzen zu ermöglichen.

Eine vertrauenswürdige Anwendung, z. B. ein cloudbasierter Dienst, generiert mit der **generateSharedAccessSignature**-Methode des **TableService**-Objekts eine SAS und stellt sie für eine nicht vertrauenswürdige oder teilweise vertrauenswürdige Anwendung, z. B. eine mobile App, bereit. Die SAS wird mithilfe einer Richtlinie generiert, die das Anfangs- und das Enddatum der Gültigkeit der SAS sowie die Zugriffsstufe definiert, die dem Inhaber der SAS gewährt wird.

Im folgenden Beispiel wird eine neue Richtlinie für den freigegebenen Zugriff generiert, die dem SAS-Inhaber erlaubt, die Tabelle abzufragen ('r'), und 100 Minuten nach ihrer Erstellung abläuft.

	var startDate = new Date();
	var expiryDate = new Date(startDate);
	expiryDate.setMinutes(startDate.getMinutes() + 100);
	startDate.setMinutes(startDate.getMinutes() - 100);

	var sharedAccessPolicy = {
	  AccessPolicy: {
	    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
	    Start: startDate,
	    Expiry: expiryDate
	  },
	};

	var tableSAS = tableSvc.generateSharedAccessSignature('mytable', sharedAccessPolicy);
	var host = tableSvc.host;

Beachten Sie, dass die Hostinformationen ebenfalls angegeben werden müssen, da sie erforderlich sind, wenn der SAS-Inhaber versucht auf die Tabelle zuzugreifen.

Die Clientanwendung verwendet die SAS dann zusammen mit **TableServiceWithSAS**, um Vorgänge für die Tabelle auszuführen. Im folgenden Beispiel wird eine Verbindung mit der Tabelle hergestellt und eine Abfrage ausgeführt.

	var sharedTableService = azure.createTableServiceWithSas(host, tableSAS);
	var query = azure.TableQuery()
	  .where('PartitionKey eq ?', 'hometasks');

	sharedTableService.queryEntities(query, null, function(error, result, response) {
	  if(!error) {
	    // result contains the entities
	  }
	});

Da die SAS nur mit Abfragezugriff generiert wurde, wird beim Versuch, Entitäten einzufügen, zu aktualisieren oder zu löschen, ein Fehler zurückgegeben.

### Zugriffssteuerungslisten

Sie können auch eine Zugriffssteuerungsliste (Access Control List, ACL) verwenden, um die Zugriffsrichtlinie für eine SAS festzulegen. Dies ist nützlich, wenn Sie mehreren Clients Zugriff auf die Tabelle gewähren, aber für jeden Client andere Zugriffsrichtlinien angeben möchten.

Eine ACL wird in einem Array von Zugriffsrichtlinien implementiert, wobei jeder Richtlinie eine ID zugeordnet wird. Im folgenden Beispiel werden zwei Richtlinien definiert, eine für "user1" und eine für "user2":

	var sharedAccessPolicy = {
	  user1: {
	    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
	    Start: startDate,
	    Expiry: expiryDate
	  },
	  user2: {
	    Permissions: azure.TableUtilities.SharedAccessPermissions.ADD,
	    Start: startDate,
	    Expiry: expiryDate
	  }
	};

Im folgenden Beispiel wird zunächst die aktuelle ACL für die Tabelle **hometasks** abgerufen. Anschließend werden die neuen Richtlinien mit **setTableAcl** hinzugefügt. Dieser Ansatz ermöglicht Folgendes:

	var extend = require('extend');
	tableSvc.getTableAcl('hometasks', function(error, result, response) {
    if(!error){
	    var newSignedIdentifiers = extend(true, result.signedIdentifiers, sharedAccessPolicy);
	    tableSvc.setTableAcl('hometasks', newSignedIdentifiers, function(error, result, response){
	      if(!error){
	        // ACL set
	      }
	    });
	  }
	});

Nachdem die ACL \(festgelegt wurde, können Sie) basierend auf der ID für eine Richtlinie eine SAS erstellen. Im folgenden Beispiel wird eine neue SAS für 'user2' erstellt:

	tableSAS = tableSvc.generateSharedAccessSignature('hometasks', { Id: 'user2' });

## Nächste Schritte

Weitere Informationen finden Sie in den folgenden Ressourcen.

-   [Azure Storage Team Blog][] \(in englischer Sprache)
-   [Azure Storage SDK for Node.js][]-Repository auf GitHub \(in englischer Sprache)
-   [Node.js Developer Center \(in englischer Sprache)](/develop/nodejs/)

  [Azure Storage SDK for Node.js]: https://github.com/Azure/azure-storage-node
  [OData.org]: http://www.odata.org/
  [Using the REST API]: http://msdn.microsoft.com/library/azure/hh264518.aspx
  [Azure Portal]: portal.azure.com

  [Node.js Cloud Service]: ../cloud-services-nodejs-develop-deploy-app.md
  [Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
  [Website with WebMatrix]: ../web-sites-nodejs-use-webmatrix.md
  [Node.js Cloud Service with Storage]: ../storage-nodejs-use-table-storage-cloud-service-app.md
  [Node.js-Web-App mit Azure-Tabellenspeicherdienst]: ../storage-nodejs-use-table-storage-web-site.md
  [Create and deploy a Node.js application to an Azure website]: ../web-sites-nodejs-develop-deploy-mac.md

<!---HONumber=AcomDC_0817_2016--->