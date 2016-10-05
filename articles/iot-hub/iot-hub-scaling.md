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
 ms.date="09/19/2016"
 ms.author="elioda"/>

# Skalieren von IoT Hub

Azure IoT Hub kann bis zu einer Million gleichzeitig verbundener Geräte unterstützen. Weitere Informationen finden Sie unter [IoT Hub – Preise][lnk-pricing]. Jede IoT Hub-Einheit lässt eine bestimmte Anzahl von täglichen Nachrichten zu.

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
| S3 | Bis zu 814 MB/Minute pro Einheit<br/>(1.144,4 GB/Tag/Einheit) | Durchschnittlich 208.333 Nachrichten/Minute pro Einheit<br/>(300 Millionen Nachrichten/Tag pro Einheit) |

## Durchsatz von Identitätsregistrierungsvorgängen

IoT Hub-Identitätsregistrierungsvorgänge sollten keine Laufzeitvorgänge sein, da sie sich größtenteils auf die Gerätebereitstellung beziehen.

Genaue Zahlen zur Burst-Leistung finden Sie unter [IoT Hub-Kontingente und -Drosselungen][].

## Sharding (Horizontales Partitionieren)

Eine einzelne IoT Hub-Einheit kann zwar auf Millionen von Geräten skaliert werden, aber es kann sein, dass Ihre Lösung bestimmte Leistungsmerkmale benötigt, die von einer einzelnen IoT Hub-Einheit nicht gewährleistet werden können. In diesem Fall wird empfohlen, Ihre Geräte auf mehrere IoT Hubs zu partitionieren. Mehrere IoT Hubs glätten Datenverkehrsbursts und erzielen den erforderlichen Durchsatz oder die erforderlichen Vorgangsraten.

## Nächste Schritte

Weitere Informationen zu den Funktionen von IoT Hub finden Sie unter:

- [Entwicklerhandbuch][lnk-devguide]
- [Erkunden der Geräteverwaltung mithilfe der Beispielbenutzeroberfläche][lnk-dmui]
- [Simulieren eines Geräts mit dem Gateway SDK][lnk-gateway]
- [Verwalten von IoT Hubs über das Azure-Portal][lnk-portal]

[lnk-pricing]: https://azure.microsoft.com/pricing/details/iot-hub
[IoT Hub-Kontingente und -Drosselungen]: iot-hub-devguide.md#throttling

[lnk-design]: iot-hub-guidance.md
[lnk-mqtt]: iot-hub-mqtt-support.md
[lnk-devices]: iot-hub-tested-configurations.md
[lnk-protocols]: iot-hub-protocol-gateway.md
[lnk-compare]: iot-hub-compare-event-hubs.md
[lnk-scaling]: iot-hub-scaling.md
[lnk-devguide]: iot-hub-devguide.md
[lnk-dmui]: iot-hub-device-management-ui-sample.md
[lnk-gateway]: iot-hub-linux-gateway-sdk-simulated-device.md
[lnk-portal]: iot-hub-manage-through-portal.md

<!---HONumber=AcomDC_0921_2016-->