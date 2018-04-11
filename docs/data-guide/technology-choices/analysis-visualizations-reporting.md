---
title: Auswählen einer Technologie für die Datenanalyse und Berichterstellung
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 830c61bba64a6971c815330887e5cdcc4f2b5f56
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>Auswählen einer Technologie für die Datenanalyse in Azure

Ziel der meisten Big Data-Lösungen ist es, über Analysen und Berichte Einblicke in die Daten zu gewinnen. Beispiele hierfür sind vorkonfigurierte Berichte und Visualisierungen oder die interaktive Datenuntersuchung. 

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>Welche Möglichkeiten stehen Ihnen bei der Wahl der Technologie für die Datenanalyse zur Verfügung?

Je nach Ihren Anforderungen haben Sie für die Analyse, Visualisierung und Berichterstellung in Azure mehrere Optionen:

- [Power BI](/power-bi/)
- [Jupyter-Notebooks](https://jupyter.readthedocs.io/en/latest/index.html)
- [Zeppelin-Notebooks](https://zeppelin.apache.org/)
- [Microsoft Azure Notebooks](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

[Power BI](/power-bi/) ist eine Suite mit Business Analytics-Tools. Sie ermöglicht die Herstellung einer Verbindung mit Hunderten von Datenquellen und kann für Ad-hoc-Analysen verwendet werden. Die derzeit verfügbaren Datenquellen sind in [dieser Liste](/power-bi/desktop-data-sources) aufgeführt. Verwenden Sie [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/), um Power BI in Ihre eigenen Anwendungen zu integrieren, ohne dass eine zusätzliche Lizenzierung erforderlich ist.

Organisationen können Power BI verwenden, um Berichte zu erstellen und für die gesamte Organisation zu veröffentlichen. Jeder kann personalisierte Dashboards mit Integration von Governance und [Sicherheit](/power-bi/service-admin-power-bi-security) erstellen. Für Power BI wird [Azure Active Directory](/azure/active-directory/) (Azure AD) zum Authentifizieren von Benutzern eingesetzt, die sich am Power BI-Dienst anmelden. Die Power BI-Anmeldeinformationen werden jeweils verwendet, wenn ein Benutzer versucht, auf Ressourcen zuzugreifen, für die eine Authentifizierung erforderlich ist.

### <a name="jupyter-notebooks"></a>Jupyter-Notebooks 

[Jupyter-Notebooks](https://jupyter.readthedocs.io/en/latest/index.html) verfügen über eine browserbasierte Shell, mit deren Hilfe Data Scientists *Notebook*-Dateien erstellen können, die Python-, Scala- oder R-Code und Markdowntext enthalten. Dies ist eine effektive Möglichkeit zur Kollaboration, indem der Code und Ergebnisse in einem zentralen Dokument freigegeben und dokumentiert werden.

Die meisten Varianten von HDInsight-Clustern, z.B. Spark oder Hadoop, sind [mit Jupyter-Notebooks vorkonfiguriert](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels), um die Interaktion mit Daten und die Übermittlung von Aufträgen zur Verarbeitung zu ermöglichen. Je nach verwendetem Typ des HDInsight-Clusters werden einer oder mehrere Kernel für die Interpretation und Ausführung Ihres Codes bereitgestellt. Spark-Cluster in HDInsight verfügen beispielsweise über Spark-bezogene Kernel, aus denen Sie wählen können, um Python- oder Scala-Code mit dem Spark-Modul auszuführen.

Jupyter-Notebooks sind eine hervorragende Umgebung zum Analysieren, Visualisieren und Verarbeiten Ihrer Daten vor der Erstellung von anspruchsvolleren Visualisierungen mit einem BI- oder Berichterstellungstool wie Power BI.

### <a name="zeppelin-notebooks"></a>Zeppelin-Notebooks

[Zeppelin-Notebooks](https://zeppelin.apache.org/) sind eine weitere Option für eine browserbasierte Shell, wobei die Funktionalität mit Jupyter vergleichbar ist. Für einige HDInsight-Cluster sind [Zeppelin-Notebooks bereits vorkonfiguriert](/azure/hdinsight/spark/apache-spark-zeppelin-notebook). Wenn Sie aber einen Cluster vom Typ [HDInsight Interactive Query](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) (Hive LLAP) verwenden, ist [Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) derzeit das einzige Notebook, das Sie zum Ausführen von interaktiven Hive-Abfragen einsetzen können. Falls Sie einen [in die Domäne eingebundenen HDInsight-Cluster](/azure/hdinsight/domain-joined/apache-domain-joined-introduction) nutzen, sind Zeppelin-Notebooks außerdem der einzige Typ, bei dem Sie unterschiedliche Benutzeranmeldungen zuweisen können, um den Zugriff auf Notebooks und die zugrunde liegenden Hive-Tabellen zu steuern.

### <a name="microsoft-azure-notebooks"></a>Microsoft Azure Notebooks

[Azure Notebooks](https://notebooks.azure.com/) ist ein auf Jupyter-Notebooks basierender Onlinedienst, mit dem Data Scientists Jupyter-Notebooks in cloudbasierten Bibliotheken erstellen, ausführen und freigeben können. Azure Notebooks verfügt über Ausführungsumgebungen für Python 2, Python 3, F# und R und mehrere Diagrammerstellungsbibliotheken zum Visualisieren Ihrer Daten, z.B. ggplot, matplotlib, bokeh und seaborn.

Im Gegensatz zu Jupyter-Notebooks, die in einem HDInsight-Cluster ausgeführt werden und mit dem Standardspeicherkonto des Clusters verbunden sind, werden bei Azure Notebooks keine Daten bereitgestellt. Sie müssen auf unterschiedliche Arten [Daten laden](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb), z.B. Daten von einer Onlinequelle herunterladen, mit Azure-Blobs oder Table Storage interagieren, eine Verbindung mit einer SQL-Datenbank herstellen oder Daten mit dem Kopier-Assistenten für Azure Data Factory laden.

Hauptvorteile:

* Kostenloser Dienst: Sie benötigen kein Azure-Abonnement.
* Keine lokale Installation von Jupyter und den unterstützenden R- oder Python-Distributionen erforderlich: Ein Browser genügt.
* Verwalten Sie Ihre eigenen Onlinebibliotheken, und greifen Sie von einem beliebigen Gerät aus darauf zu.
* Geben Sie Ihre Notebooks für Projektmitarbeiter frei.

Zu berücksichtigende Aspekte:

* Die Notebooks stehen offline nicht zur Verfügung.
* Die eingeschränkten Verarbeitungsfunktionen des kostenlosen Notebookdiensts reichen für umfangreiche oder komplexe Modelle unter Umständen nicht aus.

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie zunächst die folgenden Fragen, um die Auswahlmöglichkeiten einzuschränken:

- Müssen Sie eine Verbindung mit einer großen Zahl von Datenquellen herstellen und einen zentralen Ort zur Erstellung von Berichten für Daten bereitstellen, die in Ihrer gesamten Domäne verteilt sind? Wenn ja, sollten Sie eine Option wählen, mit der Sie eine Verbindung mit Hunderten von Datenquellen herstellen können.

- Möchten Sie dynamische Visualisierungen in eine externe Website oder Anwendung einbetten? Wenn ja, sollten Sie eine Option wählen, die über Funktionen für das Einbetten verfügt.

- Möchten Sie Ihre Visualisierungen und Berichte im Offlinezustand entwerfen? Wenn ja, sollten Sie eine Option mit Offlinefunktionen wählen.

- Benötigen Sie eine hohe Verarbeitungsleistung, um große oder komplexe KI-Modelle zu trainieren oder mit sehr großen Datasets zu arbeiten? Wenn ja, sollten Sie eine Option wählen, bei der eine Verbindung mit einem Big Data-Cluster hergestellt werden kann.

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede in Bezug auf die Funktionen zusammengefasst: 

### <a name="general-capabilities"></a>Allgemeine Funktionen

| | Power BI | Jupyter-Notebooks | Zeppelin-Notebooks | Microsoft Azure Notebooks |
| --- | --- | --- | --- | --- |
| Verbindungsherstellung mit einem Big Data-Cluster zur erweiterten Verarbeitung | Ja | Ja | Ja | Nein  |
| Verwalteter Dienst | Ja | Ja <sup>1</sup> | Ja <sup>1</sup> | Ja |
| Verbindungsherstellung mit Hunderten von Datenquellen | Ja | Nein  | Nein  | Nein  |
| Offlinefunktionen | Ja <sup>2</sup> | Nein  | Nein  | Nein  |
| Einbettung von Funktionen | Ja | Nein  | Nein  | Nein  |
| Automatische Datenaktualisierung | Ja | Nein  | Nein  | Nein  |
| Zugriff auf eine große Zahl von Open-Source-Paketen | Nein  | Ja <sup>3</sup> | Ja <sup>3</sup> | Ja <sup>4</sup> |
| Optionen für Datentransformation/-bereinigung | [Power Query](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/), R | 40 Sprachen, z.B. Python, R, Julia und Scala | 20+ Interpreter, z.B. Python, JDBC und R | Python, F#, R |
| Preise | Kostenlos für Power BI Desktop (Erstellung), siehe Hostingoptionen unter [Preise](https://powerbi.microsoft.com/pricing/) | Kostenlos | Kostenlos | Kostenlos |
| Kollaboration mehrerer Benutzer | [Ja](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | Ja (per Freigabe oder über einen Mehrbenutzer-Server wie [JupyterHub](https://github.com/jupyterhub/jupyterhub)) | Ja | Ja (per Freigabe) |

[1] Bei Verwendung als Teil eines verwalteten HDInsight-Clusters.

[2] Bei Verwendung von Power BI Desktop.

[2] Sie können das [Maven-Repository](http://search.maven.org/) nach Paketen durchsuchen, die von der Community bereitgestellt wurden.

[3] Python-Pakete können entweder per pip oder conda installiert werden. R-Pakete können über CRAN oder GitHub installiert werden. Pakete in F# können über „nuget.org“ mit dem [Paket-Abhängigkeits-Manager](https://fsprojects.github.io/Paket/) installiert werden.

