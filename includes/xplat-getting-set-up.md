<properties services="virtual-machines" title="Setting up Azure CLI for service management" authors="squillace" solutions="" manager="timlt" editor="tysonn" />

<tags
   ms.service="virtual-machine"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="linux"
   ms.workload="infrastructure"
   ms.date="04/13/2015"
   ms.author="rasquill" />

## Verwenden der Azure-Befehlszeilenschnittstelle

Die folgenden Schritte unterstützen Sie bei der Verwendung der Azure-Befehlszeilenschnittstelle mit der neuesten Version und dem richtigen Abonnement. Wenn Sie die Azure-Befehlszeilenschnittstelle noch installieren und mit Ihrem Konto verbinden müssen, lesen Sie zunächst [Azure-Befehlszeilenschnittstelle (Azure-CLI)](xplat-cli-install.md).

### Schritt 1: Aktualisieren der Azure-CLI-Version

Um die Azure-Befehlszeilenschnittstelle für Befehle im Dienstverwaltungsmodus zu verwenden, sollten Sie über eine möglichst neue Version verfügen. Geben Sie zum Überprüfen Ihrer Version `azure --version` ein. Die Ausgabe sollte folgendermaßen aussehen:

    $ azure --version
    0.8.17 (node: 0.10.25)

Wenn Sie Ihre Version der Azure-Befehlszeilenschnittstelle aktualisieren möchten, lesen Sie die Informationen unter [Azure-CLI](https://github.com/Azure/azure-xplat-cli).

### Schritt 2: Festlegen von Azure-Konto und -Abonnement

Nachdem Sie die Azure-Befehlszeilenschnittstelle mit dem zu verwendenden Konto verbunden haben, können Sie mehrere Abonnements verwenden. Wenn Sie dies tun, sollten Sie die für Ihr Konto verfügbaren Abonnements durch Eingabe von `azure account list` auflisten und dann das Abonnement auswählen, das Sie verwenden möchten. Geben Sie dazu `azure account set <subscription id or name> true` ein, wobei _subscription id or name_ entweder die ID oder der Name des Abonnements ist, das Sie in der aktuellen Sitzung verwenden möchten. Folgendes sollte angezeigt werden:

    $ azure account set "Visual Studio Ultimate with MSDN" true
    info:    Executing command account set
    info:    Setting subscription to "Visual Studio Ultimate with MSDN" with id "0e220bf6-5caa-4e9f-8383-51f16b6c109f".
    info:    Changes saved
    info:    account set command OK

> [AZURE.NOTE] Wenn Sie noch nicht über ein Azure-Konto, aber bereits über ein MSDN-Abonnement verfügen, erhalten Sie ein kostenloses Azure-Guthaben, indem Sie Ihren [Azure-Vorteil für MSDN-Abonnenten](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) aktivieren. Sie können aber auch ein kostenloses Konto verwenden. Beides ist für den Azure-Zugriff möglich.

<!---HONumber=AcomDC_0128_2016-->