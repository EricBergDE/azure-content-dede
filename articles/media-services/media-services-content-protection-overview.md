<properties 
	pageTitle="Schützen von Inhalten – Übersicht | Microsoft Azure" 
	description="In diesem Artikel finden Sie eine Übersicht über die Content Protection mit Media Services." 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/19/2016" 
	ms.author="juliako"/>

#Schützen von Inhalten – Übersicht


Microsoft Azure Media Services ermöglicht die Sicherung Ihrer Medien ab dem Zeitpunkt, an dem sie Ihren Computer durch Speicherung, Verarbeitung und Übermittlung verlassen. Mit Media Services können Sie Inhalte dynamisch verschlüsselt übermitteln, und zwar mit AES (Advanced Encryption Standard unter Verwendung eines 128-Bit-Verschlüsselungsschlüssels) und Common Encryption (CENC, allgemeine Verschlüsselung) mit PlayReady- oder Widevine-DRM. Media Services bietet auch einen Dienst für die Übermittlung von AES-Schlüsseln und PlayReady-Lizenzen an autorisierte Clients. Sie können sich auch von folgenden AMS-Partnern bei der Übermittlung von Widevine-Lizenzen unterstützen lassen: [Axinom](http://www.axinom.com/press/ibc-axinom-drm-6/), [EZDRM](http://ezdrm.com/), [CastLabs](http://castlabs.com/company/partners/azure/).

- Die folgende Abbildung veranschaulicht den Workflow für dynamische Common Encryption mit PlayReady- und/oder Widevine-DRM. Weitere Informationen finden Sie unter [Verwenden von dynamischer allgemeiner Verschlüsselung mit PlayReady- und/oder Widevine-DRM](media-services-protect-with-drm.md).

![Schützen mit PlayReady](./media/media-services-content-protection-overview/media-services-content-protection-with-drm.png)


- Die folgende Abbildung veranschaulicht den Workflow für dynamische AES-128-Verschlüsselung. Ausführliche Informationen finden Sie unter [Verwenden der dynamischen AES-128-Verschlüsselung und des Schlüsselübermittlungsdiensts](media-services-protect-with-aes128.md).

![Schützen mit AES-128](./media/media-services-content-protection-overview/media-services-content-protection-with-aes.png)

>[AZURE.NOTE]Damit Sie die dynamische Verschlüsselung verwenden können, müssen Sie zunächst mindestens eine reservierte Einheit für das Streaming auf dem Streamingendpunkt abrufen, auf dem Sie verschlüsselte Inhalte streamen möchten.

##Optionen zur Medienobjektverschlüsselung

Je nach Art der Inhalte, die Sie hochladen, speichern und bereitstellen möchten, stellt Media Services verschiedene Verschlüsselungsoptionen zur Verfügung, aus denen Sie auswählen können.

###Keine

Es wird keine Verschlüsselung verwendet. Dies ist der Standardwert. Bei Verwendung dieser Option sind Ihre Inhalte während der Übertragung oder im Ruhezustand im Speicher nicht geschützt.

Wenn Sie planen, eine MP4-Datei über progressives Herunterladen zu übermitteln, verwenden Sie diese Option zum Hochladen der Inhalte.

###StorageEncrypted

Verwenden Sie **StorageEncrypted**, um Ihre unverschlüsselten Inhalte lokal mithilfe der AES-256-Bit-Verschlüsselung zu verschlüsseln und sie dann in Azure Storage hochzuladen, wo sie verschlüsselt im Ruhezustand gespeichert werden. Medienobjekte, die durch Speicherverschlüsselung geschützt sind, werden automatisch entschlüsselt, vor der Codierung in einem verschlüsselten Dateisystem platziert und optional vor dem Hochladen als neues Ausgabemedienobjekt erneut verschlüsselt. Der primäre Anwendungsfall für die Speicherverschlüsselung ist, wenn Sie Ihre qualitativ hochwertigen Eingabemediendateien mit starker Verschlüsselung beim Speichern im Ruhezustand auf dem Datenträger sichern möchten.

Um ein speicherverschlüsseltes Medienobjekt zu übermitteln, müssen Sie die Übermittlungsrichtlinie des Medienobjekts konfigurieren und Media Services so mitteilen, wie die Inhalte bereitgestellt werden sollen. Bevor das Medienobjekt gestreamt werden kann, wird die Speicherverschlüsselung vom Streamingserver entfernt und der Inhalt mithilfe der angegebenen Übermittlungsrichtlinie (AES, Common Encryption oder unverschlüsselt) gestreamt.

####Details zur Implementierung

Die AMS-Speicherverschlüsselung wendet die Verschlüsselung im **AES-CTR**-Modus auf die gesamte Datei an. AES-CTR ist eine Blockchiffre, die Daten beliebiger Länge ohne Auffüllen verschlüsseln kann. AES-CTR verschlüsselt einen Zählerblock mit dem AES-Algorithmus und kombiniert die Ausgabe von AES per XOR-Operation mit den zu verschlüsselnden oder zu entschlüsselnden Daten. Der verwendete Zählerblock wird erstellt, indem der InitializationVector-Wert in die Bytes 0 bis 7 des Zählerwerts kopiert und die Bytes 8 bis 15 des Zählerwerts auf 0 gesetzt werden. Im 16 Byte langen Zählerblock werden die Bytes 8 bis 15 (also die niedrigstwertigen Bytes) als einfache 64-Bit-Ganzzahl ohne Vorzeichen verwendet, die für jeden nachfolgenden Block verarbeiteter Daten um eins erhöht wird. Die Reihenfolge der Netzwerkbytes wird beibehalten. Beachten Sie Folgendes: Wenn diese Ganzzahl ihren Maximalwert erreicht (0xFFFFFFFFFFFFFFFF), wird durch weitere Erhöhung der Blockzähler auf Null zurückgesetzt (Bytes 8 bis 15), ohne Auswirkung auf die restlichen 64 Bits des Zählers (also Bytes 0 bis 7). Um die Sicherheit der Verschlüsselung im AES-CTR-Modus zu gewährleisten, muss der InitializationVector-Wert für jede KID für jede Datei eindeutig sein, und die Dateien müssen weniger als 2^64 Blöcke lang sein. Damit wird sichergestellt, dass ein Zählerwert nie mit einem vorhandenen Schlüssel wiederverwendet wird. Weitere Informationen zum CTR-Modus finden Sie [auf dieser Wiki-Seite](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CTR) (der Wiki-Artikel verwendet den Begriff „Nonce“ anstelle von „InitializationVector“).

Wenn Sie wissen möchten, wie der grundlegende Algorithmus funktioniert, sehen Sie sich die AMS .NET-Implementierung der folgenden Methoden an:

- [ApplyEncryptionTransform](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.BlobTransfer/BlobTransferBase.cs)
- [AesCtr](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/FileEncryptionTransform.cs)


###CommonEncryptionProtected

Verwenden Sie **CommonEncryptionProtected**, wenn Sie Inhalte mit Common Encryption verschlüsseln (oder bereits verschlüsselte Inhalte hochladen) möchten. PlayReady und Widewine werden gemäß der Common Encryption (CENC)-Spezifikation verschlüsselt und von AMS unterstützt.

###EnvelopeEncryptionProtected

Verwenden Sie **EnvelopeEncryptionProtected**, wenn Sie HTTP Live Streaming-Inhalte (HLS) schützen (oder bereits geschützte HLS-Inhalte hochladen) möchten, die mit dem Advanced Encryption Standard (AES) verschlüsselt sind. Beachten Sie, dass beim Hochladen von bereits mit AES verschlüsseltem HLS die Verschlüsselung mithilfe von Transform Manager erfolgt sein muss.

##Dynamische Verschlüsselung

Mit Microsoft Azure Media Services können Sie Inhalte (dynamisch) verschlüsselt übermitteln, und zwar mit AES (Advanced Encryption Standard unter Verwendung eines 128-Bit-Verschlüsselungsschlüssels) und PlayReady- und/oder Widevine-DRM.

Zur Zeit können Sie die folgenden Streamingformate verschlüsseln: HLS, MPEG DASH und Smooth Streaming. Das HDS-Streamingformat oder progressive Downloads können nicht verschlüsselt werden.

Wenn ein Medienobjekt durch Media Services verschlüsselt werden soll, müssen Sie dem Medienobjekt einen Verschlüsselungsschlüssel (CommonEncryption oder EnvelopeEncryption) zuordnen und zusätzlich Autorisierungsrichtlinien für den Schlüssel konfigurieren.

Außerdem müssen Sie die Übermittlungsrichtlinie des Medienobjekts konfigurieren. Wenn Sie ein speicherverschlüsseltes Medienobjekt streamen möchten, geben Sie durch Konfigurieren der Übermittlungsrichtlinie an, wie die Übermittlung erfolgen soll.

Wenn ein Player einen Stream anfordert, verwendet Media Services den angegebenen Schlüssel, um den Inhalt mit AES oder Common Encryption dynamisch zu verschlüsseln. Um den Stream zu entschlüsseln, fordert der Player den Schlüssel vom Schlüsselübermittlungsdienst an. Um zu entscheiden, ob der Benutzer berechtigt ist, den Schlüssel zu erhalten, wertet der Dienst die Autorisierungsrichtlinien aus, die Sie für den Schlüssel angegeben haben.

>[AZURE.NOTE]Um die dynamische Verschlüsselung nutzen zu können, ist mindestens eine On-Demand-Streamingeinheit für den Streamingendpunkt erforderlich, aus dem die verschlüsselten Inhalte bereitgestellt werden sollen. Weitere Informationen finden Sie unter [Skalieren von Media Services](media-services-portal-manage-streaming-endpoints.md).

##Dienst für die Übermittlung von Lizenzen und Schlüsseln

Media Services bietet einen Dienst zum Übermitteln von DRM-Lizenzen (PlayReady und Widevine) und unverschlüsselten AES-Schlüsseln für autorisierte Clients. Sie können das klassische Azure-Portal, die REST-API oder das Media Services-SDK für .NET zum Konfigurieren von Autorisierungs- und Authentifizierungsrichtlinien für Ihre Lizenzen und Schlüssel verwenden.

Beachten Sie, dass Sie über das Portal eine AES-Richtlinie (zur Anwendung auf alle AES-verschlüsselten Inhalte) und eine PlayReady-Richtlinie (zur Anwendung auf alle PlayReady-verschlüsselten Inhalte) konfigurieren können. Verwenden Sie das Media Services-SDK für .NET, wenn Sie mehr Kontrolle über die Konfigurationen benötigen.

##DRM-Lizenzen

###PlayReady-Lizenz

Media Services bietet einen Dienst für die Übermittlung von PlayReady-Lizenzen. Wenn der Endbenutzer-Player (z. B. Silverlight) versucht, Ihre durch PlayReady geschützten Inhalte wiederzugeben, wird eine Anforderung zum Erwerben einer Lizenz an den Lizenzübermittlungsdienst gesendet. Wenn der Lizenzdienst die Anforderung genehmigt, wird die Lizenz ausgegeben. Diese wird an den Client gesendet und kann zum Entschlüsseln und Wiedergeben des angegebenen Inhalts verwendet werden.

Lizenzen enthalten die Rechte und Einschränkungen, die von der PlayReady-DRM-Laufzeit durchgesetzt werden sollen, wenn ein Benutzer versucht, geschützte Inhalte wiederzugeben. Media Services bietet APIs, mit denen Sie Ihre PlayReady-Lizenzen konfigurieren können. Weitere Informationen finden Sie unter [Media Services PlayReady-Lizenzvorlage – Übersicht](media-services-playready-license-template-overview.md).

###Widevine-Lizenz

AMS ermöglicht Ihnen auch, MPEG DASH mit Widevine-DRM-Verschlüsselung bereitzustellen. PlayReady und Widevine werden gemäß der Common Encryption (CENC)-Spezifikation verschlüsselt. Sie können das [AMS .NET SDK](https://www.nuget.org/packages/windowsazure.mediaservices/) (ab Version 3.5.1) oder die REST-API verwenden, um die AssetDeliveryConfiguration für die Verwendung von Widevine zu konfigurieren.

Ab Media Services .NET SDK, Version 3.5.2, ermöglicht Media Services Ihnen die Konfiguration der [Widevine-Lizenzvorlage](media-services-widevine-license-template-overview.md) und das Abrufen von Widevine-Lizenzen. Sie können sich auch von folgenden AMS-Partnern bei der Übermittlung von Widevine-Lizenzen unterstützen lassen: [Axinom](http://www.axinom.com/press/ibc-axinom-drm-6/), [EZDRM](http://ezdrm.com/), [CastLabs](http://castlabs.com/company/partners/azure/).

##Token-Einschränkung

Die Autorisierungsrichtlinie für Inhaltsschlüssel kann eine oder mehrere Autorisierungseinschränkungen aufweisen: offen oder Tokeneinschränkung. Die durch Token eingeschränkte Richtlinie gilt nur zusammen mit einem Token, das von einem Secure Token Service (STS) ausgestellt wurde. Media Services unterstützt Token im Simple Web Tokens (SWT)-Format und JSON Web Token (JWT)-Format. Secure Token Services werden von Media Services nicht bereitgestellt. Sie können einen benutzerdefinierten STS erstellen oder Microsoft Azure ACS zum Ausstellen von Token nutzen. Der STS muss für die Erstellung eines mit dem angegebenen Schlüssel signierten Tokens und die Ausstellungsansprüche konfiguriert sein, die Sie in der Konfiguration der Tokeneinschränkung angegeben haben. Der Schlüsselübermittlungsdienst von Media Services gibt den angeforderten Schlüssel (oder die Lizenz) an den Client zurück, wenn das Token gültig ist und die Ansprüche im Token mit den für den Schlüssel (oder die Lizenz) konfigurierten Ansprüchen übereinstimmen.

Bei der Konfiguration der Richtlinie mit Token-Einschränkung müssen die Parameter PrimaryVerificationKey, Issuer und Audience angegeben werden. PrimaryVerificationKey enthält den Schlüssel, mit dem das Token signiert wurde, und Issuer ist der STS (Secure Token Service), von dem das Token ausgestellt wurde. Audience (manchmal auch Scope) beschreibt den Verwendungszweck des Tokens oder die Ressource, auf die durch das Token Zugriff gewährt wird. Der Schlüsselübermittlungsdienst von Media Services überprüft, ob die Werte im Token mit den Werten in der Vorlage übereinstimmen.

##Häufige Szenarios

###Schützen von Inhalten im Speicher, Übermitteln dynamisch verschlüsselter Streamingmedien, Verwenden des AMS-Schlüssel-/Lizenzübermittlungsdiensts

1. Nehmen Sie eine Zwischendatei (Mezzanine File) hoher Qualität in ein Medienobjekt auf. Wenden Sie die Speicherverschlüsselung auf das Medienobjekt an.
2. Konfigurieren von Streamingendpunkten
1. Führen Sie eine Codierung in einen MP4-Satz mit adaptiver Bitrate durch. Wenden Sie die Speicherverschlüsselung auf das Ausgabemedienobjekt an.
1. Erstellen Sie einen Inhaltsverschlüsselungsschlüssel für das Medienobjekt, das während der Wiedergabe dynamisch verschlüsselt werden soll.
2. Konfigurieren Sie Autorisierungsrichtlinien für Inhaltsschlüssel.
1. Konfigurieren Sie Übermittlungsrichtlinien für Medienobjekte (wird zur dynamischen Paketerstellung und zur dynamischen Verschlüsselung verwendet).
1. Veröffentlichen Sie das Medienobjekt durch Erstellen eines OnDemand-Locators.
1. Streamen Sie die veröffentlichten Inhalte.

Weitere Informationen finden Sie unter [Schutz mit AES](media-services-protect-with-aes128.md) und [Schutz mit PlayReady und/oder Widevine](media-services-protect-with-drm.md).


###Verwenden von Media Service-Schlüssel und Lizenzübermittlungsdienst mit eigenen Verschlüsselungs- und Streamingdiensten

Weitere Informationen finden Sie unter [How to integrate Azure PlayReady License service with your own encryptor/streaming server](http://mingfeiy.com/integrate-azure-playready-license-service-encryptorstreaming-server) (Integrieren des Azure PlayReady-Lizenzierungsdiensts auf einem Verschlüsselungs-/Streamingserver, in englischer Sprache).

###Integrieren in Partneranwendungen

[Übermitteln von DRM-Lizenzen an Azure Media Services mithilfe von castLabs](media-services-castlabs-integration.md)

##Media Services-Lernpfade

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Feedback geben

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


##Verwandte Links

[Announcing PlayReady as a service and AES dynamic encryption with Azure Media Services ("Ankündigen von PlayReady als Dienst und dynamische AES-Verschlüsselung mit Azure Media Services", in englischer Sprache)](http://mingfeiy.com/playready)

[Azure Media Services PlayReady license delivery pricing explained ("Erläuterung der Preisgestaltung bei der Azure Media Services PlayReady-Lizenzübermittlung", in englischer Sprache)](http://mingfeiy.com/playready-pricing-explained-in-azure-media-services)

[How to debug for AES encrypted stream in Azure Media Services ("Debuggen AES-verschlüsselter Streams in Azure Media Services", in englischer Sprache)](http://mingfeiy.com/debug-aes-encrypted-stream-azure-media-services)

[JWT-Tokenauthentifizierung](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)

[Integrieren einer Azure Media Services-OWIN MVC-basierten App in Azure Active Directory und Einschränken der Übermittlung von Inhaltsschlüsseln auf Grundlage von JWT-Ansprüchen](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)

[Ausstellen von Token mithilfe von Azure ACS](http://mingfeiy.com/acs-with-key-services)

[content-protection]: ./media/media-services-content-protection-overview/media-services-content-protection.png

<!---HONumber=AcomDC_0921_2016-->