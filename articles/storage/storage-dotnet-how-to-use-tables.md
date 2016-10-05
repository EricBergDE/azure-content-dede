<properties
	pageTitle="Erste Schritte mit Azure Table Storage mit .NET | Microsoft Azure"
	description="Speichern Sie strukturierte Daten mit Azure Table Storage, einem NoSQL-Datenspeicher, in der Cloud."
	services="storage"
	documentationCenter=".net"
	authors="tamram"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="hero-article"
	ms.date="09/20/2016"
	ms.author="gusapost;tamram"/>


# Erste Schritte mit Azure Table Storage mit .NET

[AZURE.INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
<br/>
[AZURE.INCLUDE [storage-try-azure-tools-tables](../../includes/storage-try-azure-tools-tables.md)]

## Übersicht

Azure Table Storage ist ein Dienst, bei dem strukturierte NoSQL-Daten in der Cloud gespeichert werden. Bei Table Storage handelt es sich um einen Schlüssel-/Attributspeicher mit einem schemalosen Design. Aufgrund der Schemalosigkeit von Table Storage ist es einfach, Ihre Daten an die Entwicklung Ihrer Anwendungen anzupassen. Der Datenzugriff ist für alle Arten von Anwendungen schnell und kostengünstig. Table Storage ist in der Regel erheblich günstiger als herkömmliche SQL-Lösungen für ähnliche Datenmengen.

Mit Table Storage können Sie flexible Datasets wie Benutzerdaten für Webanwendungen, Adressbücher, Geräteinformationen und jegliche Art von Metadaten speichern, die Ihr Dienst erfordert. Sie können eine beliebige Anzahl von Entitäten in einer Tabelle speichern, und ein Speicherkonto kann eine beliebige Anzahl von Tabellen enthalten (bis zur Speicherkapazitätsgrenze eines Speicherkontos).

### Informationen zu diesem Lernprogramm

In diesem Tutorial wird gezeigt, wie Sie .NET-Code für einige häufige Szenarien mit Verwendung von Azure Table Storage schreiben, z. B. das Erstellen und Löschen einer Tabelle und das Einfügen, Aktualisieren, Löschen und Abfragen von Tabellendaten.

**Geschätzter Zeitaufwand:** 45 Minuten

**Voraussetzungen:**

- [Microsoft Visual Studio](https://www.visualstudio.com/de-DE/visual-studio-homepage-vs.aspx)
- [Azure Storage-Clientbibliothek für .NET](https://www.nuget.org/packages/WindowsAzure.Storage/)
- [Azure Configuration Manager für .NET](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)
- Ein [Azure Storage-Konto](storage-create-storage-account.md#create-a-storage-account)

[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../../includes/storage-dotnet-client-library-version-include.md)]

### Weitere Beispiele

Weitere Beispiele für die Verwendung von Table Storage finden Sie unter [Getting Started with Azure Table Storage in .NET](https://azure.microsoft.com/documentation/samples/storage-table-dotnet-getting-started/) (Erste Schritte mit Azure Table Storage in .NET). Sie können die Beispielanwendung herunterladen und ausführen oder den Code auf GitHub durchsuchen.


[AZURE.INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-development-environment-include](../../includes/storage-development-environment-include.md)]

### Hinzufügen von Namespace-Deklarationen

Fügen Sie am Anfang der Datei `program.cs` die folgenden `using`-Anweisungen ein:

	using Microsoft.Azure; // Namespace for CloudConfigurationManager
	using Microsoft.WindowsAzure.Storage; // Namespace for CloudStorageAccount
    using Microsoft.WindowsAzure.Storage.Table; // Namespace for Table storage types

### Analysieren der Verbindungszeichenfolge

[AZURE.INCLUDE [storage-cloud-configuration-manager-include](../../includes/storage-cloud-configuration-manager-include.md)]

### Erstellen des Tabellenspeicherdienst-Clients

Mit der **CloudTableClient**-Klasse können Sie im Tabellenspeicher gespeicherte Tabellen und Entitäten abrufen. Hier sehen Sie eine Möglichkeit zum Erstellen des Dienstclients:

	// Create the table client.
	CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

Jetzt können Sie Code schreiben, der Daten aus dem Tabellenspeicher liest und Daten in den Tabellenspeicher schreibt.

## Erstellen einer Tabelle

Dieses Beispiel zeigt, wie Sie eine Tabelle erstellen, falls sie nicht bereits vorhanden ist:

	// Retrieve the storage account from the connection string.
	CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
	    CloudConfigurationManager.GetSetting("StorageConnectionString"));

	// Create the table client.
	CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

	// Retrieve a reference to the table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create the table if it doesn't exist.
    table.CreateIfNotExists();

## Hinzufügen einer Entität zu einer Tabelle

Entitäten werden C#-Objekten zugeordnet, indem eine benutzerdefinierte Klasse verwendet wird, die aus **TableEntity** abgeleitet wurde. Erstellen Sie eine Klasse, mit der die Eigenschaften der Entität definiert werden, um eine Entität zu einer Tabelle hinzuzufügen. Mit dem folgenden Code wird eine Entitätsklasse definiert, die den Vornamen des Kunden als Zeilenschlüssel und den Nachnamen als Partitionsschlüssel verwendet. In Kombination miteinander wird mit dem Partitions- und Zeilenschlüssel eine Entität in der Tabelle eindeutig identifiziert. Entitäten mit demselben Partitionsschlüssel können schneller abgerufen werden als Entitäten mit unterschiedlichen Partitionsschlüsseln, die Verwendung verschiedener Partitionsschlüssel ermöglicht jedoch eine größere Skalierbarkeit paralleler Vorgänge. Bei jeder Eigenschaft, die im Tabellendienst gespeichert werden soll, muss es sich um eine öffentliche Eigenschaft eines unterstützten Typs handeln, die sowohl `get` als auch `set` verfügbar macht. Darüber hinaus *muss* der Entitätstyp einen parameterlosen Konstruktor verfügbar machen.

    public class CustomerEntity : TableEntity
    {
        public CustomerEntity(string lastName, string firstName)
        {
            this.PartitionKey = lastName;
            this.RowKey = firstName;
        }

        public CustomerEntity() { }

        public string Email { get; set; }

        public string PhoneNumber { get; set; }
    }

Tabellenvorgänge, die Entitäten umfassen, werden über das **CloudTable**-Objekt ausgeführt, das Sie im Abschnitt "Erstellen einer Tabelle" erstellt haben. Der durchzuführende Vorgang wird durch ein **TableOperation**-Objekt dargestellt. Im folgenden Codebeispiel wird die Erstellung des **CloudTable**-Objekts und danach eines **CustomerEntity**-Objekts gezeigt. Um den Vorgang vorzubereiten, wird ein **TableOperation**-Objekt erstellt, um die Kundenentität in die Tabelle einzufügen. Schließlich wird der Vorgang durch den Aufruf von **CloudTable.Execute** ausgeführt.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

	// Create the CloudTable object that represents the "people" table.
	CloudTable table = tableClient.GetTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.Email = "Walter@contoso.com";
    customer1.PhoneNumber = "425-555-0101";

    // Create the TableOperation object that inserts the customer entity.
    TableOperation insertOperation = TableOperation.Insert(customer1);

    // Execute the insert operation.
    table.Execute(insertOperation);

## Einfügen eines Entitätsbatchs

Sie können einen Entitätsbatch in einem Schreibvorgang in eine Tabelle einfügen. Beachten Sie im Zusammenhang mit Batchvorgängen Folgendes:

-  Sie können Aktualisierungs-, Lösch- und Einfügevorgänge in einem einzigen Batchvorgang ausführen.
-  Ein einzelner Batchvorgang kann bis zu 100 Entitäten umfassen.
-  Alle Entitäten in einem Batchvorgang müssen über denselben Partitionsschlüssel verfügen.
-  Eine Abfrage kann als Batchvorgang durchgeführt werden, dabei muss es sich jedoch um den einzigen Vorgang im Batch handeln.

<!-- -->
Im folgenden Codebeispiel werden zwei Entitätsobjekte erstellt, und jedes dieser Objekte wird über die **Insert**-Methode zu **TableBatchOperation** hinzugefügt. Danach wird **CloudTable.Execute** aufgerufen, um den Vorgang auszuführen.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

	// Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create the batch operation.
    TableBatchOperation batchOperation = new TableBatchOperation();

    // Create a customer entity and add it to the table.
	CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
	customer1.Email = "Jeff@contoso.com";
	customer1.PhoneNumber = "425-555-0104";

	// Create another customer entity and add it to the table.
	CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
	customer2.Email = "Ben@contoso.com";
	customer2.PhoneNumber = "425-555-0102";

	// Add both customer entities to the batch insert operation.
	batchOperation.Insert(customer1);
	batchOperation.Insert(customer2);

	// Execute the batch operation.
	table.ExecuteBatch(batchOperation);

## Abrufen aller Entitäten einer Partition

Verwenden Sie ein **TableQuery**-Objekt, um eine Tabelle für alle Entitäten in einer Partition abzurufen. Im folgenden Codebeispiel wird ein Filter für Entitäten erstellt, wobei "Smith" der Partitionsschlüssel ist. In diesem Beispiel werden die Felder der einzelnen Entitäten in den Abfrageergebnissen an die Konsole ausgegeben.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Construct the query operation for all customer entities where PartitionKey="Smith".
    TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

    // Print the fields for each customer.
    foreach (CustomerEntity entity in table.ExecuteQuery(query))
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
            entity.Email, entity.PhoneNumber);
    }

## Abrufen eines Entitätsbereichs in einer Partition

Wenn Sie nicht alle Entitäten in einer Partition abrufen möchten, können Sie einen Bereich angeben, indem Sie den Partitionsschlüsselfilter mit einem Zeilenschlüsselfilter kombinieren. Im folgenden Codebeispiel werden zwei Filter eingesetzt, um alle Entitäten in der Partition „Smith“ abzurufen, deren Zeilenschlüssel (Vorname) mit einem Buchstaben vor dem Buchstaben E im Alphabet beginnen. Danach werden die Abfrageergebnisse gedruckt.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

	// Create the table query.
    TableQuery<CustomerEntity> rangeQuery = new TableQuery<CustomerEntity>().Where(
        TableQuery.CombineFilters(
            TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"),
            TableOperators.And,
            TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "E")));

    // Loop through the results, displaying information about the entity.
    foreach (CustomerEntity entity in table.ExecuteQuery(rangeQuery))
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
            entity.Email, entity.PhoneNumber);
    }

## Abrufen einer einzelnen Entität

Sie können eine Abfrage schreiben, um eine einzelne bestimmte Entität abzurufen. Im folgenden Code wird **TableOperation** verwendet, um den Kunden "Ben Smith" anzugeben. Diese Methode gibt anstelle einer Sammlung nur eine Entität zurück, und bei dem zurückgegebenen Wert in **TableResult.Result** handelt es sich um ein **CustomerEntity**-Objekt. Die Angabe beider Schlüssel, Partition und Zeile, in einer Abfrage ist die schnellste Möglichkeit, um eine einzelne Entität aus dem Tabellenspeicherdienst abzurufen.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the retrieve operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Print the phone number of the result.
	if (retrievedResult.Result != null)
	   Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
	else
	   Console.WriteLine("The phone number could not be retrieved.");

## Ersetzen einer Entität

Um eine Entität zu aktualisieren, rufen Sie sie aus dem Tabellendienst ab, ändern Sie das Entitätsobjekt, und speichern Sie die Änderungen dann im Tabellendienst. Mit dem folgenden Code wird die Telefonnummer eines vorhandenen Kunden geändert. Anstelle von **Insert** wird in diesem Code **Replace** aufgerufen. Dadurch wird die Entität auf dem Server vollständig ersetzt, sofern die Entität auf dem Server sich seit dem letzten Abruf nicht geändert hat. In diesem Fall wäre der Vorgang nicht erfolgreich. Auf diese Weise soll verhindert werden, dass eine Änderung, die zwischen dem Abruf und der Aktualisierung durch eine andere Komponente der Anwendung vorgenommen wurde, unabsichtlich überschrieben wird. Die richtige Vorgehensweise in diesem Fall besteht darin, die Entität erneut abzurufen, Ihre Änderungen vorzunehmen (falls diese noch gültig sind) und dann einen anderen **Replace**-Vorgang auszuführen. Im nächsten Abschnitt erfahren Sie, wie Sie dieses Verhalten überschreiben.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity object.
    CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

    if (updateEntity != null)
	{
	   // Change the phone number.
	   updateEntity.PhoneNumber = "425-555-0105";

	   // Create the Replace TableOperation.
	   TableOperation updateOperation = TableOperation.Replace(updateEntity);

	   // Execute the operation.
	   table.Execute(updateOperation);

	   Console.WriteLine("Entity updated.");
	}

	else
	   Console.WriteLine("Entity could not be retrieved.");

## Einfügen-oder-Ersetzen einer Entität

**Ersetzungsvorgänge** sind nicht erfolgreich, wenn die Entität seit dem letzten Abruf vom Server geändert wurde. Darüber hinaus müssen Sie zuerst die Entität vom Server abrufen, damit der **Ersetzungsvorgang** erfolgreich ist. Manchmal ist jedoch nicht bekannt, ob die Entität auf dem Server vorhanden ist, und die darin aktuell gespeicherten Werte sind nicht relevant. In diesem Fall sollten letztere durch die Aktualisierung vollständig überschrieben werden. Dazu würden Sie einen **InsertOrReplace**-Vorgang verwenden. Bei diesem Vorgang wird die Entität eingefügt, falls sie nicht vorhanden ist, oder ersetzt, falls sie vorhanden ist, unabhängig davon, wann die letzte Aktualisierung stattgefunden hat. Im folgenden Codebeispiel wird die Kundenentität für Ben Smith abgerufen, anschließend jedoch über **InsertOrReplace** erneut auf dem Server gespeichert. Alle Änderungen, die zwischen dem Abruf- und dem Aktualisierungsvorgang an der Entität vorgenommen wurden, werden überschrieben.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity object.
    CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

    if (updateEntity != null)
	{
	   // Change the phone number.
	   updateEntity.PhoneNumber = "425-555-1234";

	   // Create the InsertOrReplace TableOperation.
	   TableOperation insertOrReplaceOperation = TableOperation.InsertOrReplace(updateEntity);

	   // Execute the operation.
	   table.Execute(insertOrReplaceOperation);

	   Console.WriteLine("Entity was updated.");
	}

	else
	   Console.WriteLine("Entity could not be retrieved.");

## Abfragen einer Teilmenge von Entitätseigenschaften

Mit einer Tabellenabfrage können nicht nur alle, sondern auch nur einige Eigenschaften aus einer Entität abgerufen werden. Mit dieser Technik, der sogenannten Projektion, wird die Bandbreite reduziert und die Abfrageleistung gesteigert, vor allem bei großen Entitäten. Mit der Abfrage im folgenden Code werden nur die E-Mail-Adressen von Entitäten in der Tabelle zurückgegeben. Dies geschieht durch die Verwendung einer Abfrage von **DynamicTableEntity** und **EntityResolver**. Weitere Informationen zur Projektion finden Sie im Blogbeitrag [Microsoft Azure Tables: Introducing Upsert and Query Projection][]. Beachten Sie, dass Projektion nicht auf dem lokalen Speicheremulator unterstützt wird, weshalb dieser Code nur ausgeführt wird, wenn Sie ein Konto für den Tabellendienst verwenden.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Define the query, and select only the Email property.
    TableQuery<DynamicTableEntity> projectionQuery = new TableQuery<DynamicTableEntity>().Select(new string[] { "Email" });

    // Define an entity resolver to work with the entity after retrieval.
    EntityResolver<string> resolver = (pk, rk, ts, props, etag) => props.ContainsKey("Email") ? props["Email"].StringValue : null;

    foreach (string projectedEmail in table.ExecuteQuery(projectionQuery, resolver, null, null))
    {
        Console.WriteLine(projectedEmail);
    }

## Löschen einer Entität

Sie können eine Entität problemlos nach dem Abrufen löschen. Verwenden Sie dazu dasselbe Muster wie für das Aktualisieren einer Entität. Durch den nachstehenden Code wird eine Kundenentität aufgerufen und gelöscht.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that expects a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity.
    CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

    // Create the Delete TableOperation.
	if (deleteEntity != null)
	{
	   TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

	   // Execute the operation.
	   table.Execute(deleteOperation);

	   Console.WriteLine("Entity deleted.");
	}

	else
	   Console.WriteLine("Could not retrieve the entity.");

## Löschen einer Tabelle

Schließlich wird mit dem folgenden Codebeispiel eine Tabelle aus einem Speicherkonto gelöscht. Eine gelöschte Tabelle kann für eine bestimmte Zeitdauer nach dem Löschvorgang nicht neu erstellt werden.

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Delete the table it if exists.
    table.DeleteIfExists();

## Asynchrones Abrufen von Entitäten in Seiten

Wenn Sie eine große Anzahl von Entitäten lesen und die Entitäten verarbeiten/anzeigen möchten, während sie abgerufen werden, statt zu warten, bis sie alle zurückgeben worden sind, können Sie Entitäten mit einer segmentierten Abfrage abrufen. In diesem Beispiel wird gezeigt, wie Sie mit dem Async-Await-Muster die Ergebnisse seitenweise zurückgeben, sodass die Ausführung nicht blockiert ist, während Sie auf die Rückgabe eines großen Ergebnissatzes warten. Weitere Informationen zur Verwendung des Async-Await-Musters in .NET finden Sie unter [Asynchrone Programmierung mit Async und Await (C# und Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx).

    // Initialize a default TableQuery to retrieve all the entities in the table.
    TableQuery<CustomerEntity> tableQuery = new TableQuery<CustomerEntity>();

    // Initialize the continuation token to null to start from the beginning of the table.
    TableContinuationToken continuationToken = null;

    do
    {
        // Retrieve a segment (up to 1,000 entities).
        TableQuerySegment<CustomerEntity> tableQueryResult =
            await table.ExecuteQuerySegmentedAsync(tableQuery, continuationToken);

        // Assign the new continuation token to tell the service where to
        // continue on the next iteration (or null if it has reached the end).
        continuationToken = tableQueryResult.ContinuationToken;

        // Print the number of rows retrieved.
        Console.WriteLine("Rows retrieved {0}", tableQueryResult.Results.Count);

    // Loop until a null continuation token is received, indicating the end of the table.
    } while(continuationToken != null);

## Nächste Schritte

Nachdem Sie sich nun mit den Grundlagen des Tabellenspeichers vertraut gemacht haben, folgen Sie diesen Links, um zu erfahren, wie komplexere Speicheraufgaben ausgeführt werden.

- Weitere Beispiele für Tabellenspeicher finden Sie unter [Getting Started with Azure Table Storage in .NET](https://azure.microsoft.com/documentation/samples/storage-table-dotnet-getting-started/) (Erste Schritte mit Azure Table Storage in .NET).
- In der Referenzdokumentation für den Tabellenspeicherdienst finden Sie alle Details zu verfügbaren APIs:
    - [Referenz zur Storage-Clientbibliothek für .NET](http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)
    - [REST-API-Referenz](http://msdn.microsoft.com/library/azure/dd179355)
- Erfahren Sie, wie Sie mithilfe des [Azure WebJobs SDK](../app-service-web/websites-dotnet-webjobs-sdk-get-started.md) den geschriebenen Code so vereinfachen, dass er mit Azure Storage funktioniert.
- Weitere Informationen zu zusätzlichen Optionen für das Speichern von Daten in Azure finden Sie in den anderen Featureleitfäden.
    - [Erste Schritte mit Azure Blob Storage mit .NET](storage-dotnet-how-to-use-blobs.md) zum Speichern unstrukturierter Daten
    - [Verwenden von Azure SQL-Datenbank in .NET-Anwendungen](sql-database-dotnet-how-to-use.md) zum Speichern relationaler Daten

  [Download and install the Azure SDK for .NET]: /develop/net/
  [Creating an Azure Project in Visual Studio]: http://msdn.microsoft.com/library/azure/ee405487.aspx

  [Blob5]: ./media/storage-dotnet-how-to-use-table-storage/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-table-storage/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-table-storage/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-table-storage/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-table-storage/blob9.png

  [Microsoft Azure Tables: Introducing Upsert and Query Projection]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx
  [.NET Client Library reference]: http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409
  [Azure Storage Team blog]: http://blogs.msdn.com/b/windowsazurestorage/
  [Configure Azure Storage connection strings]: http://msdn.microsoft.com/library/azure/ee758697.aspx
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
  [How to: Programmatically access Table storage]: #tablestorage

<!---HONumber=AcomDC_0921_2016-->