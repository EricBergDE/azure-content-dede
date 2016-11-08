<properties
   pageTitle="Operations Management Suite (OMS) overview | Microsoft Azure"
   description="Microsoft Operations Management Suite (OMS) is Microsoft's cloud-based IT management solution that helps you manage and protect your on-premises and cloud infrastructure.  This article identifies the different services included in OMS and provides links to their detailed content."
   services="operations-management-suite"
   documentationCenter=""
   authors="bwren"
   manager="jwhit"
   editor="tysonn" />
<tags
   ms.service="operations-management-suite"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/24/2016"
   ms.author="bwren" />

# Was ist die Operations Management Suite (OMS)?

Die Microsoft Operations Management Suite (OMS) ist Microsofts cloud-basierte IT-Management Lösung mit deren Hilfe On-Premises und Cloud Infrastrukturen verwatet und geschützt werden können.  Seit OMS als Cloud-Dienst implementiert wurde, lässt sich die Lösung schnell und einfach bereitstellen und mit minimalem Aufwand in die Infrastruktur implementieren.  Neue Funktionen werden automatisch bereitgestellt, so dass Wartungs- und Upgrade-Kosten eingespart werden können.

Zusätzlich zu den Diensten die OMS direkt bereitstellt, kann eine Integration mit bestehenden System Center Komponenten, wie dem System Center Operations Manager, erfolgen.  Durch diese Integration kann das bestehende Investment in Management-Lösungen erhalten bleiben und eine Erweiterung in die Cloud erfolgen.  System Center und OMS können gemeinsam arbeiten um eine vollständig hybride Management-Erfahrung zu ermöglichen.

Die nachfolgenden Abschnitte liefern einen Überblick über die verschiedenen Bereiche in denen OMS Mehrwerte liefern kann und in welchen Diensten diese enthalten sind.  Vor dem Einblick in die detaillierte Dokumentation der einzelnen OMS Komponenten, lohnt sich ein Blick in die OMS-Architektur um einen Überblick über die OMS Komponenten zu erhalten.


## ![Insight and Analytics](media/operations-management-suite-overview/icon-insight-analytics.png) Insights und Analytics

[Log Analytics](http://azure.microsoft.com/documentation/services/log-analytics) hilft Ihnen Log- und Performance-Daten, die durch das Betriebssystem oder Ihrer Applikationen erzeugt werden, zu sammeln, auszuwerten, zu durchsuchen und darauf zu reagieren.  Sie erhalten Echtzeit-Einblicke in ihr operatives Geschäft mittels der integrierten Suche und angepasster Dashboards.  Mit diesen Tools können sie Millionen Datensätze all Ihrer Workloads und Server unabhängig vom physischen Standort analysieren.

Sie können einfach Lösungen zu Log Analytics hinzufügen, die definieren, welche Daten gesammelt werden und mit welcher Logik diese auszuwerten sind.  Diese Lösungen können weitere Funktionen enthalten, die automatisch und mit geringer oder ohne zusätzliche Konfiguration an die Agents übermittelt werden.  Zusätzlich zu den Analyse-Tools die mit den jeweiligen Lösungen geliefert werden, besteht jederzeit die Möglichkeit angepasste Suchen über den gesamten Datenbestand durhzuführen um Zusammenhänge in Daten von Systemen und Appliaktionen zu ermitteln.


## ![Automation & Control](media/operations-management-suite-overview/icon-automation-control.png) Automation und Kontrolle

Azure Automation automatisiert administrative Prozesse mit [Runbooks](../automation/automation-runbook-types.md) die auf PowerShell basieren und in der Azure Cloud ausgeführt werden.  Runbooks können alle Produkte und Dienste erfassen, die mittels PowerShell verwaltbar sind, auch wenn diese in anderen Cloud Services, wie Amazon Web Services (AWS), betrieben werden.  Runbooks können zudem auch auf Servern in Ihrem lokalen Rechenzentrum ausgeführt werden um lokale Ressourcen zu verwalten.

Azure Automation stellt Configuration Management mit [PowerShell DSC](../automation/automation-dsc-overview.md) zur Verfügung.  Sie können DSC Ressourcen in Azure hosten, erstellen und verwalten und diese auf Cloud und On-Premises Systeme anwenden um deren Konfiguration zu beschreiben und automatisch anzuwenden.


## ![Protection and Recovery](media/operations-management-suite-overview/icon-protection-recovery.png) Schutz und Wiederherstellung

[Azure Backup](http://azure.microsoft.com/documentation/services/backup) schützt Ihre Applikationen und Daten und sichert diese für Jahre, ohne einen initialen Invest in eine Backup-Infrastruktur zu erfordern und bei geringen Betriebskosten.  It can backup data from physical and virtual Windows servers in addition to application workloads such as SQL Server and SharePoint.  It can also be used by System Center Data Protection Manager (DPM) to replicate protected data to Azure for redundancy and long term storage.

[Azure Site Recovery](http://azure.microsoft.com/documentation/services/site-recovery) unterstützt ihre Business Continuity und Disaster Recovery (BCDR) Strategie indem Replikation, Failover und Wiederherstellung von On-Premises Hyper-V VMs, VMware VMs und physischen Windows/Linux Servern orchestriert werden.  Sie können Maschinen in ein zweites Rechenzentrum replizieren oder ihr Rechenzentrum erweitern, indem Mschinen nach Azure repliziert werden.  Site Recovery ermöglicht ein Failover und die Wiederherstellung von Workloads ohne zusätzlichen Aufwand zur Verfügung. Es integriert sich in Disaster Recovery Mechanismen wie SQL Server AlwaysOn und ermöglicht ein Failover von Workloads die über diverse Maschinen verteilt laufen mit Hilfe von Wiederherstellungsplänen.


## ![OMS Security and Compliance](media/operations-management-suite-overview/icon-security-compliance.png) Sicherheit und Compliance
Sicherheit und Compliance hilft dabei Risiken in Ihrer Infrastruktur zu identifizieren, analysieren und zu beheben.  Diese Funktionen von OMS werden durch mehrere Lösungen in Log Analytics implementiert. Sie analysieren Log-Daten und Konfigurationen der Agents um die kontinuierliche Sicherheit der Umgebung sicherzustellen.

- Die [Security und Audit Lösung](oms-security-getting-started.md ) sammelt und analysiert Security Events auf verwalteten Systemen um ungewöhnliche Aktivitäten zu identifizieren.
- Die [Antimalware Lösung](log-analytics-malware.md ) berichtet über den Status des Antimalware Schutzes auf verwalteten Systemen.
- Die System Updates Lösung führt eine Analyse der Security Updates und anderer Updates auf Ihren verwalteten Systemen aus um Systeme zu identifzieren, welche ausstehende Updates haben.


## Nächste Schritte
- Lernen sie mehr über [Log Analytics](http://azure.microsoft.com/documentation/services/log-analytics).
- Lernen sie mehr über [Azure Automation](../automation/automation-intro.md).
- Lernen sie mehr über [Azure Backup](http://azure.microsoft.com/documentation/services/backup).
- Lernen sie mehr über [Azure Site Recovery](http://azure.microsoft.com/documentation/services/site-recovery).
