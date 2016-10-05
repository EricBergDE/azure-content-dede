<properties 
	pageTitle="Arbeiten mit einem mobilen JavaScript-Back-End-Dienst" 
	description="Stellt Beispiele zum Definieren, Registrieren und Verwenden von Serverskripts in Azure Mobile Services bereit." 
	services="mobile-services" 
	documentationCenter="" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="javascript" 
	ms.devlang="multiple" 
	ms.topic="article" 
	ms.date="07/21/2016" 
	ms.author="ricksal"/>


# Arbeiten mit einem mobilen JavaScript-Back-End-Dienst

Dieser Artikel enthält ausführliche Informationen und Beispiele zum Arbeiten mit einem JavaScript-Back-End in Azure Mobile Services.

##<a name="intro"></a>Einführung

In einem mobilen JavaScript-Back-End-Dienst können Sie benutzerdefinierte Geschäftslogik in Form von JavaScript-Code definieren, der auf dem Server gespeichert und ausgeführt wird. Dieser serverseitige Skriptcode ist einer der folgenden Serverfunktionen zugewiesen:

+ [Lese-, Schreib-, Änderungs- oder Löschvorgänge in einer bestimmten Tabelle][Table operations].
+ [Geplante Aufträge][Job Scheduler].
+ [Definieren von HTTP-Methoden in einer benutzerdefinierten API][Custom API anchor].

Die Signatur der main-Funktion im Serverskript hängt davon ab, in welchem Kontext das Skript verwendet wird. Sie können auch CommonScript-Code als nodes.js-Module definieren, die von verschiedenen Skripts genutzt werden. Weitere Informationen finden Sie unter [Quellcodeverwaltung und freigegebener Code][Source control, shared code, and helper functions].

Beschreibungen zu einzelnen Serverskript-Objekten und -Funktionen finden Sie unter [Skriptreferenz für Mobile Services-Server].


##<a name="table-scripts"></a>Tabellenvorgänge

Ein Tabellenvorgangsskript ist ein Serverskript, das für einen Vorgang auf einer Tabelle registriert ist: insert (Einfügen), read (Lesen), update (Aktualisieren) oder delete bzw. *del* (Löschen). In diesem Abschnitt wird beschrieben, wie Sie mit Tabellenvorgänge in einem JavaScript-Back-End arbeiten. Darin sind die folgenden Abschnitte enthalten:

+ [Übersicht über Tabellenvorgänge][Basic table operations]
+ [Gewusst wie: Registrierung für Tabellenvorgänge]
+ [Gewusst wie: Überschreiben der Standardantwort]
+ [Gewusst wie: Überschreiben des Ausführungserfolgs]
+ [Gewusst wie: Überschreiben der Standard-Fehlerbehandlung]
+ [Gewusst wie: Generieren eindeutiger ID-Werte](#generate-guids)
+ [Gewusst wie: Hinzufügen benutzerdefinierter Parameter]
+ [Gewusst wie: Arbeiten mit Tabellenbenutzern][How to: Work with users]

###<a name="basic-table-ops"></a>Übersicht über Tabellenvorgänge

Der Name des Skripts muss mit dem Vorgang übereinstimmen, für die es registriert ist. Pro Tabellenvorgang kann nur ein Skript registriert werden. Das Skript wird jedes mal ausgeführt, wenn der entsprechende Vorgang durch eine REST-Anforderung aufgerufen wird, z. B. beim Empfang einer POST-Anforderung zum Einfügen eines Elements in die Tabelle. Mobile Services speichert den Status zwischen einzelnen Skriptausführungen nicht. Da bei jeder Skriptausführung ein neuer globaler Kontext erstellt wird, werden sämtliche im Skript definierten Statusvariablen neu initialisiert. Falls Sie den Status zwischen einzelnen Anforderungen speichern möchten, erstellen Sie eine Tabelle in Ihrem Mobile Service und speichern Sie den Status in dieser Tabelle. Weitere Informationen finden Sie unter [Gewusst wie: Tabellenzugriff in Skripts].

Skripts für Tabellenvorgänge dienen zur Umsetzung von benutzerdefinierter Geschäftslogik bei der Ausführung des Vorgangs. Das folgende Skript verhindert z. B. Einfügevorgänge, wenn die Zeichenfolge im Feld `text` länger als zehn Zeichen ist:

	function insert(item, user, request) {
	    if (item.text.length > 10) {
	        request.respond(statusCodes.BAD_REQUEST, 
				'Text length must be less than 10 characters');
	    } else {
	        request.execute();
	    }
	}

Tabellen-Skriptfunktionen nehmen immer drei Argumente entgegen.

- Das erste Argument variiert je nach Tabellenvorgang.

	- Für Insert- und Update-Vorgänge ist dies ein **Item**-Objekt mit einer JSON-Abbildung der vom entsprechenden Vorgang betroffenen Zeile. Auf diese Weise können Sie die Spaltenwerte mit Namen ansprechen, z. B. *item.Owner*, wobei *Owner* einer der Namen in der JSON-Darstellung ist.
	- Für Delete-Vorgänge ist dies die ID des zu löschenden Eintrags.
	- Für Select-Vorgänge ist das erste Argument ein [Suchabfrageobjekt], welches das zurückzuliefernde Rowset angibt.

- Das zweite Argument ist immer ein [Benutzerobjekt][User object] und repräsentiert den Benutzer, der die Anforderung übermittelt hat.

- Das dritte Argument ist immer ein [Anforderungsobjekt][request object], mit dem Sie den angeforderten Vorgang und die an den Client geschickte Antwort steuern können.

Dies sind die kanonischen Signaturen der main-Funktion für die Tabellenvorgänge:

+ [Einfügen][insert function]\: `function insert (item, user, request) { ... }`
+ [Aktualisieren][update function]\: `function update (item, user, request) { ... }`
+ [Löschen][delete function]\: `function del (id, user, request) { ... }`
+ [Lesen][read function]\: `function read (query, user, request) { ... }`

>[AZURE.NOTE]Funktionen, die für den Delete-Vorgang registriert sind, müssen _del_ genannt werden, da delete ein reserviertes Schlüsselwort in JavaScript ist.

Jedes Serverskript hat eine main-Funktion und weitere optionale Hilfsfunktionen. Auch wenn ein Serverskript für eine bestimmte Tabelle erstellt wurde, kann das Skript auf andere Tabellen in derselben Datenbank verweisen. Sie können auch allgemeine Funktionen als Module definieren, die von verschiedenen Skripts genutzt werden. Weitere Informationen finden Sie unter [Quellcodeverwaltung und freigegebener Code][Source control, shared code, and helper functions].

###<a name="register-table-scripts"></a>Gewusst wie: Registrieren von Tabellenskripts

Sie können Serverskripts, die für einen Tabellenvorgang registriert sind, auf folgende Arten definieren:

+ Im [klassischen Azure-Portal]. Sie finden die Skripts für Tabellenvorgänge in der Registerkarte **Skripts** für eine bestimmte Tabelle. Hier sehen Sie den Standardcode, der für das Insert-Skript der Tabelle `TodoItem` registriert ist. Sie können diesen Code mit Ihrer eigenen benutzerdefinierten Geschäftslogik überschreiben.

	![1][1]
	
	Weitere Informationen hierzu finden Sie unter [Validate and modify data in Mobile Services by using server scripts] \(Validierung und Änderung von Daten in Mobile Services mithilfe von Serverskripts, in englischer Sprache).

+ Mithilfe von Quellcodeverwaltung. Falls Sie die Quellcodeverwaltung aktiviert haben, erstellen Sie einfach eine Datei mit dem Namen "<em>`<table>`</em>.<em>`<operation>`</em>.js" im Unterordner ".\\service\\table" in Ihrem Git-Repository, wobei <em>`<table>`</em> der Name der Tabelle und <em>`<operation>`</em> der registrierte Tabellenvorgang ist. Weitere Informationen finden Sie unter [Quellcodeverwaltung und freigegebener Code][Source control, shared code, and helper functions].

+ Über die Eingabeaufforderung im Azure-Befehlszeilentool. Weitere Informationen finden Sie unter [Arbeiten mit dem Befehlszeilentool].


Tabellenvorgänge müssen mindestens eine der folgenden Funktionen des [Anforderungsobjekt] aufrufen, um sicherzustellen, dass der Client eine Antwort erhält.
 
+ **execute-Funktion**: Der Vorgang wird wie angefordert ausgeführt und die Standardantwort zurückgeliefert.
 
+ **respond-Funktion**: Eine benutzerdefinierte Antwort wird zurückgegeben.

> [AZURE.IMPORTANT] Falls ein Skript einen Codepfad enthält, in dem weder **execute** noch **respond** aufgerufen wird, kann es passieren, dass der Vorgang nicht reagiert.

Das folgende Skript ruft die **execute**-Funktion auf, um den vom Client angeforderten Datenvorgang abzuschließen:

	function insert(item, user, request) { 
	    request.execute(); 
	}

In diesem Beispiel wird das Element in die Datenbank eingefügt und der entsprechende Statuscode an den Benutzer zurückgeliefert.

Beim Aufruf der **execute**-Funktion wird der Wert für `item`, [query][query object] oder `id`, der als erstes Argument an die Skriptfunktion übergeben wurde, zum Ausführen des Vorgangs verwendet. Für Insert-, Update- oder Query-Vorgänge können Sie Element oder Abfrage modifizieren, bevor Sie **execute** aufrufen:

	function insert(item, user, request) { 
	    item.scriptComment =
			'this was added by a script and will be saved to the database'; 
	    request.execute(); 
	} 
 
	function update(item, user, request) { 
	    item.scriptComment = 
			'this was added by a script and will be saved to the database'; 
	    request.execute(); 
	} 

	function read(query, user, request) { 
		// Only return records for the current user 	    
		query.where({ userid: user.userId}); 
	    request.execute(); 
	}
 
>[AZURE.NOTE]In Delete-Skripts haben Änderungen am Wert der übergebenen userId-Variable keinen Einfluss darauf, welcher Datensatz gelöscht wird.

Weitere Beispiele finden Sie unter [Lesen und Schreiben von Daten], [Ändern der Anforderung] und [Überprüfen von Daten].


###<a name="override-response"></a>Gewusst wie: Überschreiben der Standardantwort

Sie können mithilfe von Skripts auch eine Prüflogik implementieren und das Standard-Antwortverhalten überschreiben. Falls die Überprüfung fehlschlägt, rufen Sie einfach die **respond**-Funktion anstelle der **execute**-Funktion auf und schreiben die Antwort an den Client:

	function insert(item, user, request) {
	    if (item.userId !== user.userId) {
	        request.respond(statusCodes.FORBIDDEN, 
	        'You may only insert records with your userId.');
	    } else {
	        request.execute();
	    }
	}

In diesem Beispiel wird die Anforderung abgelehnt, wenn das eingefügte Element keine `userId`-Eigenschaft hat, die mit der `userId` des [Benutzerobjekts] übereinstimmt, das für den authentifizierten Client angegeben wurde. In diesem Fall wird der Datenbankvorgang (*insert*) nicht ausgeführt, und der Client erhält eine Antwort mit HTTP-Statuscode 403 und einer benutzerdefinierten Fehlermeldung. Weitere Beispiele finden Sie unter [Ändern der Antwort].

###<a name="override-success"></a>Gewusst wie: Überschreiben des Ausführungserfolgs

Die **execute**-Funktion in Tabellenvorgängen schreibt die Antworten standardmäßig automatisch. Sie können jedoch zwei optionale Parameter an die execute-Funktion übergeben, die dieses Verhalten im Erfolgs- und/oder Fehlerfall außer Kraft setzen.

Wenn Sie beim Aufruf von execute einen **success**-Handler übergeben, können Sie die Ergebnisse einer Anforderung modifizieren, bevor Sie diese in die Antwort schreiben. Im folgenden Beispiel wird `execute({ success: function(results) { ... })` aufgerufen, um zusätzliche Vorgänge auszuführen, nachdem Daten aus der Datenbank gelesen wurden, jedoch bevor die Antwort geschrieben wird:

	function read(query, user, request) {
	    request.execute({
	        success: function(results) {
	            results.forEach(function(r) {
	                r.scriptComment = 
	                'this was added by a script after querying the database';
	            });
	            request.respond();
	        }
	    });
	}

Wenn Sie der **execute**-Funktion einen **success**-Handler übergeben, müssen Sie außerdem die **respond**-Funktion als Teil des **success**-Handlers aufrufen, um der Runtime mitzuteilen, dass das Skript abgeschlossen wurde und eine Antwort geschrieben werden kann. Wenn Sie **respond** ohne Argumente aufrufen, generiert Mobile Services die Standardantwort.

>[AZURE.NOTE]Sie können **respond** erst dann ohne Argumente aufrufen, um die Standardantwort zu erhalten, nachdem Sie die **execute**-Funktion aufgerufen haben.
 
###<a name="override-error"></a>Gewusst wie: Überschreiben der Standard-Fehlerbehandlung

Die **execute**-Funktion kann fehlschlagen, wenn die Datenbankverbindung abbricht oder ein ungültiges Objekt bzw. eine ungültige Abfrage übergeben wird. Standardmäßig werden Fehler vom Serverskript in das Protokoll geschrieben, und der Client erhält eine Antwort mit einer Fehlermeldung. Dank der Standard-Fehlerbehandlung von Mobile Services müssen Sie eventuelle Fehler in Ihrem Dienst nicht behandeln.

Sie können die Standard-Fehlerbehandlung durch eine eigene Fehlerbehandlung überschreiben, falls Sie bestimmte Maßnahmen ergreifen oder über das globale Konsolenobjekt zusätzliche Detailinformationen in das Protokoll schreiben möchten. Übergeben Sie hierzu einen **error**-Handler an die **execute**-Funktion:

	function update(item, user, request) { 
	  request.execute({ 
	    error: function(err) { 
	      // Do some custom logging, then call respond. 
	      request.respond(); 
	    } 
	  }); 
	}
 

Wenn Sie einen error-Handler übergeben, liefert Mobile Services ein Fehlerergebnis an den Client zurück, wenn **respond** aufgerufen wird.

Sie können bei Bedarf auch einen **success**- und einen **error**-Handler übergeben.

###<a name="generate-guids"></a>Gewusst wie: Generieren eindeutiger ID-Werte

Mobile Services unterstützt eindeutige benutzerdefinierte Zeichenfolgenwerte für die Spalte **id** der Tabelle. Auf diese Weise können Anwendungen benutzerdefinierte Werte wie E-Mail-Adressen oder Benutzernamen für die ID verwenden.

Zeichenfolgen-IDs bieten Ihnen die folgenden Vorteile:

+ IDs werden ohne Roundtrip zur Datenbank generiert.
+ Datensätze aus unterschiedlichen Tabellen oder Datenbanken lassen sich leichter zusammenführen.
+ ID-Werte lassen sich möglicherweise leichter in die Anwendungslogik integrieren.

Wenn der ID-Wert einer Zeichenfolge für eingefügte Datensätze nicht festgelegt ist, generiert Mobile Services einen eindeutigen Wert für die ID. Sie können eigene eindeutige ID-Werte in Serverskripts generieren. Das folgende Skript generiert eine benutzerdefinierte GUID und weist diese der ID eines neuen Eintrags zu. Dieser Wert ähnelt dem ID-Wert, den Mobile Services generieren würde, wenn Sie für die ID eines Datensatzes keinen Wert übergeben.

	// Example of generating an id. This is not required since Mobile Services
	// will generate an id if one is not passed in.
	item.id = item.id || newGuid();
	request.execute();

	function newGuid() {
		var pad4 = function(str) { return "0000".substring(str.length) + str; };
		var hex4 = function () { return pad4(Math.floor(Math.random() * 0x10000 /* 65536 */ ).toString(16)); };
		return (hex4() + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + hex4() + hex4());
	}


Wenn eine Anwendung einen Wert für eine ID übergibt, speichert Mobile Services diesen Wert unverändert. Dies beinhaltet auch vor- und nachstehende Leerzeichen. Leerzeichen werden nicht aus dem Wert gekürzt.

Der `id`-Wert muss eindeutig sein und darf keine Zeichen aus den folgenden Sätzen enthalten:

+ Steuerzeichen: [0x0000-0x001F] und [0x007F-0x009F]. Weitere Informationen finden Sie unter [ASCII-Steuerzeichen C0 und C1](http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set).
+  Druckbare Zeichen: **"**(0 x 0022), **+** (0x002B), **/** (0x002F), **?** (0x003F), **\\** (0x005C), **`** (0x0060)
+  Die IDs "." und ".."

Alternativ können Sie auch ganzzahlige IDs für Ihre Tabellen verwenden. Um ganzzahlige IDs zu verwenden, müssen Sie bei der Tabellenerstellung für den `mobile table create`-Befehl die Option `--integerId` verwenden. Dieser Befehl wird in der Befehlszeilenschnittstelle (CLI) für Azure verwendet. Weitere Informationen zur CLI finden Sie unter [CLI to manage Mobile Services tables](../virtual-machines-command-line-tools.md#Mobile_Tables) (CLI für Tabellen in mobilen Diensten, in englischer Sprache).


###<a name="access-headers"></a>Gewusst wie: Zugreifen auf benutzerdefinierte Parameter

Wenn Sie eine Anforderung an Ihren Mobile Service schicken, können Sie die Anforderungs-URI um benutzerdefinierte Parameter erweitern, um Ihrem Tabellenvorgangs-Skript mitzuteilen, wie eine bestimmte Anforderung behandelt werden soll. Anschließend ändern Sie Ihr Skript, sodass dieses den Parameter ausliest und den Verarbeitungspfad ermittelt.

Die folgende URI für eine POST-Anforderung weist den Service z. B. an, keine Einfügung von *TodoItem* s mit demselben Textwert zuzulassen:

		https://todolist.azure-mobile.net/tables/TodoItem?duplicateText=false

Diese benutzerdefinierten Abfrageparameter können als JSON-Werte der **parameters**-Eigenschaft des [Anforderungsobjekts] abgerufen werden. Das **Anforderungsobjekt** wird von Mobile Services für alle Funktionen bereitgestellt, die für einen Tabellenvorgang registriert sind. Das folgende Serverskript für den insert-Vorgang prüft den Wert des Parameters `duplicateText`, bevor der insert-Vorgang ausgeführt wird:

		function insert(item, user, request) {
		    var todoItemTable = tables.getTable('TodoItem');
		    // Check the supplied custom parameter to see if
		    // we should allow duplicate text items to be inserted.		   
		    if (request.parameters.duplicateText === 'false') {
		        // Find all existing items with the same text
		        // and that are not marked 'complete'. 
		        todoItemTable.where({
		            text: item.text,
		            complete: false
		        }).read({
		            success: insertItemIfNotComplete
		        });
		    } else {
		        request.execute();
		    }

		    function insertItemIfNotComplete(existingItems) {
		        if (existingItems.length > 0) {
		            request.respond(statusCodes.CONFLICT, 
                        "Duplicate items are not allowed.");
		        } else {
		            // Insert the item as normal. 
		            request.execute();
		        }
		    }
		}

Beachten Sie, dass in **insertItemIfNotComplete** die **execute**-Funktion des [Anforderungsobjekts] aufgerufen wird, um das Element einzufügen, falls kein duplizierter Text vorhanden ist. Andernfalls wird die **respond**-Funktion aufgerufen, um den Client über den duplizierten Text zu informieren.

Beachten Sie die Syntax des Aufrufs an die **success**-Funktion im obigen Code:

 		        }).read({
		            success: insertItemIfNotComplete
		        });

In JavaScript ist dies eine kompakte Version des folgenden Codes:

		success: function(results) 
		{ 
			insertItemIfNotComplete(results); 
		}


###<a name="work-with-users"></a>Gewusst wie: Arbeiten mit Benutzern

In Azure Mobile Services können Sie Benutzer mithilfe von Identitätsanbietern authentifizieren. Weitere Informationen finden Sie unter [Erste Schritte mit der Authentifizierung]. Wenn ein authentifizierter Benutzer einen Tabellenvorgang aufruft, verwendet Mobile Services das [Benutzerobjekt], um Informationen über den Benutzer an die registrierte Skriptfunktion zu liefern. Die **userId**-Eigenschaft kann zum Speichern und Abrufen benutzerspezifischer Informationen verwendet werden. Im folgenden Beispiel wird die owner-Eigenschaft eines Elements anhand der **userId** eines authentifizierten Benutzers gesetzt:

	function insert(item, user, request) {
	    item.owner = user.userId;
	    request.execute();
	}

Im nächsten Beispiel wird ein zusätzlicher Filter für die Abfrage anhand der **userId** eines authentifizierten Benutzers hinzugefügt: Dieser Filter schränkt das Ergebnis auf Elemente ein, die dem aktuellen Benutzer gehören:

	function read(query, user, request) {
	    query.where({
	        owner: user.userId
	    });
	    request.execute();
	}

##<a name="custom-api"></a> Benutzerdefinierte APIs

In diesem Abschnitt wird beschrieben, wie Sie benutzerdefinierte API-Endpunkte erstellen und mit ihnen arbeiten. Darin sind die folgenden Abschnitte enthalten:
	
+ [Übersicht über benutzerdefinierte APIs](#custom-api-overview)
+ [Gewusst wie: Definieren einer benutzerdefinierten API]
+ [Gewusst wie: Implementieren von HTTP-Methoden]
+ [Gewusst wie: Senden und Empfangen von Daten als XML]
+ [Gewusst wie: Arbeiten mit Benutzern und Headern in benutzerdefinierten APIs]
+ [Gewusst wie: Definieren mehrerer Routen in einer benutzerdefinierten API]

###<a name="custom-api-overview"></a>Übersicht über benutzerdefinierte APIs

Benutzerdefinierte APIs sind Endpunkte in Ihrem Mobile Service, die von einer oder mehreren der Standard-HTTP-Methoden aufgerufen werden: GET, POST, PUT, PATCH, DELETE. Ein separater Funktionsexport kann für jede HTTP-Methode definiert werden, die von der benutzerdefinierten API unterstützt wird, alles zusammen in einer Skriptdatei. Das registrierte Skript wird aufgerufen, wenn eine Anforderung an die benutzerdefinierte API über die entsprechende Methode empfangen wird. Weitere Informationen finden Sie unter [Benutzerdefinierte API].

Bei Aufrufen von Funktionen einer benutzerdefinierten API durch die Mobile Services-Runtime werden sowohl [Anforderungs-][request object] als auch [Antwort-][response object]Objekt mitgeliefert. Diese Objekte machen die Funktionen der [express.js-Bibliothek] verfügbar, die wiederum von Ihren Skripts verwendet werden können. Die folgende benutzerdefinierte API namens **hello** ist ein sehr einfaches Beispiel und gibt als Antwort auf eine POST-Anforderung _Hello, world!_ zurück:

		exports.post = function(request, response) {
		    response.send(200, "{ message: 'Hello, world!' }");
		} 

Die **send**-Funktion des [Antwortobjekts] gibt die gewünschte Antwort an den Client zurück. Sie können diesen Code aufrufen, indem Sie eine POST-Anforderung an die folgende URL schicken:

		https://todolist.azure-mobile.net/api/hello  

Der globale Status bleibt zwischen Ausführungen erhalten.

###<a name="define-custom-api"></a>Gewusst wie: Definieren einer benutzerdefinierten API

Sie können Serverskripts, die für HTTP-Methoden in einem Endpunkt einer benutzerdefinierten API registriert sind, auf folgende Arten definieren:

+ Im [klassischen Azure-Portal]. Skripts für benutzerdefinierte APIs werden in der Registerkarte **API** erstellt und bearbeitet. Der Serverskript-Code befindet sich in der Registerkarte **Skripts** der jeweiligen benutzerdefinierten API. Im Folgenden wird das Skript gezeigt, das durch eine POST-Anforderung an den Endpunkt der benutzerdefinierten `CompleteAll`-API aufgerufen wird.

	![2][2]
	
	Zugriffsberechtigungen für Methoden von benutzerdefinierten APIs werden in der Registerkarte Berechtigungen zugewiesen. Informationen zur Erstellung der API finden Sie unter [Call a custom API from the client] \(Aufrufen benutzerdefinierter APIs vom Client, in englischer Sprache.

+ Mithilfe von Quellcodeverwaltung. Falls Sie die Quellcodeverwaltung aktiviert haben, erstellen Sie einfach eine Datei mit dem Namen "<em>`<custom_api>`</em>.js" im Unterordner ".\\service\\api" in Ihrem Git-Repository, wobei <em>`<custom_api>`</em> der Name der benutzerdefinierten API ist, die registriert wird. Die Skriptdatei enthält eine _exported_-Funktion für jede HTTP-Methode, die von der benutzerdefinierten API verfügbar gemacht wird. Die Berechtigungen werden in einer separaten .json-Datei definiert. Weitere Informationen finden Sie unter [Quellcodeverwaltung und freigegebener Code][Source control, shared code, and helper functions].

+ Über die Eingabeaufforderung im Azure-Befehlszeilentool. Weitere Informationen finden Sie unter [Arbeiten mit dem Befehlszeilentool].

###<a name="handle-methods"></a>Gewusst wie: Implementieren von HTTP-Methoden

Benutzerdefinierte APIs können eine oder mehrere der HTTP-Methoden GET, POST, PUT, PATCH und DELETE verarbeiten. Für jede von der benutzerdefinierten API verarbeitete HTTP-Methode wird eine exported-Funktion definiert. Eine einzelne Codedatei einer benutzerdefinierten API kann eine oder alle der folgenden Funktionen exportieren:

		exports.get = function(request, response) { ... };
		exports.post = function(request, response) { ... };
		exports.patch = function(request, response) { ... };
		exports.put = function(request, response) { ... };
		exports.delete = function(request, response) { ... };

Der Endpunkt der benutzerdefinierten API kann nur mit den HTTP-Methoden aufgerufen werden, die im Serverskript implementiert sind, ansonsten wird eine Fehlerantwort 405 (Methode nicht erlaubt) zurückgegeben. Für die einzelnen HTTP-Methoden können separate Berechtigungsstufen vergeben werden.

###<a name="api-return-xml"></a>Gewusst wie: Senden und Empfangen von Daten als XML

Wenn Clients Daten speichern und abrufen, verwendet Mobile Services JavaScript Object Notation (JSON) für die Darstellung der Daten im Nachrichtentext. In manchen Szenarien macht es jedoch mehr Sinn, stattdessen eine XML-Nutzlast zu verwenden. Windows Store-Apps haben z. B. eine eingebaute Funktion für periodische Benachrichtigungen, die nur mit XML-Diensten funktioniert. Weitere Informationen finden Sie unter [Define a custom API that supports periodic notifications] \(Definieren benutzerdefinierter APIs mit Unterstützung für periodische Benachrichtigungen, in englischer Sprache).

Die folgende **OrderPizza**-Funktion einer benutzerdefinierten API liefert ein einfaches XML-Dokument als Nutzlast der Antwort zurück.

		exports.get = function(request, response) {
		  response.set('content-type', 'application/xml');
		  var xml = '<?xml version="1.0"?><PizzaOrderForm><PizzaOrderForm/>';
		  response.send(200, xml);
		};

Sie können diese Funktion aufrufen, indem Sie eine HTTP GET-Anforderung an den folgenden Endpunkt schicken:

		https://todolist.azure-mobile.net/api/orderpizza

###<a name="get-api-user"></a>Gewusst wie: Arbeiten mit Benutzern und Headern in benutzerdefinierten APIs

In Azure Mobile Services können Sie Benutzer mithilfe von Identitätsanbietern authentifizieren. Weitere Informationen finden Sie unter [Get started with authentication] \(Erste Schritte zur Authentifizierung, in englischer Sprache). Wenn ein authentifizierter Benutzer eine benutzerdefinierte API aufruft, verwendet Mobile Services das [Benutzerobjekt], um Informationen über den Benutzer an den Code der benutzerdefinierten API zu liefern. Das [Benutzerobjekt] ist über die user-Eigenschaft des [Anforderungsobjekts] zugänglich. Die **userId**-Eigenschaft kann zum Speichern und Abrufen benutzerspezifischer Informationen verwendet werden.

In der folgenden **OrderPizza**-Funktion einer benutzerdefinierten API wird die owner-Eigenschaft eines Elements anhand der **userId** eines authentifizierten Benutzers gesetzt:

		exports.post = function(request, response) {
			var userTable = request.service.tables.getTable('user');
			userTable.lookup(request.user.userId, {
				success: function(userRecord) {
					callPizzaAPI(userRecord, request.body, function(orderResult) {
						response.send(201, orderResult);
					});
				}
			});
		
		};

Sie können diese Funktion aufrufen, indem Sie eine HTTP POST-Anforderung an den folgenden Endpunkt schicken:

		https://<service>.azure-mobile.net/api/orderpizza

Sie können auch auf bestimmte HTTP-Header des [Anforderungsobjekts] zugreifen, wie der folgende Code zeigt:

		exports.get = function(request, response) {    
    		var header = request.header('my-custom-header');
    		response.send(200, "You sent: " + header);
		};

In diesem einfachen Beispiel wird der benutzerdefinierte Header `my-custom-header` gelesen und dann der Wert in der Antwort zurückgegeben:

###<a name="api-routes"></a>Gewusst wie: Definieren mehrerer Routen in einer benutzerdefinierten API

Mit Mobile Services können Sie mehrere Pfade oder Routen in benutzerdefinierten APIs definieren. HTTP GET-Anforderungen an die folgenden URLs in einer benutzerdefinierten **Taschenrechner**-API werden z. B. die **add-** bzw. die **subtract-**Funktion aufrufen:

+ `https://<service>.azure-mobile.net/api/calculator/add`
+ `https://<service>.azure-mobile.net/api/calculator/sub`

Mehrere Routen werden durch den Export einer **register**-Funktion definiert, der ein **api**-Objekt übergeben wird (ähnlich dem [express-Objekt in express.js]), das wiederum zum Registrieren von Routen innerhalb des Endpunkts der benutzerdefinierten API verwendet wird. Das folgende Beispiel implementiert die **add-** und **sub-**-Methoden in der benutzerdefinierten **Taschenrechner**-API:

		exports.register = function (api) {
		    api.get('add', add);
		    api.get('sub', subtract);
		}
		
		function add(req, res) {
		    var result = parseInt(req.query.a) + parseInt(req.query.b);
		    res.send(200, { result: result });
		}
		
		function subtract(req, res) {
		    var result = parseInt(req.query.a) - parseInt(req.query.b);
		    res.send(200, { result: result });
		}

Das an die **register**-Funktion übergebene **api**-Objekt macht eine Funktion für jede HTTP-Methode verfügbar (**get**, **post**, **put**, **patch**, **delete**). Diese Funktionen registrieren eine Route zu einer bestimmten Funktion für die entsprechende HTTP-Methode. Jede Funktion nimmt zwei Parameter entgegen, den Namen der Route und die Funktion, die für die Route registriert ist.

Die beiden Routen im obigen Beispiel für eine benutzerdefinierte API können wie folgt durch HTTP GET-Anforderungen aufgerufen werden (jeweils mit der Antwort):

+ `https://<service>.azure-mobile.net/api/calculator/add?a=1&b=2`

		{"result":3}

+ `https://<service>.azure-mobile.net/api/calculator/sub?a=3&b=5`

		{"result":-2}

##<a name="scheduler-scripts"></a>Auftragsplaner

Mit Mobile Services können Sie Serverskripts definieren, die entweder als Aufträge nach festem Zeitplan oder bedarfsgesteuert über das klassische Azure-Portal ausgeführt werden können. Planmäßige Aufträge sind hilfreich für periodische Aufgaben wie z. B: Aufräumvorgänge in Tabellendaten und Batchverarbeitungen. Weitere Informationen finden Sie unter [Geplante Aufträge].

Skripts, die für geplante Aufträge registriert sind, haben eine Hauptfunktion mit demselben Namen wie der geplante Auftrag. Da geplante Aufträge nicht über HTTP-Anforderungen aufgerufen werden, existiert kein Kontext, der von der Server-Runtime übergeben werden kann, und die Funktion nimmt keine Parameter entgegen. Wie auch andere Skriptarten können Sie Unterfunktionen definieren und freigegebene Module einbinden. Weitere Informationen finden Sie unter [Quellcodeverwaltung, freigegebener Code und Hilfsfunktionen].

###<a name="scheduler-scripts"></a>Gewusst wie: Definieren geplanter Auftragsskripts

Serverskripts können Aufträgen zugewiesen werden, die im Mobile Service-Scheduler definiert sind. Diese Skripts gehören zum jeweiligen Auftrag und werden nach Auftragsplaner ausgeführt. (Sie können Aufträge auch über das [klassische Azure-Portal] bei Bedarf ausführen.) Skripts, die geplante Aufträge definieren, haben keine Parameter, da Mobile Services keine Daten an sie übergeben. Sie werden stattdessen als normale JavaScript-Funktion aufgerufen und interagieren nicht direkt mit Mobile Services.

Geplante Aufträge können auf zwei Arten definiert werden:

+ Im [klassischen Azure-Portal] auf der Registerkarte **Skript** im Scheduler:

	![3][3]

	Weitere Informationen hierzu finden Sie unter [Schedule backend jobs in Mobile Services] \(Planen von Back-End-Aufgaben in Mobile Services, in englischer Sprache).

+ Über die Eingabeaufforderung im Azure-Befehlszeilentool. Weitere Informationen finden Sie unter [Arbeiten mit dem Befehlszeilentool].

>[AZURE.NOTE]Falls Sie die Quellcodeverwaltung aktiviert haben, können Sie die Skriptdateien der geplanten Aufträge direkt im Unterordner .\\service\\scheduler in Ihrem Git-Repository bearbeiten. Weitere Informationen finden Sie unter [Gewusst wie: Freigeben von Code mithilfe der Quellcodeverwaltung].

##<a name="shared-code"></a>Quellcodeverwaltung, freigegebener Code und Hilfsfunktionen

In diesem Abschnitt wird gezeigt, wie Sie die Quellcodeverwaltung nutzen, um eigene benutzerdefinierte "node.js"-Module, Wiederverwendungsstrategien für freigegebenen und sonstigen Code hinzufügen, einschließlich der folgenden Abschnitte:

+ [Übersicht über die Nutzung von freigegebenen Code](#leverage-source-control)
+ [Gewusst wie: Laden von Node.js-Modulen]
+ [Gewusst wie: Arbeiten mit Hilfsfunktionen]
+ [Gewusst wie: Freigeben von Code mithilfe der Quellcodeverwaltung]
+ [Gewusst wie: Arbeiten mit App-Einstellungen]

###<a name="leverage-source-control"></a>Übersicht über die Nutzung von freigegebenen Code

Da Mobile Services auf dem Server Node.js verwendet, haben Ihre Skripts bereits Zugriff auf die eingebauten Node.js-Module. Sie können in Ihrer Quellcodeverwaltung eigene Module definieren oder andere Node.js-Module zu Ihrem Dienst hinzufügen.

Es folgt eine Liste hilfreicher Module, die Sie über die globale **require**-Funktion in Ihre Skripts einbinden können:

+ **azure**: Macht die Funktionen des Azure SDK für Node.js verfügbar. Weitere Informationen finden Sie unter [Azure SDK für Node.js].
+ **crypto**: Macht die Verschlüsselungsfunktionen von Open SSL verfügbar. Weitere Informationen finden Sie in der [Node.js-Dokumentation][crypto API].
+ **path**: Enthält Werkzeuge zum Arbeiten mit Dateipfaden. Weitere Informationen finden Sie in der [Node.js-Dokumentation][path API].
+ **querystring**: Enthält Werkzeuge zum Arbeiten mit Abfragezeichenfolgen. Weitere Informationen finden Sie in der [Node.js-Dokumentation][querystring API].
+ **request**: Verschickt HTTP-Anforderungen an externe REST-Dienste wie z. B. Twitter und Facebook. Weitere Informationen finden Sie unter [HTTP-Anforderungen senden].
+ **sendgrid**: Verschickt E-Mails über den Sendgrid-E-Mail-Dienst in Azure. Weitere Informationen finden Sie unter [Send email from Mobile Services with SendGrid] \(E-Mail-Versand in Mobile Services mit SendGrid, in englischer Sprache).
+ **url**: Enthält Werkzeuge zum Analysieren und Auflösen von URLs. Weitere Informationen finden Sie in der [Node.js-Dokumentation][url API].
+ **util**: Enthält verschiedene Werkzeuge wie z. B. Zeichenfolgenformatierung und Objekttypprüfungen. Weitere Informationen finden Sie in der [Node.js-Dokumentation][util API].
+ **zlib**: Macht Komprimierungsmethoden wie z. B. "gzip" und "deflate" verfügbar. Weitere Informationen finden Sie in der [Node.js-Dokumentation][zlib API].

###<a name="modules-helper-functions"></a>Gewusst wie: Einbinden von Modulen

Mobile Services bietet eine Reihe von Modulen an, die mithilfe der globalen **require**-Funktion in Skripts eingebunden werden können. Ein Skript kann z. B. **request** einbinden, um HTTP-Anforderungen zu schicken:

	function update(item, user, request) { 
	    var httpRequest = require('request'); 
	    httpRequest('http://www.google.com', function(err, response, body) { 
	    	... 
	    }); 
	} 


###<a name="shared-code-source-control"></a>Gewusst wie: Freigeben von Code mithilfe der Quellcodeverwaltung

Sie können über Ihre Quellcodeverwaltung und den Node.js-Paket-Manager (npm) steuern, welche Module für Ihren Mobile Service zur Verfügung stehen. Dies kann auf zwei Arten erreicht werden:

+ Für die von npm bereitgestellten und installierten Module verwenden Sie die Datei package.json, um anzugeben, welche Pakete von Ihrem Mobile Service installiert werden sollen. Auf diese Weise hat Ihr Dienst immer Zugriff auf die neueste Version der installierten Pakete. Die Datei "package.json" befindet sich im Verzeichnis `.\service`. Weitere Informationen finden Sie unter [Support for package.json in Azure Mobile Services] \(package.json-Unterstützung in Azure Mobile Services, in englischer Sprache).

+ Private oder benutzerdefinierte Module können Sie mithilfe von npm manuell im Verzeichnis `.\service\node_modules` der Quellcodeverwaltung installieren. Ein Beispiel zum manuellen Hochladen von Modulen finden Sie unter [Leverage shared code and Node.js modules in your server scripts] \(Einbinden von freigegebenem Code und Node.js-Modulen in Ihren Serverskripts, in englischer Sprache).

	>[AZURE.NOTE]Wenn `node_modules` in der Verzeichnishierarchie bereits vorhanden ist, wird mit npm das Unterverzeichnis `\node-uuid` in diesem Verzeichnis erstellt, anstatt ein neues Verzeichnis `node_modules` im Repository zu erstellen. Löschen Sie in diesem Fall das vorhandene `node_modules`-Verzeichnis.

Nachdem Sie die Daten package.json oder Ihre benutzerdefinierten Module im Repository für Ihren Mobile Service übernommen haben, können Sie die Module über deren Namen mit **require** einbinden.

>[AZURE.NOTE] Module, die Sie in package.json angeben oder auf Ihren Mobile Service hochladen, werden ausschließlich in Ihrem Serverskript-Code verwendet. Diese Module werden nicht von der Mobile Services-Runtime verwendet.

###<a name="helper-functions"></a>Gewusst wie: Arbeiten mit Hilfsfunktionen

Zusätzlich zum Einbinden von Modulen mit dem Befehl require können Serverskripts auch Hilfsfunktionen einbinden. Diese Funktionen sind separat von der main-Funktion und können zur Faktorisierung von Code im Skript verwendet werden.

Im folgenden Beispiel wird ein Tabellenskript für den insert-Vorgang registriert, das die Hilfsfunktion **handleUnapprovedItem** per include einbindet:


	function insert(item, user, request) {
	    if (!item.approved) {
	        handleUnapprovedItem(item, user, request);
	    } else {
	        request.execute();
	    }
	}
	
	function handleUnapprovedItem(item, user, request) {
	    // Do something with the supplied item, user, or request objects.
	}
 
Hilfsfunktionen müssen in Skripts nach der main-Funktion definiert werden. Sie müssen alle Variablen in Ihrem Skript deklarieren. Nicht deklarierte Variablen verursachen einen Fehler.

Hilfsfunktionen können an einer Stelle definiert und von mehreren Serverskripts verwendet werden. Um eine Funktion zwischen Skripts freizugeben, müssen Funktionen exportiert werden, und die Skriptdatei muss im Verzeichnis `.\service\shared` enthalten sein. Es folgt ein Beispiel für den Export einer freigegebenen Funktion in eine Datei `.\services\shared\helpers.js`:

		exports.handleUnapprovedItem = function (tables, user, callback) {
		    
		    // Do something with the supplied tables or user objects and 
			// return a value to the callback function.
		};
 
Anschließend können Sie Funktion in einem Tabellenvorgangsskript verwenden:

		function insert(item, user, request) {
		    var helper = require('../shared/helper');
		    helper.handleUnapprovedItem(tables, user, function(result) {
		        	
					// Do something based on the result.
		            request.execute();
		        }
		    }
		}

In diesem Beispiel müssen Sie sowohl ein [Tabellenobjekt] als auch ein [Benutzerobjekt] an die freigegebene Funktion übergeben. Dies liegt daran, dass freigegebene Skripts keinen Zugriff auf das globale [Tabellenobjekt] haben, und das [Benutzerobjekt] nur im Kontext einer Anforderung existiert.

Skriptdateien können entweder mithilfe der [Quellcodeverwaltung][How to: Share code by using source control] oder über das [Befehlszeilentool][Using the command line tool] in das freigegebene Verzeichnis kopiert werden.

###<a name="app-settings"></a>Gewusst wie: Arbeiten mit App-Einstellungen

Mit Mobile Services können Sie Werte wie z. B. App-Einstellungen sicher speichern und Ihren Serverskripts zur Laufzeit zur Verfügung stellen. Wenn Sie Daten zu den App-Einstellungen Ihres Mobile Service hinzufügen, werden die Name-Wert-Paare verschlüsselt gespeichert und Sie können in Ihren Serverskripts darauf zugreifen, ohne die Daten hart in der Skriptdatei codieren zu müssen. Weitere Informationen finden Sie unter [App-Einstellungen].

Die folgende benutzerdefinierte API verwendet das übergebene [Serviceobjekt], um den Wert einer App-Einstellung abzurufen.

		exports.get = function(request, response) {
		
			// Get the MY_CUSTOM_SETTING value from app settings.
		    var customSetting = 
		        request.service.config.appSettings.my_custom_setting;
				
			// Do something and then send a response.

		}

Im folgenden Code werden Werte eines Twitter-Zugriffstokens über das Konfigurationsmodul abgerufen, in den App-Einstellungen gespeichert und dort wiederum im Skript eines geplanten Auftrags verwendet.

		// Get the service configuration module.
		var config = require('mobileservice-config');

		// Get the stored Twitter consumer key and secret. 
		var consumerKey = config.twitterConsumerKey,
		    consumerSecret = config.twitterConsumerSecret
		// Get the Twitter access token from app settings.    
		var accessToken= config.appSettings.TWITTER_ACCESS_TOKEN,
		    accessTokenSecret = config.appSettings.TWITTER_ACCESS_TOKEN_SECRET;

Beachten Sie, dass dieser Code auch die Consumer Key-Werte von Twitter abruft und in der Registerkarte **Identität** im Portal speichert. Da das **config-Objekt** in Skripts für Tabellenvorgänge und geplante Aufträge nicht verfügbar ist, muss das Konfigurationsmodul stattdessen auf die App-Einstellungen zugreifen. Ein vollständiges Beispiel finden Sie unter [Schedule backend jobs in Mobile Services] \(Planen von Back-End-Aufgaben in Mobile Services, in englischer Sprache).

<h2><a name="command-prompt"></a>Arbeiten mit dem Befehlszeilentool</h2>

In Mobile Services können Sie Serverskripts mithilfe des Azure-Befehlszeilentools erstellen, bearbeiten und löschen. Stellen Sie vor dem Hochladen der Skripts sicher, dass Sie die folgende Verzeichnisstruktur verwenden:

![4][4]

Diese Verzeichnisstruktur entspricht dem Git-Repository, falls Sie Quellcodeverwaltung verwenden.

Beim Hochladen von Skriptdateien im Befehlszeilentool müssen Sie zuerst zum Verzeichnis `.\services` navigieren. Der folgende Befehl lädt ein Skript namens `todoitem.insert.js` aus dem Unterverzeichnis `table` hoch:

		~$azure mobile script upload todolist table/todoitem.insert.js
		info:    Executing command mobile script upload
		info:    mobile script upload command OK

Der folgende Befehl gibt Informationen über sämtliche Skriptdateien in Ihrem Mobile Service zurück:

		~$ azure mobile script list todolist
		info:    Executing command mobile script list
		+ Retrieving script information
		info:    Table scripts
		data:    Name                       Size
		data:    -------------------------  ----
		data:    table/channels.insert      1980
		data:    table/TodoItem.insert      5504
		data:    table/TodoItem.read        64
		info:    Shared scripts
		data:    Name              Size
		data:    ----------------  ----
		data:    shared/helper.js  62
		data:    shared/uuid.js    7452
		info:    Scheduled job scripts
		data:    Job name    Script name           Status    Interval     Last run  Next run
		data:    ----------  --------------------  --------  -----------  --------  --------
		data:    getUpdates  scheduler/getUpdates  disabled  15 [minute]  N/A       N/A
		info:    Custom API scripts
		data:    Name                    Get          Put          Post         Patch        Delete
		data:    ----------------------  -----------  -----------  -----------  -----------  -----------
		data:    completeall             application  application  application  application  application
		data:    register_notifications  application  application  user         application  application
		info:    mobile script list command OK

Weitere Informationen finden Sie unter [Commands to manage Azure Mobile Services] \(Befehle zum Verwalten von Azure Mobile Services, in englischer Sprache).

##<a name="working-with-tables"></a>Arbeiten mit Tabellen

In diesem Abschnitt werden Strategien für die direkte Arbeit mit Tabellendaten einer SQL-Datenbank ausführlich beschrieben, einschließlich der folgenden Abschnitte:

+ [Übersicht über das Arbeiten mit Tabellen](#overview-tables)
+ [Gewusst wie: Tabellenzugriff in Skripts]
+ [Gewusst wie: Ausführen von Masseneinfügungen]
+ [Gewusst wie: Zuordnen von JSON-Typen zu Datenbanktypen]
+ [Gewusst wie: Tabellenzugriff mit Transact-SQL]

###<a name="overview-tables"></a>Übersicht über das Arbeiten mit Tabellen

Serverskripts in Mobile Services benötigen häufig Zugriff auf Tabellen in der Datenbank. Zum Beispiel weil Mobile Services den Status zwischen Skriptausführungen nicht speichert und alle Daten, die zwischen Skriptausführungen erhalten bleiben sollen, in Tabellen gespeichert werden müssen. Es kann auch passieren, dass Sie Einträge in einer Berechtigungstabelle auslesen oder Überwachungsdaten anstelle von normalen Protokolleinträgen speichern müssen, die nur eine begrenzte Lebensdauer haben und nicht programmgesteuert abrufbar sind.

Mobile Services bietet zwei Methoden für den Zugriff auf Tabellen, entweder über das [Tabellenobjekt] als Proxy oder über Transact-SQL-Abfragen an das [mssql-Objekt]. Das [Tabellenobjekt] erleichtert den Zugriff auf Tabellendaten aus Ihrem Serverskriptcode heraus. Das [mssql-Objekt] unterstützt komplexere Datenvorgänge und bietet die größte Flexibilität.

###<a name="access-tables"></a>Gewusst wie: Tabellenzugriff in Skripts

Der einfachste Weg, um in Ihrem Skript auf Tabellen zuzugreifen, ist das [Tabellenobjekt]. Die **getTable**-Funktion gibt eine [Tabellenobjekt]-Instanz zurück, die als Proxy für den Zugriff auf die angeforderte Tabelle dient. Sie können anschließend Funktionen des Proxy aufrufen, um Daten auszulesen oder zu ändern.

Skripts, die sowohl für Tabellenvorgänge als auch für geplante Aufträge registriert sind, können auf das [Tabellenobjekt] als globales Objekt zugreifen. Die folgende Codezeile ruft einen Proxy für die Tabelle *TodoItems* aus dem globalen [Tabellenobjekt] ab:

		var todoItemsTable = tables.getTable('TodoItems');

Skripts in benutzerdefinierten APIs können über die <strong>service</strong>-Eigenschaft des übergebenen [Anforderungsobjekts] auf das [Tabellenobjekt] zugreifen. Die folgende Codezeile ruft das [Tabellenobjekt] aus der Anforderung ab:

		var todoItemsTable = request.service.tables.getTable('TodoItem');

> [AZURE.NOTE] Freigegebene Funktionen können nicht direkt auf das **Tabellenobjekt** zugreifen. In freigegebenen Funktionen müssen Sie das Tabellenobjekt an die Funktion übergeben.

Sobald Sie ein [Tabellenobjekt] haben, können Sie die folgenden Funktionen für Tabellenvorgänge aufrufen: insert, update, delete oder read. Das folgende Beispiel ruft Berechtigungen eines Benutzers aus einer Berechtigungstabelle ab:

	function insert(item, user, request) {
		var permissionsTable = tables.getTable('permissions');
	
		permissionsTable
			.where({ userId: user.userId, permission: 'submit order'})
			.read({ success: checkPermissions });
			
		function checkPermissions(results) {
			if(results.length > 0) {
				// Permission record was found. Continue normal execution.
				request.execute();
			} else {
				console.log('User %s attempted to submit an order without permissions.', user.userId);
				request.respond(statusCodes.FORBIDDEN, 'You do not have permission to submit orders.');
			}
		}
	}

Das nächste Beispiel schreibt Überwachungsinformationen in eine **Überwachungstabelle**:

	function update(item, user, request) {
		request.execute({ success: insertAuditEntry });
		
		function insertAuditEntry() {
			var auditTable = tables.getTable('audit');
			var audit = {
				record: 'checkins',
				recordId: item.id,
				timestamp: new Date(),
				values: JSON.stringify(item)
			};
			auditTable.insert(audit, {
				success: function() {
					// Write to the response now that all data operations are complete
					request.respond();
				}
			});
		}
	}

Das letzte Beispiel ist das Codebeispiel hier: [Gewusst wie: Zugreifen auf benutzerdefinierte Parameter][How to: Add custom parameters].

###<a name="bulk-inserts"></a>Gewusst wie: Ausführen von Masseneinfügungen

Wenn Sie mithilfe einer **for-** oder **while-**Schleife große Mengen (z. B. 1000) von Datensätzen in eine Tabelle einfügen, können SQL-Verbindungslimits dazu führen, dass nicht alle Datensätze eingefügt werden. Möglicherweise wird die Anforderung nie beendet, oder sie liefert einen HTTP 500 Interner Serverfehler zurück. Um dieses Problem zu vermeiden, können Sie die Datensätze stapelweise einfügen (z. B. 10 auf einmal). Nach Abschluss des ersten Stapels übermitteln Sie den zweiten Stapel, und so weiter.

Mit dem folgenden Skript können Sie die Stapelgröße für das parallele Einfügen von Datensätzen festlegen. Sie sollten diese Anzahl jedoch möglichst klein halten. Die Funktion **insertItems** ruft sich selbst rekursiv auf, wenn ein asynchroner insert-Stapel abgeschlossen wurde. Die for-Schleife am Ende fügt die Datensätze nacheinander ein und ruft bei Erfolg **insertComplete** und im Fehlerfall **errorHandler** auf. **insertComplete** steuert, ob **insertItems** rekursiv für den nächsten Batch aufgerufen wird oder ob der Auftrag abgeschlossen ist und das Skript beendet werden muss.

		var todoTable = tables.getTable('TodoItem');
		var recordsToInsert = 1000;
		var batchSize = 10; 
		var totalCount = 0;
		var errorCount = 0; 
		
		function insertItems() {        
		    var batchCompletedCount = 0;  
		
		    var insertComplete = function() { 
		        batchCompletedCount++; 
		        totalCount++; 
		        if(batchCompletedCount === batchSize || totalCount === recordsToInsert) {                        
		            if(totalCount < recordsToInsert) {
		                // kick off the next batch 
		                insertItems(); 
		            } else { 
		                // or we are done, report the status of the job 
		                // to the log and don't do any more processing 
		                console.log("Insert complete. %d Records processed. There were %d errors.", totalCount, errorCount); 
		            } 
		        } 
		    }; 
		
		    var errorHandler = function(err) { 
		        errorCount++; 
		        console.warn("Ignoring insert failure as part of batch.", err); 
		        insertComplete(); 
		    };
		
		    for(var i = 0; i < batchSize; i++) { 
		        var item = { text: "This is item number: " + totalCount + i }; 
		        todoTable.insert(item, { 
		            success: insertComplete, 
		            error: errorHandler 
		        }); 
		    } 
		} 
		
		insertItems(); 


Das gesamte Codebeispiel und eine begleitende Besprechung finden Sie in diesem [Blogeintrag](http://blogs.msdn.com/b/jpsanders/archive/2013/03/20/server-script-to-insert-table-items-in-windows-azure-mobile-services.aspx). Wenn Sie diesen Code verwenden, können Sie ihn an Ihre speziellen Anforderungen anpassen und gründlich testen.

###<a name="JSON-types"></a>Gewusst wie: Zuordnen von JSON-Typen zu Datenbanktypen

Client- und Mobile Service-Datenbank verwenden unterschiedliche Datentypen. Manche dieser Datentypen lassen sich einfach zuordnen, andere dagegen weniger einfach. Mobile Services führt bei der Zuordnung eine Reihe von Typumwandlungen durch:

- Die spezifischen Typen der Clientsprache werden im JSON-Format serialisiert.
- Die JSON-Darstellung wird nach JavaScript übersetzt, bevor sie in den Serverskripts auftaucht.
- Die JavaScript-Daten werden beim Speichern mit dem [Tabellenobjekt] in SQL-Datenbanktypen transformiert.

Die Transformation vom Clientschema nach JSON unterscheidet sich je nach Plattform. JSON.NET wird im Windows Store und auf Windows Phone-Clients verwendet. Der Android-Client verwendet die gson-Bibliothek. Der iOS-Client verwendet die NSJSONSerialization-Klasse. Diese Bibliotheken verwenden jeweils ihr Standard-Serialisierungsverhalten, allerdings werden Datumsobjekte in JSON-Zeichenfolgen umgewandelt, in denen das Datum nach ISO 8601 codiert ist.

Beim Schreiben von Serverskripts, die [insert-], [update-], [read-] oder [delete-]-Funktionen verwenden, haben Sie Zugriff auf die JavaScript-Darstellung Ihrer Daten. Mobile Services verwendet die Deserialisierungsfunktion von Node.js ([JSON.parse](http://es5.github.io/#x15.12)), um eingehendes JSON in JavaScript-Objekte zu transformieren. Mobile Services führt jedoch eine Transformation durch, um **Date**-Objekte aus ISO 8601-Zeichenfolgen zu extrahieren.

Wenn Sie das [Tabellenobjekt] oder das [mssql-Objekt] verwenden oder Ihre Skripts ausführen, werden die deserialisierten JavaScript-Objekte in Ihre SQL-Datenbank eingefügt. Bei diesem Vorgang werden die Objekteigenschaften auf T-SQL-Datentypen abgebildet:

JavaScript-Eigenschaft|T-SQL-Datentyp
---|---
Number|Float(53)
Boolean|Bit
Date|DateTimeOffset(3)|
String|Nvarchar(max)
Buffer|Nicht unterstützt
Object|Nicht unterstützt
Array|Nicht unterstützt
Stream|Nicht unterstützt

###<a name="TSQL"></a>Tabellenzugriff mit Transact-SQL

Der einfachste Weg, um in einem Serverskript mit Tabellendaten zu arbeiten, ist das [Tabellenobjekt] als Proxy. Manche erweiterte Szenarien wie z. B. join-Abfragen und andere komplexe Abfragen sowie das Aufrufen von gespeicherten Prozeduren werden jedoch vom [Tabellenobjekt] nicht unterstützt. In diesen Fällen müssen Sie Transact-SQL-Anweisungen über das [mssql-Objekt] direkt auf den relationalen Tabellen ausführen. Dieses Objekt bietet die folgenden Funktionen:

- **query**: führt eine Abfrage in Form einer TSQL-Zeichenfolge aus. Die Ergebnisse werden an den **success**-Rückruf des **options**-Objekts zurückgegeben. Die Abfrage kann Parameter enthalten, falls der *params*-Parameter vorhanden ist.
- **queryRaw**: wie *query*, allerdings wird das Resultset der Abfrage im "raw"-Format zurückgegeben (siehe Beispiel unten).
- **open**: öffnet eine Verbindung zu Ihrer Mobile Services-Datenbank. Anschließend können Sie über das Verbindungsobjekt Datenbankvorgänge wie z. B. Transaktionen ausführen.

Mit diesen Methoden erhalten Sie eine immer tiefgehendere Kontrolle über die Verarbeitung der Abfrage.

+ [Gewusst wie: Ausführen statischer Abfragen]
+ [Gewusst wie: Ausführen dynamischer Abfragen]
+ [Gewusst wie: Verknüpfen relationaler Tabellen]
+ [Gewusst wie: Ausführen einer Abfrage, die Ergebnisse im *raw*-Format zurückgibt]
+ [Gewusst wie: Zugreifen auf eine Datenbankverbindung]

####<a name="static-query"></a>Gewusst wie: Ausführen statischer Abfragen

Die folgende Abfrage hat keine Parameter und gibt drei Datensätze aus der Tabelle `statusupdate` zurück. Das Rowset ist im JSON-Standardformat.

		mssql.query('select top 3 * from statusupdates', {
		    success: function(results) {
		        console.log(results);
		    },
            error: function(err) {
                console.log("error is: " + err);
			}
		});


####<a name="dynamic-query"></a>Gewusst wie: Ausführen einer parametrisierten dynamischen Abfrage

Das folgende Beispiel implementiert eine benutzerdefinierte Autorisierung, indem die Berechtigungen der einzelnen Benutzer aus der Tabelle permissions gelesen werden. Der Platzhalter (?) wird beim Ausführen der Abfrage durch den übergebenen Parameter ersetzt.

		    var sql = "SELECT _id FROM permissions WHERE userId = ? AND permission = 'submit order'";
		    mssql.query(sql, [user.userId], {
		        success: function(results) {
		            if (results.length > 0) {
		                // Permission record was found. Continue normal execution. 
		                request.execute();
		            } else {
		                console.log('User %s attempted to submit an order without permissions.', user.userId);
		                request.respond(statusCodes.FORBIDDEN, 'You do not have permission to submit orders.');
		            }
		        },
            	error: function(err) {
                	console.log("error is: " + err);
				}	
		    });


####<a name="joins"></a>Gewusst wie: Verknüpfen relationaler Tabellen

Sie können zwei Tabellen mithilfe der **query**-Methode des [mssql-Objekts] verknüpfen, indem Sie den TSQL-Code übergeben, der die Verknüpfungsanweisung enthält. Nehmen wir an, unsere Tabelle **ToDoItem** enthält einige Datensätze und jeder Datensatz hat eine Eigenschaft **priority**, die einer Spalte in der Tabelle entspricht. Ein Element kann folgendermaßen aussehen:

		{ text: 'Take out the trash', complete: false, priority: 1}

Nehmen wir außerdem an, wir haben eine zweite Tabelle namens **Priority**, deren Spalten eine **Prioritätsnummer** und eine **Textbeschreibung enthalten**. Die Prioritätsnummer 1 hat z. B. die Beschreibung "Kritisch", und das Objekt sieht folgendermaßen aus:

		{ number: 1, description: 'Critical'}

Wir können nun die **Prioritätsnummer** in unserem Element durch die Textbeschreibung der Prioritätsnummer ersetzen. Wir erreichen dies mit einer relationalen Verknüpfung der beiden Tabellen.

		mssql.query('SELECT t.text, t.complete, p.description FROM ToDoItem as t INNER JOIN Priority as p ON t.priority = p.number', {
			success: function(results) {
				console.log(results);
			},
            error: function(err) {
                console.log("error is: " + err);
		});
	
Das Skript verknüpft die beiden Tabellen und schreibt die Ergebnisse in das Protokoll. Die resultierenden Objekte könnten folgendermaßen aussehen:

		{ text: 'Take out the trash', complete: false, description: 'Critical'}


####<a name="raw"></a>Gewusst wie: Ausführen einer Abfrage, die Ergebnisse im *raw*-Format zurückgibt

Dieses Beispiel führt die Abfrage wie zuvor aus, gibt jedoch das Resultset im "raw"-Format zurück, das Sie Zeile für Zeile und Spalte für Spalte durchlaufen müssen. Dies macht z. B. Sinn, wenn Sie Zugriff auf Datentypen benötigen, die von Mobile Services nicht unterstützt werden. Dieser Code schreibt die Ausgabe einfach in das Konsolenprotokoll, sodass Sie das "raw"-Format inspizieren können.

		mssql.queryRaw('SELECT * FROM ToDoItem', {
		    success: function(results) {
		        console.log(results);
		    },
            error: function(err) {
                console.log("error is: " + err);
			}
		});

Unten finden Sie die Ausgabe dieser Abfrage. Sie enthält Metadaten der einzelnen Tabellenspalten, gefolgt von einer Darstellung der Zeilen und Spalten.

		{ meta: 
		   [ { name: 'id',
		       size: 19,
		       nullable: false,
		       type: 'number',
		       sqlType: 'bigint identity' },
		     { name: 'text',
		       size: 0,
		       nullable: true,
		       type: 'text',
		       sqlType: 'nvarchar' },
		     { name: 'complete',
		       size: 1,
		       nullable: true,
		       type: 'boolean',
		       sqlType: 'bit' },
		     { name: 'priority',
		       size: 53,
		       nullable: true,
		       type: 'number',
		       sqlType: 'float' } ],
		  rows: 
		   [ [ 1, 'good idea for the future', null, 3 ],
		     [ 2, 'this is important but not so much', null, 2 ],
		     [ 3, 'fix this bug now', null, 0 ],
		     [ 4, 'we need to fix this one real soon now', null, 1 ],
		   ] }

####<a name="connection"></a>Gewusst wie: Zugreifen auf eine Datenbankverbindung

Mit der **open**-Methode erhalten Sie Zugriff auf die Datenbankverbindung. Dies ist z. B. nützlich, wenn Sie Datenbanktransaktionen verwenden möchten.

Bei einem erfolgreichen Aufruf von **open** wird die Datenbankverbindung als Parameter an die **success**-Funktion übergeben. Sie können die folgenden Funktionen des **connection**-Objekts aufrufen: *close*, *queryRaw*, *query*, *beginTransaction*, *commit* und *rollback*.

		    mssql.open({
		        success: function(connection) {
		            connection.query(//query to execute);
		        },
	            error: function(err) {
	                console.log("error is: " + err);
				}
		    });

##<a name="debugging"></a>Debuggen und Problembehandlung

Debugging und Problembehandlung Ihrer Serverskript erledigen Sie am Besten, indem Sie in das Dienst-Protokoll schreiben. Mobile Services schreibt standardmäßig alle Fehler, die bei der Ausführung von Dienstskripten auftreten, in die Dienst-Protokolle. Ihre Skripts können ebenfalls in die Protokolle schreiben. Dies ist ein guter Weg, um Ihre Skripts zu debuggen und deren Verhalten zu überprüfen.

###<a name="write-to-logs"></a>Gewusst wie: Schreiben von Ausgaben in die Mobile Services-Protokolle

Verwenden Sie das globale [Konsolenobjekt], um in die Protokolle zu schreiben. Mit den Funktionen **log** oder **info** können Sie Warnungen auf der Informationsebene schreiben. Die Funktionen **warning** und **error** schreiben Einträge auf ihrer jeweiligen Ebene, die in den Protokollen hervorgehoben werden.

> [AZURE.NOTE] Um die Protokolle für Ihren Mobile Service anzuzeigen, melden Sie sich beim [klassischen Azure-Portal](https://manage.windowsazure.com/) an, wählen Sie Ihren Mobile Service aus, und wählen Sie die Registerkarte **Protokolle** aus.

Sie können außerdem die Protokollfunktionen des [Konsolenobjekts] verwenden, um Ihre Nachrichten mithilfe von Parametern zu formatieren. Im folgenden Beispiel wird ein JSON-Objekt als Parameter an die Nachrichten-Zeichenfolge übergeben:

	function insert(item, user, request) {
	    console.log("Inserting item '%j' for user '%j'.", item, user);  
	    request.execute();
	}

Beachten Sie, dass die Zeichenfolge `%j` als Platzhalter für ein JSON-Objekt verwendet wird und dass die Parameter in sequenzieller Reihenfolge angegeben werden.

Um Ihr Protokoll nicht zu überladen, sollten Sie Aufrufe von console.log() entfernen oder auskommentieren, die Sie im Produktionsalltag nicht benötigen.

<!-- Anchors. -->
[Introduction]: #intro
[Table operations]: #table-scripts
[Basic table operations]: #basic-table-ops
[Gewusst wie: Registrierung für Tabellenvorgänge]: #register-table-scripts
[How to: Define table scripts]: #execute-operation
[Gewusst wie: Überschreiben der Standardantwort]: #override-response
[How to: Modify an operation]: #modify-operation
[How to: Override success and error]: #override-success-error
[Gewusst wie: Überschreiben des Ausführungserfolgs]: #override-success
[Gewusst wie: Überschreiben der Standard-Fehlerbehandlung]: #override-error
[Gewusst wie: Tabellenzugriff in Skripts]: #access-tables
[How to: Add custom parameters]: #access-headers
[Gewusst wie: Hinzufügen benutzerdefinierter Parameter]: #access-headers
[How to: Work with users]: #work-with-users
[How to: Define scheduled job scripts]: #scheduler-scripts
[How to: Refine access to tables]: #authorize-tables
[Gewusst wie: Tabellenzugriff mit Transact-SQL]: #TSQL
[Gewusst wie: Ausführen statischer Abfragen]: #static-query
[Gewusst wie: Ausführen dynamischer Abfragen]: #dynamic-query
[Gewusst wie: Ausführen einer Abfrage, die Ergebnisse im *raw*-Format zurückgibt]: #raw
[Gewusst wie: Zugreifen auf eine Datenbankverbindung]: #connection
[Gewusst wie: Verknüpfen relationaler Tabellen]: #joins
[Gewusst wie: Ausführen von Masseneinfügungen]: #bulk-inserts
[Gewusst wie: Zuordnen von JSON-Typen zu Datenbanktypen]: #JSON-types
[Gewusst wie: Laden von Node.js-Modulen]: #modules-helper-functions
[How to: Write output to the mobile service logs]: #write-to-logs
[Source control, shared code, and helper functions]: #shared-code
[Quellcodeverwaltung, freigegebener Code und Hilfsfunktionen]: #shared-code
[Using the command line tool]: #command-prompt
[Arbeiten mit dem Befehlszeilentool]: #command-prompt
[Working with tables]: #working-with-tables
[Custom API anchor]: #custom-api
[Gewusst wie: Definieren einer benutzerdefinierten API]: #define-custom-api
[How to: Share code by using source control]: #shared-code-source-control
[Gewusst wie: Freigeben von Code mithilfe der Quellcodeverwaltung]: #shared-code-source-control
[Gewusst wie: Arbeiten mit Hilfsfunktionen]: #helper-functions
[Debugging and troubleshooting]: #debugging
[Gewusst wie: Implementieren von HTTP-Methoden]: #handle-methods
[Gewusst wie: Arbeiten mit Benutzern und Headern in benutzerdefinierten APIs]: #get-api-user
[How to: Access custom API request headers]: #get-api-headers
[Job Scheduler]: #scheduler-scripts
[Gewusst wie: Definieren mehrerer Routen in einer benutzerdefinierten API]: #api-routes
[Gewusst wie: Senden und Empfangen von Daten als XML]: #api-return-xml
[Gewusst wie: Arbeiten mit App-Einstellungen]: #app-settings

[1]: ./media/mobile-services-how-to-use-server-scripts/1-mobile-insert-script-users.png
[2]: ./media/mobile-services-how-to-use-server-scripts/2-mobile-custom-api-script.png
[3]: ./media/mobile-services-how-to-use-server-scripts/3-mobile-schedule-job-script.png
[4]: ./media/mobile-services-how-to-use-server-scripts/4-mobile-source-local-cli.png

<!-- URLs. -->
[Skriptreferenz für Mobile Services-Server]: http://msdn.microsoft.com/library/windowsazure/jj554226.aspx
[Schedule backend jobs in Mobile Services]: /develop/mobile/tutorials/schedule-backend-tasks/
[request object]: http://msdn.microsoft.com/library/windowsazure/jj554218.aspx
[Anforderungsobjekt]: http://msdn.microsoft.com/library/windowsazure/jj554218.aspx
[Anforderungsobjekts]: http://msdn.microsoft.com/library/windowsazure/jj554218.aspx
[response object]: http://msdn.microsoft.com/library/windowsazure/dn303373.aspx
[Antwortobjekts]: http://msdn.microsoft.com/library/windowsazure/dn303373.aspx
[User object]: http://msdn.microsoft.com/library/windowsazure/jj554220.aspx
[Benutzerobjekt]: http://msdn.microsoft.com/library/windowsazure/jj554220.aspx
[Benutzerobjekts]: http://msdn.microsoft.com/library/windowsazure/jj554220.aspx
[push object]: http://msdn.microsoft.com/library/windowsazure/jj554217.aspx
[insert function]: http://msdn.microsoft.com/library/windowsazure/jj554229.aspx
[insert-]: http://msdn.microsoft.com/library/windowsazure/jj554229.aspx
[update function]: http://msdn.microsoft.com/library/windowsazure/jj554214.aspx
[delete function]: http://msdn.microsoft.com/library/windowsazure/jj554215.aspx
[read function]: http://msdn.microsoft.com/library/windowsazure/jj554224.aspx
[update-]: http://msdn.microsoft.com/library/windowsazure/jj554214.aspx
[delete-]: http://msdn.microsoft.com/library/windowsazure/jj554215.aspx
[read-]: http://msdn.microsoft.com/library/windowsazure/jj554224.aspx
[query object]: http://msdn.microsoft.com/library/windowsazure/jj613353.aspx
[Suchabfrageobjekt]: http://msdn.microsoft.com/library/windowsazure/jj613353.aspx
[apns object]: http://msdn.microsoft.com/library/windowsazure/jj839711.aspx
[mpns object]: http://msdn.microsoft.com/library/windowsazure/jj871025.aspx
[wns object]: http://msdn.microsoft.com/library/windowsazure/jj860484.aspx
[Tabellenobjekt]: http://msdn.microsoft.com/library/windowsazure/jj554210.aspx
[Tabellenobjekt]: http://msdn.microsoft.com/library/windowsazure/jj614364.aspx
[mssql-Objekt]: http://msdn.microsoft.com/library/windowsazure/jj554212.aspx
[mssql-Objekts]: http://msdn.microsoft.com/library/windowsazure/jj554212.aspx
[Konsolenobjekt]: http://msdn.microsoft.com/library/windowsazure/jj554209.aspx
[Konsolenobjekts]: http://msdn.microsoft.com/library/windowsazure/jj554209.aspx
[Lesen und Schreiben von Daten]: http://msdn.microsoft.com/library/windowsazure/jj631640.aspx
[Überprüfen von Daten]: http://msdn.microsoft.com/library/windowsazure/jj631638.aspx
[Ändern der Anforderung]: http://msdn.microsoft.com/library/windowsazure/jj631635.aspx
[Ändern der Antwort]: http://msdn.microsoft.com/library/windowsazure/jj631631.aspx
[klassische Azure-Portal]: https://manage.windowsazure.com/
[klassischen Azure-Portal]: https://manage.windowsazure.com/
[Geplante Aufträge]: http://msdn.microsoft.com/library/windowsazure/jj860528.aspx
[Validate and modify data in Mobile Services by using server scripts]: /develop/mobile/tutorials/validate-modify-and-augment-data-dotnet/
[Commands to manage Azure Mobile Services]: ../virtual-machines-command-line-tools.md#Mobile_Scripts
[Windows Store Push]: /develop/mobile/tutorials/get-started-with-push-dotnet/
[Windows Phone Push]: /develop/mobile/tutorials/get-started-with-push-wp8/
[iOS Push]: /develop/mobile/tutorials/get-started-with-push-ios/
[Android Push]: /develop/mobile/tutorials/get-started-with-push-android/
[Azure SDK für Node.js]: http://go.microsoft.com/fwlink/p/?LinkId=275539
[HTTP-Anforderungen senden]: http://msdn.microsoft.com/library/windowsazure/jj631641.aspx
[Send email from Mobile Services with SendGrid]: /develop/mobile/tutorials/send-email-with-sendgrid/
[Get started with authentication]: http://go.microsoft.com/fwlink/p/?LinkId=287177
[Erste Schritte mit der Authentifizierung]: http://go.microsoft.com/fwlink/p/?LinkId=287177
[crypto API]: http://go.microsoft.com/fwlink/p/?LinkId=288802
[path API]: http://go.microsoft.com/fwlink/p/?LinkId=288803
[querystring API]: http://go.microsoft.com/fwlink/p/?LinkId=288804
[url API]: http://go.microsoft.com/fwlink/p/?LinkId=288805
[util API]: http://go.microsoft.com/fwlink/p/?LinkId=288806
[zlib API]: http://go.microsoft.com/fwlink/p/?LinkId=288807
[Benutzerdefinierte API]: http://msdn.microsoft.com/library/windowsazure/dn280974.aspx
[Call a custom API from the client]: /develop/mobile/tutorials/call-custom-api-dotnet/#define-custom-api
[express.js-Bibliothek]: http://go.microsoft.com/fwlink/p/?LinkId=309046
[Define a custom API that supports periodic notifications]: /develop/mobile/tutorials/create-pull-notifications-dotnet/
[express-Objekt in express.js]: http://expressjs.com/api.html#express
[Store server scripts in source control]: /develop/mobile/tutorials/store-scripts-in-source-control/
[Leverage shared code and Node.js modules in your server scripts]: /develop/mobile/tutorials/store-scripts-in-source-control/#use-npm
[Serviceobjekt]: http://msdn.microsoft.com/library/windowsazure/dn303371.aspx
[App-Einstellungen]: http://msdn.microsoft.com/library/dn529070.aspx
[config module]: http://msdn.microsoft.com/library/dn508125.aspx
[Support for package.json in Azure Mobile Services]: http://go.microsoft.com/fwlink/p/?LinkId=391036
 

<!---HONumber=AcomDC_0727_2016-->

