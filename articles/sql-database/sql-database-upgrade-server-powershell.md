<properties
	pageTitle="Durchführen eines Upgrades auf Azure SQL-Datenbank V12 mithilfe von PowerShell | Microsoft Azure"
	description="Erläutert das Upgrade auf Azure SQL-Datenbank V12, einschließlich der Aktualisierung von Web- und Business-Datenbanken, sowie das Upgrade eines V11-Servers mit direkter Migration der zugehörigen Datenbanken in einen Pool für elastische Datenbanken mithilfe von PowerShell."
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/19/2016"
	ms.author="sstein"/>

# Upgrade auf Azure SQL-Datenbank V12 mithilfe von PowerShell


> [AZURE.SELECTOR]
- [Azure-Portal](sql-database-upgrade-server-portal.md)
- [PowerShell](sql-database-upgrade-server-powershell.md)


SQL-Datenbank V12 ist die neueste Version, sodass eine Aktualisierung auf SQL-Datenbank V12 empfohlen wird. SQL-Datenbank V12 verfügt über viele [Vorteile gegenüber der vorherigen Version](sql-database-v12-whats-new.md), einschließlich:

- Höhere Kompatibilität mit SQL Server.
- Verbesserte Premium-Leistung und neue Leistungsstufen.
- [Pools für elastische Datenbanken](sql-database-elastic-pool.md).

Dieser Artikel enthält Anweisungen zum Aktualisieren von vorhandenen SQL-Datenbank V11-Servern und -Datenbanken auf SQL-Datenbank V12.

Bei der Aktualisierung auf V12 führen Sie ein Upgrade aller Web- und Business-Datenbanken auf eine neue Dienstebene durch, daher stehen Anleitungen für das Upgrade von Web- und Business-Datenbanken zur Verfügung.

Zudem kann eine Migration zu einem [Pool für elastische Datenbanken](sql-database-elastic-pool.md) kostengünstiger sein als die Aktualisierung auf einzelne Leistungsstufen (Tarife) für einzelne Datenbanken. Pools vereinfachen zudem das Datenbankmanagement, da Sie nur die Leistungseinstellungen für den Pool verwalten müssen, und nicht die Leistungsmerkmale der einzelnen Datenbanken. Wenn Sie über Datenbanken auf mehreren Servern verfügen, sollten Sie sie auf den gleichen Server verschieben und von der Zusammenfassung der Datenbanken in einem gemeinsamen Pool profitieren.

Führen Sie die Schritte in diesem Artikel aus, um Datenbanken von V11-Servern ganz einfach direkt in Pools für elastische Datenbanken zu migrieren.

Beachten Sie, dass Ihre Datenbanken während des gesamten Upgradevorgangs online bleiben und weiterhin funktionieren. Zum Zeitpunkt des tatsächlichen Übergangs auf die neue Leistungsstufe können die Datenbankverbindungen ggf. für einen kurzen Zeitraum – i. d. R. 90 Sekunden, eventuell jedoch bis zu 5 Minuten – unterbrochen werden. Wenn eine Anwendung über [Maßnahmen zur Behandlung vorübergehender Fehler bei Verbindungsunterbrechungen](sql-database-connectivity-issues.md) verfügt, ist es ausreichend, am Ende des Upgrades Schutz vor unterbrochenen Verbindungen vorzusehen.

Das Upgrade auf SQL-Datenbank V12 kann nicht rückgängig gemacht werden. Nach dem Upgrade kann der Server nicht auf V11 zurückgesetzt werden.

Nach dem Upgrade auf V12 stehen [Empfehlungen zur Dienstebene](sql-database-service-tier-advisor.md) und [Empfehlungen zu elastischen Pools](sql-database-elastic-pool-create-portal.md) erst dann zur Verfügung, wenn der Dienst Zeit für die Evaluierung der Workloads auf dem neuen Server hat. Der Empfehlungsverlauf für V11-Server gilt nicht für V12-Server und wird daher nicht beibehalten.

## Vorbereiten des Upgrades

- **Aktualisieren aller Web- und Business-Datenbanken**: Informationen hierzu finden Sie im Abschnitt [Durchführen eines Upgrades für alle Web- und Business-Datenbanken](sql-database-v12-upgrade.md#upgrade-all-web-and-business-databases) weiter unten, oder verwenden Sie [PowerShell zum Aktualisieren von Datenbanken und Server](sql-database-upgrade-server-powershell.md).
- **Überprüfen und Beenden der Georeplikation**: Wenn Ihre Azure SQL-Datenbank für Georeplikation konfiguriert ist, sollten Sie ihre aktuelle Konfiguration dokumentieren und die [Georeplikation beenden](sql-database-geo-replication-portal.md#remove-secondary-database). Nachdem das Upgrade abgeschlossen ist, können Sie Ihre Datenbank erneut für die Georeplikation konfigurieren.
- **Öffnen Sie folgende Ports, falls Sie Clients auf einer Azure-VM verwenden**: Wenn Ihr Clientprogramm eine Verbindung mit SQL-Datenbank V12 herstellt und der Client auf einem virtuellen Azure-Computer ausgeführt wird, müssen Sie die Portbereiche 11000 – 11999 und 14000 – 14999 auf dem virtuellen Computer öffnen. Weitere Informationen finden Sie unter [Ports für SQL-Datenbank V12](sql-database-develop-direct-route-ports-adonet-v12.md).


## Voraussetzungen

Für das Upgrade eines Servers auf V12 mithilfe von PowerShell muss die aktuelle Version von Azure PowerShell installiert sein und ausgeführt werden. Weitere Informationen finden Sie unter [Installieren und Konfigurieren von Azure PowerShell](../powershell-install-configure.md).


## Konfigurieren der Anmeldeinformationen und Auswählen des Abonnements

Um PowerShell-Cmdlets für Ihr Azure-Abonnement ausführen zu können, müssen Sie zunächst den Zugriff auf Ihr Azure-Konto herstellen. Führen Sie das folgende Cmdlet aus. Es wird eine Anmeldeseite angezeigt, auf der Sie Ihre Anmeldeinformationen eingeben müssen. Verwenden Sie die E-Mail-Adresse und das Kennwort für die Anmeldung beim Azure-Portal.

	Add-AzureRmAccount

Nach der erfolgreichen Anmeldung sollten einige Informationen auf dem Bildschirm angezeigt werden, wie die ID, mit der Sie sich angemeldet haben, und die Azure-Abonnements, auf die Sie zugreifen können.

Zur Auswahl des Abonnements, mit dem Sie arbeiten möchten, benötigen Sie Ihre Abonnement-ID (**-SubscriptionId**) oder den Abonnementnamen (**-SubscriptionName**). Sie können diese Informationen aus dem vorherigen Schritt kopieren. Wenn Sie über mehrere Abonnements verfügen, können Sie diese über das Cmdlet **Get-AzureRmSubscription** abrufen und die gewünschten Abonnementinformationen aus dem Resultset kopieren.

Führen Sie das folgende Cmdlet mit den Informationen zu Ihrem Abonnement aus, um Ihr aktuelles Abonnement festlegen:

	Set-AzureRmContext -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

Die folgenden Befehle werden für das soeben ausgewählte Abonnement ausgeführt.

## Empfehlungen erhalten

Zum Abrufen der Empfehlung für das Serverupgrade führen Sie das folgende Cmdlet aus:

    $hint = Get-AzureRmSqlServerUpgradeHint -ResourceGroupName “resourcegroup1” -ServerName “server1”

Weitere Informationen finden Sie unter [Erstellen eines Pools für elastische Datenbanken](sql-database-elastic-pool-create-portal.md) und [Tarifempfehlungen für Azure SQL-Datenbank](sql-database-service-tier-advisor.md).



## Starten des Upgrades

Zum Starten des Upgrades des Servers führen Sie das folgende Cmdlet aus:

    Start-AzureRmSqlServerUpgrade -ResourceGroupName “resourcegroup1” -ServerName “server1” -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


Mit dem Ausführen dieses Befehls beginnt der Upgradevorgang. Sie können die Ausgabe der Empfehlung anpassen und diesem Cmdlet die bearbeitete Empfehlung bereitstellen.


## Aktualisieren eines Servers


    # Adding the account
    #
    Add-AzureRmAccount

    # Setting the variables
    #
    $SubscriptionName = 'YOUR_SUBSCRIPTION'
    $ResourceGroupName = 'YOUR_RESOURCE_GROUP'
    $ServerName = 'YOUR_SERVER'

    # Selecting the right subscription
    #
    Set-AzureRmContext -SubscriptionName $SubscriptionName

    # Getting the upgrade recommendations
    #
    $hint = Get-AzureRmSqlServerUpgradeHint -ResourceGroupName $ResourceGroupName -ServerName $ServerName

    # Starting the upgrade process
    #
    Start-AzureRmSqlServerUpgrade -ResourceGroupName $ResourceGroupName -ServerName $ServerName -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


## Benutzerdefinierte Upgradezuordnung

Wenn die Vorschläge für Ihren Server und das Geschäftsszenario nicht geeignet sind, können Sie auswählen, wie das Upgrade Ihrer Datenbanken durchgeführt wird und sie entweder einzelnen oder elastischen Datenbanken zuordnen.

Die Parameter "ElasticPoolCollection" und "DatabaseCollection" sind optional:

    # Creating elastic pool mapping
    #
    $elasticPool = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeRecommendedElasticPoolProperties
    $elasticPool.DatabaseDtuMax = 100
    $elasticPool.DatabaseDtuMin = 0
    $elasticPool.Dtu = 800
    $elasticPool.Edition = "Standard"
    $elasticPool.DatabaseCollection = ("DB1", “DB2”, “DB3”, “DB4”)
    $elasticPool.Name = "elasticpool_1"
    $elasticPool.StorageMb = 800

    # Creating single database mapping for 2 databases. DBMain1 mapped to S0 and DBMain2 mapped to S2
    #
    $databaseMap1 = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeDatabaseProperties
    $databaseMap1.Name = "DBMain1"
    $databaseMap1.TargetEdition = "Standard"
    $databaseMap1.TargetServiceLevelObjective = "S0"

    $databaseMap2 = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeDatabaseProperties
    $databaseMap2.Name = "DBMain2"
    $databaseMap2.TargetEdition = "Standard"
    $databaseMap2.TargetServiceLevelObjective = "S2"

    # Starting the upgrade
    #
    Start-AzureRmSqlServerUpgrade –ResourceGroupName resourcegroup1 –ServerName server1 -ServerVersion 12.0 -DatabaseCollection @($databaseMap1, $databaseMap2) -ElasticPoolCollection @($elasticPool)



## Überwachen von Datenbanken nach dem Upgrade auf SQL-Datenbank V12


Nach dem Upgrade wird empfohlen, die Datenbank aktiv zu überwachen, um sicherzustellen, dass Anwendungen mit der gewünschten Leistung ausgeführt werden. Außerdem sollte die Nutzung optimiert werden.

Zusätzlich zur Überwachung einzelner Datenbanken können Sie [mithilfe des Portals](sql-database-elastic-pool-manage-portal.md) oder mit [PowerShell](sql-database-elastic-pool-manage-powershell.md) auch Pools für elastische Datenbanken überwachen.


**Ressourcenverbrauchsdaten:** Für Basic-, Standard- und Premium-Datenbanken sind Ressourcenverbrauchsdaten über die dynamische Verwaltungssicht (DMV) [sys.dm_ db_ resource\_stats](http://msdn.microsoft.com/library/azure/dn800981.aspx) in der Benutzerdatenbank verfügbar. Diese dynamische Verwaltungssicht stellt Informationen zum Ressourcenverbrauch für die vorhergehende Betriebsstunde zur Verfügung, beinahe in Echtzeit und mit einer Genauigkeit von 15 Sekunden. Der prozentuale DTU-Verbrauch für ein Intervall wird als maximaler prozentualer Verbrauch von CPU-, E/A- und Protokollressourcen berechnet. Die folgende Abfrage berechnet den durchschnittlichen prozentualen DTU-Verbrauch während der letzten Stunde:

    SELECT end_time
    	 , (SELECT Max(v)
             FROM (VALUES (avg_cpu_percent)
                         , (avg_data_io_percent)
                         , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.dm_db_resource_stats
    ORDER BY end_time DESC;

Zusätzliche Überwachungsinformationen:

- [Leitfaden zur Azure SQL-Datenbankleistung für Einzeldatenbanken](http://msdn.microsoft.com/library/azure/dn369873.aspx)
- [Überlegungen zum Preis und zur Leistung eines Pools für elastische Datenbanken](sql-database-elastic-pool-guidance.md)
- [Überwachen der Azure SQL-Datenbank mit dynamischen Verwaltungssichten](sql-database-monitoring-with-dmvs.md)



**Warnungen**: Richten Sie im Azure-Portal „Warnungen“ ein, damit Sie benachrichtigt werden, wenn der DTU-Verbrauch für eine aktualisierte Datenbank eine bestimmte hohe Ebene erreicht. Datenbankwarnungen können im Azure-Portal für verschiedene Leistungsmetriken wie DTU, CPU, E/A und die Protokollierung eingerichtet werden. Navigieren Sie zu der Datenbank, und wählen Sie **Warnungsregeln** auf dem Blatt **Einstellungen** aus.

Sie können z. B. eine E-Mail-Benachrichtigung für den "DTU Prozentsatz" festlegen, wenn der durchschnittliche prozentuale DTU-Wert 75 % innerhalb der letzten 5 Minuten überschreitet. Weitere Informationen zur Konfiguration von Warnungsbenachrichtigungen finden Sie unter [Empfangen von Warnungsbenachrichtigungen](../azure-portal/insights-receive-alert-notifications.md).



## Nächste Schritte

- [Erstellen eines Pools für elastische Datenbanken](sql-database-elastic-pool-create-portal.md) und Hinzufügen einiger oder aller Datenbanken zum Pool.
- [Ändern der Dienstebene und Leistungsstufe Ihrer Datenbank](sql-database-scale-up.md).



## Verwandte Informationen

- [Get-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt603582.aspx)
- [Start-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt619403.aspx)
- [Stop-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt603589.aspx)

<!---HONumber=AcomDC_0921_2016-->