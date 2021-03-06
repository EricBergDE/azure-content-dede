<properties
 pageTitle="Skalieren von Azure IoT Hub | Microsoft Azure"
 description="Beschreibt das Skalieren von Azure IoT Hub."
 services="iot-hub"
 documentationCenter=""
 authors="fsautomata"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="03/14/2016"
 ms.author="elioda"/>

# Skalieren von IoT Hub

Azure IoT Hub kann bis zu einer Million gleichzeitig verbundener Geräte unterstützen. Weitere Informationen finden Sie unter [IoT Hub – Preise][lnk-pricing]. Jede IoT Hub-Einheit lässt eine Anzahl von täglichen Nachrichten zu.

Um Ihre Lösung richtig skalieren zu können, sollten Sie sich nach dem jeweiligen IoT Hub-Anwendungsfall richten. Achten Sie besonders auf den erforderlichen Spitzendurchsatz für die folgenden Kategorien von Vorgängen:

* D2C-Nachrichten
* C2D-Nachrichten
* Identitätsregistrierungsvorgänge

Sehen Sie sich zusätzlich zu diesen Durchsatzinformationen auch [IoT Hub-Kontingente und -Drosselungen][] an und entwerfen Sie Ihre Lösung entsprechend.

## D2C- und C2D-Nachrichtendurchsatz

Die beste Methode zum Skalieren einer IoT Hub-Lösung ist die Auswertung des Datenverkehrs pro Einheit.

D2C-Nachrichten basieren auf diesen Richtlinien für den anhaltenden Durchsatz:

| Tarif | Anhaltender Durchsatz | Anhaltende Senderate |
| ---- | -------------------- | ------------------- |
| S1 | Bis zu 1111 KB/Minute pro Einheit<br/>(1,5 GB/Tag/Einheit) | Durchschnittlich 278 Nachrichten/Minute pro Einheit<br/>(400.000 Nachrichten/Tag pro Einheit) |
| S2 | Bis zu 16 MB/Minute pro Einheit<br/>(22,8 GB/Tag/Einheit) | Durchschnittlich 4167 Nachrichten/Minute pro Einheit<br/>(6 Millionen Nachrichten/Tag pro Einheit) |

Die Leistung wird bei C2D-Nachrichten pro Gerät skaliert und jedes Gerät empfängt bis zu 5 Nachrichten pro Minute.

## Durchsatz von Identitätsregistrierungsvorgängen

IoT Hub-Identitätsregistrierungsvorgänge sollten keine Laufzeitvorgänge sein, da sie sich größtenteils auf die Gerätebereitstellung beziehen.

Genaue Zahlen zur Burst-Leistung finden Sie unter [IoT Hub-Kontingente und -Drosselungen][].

## Sharding (Horizontales Partitionieren)

Eine einzelne IoT Hub-Einheit kann zwar auf Millionen von Geräten skaliert werden, aber es kann sein, dass Ihre Lösung bestimmte Leistungsmerkmale benötigt, die von einer einzelnen IoT Hub-Einheit nicht gewährleistet werden können. In diesem Fall wird empfohlen, Ihre Geräte auf mehrere IoT Hubs zu partitionieren. Mehrere IoT Hubs glätten Datenverkehrsbursts und erzielen den erforderlichen Durchsatz oder die erforderlichen Vorgangsraten.

## Nächste Schritte

Folgen Sie diesen Links, um mehr über Azure IoT Hub zu erfahren:

- [Erste Schritte mit IoT Hub (Lernprogramm)][lnk-get-started]
- [Was ist Azure IoT Hub?][]

[lnk-pricing]: https://azure.microsoft.com/pricing/details/iot-hub
[IoT Hub-Kontingente und -Drosselungen]: iot-hub-devguide.md#throttling

[lnk-get-started]: iot-hub-csharp-csharp-getstarted.md
[Was ist Azure IoT Hub?]: iot-hub-what-is-iot-hub.md

<!---HONumber=AcomDC_0316_2016-->