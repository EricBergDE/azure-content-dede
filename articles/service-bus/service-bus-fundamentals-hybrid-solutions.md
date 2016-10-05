<properties 
	pageTitle="Azure Service Bus | Microsoft Azure" 
	description="Enthält eine Einführung in die Verwendung von Service Bus zum Herstellen einer Verbindung für Azure-Anwendungen mit anderer Software." 
	services="service-bus" 
	documentationCenter=".net" 
	authors="sethmanheim" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="service-bus" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="get-started-article" 
	ms.date="08/31/2016" 
	ms.author="sethm"/>

# Azure-Servicebus

Ob eine Anwendung oder ein Dienst in einer Cloud betrieben wird oder lokal installiert ist – sie bzw. er muss oft mit anderen Anwendungen oder Diensten interagieren. Microsoft Azure stellt Service Bus als umfangreich einsetzbare Lösung bereit. Der vorliegende Artikel bietet einen Überblick über diese Technologie und beschreibt, wann Sie den Einsatz von Service Bus in Erwägung ziehen sollten.

## Service Bus – Grundlagen

Verschiedene Situationen erfordern verschiedene Kommunikationsarten. Manchmal ist die beste Lösung, Anwendungen Nachrichten über einfache Warteschlangen senden und empfangen zu lassen. In anderen Situationen ist eine normale Warteschlange nicht genug, sondern eine Warteschlange mit einem Veröffentlichungs- und Abonnementsmechanismus ist besser geeignet. In manchen Fällen ist nur eine Verbindung zwischen den Anwendungen erforderlich. Warteschlangen sind nicht erforderlich. Service Bus bietet alle drei Optionen, sodass Ihre Anwendungen auf mehrere verschiedene Arten miteinander interagieren können.

Service Bus ist ein Clouddienst mit mehreren Mandanten. Das bedeutet, dass der Dienst von mehreren Benutzern gemeinsam genutzt wird. Jeder Benutzer, z. B. auch ein Anwendungsentwickler, erstellt einen *Namespace* und definiert dann die erforderlichen Kommunikationsmechanismen innerhalb dieses Namespace. Abbildung 1 zeigt, wie dies aussieht.

![][1]
 
**Abbildung 1: Service Bus stellt einen mehrinstanzenfähigen Dienst zum Verbinden von Anwendungen über die Cloud bereit.**

Innerhalb eines Namespace können Sie eine oder mehr Instanzen von vier verschiedenen Kommunikationsmechanismen verwenden, von denen jeder die Anwendungen auf andere Weise verbindet. Die Auswahlmöglichkeiten sind:

- *Warteschlangen* für eine unidirektionale Kommunikation. Jede Warteschlange agiert als Zwischenstufe (manchmal auch *Broker* genannt), die gesendete Nachrichten speichert, bis diese empfangen werden. Jede Nachricht wird von einem einzelnen Empfänger empfangen.
- *Themen* für die unidirektionale Kommunikation mithilfe von *Abonnements*, wobei ein einzelnes Thema mehrere Abonnements umfassen kann. Wie bei einer Warteschlange agiert ein Thema als Broker, aber jedes Abonnement kann optional Filter verwenden, um nur Nachrichten zu empfangen, die bestimmte Kriterien erfüllen.
- *Relays* für eine bidirektionale Kommunikation. Im Gegensatz zu Warteschlangen und Themen speichert ein Relay keine gesendeten Nachrichten – es ist kein Broker. Stattdessen leitet es sie an eine Zielanwendung weiter.

Wenn Sie eine Warteschlange, ein Thema oder ein Relais erstellen, geben Sie diesem Objekt einen Namen. In Kombination mit der Bezeichnung Ihres Namespace ist dieser Name ein eindeutiger Bezeichner für das Objekt. Anwendungen können diesen Namen an Service Bus übermitteln und dann mithilfe der Warteschlange, des Themas oder des Relays miteinander kommunizieren.

Für die Verwendung eines dieser Objekte im Relayszenario können Windows-Anwendungen Windows Communication Foundation (WCF) nutzen. Für Warteschlangen und Themen können Windows-Anwendungen eine über Service Bus definierte Messaging-API verwenden. Damit diese Objekte von Nicht-Windows-Anwendungen einfacher verwendet werden können, stellt Microsoft SDKs für Java, Node.js und andere Sprachen bereit. Sie können auch mithilfe von REST-APIs über HTTP(s) auf Warteschlangen und Themen zugreifen.

Es ist wichtig zu verstehen, dass, obwohl Service Bus selbst in einer Cloud (in den Azure-Rechenzentren von Microsoft) ausgeführt wird, die Anwendungen, die es verwenden, überall laufen können. Sie können Service Bus zum Verbinden von Anwendungen auf Azure oder innerhalb Ihres eigenen Datencenters verwenden. Sie können damit auch eine Anwendung, die auf Azure oder einer anderen Cloud-Plattform läuft, mit einer lokalen Anwendung oder mit Tablets oder Telefonen verbinden. Es ist auch möglich, Haushaltsgeräte, Sensoren und andere Vorrichtungen mit einer zentralen Anwendung oder miteinander zu verbinden. Service Bus ist ein Kommunikationsmechanismus in der Cloud, auf den praktisch von überall aus zugegriffen werden kann. Wie Sie es verwenden, hängt davon ab, was Ihre Anwendungen leisten müssen.

## Warteschlangen

Angenommen, Sie entscheiden sich dafür, zwei Anwendungen über eine Service Bus-Warteschlange miteinander zu verbinden. Abbildung 2 illustriert diese Situation.

![][2]
 
**Abbildung 2: Service Bus-Warteschlangen bieten eine asynchrone unidirektionale Kommunikation.**

Der Vorgang ist einfach: Ein Sender sendet eine Nachricht an eine Service Bus-Warteschlange, und ein Empfänger greift diese Nachricht zu einem späteren Zeitpunkt auf. Eine Warteschlange kann einen einzelnen Empfänger besitzen, wie in Abbildung 2 dargestellt. Alternativ können auch mehrere Anwendungen aus der gleichen Warteschlange lesen. Im zweiten Fall wird jede Nachricht nur von einem einzelnen Empfänger gelesen. Bei einem Multicastdienst empfiehlt sich stattdessen die Verwendung eines Themas.

Jede Nachricht besteht aus zwei Teilen: einem Satz von Eigenschaften (jede davon ist ein Paar aus Schlüssel und Wert) und einem binären Nachrichtentext. Die Art der Verwendung richtet sich danach, welche Aufgaben in der Anwendung ausgeführt werden sollen. Wenn eine Anwendung z. B. eine Nachricht über einen kürzlich erfolgten Verkauf sendet, kann die Nachricht die Eigenschaften *Verkäufer="Ava"* und *Menge=10000* enthalten. Der Nachrichtentext kann ein gescanntes Bild des unterzeichneten Verkaufsvertrags enthalten oder auch leer sein.

Ein Empfänger kann eine Nachricht von einem Servicebus auf zwei verschiedene Arten lesen. Bei Verwendung der ersten Option, *ReceiveAndDelete*, wird eine Nachricht aus der Warteschlange entfernt und sofort gelöscht. Dies ist einfach, aber wenn der Empfänger ausfällt, bevor die Verarbeitung der Nachricht abgeschlossen ist, geht die Nachricht verloren. Da die Nachricht dann aus der Warteschlange entfernt worden ist, kann kein anderer Empfänger auf sie zugreifen.

Die zweiten Option, *PeekLock*, dient zur Lösung dieses Problems. Genau wie bei **ReceiveAndDelete** wird auch bei **PeekLock** eine gelesene Nachricht aus der Warteschlange entfernt. Die Nachricht wird allerdings nicht gelöscht, sondern gesperrt und für andere Empfänger unsichtbar gemacht. Anschließend wird auf eines von drei Ereignissen gewartet.

- Wenn der Empfänger die Nachricht erfolgreich verarbeitet, wird die Meldung **Complete** ausgegeben, und die Warteschlange löscht die Nachricht.
- Wenn der Empfänger entscheidet, dass er die Nachricht nicht erfolgreich verarbeiten kann, wird die Meldung **Abandon** ausgegeben. In diesem Falle hebt die Warteschlange die Sperrung der Nachricht auf und macht sie für andere Empfänger verfügbar.
- Wenn der Empfänger innerhalb eines konfigurierbaren Zeitraums (standardmäßig 60 Sekunden) keine der beiden Meldungen ausgibt, nimmt die Warteschlange an, dass der Empfänger ausgefallen ist. In diesem Fall verhält sie sich so, als hätte der Empfänger **Abandon** aufgerufen, sodass die Nachricht für andere Empfänger verfügbar wird.

Beachten Sie, was geschieht: Dieselbe Nachricht kann zweimal ausgegeben werden, möglicherweise sogar an zwei verschiedene Empfänger. Anwendungen, die Servicebus-Warteschlangen verwenden, müssen darauf vorbereitet sein. Um die Erkennung von Duplikaten zu erleichtern, verfügt jede Nachricht über die eindeutige Eigenschaft **MessageID**, die standardmäßig gleich bleibt – unabhängig davon, wie oft die Nachricht aus einer Warteschlange gelesen wird.

Warteschlangen sind in einigen Situationen sinnvoll. Sie ermöglichen eine Kommunikation zwischen Anwendungen auch dann, wenn beide zur selben Zeit ausgeführt werden, was bei Batch- und Mobilanwendungen sehr praktisch sein kann. Eine Warteschlange mit mehreren Empfängern bietet auch automatischen Lastenausgleich, da gesendete Nachrichten zwischen diesen Empfängern verteilt werden.

## Themen

So hilfreich sie generell auch sind – Warteschlangen sind nicht immer die beste Lösung. Manchmal sind Servicebus-Themen besser. Abbildung 3 illustriert diesen Gedanken.

![][3]
 
**Abbildung 3: Auf Grundlage des in einer Abonnementanwendung definierten Filters kann diese einige oder alle Nachrichten empfangen, die an ein Service Bus-Thema gesendet wurden.**

Ein *Thema* ähnelt in vielen Punkten einer Warteschlange. Sender schicken Nachrichten an ein Thema auf dieselbe Weise, auf die sie Nachrichten an eine Warteschlange schicken, und diese Nachrichten sehen genauso aus wie bei den Warteschlangen. Der große Unterschied ist, dass bei Themen jede empfangende Anwendung durch Definieren eines *Filters* ihr eigenes *Abonnement* erstellen kann. Ein Abonnement sieht dann nur die Nachrichten, die auf diesen Filter passen. Abbildung 3 zeigt z. B. einen Sender und ein Thema mit drei Abonnenten, jeder mit seinem eigenen Filter:

- Abonnent 1 empfängt nur Nachrichten, die die Eigenschaft *Verkäufer="Ava"* haben.
- Abonnent 2 empfängt Nachrichten, die die Eigenschaft *Verkäufer="Ruby"* und/oder eine Eigenschaft namens *Menge* haben, deren Wert größer ist als 100.000. Ruby könnte die Vertriebschefin sein, die sowohl ihre eigenen Verkäufe als auch diejenigen anderer Verkäufer ab einer bestimmten Menge sehen möchte.
- Abonnent 3 hat seinen Filter auf *True* gesetzt, was bedeutet, dass er alle Nachrichten empfängt. Diese Anwendung könnte z. B. zum Verwalten eines Überwachungspfads zuständig sein. Daher muss sie alle Nachrichten sehen können.

Wie bei Warteschlangen können Abonnenten eines Themas Nachrichten entweder über **ReceiveAndDelete** oder über **PeekLock** lesen. Anders als bei Warteschlangen kann jedoch eine einzelne an ein Thema gesendete Nachricht von mehreren Abonnements empfangen werden. Dieser Ansatz, der allgemein als *Veröffentlichen und Abonnieren* (oder *Pub/Sub*)bezeichnet wird, ist sinnvoll, wenn mehrere Anwendungen an den gleichen Nachrichten interessiert sind. Durch Definition eines passenden Filters kann jeder Abonnent genau den Teil aus der Nachricht herausziehen, den er mitbekommen muss.

## Relays

Sowohl Warteschlangen als auch Themen ermöglichen eine asynchrone unidirektionale Kommunikation über einen Broker. Der Verkehr fließt in nur eine Richtung, und es besteht keine direkte Verbindung zwischen Sendern und Empfängern. Aber was tun, wenn dies nicht genug ist? Angenommen, Ihre Anwendungen sollen Nachrichten sowohl senden als auch empfangen, oder sie möchten vielleicht eine direkte Verbindung zwischen ihnen herstellen, ohne einen Broker zum Speichern der Nachrichten zu verwenden. Für solche Szenarien bietet Service Bus *Relays* an, wie in Abbildung 4 gezeigt.

![][4]
 
**Abbildung 4: Service Bus Relay ermöglicht eine synchrone bidirektionale Kommunikation zwischen Anwendungen.**

Die naheliegende Frage lautet: Welchen Grund gibt es für den Einsatz eines Relays? Selbst wenn ich keine Warteschlangen benötige, wieso sollten Anwendungen über einen Clouddienst kommunizieren und nicht direkt miteinander interagieren? Die Antwort ist, dass eine direkte Kommunikation schwieriger sein kann, als Sie denken.

Nehmen wir an, Sie möchten zwei lokale Anwendungen miteinander verbinden, die beide in Datencentern Ihres Unternehmens laufen. Jede dieser Anwendungen befindet sich hinter einer Firewall, und wahrscheinlich nutzt jedes Rechenzentrum eine Netzwerkadressübersetzung (NAT). Die Firewall blockiert alle eingehenden Nachrichten auf fast allen Ports, und NAT bedeutet, dass die Computer, auf denen jede Anwendung läuft, keine feste IP-Adresse haben, auf die Sie von außerhalb des Rechenzentrums nicht direkt zugreifen können. Ohne zusätzliche Hilfe ist die Verbindung dieser Anwendungen über das öffentliche Internet problematisch.

Ein Service Bus Relay kann diese Hilfestellung leisten. Zur bidirektionalen Kommunikation über ein Relay baut jede Anwendung eine ausgehende TCP-Verbindung mit Service Bus auf und hält diese offen. Jede Kommunikation zwischen den beiden Anwendungen läuft über diese beiden Verbindungen. Da jede Verbindung innerhalb des Rechenzentrums hergestellt wurde, erlaubt die Firewall, dass eingehender Datenverkehr an jede Anwendung gesendet wird, ohne dass neue Ports geöffnet werden müssen. Dieser Ansatz löst auch das NAT-Problem, da jede Anwendung während der gesamten Kommunikation einen konstanten Endpunkt in der Cloud aufweist. Durch den Datenaustausch über das Relay können die Anwendungen die Probleme vermeiden, die die Kommunikation erschweren können.

Um Service Bus Relay verwenden zu können, nutzen die Anwendungen Windows Communication Foundation (WCF). Service Bus bietet WCF-Anbindungen, die den Windows-Anwendungen die Interaktion über Relays erleichtern. Anwendungen, die WCF bereits verwenden, geben normalerweise nur eine dieser Anbindungen an und kommunizieren dann miteinander über ein Relay. Anders als bei Warteschlangen und Themen erfordert die Verwendung von Relays mit Windows-fremden Anwendungen einigen Programmieraufwand (sofern sie überhaupt möglich ist), da hierfür keine Standardbibliotheken zur Verfügung stehen.

Im Gegensatz zu Warteschlangen und Themen werden Relays nicht explizit durch Anwendungen erstellt. Wenn eine Anwendung zum Empfangen von Nachrichten eine TCP-Verbindung mit Service Bus aufbaut, wird ein Relay automatisch erstellt. Wenn die Verbindung ausfällt, wird das Relay gelöscht. Damit eine Anwendung ein von einem bestimmten Listener erzeugtes Relay finden kann, bietet Service Bus eine Registry, die es Anwendungen erlaubt, ein bestimmtes Relay anhand seines Namens zu finden.

Relays sind die richtige Lösung, wenn direkte Kommunikation zwischen Anwendungen erforderlich ist. Denken Sie z. B. an ein Fluglinien-Reservierungssystem, das in einem lokalen Rechenzentrum läuft und auf das von Eincheck-Terminals, Mobilgeräten und anderen Computern zugegriffen wird. Anwendungen, die auf all diesen Systemen laufen, könnten auf Service Bus Relay in der Cloud vertrauen, um immer dann zu kommunizieren, wenn sie gerade ausgeführt werden.

## Zusammenfassung

Das Verbinden von Anwendungen zählte schon immer zur Erstellung von Komplettlösungen und die Palette von Szenarien, in denen erforderlich ist, dass Anwendungen und Dienste miteinander kommunizieren, nimmt stetig zu, da Anwendungen und Geräte in zunehmenden Maße mit dem Internet verbunden sind. Service Bus stellt cloudbasierte Technologien bereit, um diese Aufgabe über Warteschlangen, Themen und Relays zu bewältigen und damit diese wesentliche Funktion leichter und breiter verfügbar zu machen.

## Nächste Schritte

Nachdem Sie nun mit den Grundlagen von Azure Service Bus vertraut sind, finden Sie unter den folgenden Links weitere Informationen.

- Gewusst wie: Verwenden von [Service Bus-Warteschlangen](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md)
- Gewusst wie: Verwenden von [Service Bus-Themen](../service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions.md)
- Gewusst wie: Verwenden von [Service Bus Relay](../service-bus-relay/service-bus-dotnet-how-to-use-relay.md)
- [Service Bus-Beispiele:](service-bus-samples.md)

[1]: ./media/service-bus-fundamentals-hybrid-solutions/SvcBus_01_architecture.png
[2]: ./media/service-bus-fundamentals-hybrid-solutions/SvcBus_02_queues.png
[3]: ./media/service-bus-fundamentals-hybrid-solutions/SvcBus_03_topicsandsubscriptions.png
[4]: ./media/service-bus-fundamentals-hybrid-solutions/SvcBus_04_relay.png

<!---HONumber=AcomDC_0928_2016-->