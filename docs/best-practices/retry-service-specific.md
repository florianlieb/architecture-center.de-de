---
title: Anleitung zu dienstspezifischen Wiederholungsmechanismen
description: "Spezifische Dienstanleitung für die Festlegung des Wiederholungsmechanismus."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 0a416bc6297c7406de92fbc695b62c39c637de8f
ms.sourcegitcommit: 1c0465cea4ceb9ba9bb5e8f1a8a04d3ba2fa5acd
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/02/2018
---
# <a name="retry-guidance-for-specific-services"></a>Wiederholungsanleitung für bestimmte Dienste

Die meisten Azure-Dienste und Client-SDKs enthalten einen Wiederholungsmechanismus. Diese unterscheiden sich jedoch, da jeder Dienst unterschiedliche Merkmale und Anforderungen hat und somit jeder Wiederholungsmechanismus für einen bestimmten Dienst optimiert ist. Dieses Handbuch fasst die Wiederholungsmechanismusfunktionen für die Mehrzahl der Azure-Dienste zusammen und enthält Informationen, mit denen Sie die Wiederholungsmechanismus für diesen Dienst verwenden, anpassen oder erweitern können.

Allgemeine Anweisungen zum Behandeln von vorübergehenden Fehlern und für Wiederholungsverbindungen und Operationen für Dienste und Ressourcen finden Sie unter [Wiederholungsanleitung](./transient-faults.md).

In der folgende Tabelle werden die Wiederholungsfunktionen für die in dieser Anleitung beschriebenen Azure-Dienste zusammengefasst.

| **Service** | **Wiederholungsfunktionen** | **Richtlinienkonfiguration** | **Umfang** | **Telemetriefunktionen** |
| --- | --- | --- | --- | --- |
| **[Azure Storage](#azure-storage-retry-guidelines)** |Systemeigen in Client |Programmgesteuert |Clientvorgänge und einzelne Vorgänge |TraceSource |
| **[SQL-Datenbank mit Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)** |Systemeigen in Client |Programmgesteuert |Global pro AppDomain |Keine |
| **[SQL-Datenbank mit Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)** |Systemeigen in Client |Programmgesteuert |Global pro AppDomain |Keine |
| **[SQL-Datenbank mit ADO.NET](#sql-database-using-adonet-retry-guidelines)** |[Polly](#transient-fault-handling-with-polly) |Deklarativ und programmatisch |Einzelne Anweisungen oder Codeblöcke |Benutzerdefiniert |
| **[Service Bus](#service-bus-retry-guidelines)** |Systemeigen in Client |Programmgesteuert |Namespace-Manager, Messaging Factory und Client |ETW |
| **[Azure Redis Cache](#azure-redis-cache-retry-guidelines)** |Systemeigen in Client |Programmgesteuert |Client |TextWriter |
| **[DocumentDB-API](#documentdb-api-retry-guidelines)** |Systemeigen im Dienst |Nicht konfigurierbar |Global |TraceSource |
| **[Azure Search](#azure-storage-retry-guidelines)** |Systemeigen in Client |Programmgesteuert |Client |ETW oder benutzerdefiniert |
| **[Azure Active Directory](#azure-active-directory-retry-guidelines)** |Nativ in ADAL-Bibliothek |Eingebettet in ADAL-Bibliothek |Intern |Keine |
| **[Service Fabric](#service-fabric-retry-guidelines)** |Systemeigen in Client |Programmgesteuert |Client |Keine | 
| **[Azure Event Hubs](#azure-event-hubs-retry-guidelines)** |Systemeigen in Client |Programmgesteuert |Client |Keine |

> [!NOTE]
> Für die meisten der integrierten Azure Mechanismen gibt es derzeit keine Möglichkeit, unterschiedliche Wiederholungsrichtlinien für verschiedene Typen von Fehler oder Ausnahmen anzuwenden, die über die in der Wiederholungsrichtlinie integrierte Funktionalität hinausgeht. Daher ist die beste gegenwärtige Anweisung zum Redaktionszeitpunkt eine Richtlinie zu konfigurieren, die die optimale durchschnittliche Leistung und Verfügbarkeit bietet. Eine Möglichkeit zur Optimierung der Richtlinie ist die Analyse von Protokolldateien, um den Typ der vorübergehenden Fehler zu bestimmen, die auftreten. Wenn z. B. die meisten Fehler mit der Netzwerkkonnektivität verknüpft sind, können Sie eine sofortige Wiederholung versuchen, anstatt langer für die erste Wiederholung zu warten.
>
>

## <a name="azure-storage-retry-guidelines"></a>Azure Storage Wiederholungsrichtlinien
Azure-Speicherdienste umfassen Tabellen- und Blob-Speicher, Dateien und Speicher-Warteschlangen.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
Wiederholungen treten auf individueller REST-Vorgangsebene auf und sind ein wesentlicher Bestandteil der Client-API-Implementierung. Das Client-Speicher SDK verwendet Klassen, die die [IExtendedRetryPolicy Schnittstelle](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx)implementieren.

Es gibt verschiedene Implementierungen der Schnittstelle. Speicherclients können Richtlinien auswählen, die speziell für den Zugriff auf Tabellen, Blobs und Warteschlangen entworfen wurden. Jede Implementierung verwendet eine andere Wiederholungsstrategie, die im Wesentlichen das Wiederholungsintervall und andere Details definiert.

Die integrierten Klassen bieten Unterstützung für linear (konstante Verzögerung) und exponentiell mit Zufallsgenerator-Wiederholungsintervallen. Es gibt auch eine Richtlinie für keine Wiederholung, wenn ein anderer Prozess Wiederholungen auf höherer Ebene behandelt. Allerdings können Sie Ihre eigenen Wiederholungsklassen implementieren, wenn Sie spezielle Anforderungen haben, die von den integrierten Klassen nicht bereitgestellt werden.

Abwechselnde Wiederholungen wechseln zwischen primären und sekundären Speicherdienststandorten, wenn Sie Lesezugriff auf georedundante Speicher (RA-GRS) verwenden und das Ergebnis der Anforderung ist ein wiederholbarer Fehler. Weitere Informationen finden Sie unter [Redundanzoptionen für Azure Storage](http://msdn.microsoft.com/library/azure/dn727290.aspx) .

### <a name="policy-configuration"></a>Richtlinienkonfiguration
Wiederholungsrichtlinien werden programmgesteuert konfiguriert. Eine typische Vorgehensweise ist das Erstellen und Ausfüllen von **TableRequestOptions**-, **BlobRequestOptions**-, **FileRequestOptions**- oder **QueueRequestOptions**-Instanzen.

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

Die Anforderungsoptioneninstanz kann dann auf dem Client festgelegt werden und alle Operationen mit dem Client verwenden die angegebene Anforderungsoptionen.

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

Sie können die Client-Anforderungsoptionen überschreiben, indem Sie eine ausgefüllte Instanz der Anforderungsoptionsklasse als Parameter an Vorgangsmethoden übergeben.

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

Sie verwenden eine **OperationContext** -Instanz, um den auszuführenden Code anzugeben, wenn eine Wiederholung auftritt und ein Vorgang abgeschlossen wurde. Dieser Code kann Informationen über den Vorgang zur Verwendung in Protokollen und Telemetrie erfassen.

    // Set up notifications for an operation
    var context = new OperationContext();
    context.ClientRequestID = "some request id";
    context.Retrying += (sender, args) =>
    {
      /* Collect retry information */
    };
    context.RequestCompleted += (sender, args) =>
    {
      /* Collect operation completion information */
    };
    var stats = await client.GetServiceStatsAsync(null, context);

Zusätzlich zur Angabe, ob ein Fehler für die Wiederholung geeignet ist, geben die erweiterten Wiederholungsrichtlinien ein **RetryContext** -Objekt zurück, das folgendes angibt: die Anzahl der Wiederholungen, die Ergebnisse der letzten Anforderung, ob die nächste Wiederholung im primären oder sekundären Standort ausgeführt werden (siehe Tabelle unten). Die Eigenschaften des **RetryContext** Objekts können verwendet werden, um zu entscheiden, ob und wann eine Wiederholung versucht wird. Weitere Informationen finden Sie unter [IExtendedRetryPolicy.Evaluate-Methode](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).

Die folgenden Tabellen enthalten die Standardeinstellungen für die integrierten Wiederholungsrichtlinien.

**Anforderungsoptionen**

| **Einstellung** | **Standardwert** | **Bedeutung** |
| --- | --- | --- |
| MaximumExecutionTime | 120 Sekunden | Maximale Ausführungszeit für die Anforderung, einschließlich aller möglichen Wiederholungsversuche. |
| ServerTimeout | Keine | Server-Timeout-Intervall für die Anforderung (der Wert wird in Sekunden gerundet). Wenn nicht angegeben, wird der Standardwert für alle Anforderungen an den Server verwendet. In der Regel ist die beste Option, diese Einstellung auszulassen, sodass die Standardeinstellung des Servers verwendet wird. | 
| LocationMode | Keine | Wenn das Speicherkonto mit der Replikationsoption des Lesezugriffs auf georedundanten Speicher (RA-GRS) erstellt wird, können Sie mit dem Speicherortmodus bestimmen, welche Stelle die Anforderung erhalten soll. Wenn zum Beispiel **PrimaryThenSecondary** angegeben ist, werden Anforderungen immer zuerst an den primären Standort gesendet. Wenn eine Anforderung fehlschlägt, wird sie an den sekundären Standort gesendet. |
| RetryPolicy | ExponentialPolicy | Details zu jeder Option finden Sie weiter unten. |

**Exponentielle Richtlinie** 

| **Einstellung** | **Standardwert** | **Bedeutung** |
| --- | --- | --- |
| maxAttempt | 3 | Anzahl der Wiederholungsversuche. |
| deltaBackoff | 4 Sekunden | Backoff-Intervall zwischen den Wiederholungen. Vielfache dieser Zeitspanne, einschließlich eines Zufallselements, werden für weitere Wiederholungsversuche verwendet. |
| MinBackoff | 3 Sekunden | Wird zu allen Wiederholungsintervallen, die aus deltaBackoff berechnet werden, hinzugefügt. Dieser Wert kann nicht geändert werden.
| MaxBackoff | 120 Sekunden | MaxBackoff wird verwendet, wenn das berechnete Wiederholungsintervall größer als MaxBackoff ist. Dieser Wert kann nicht geändert werden. |

**Lineare Richtlinie**

| **Einstellung** | **Standardwert** | **Bedeutung** |
| --- | --- | --- |
| maxAttempt | 3 | Anzahl der Wiederholungsversuche. |
| deltaBackoff | 30 Sekunden | Backoff-Intervall zwischen den Wiederholungen. |

### <a name="retry-usage-guidance"></a>Gebrauchsanleitung Wiederholungen
Beachten Sie die folgenden Richtlinien beim Zugriff auf Azure-Speicherdienste mit der Speicherclient-API:

* Verwenden der integrierten Wiederholungsrichtlinien aus dem Microsoft.WindowsAzure.Storage.RetryPolicies-Namespace, wo sie für Ihre Anforderungen geeignet sind. In den meisten Fällen sind diese Richtlinien ausreichend.
* Verwenden der **ExponentialRetry** Richtlinie in Batchvorgängen, Hintergrundaufgaben oder nicht interaktiven Szenarios. In diesen Szenarien können Sie in der Regel dem Dienst mehr Zeit für die Wiederherstellung  zur Verfügung stellen - dadurch erhöht sich die Wahrscheinlichkeit, dass der Vorgang schließlich erfolgreich ist.
* Geben Sie die Eigenschaft **MaximumExecutionTime** des **RequestOptions**-Parameters an, um die Gesamtausführungszeit zu beschränken. Berücksichtigen Sie bei der Auswahl eines Timeoutwerts aber den Typ und die Größe des Vorgangs.
* Wenn Sie eine benutzerdefinierte Wiederholung implementieren müssen, vermeiden Sie das Erstellen von Wrappern um die Speicherclienten-Klassen. Verwenden Sie stattdessen die Funktionen zum Erweitern der vorhandenen Richtlinien über die **IExtendedRetryPolicy** Schnittstelle.
* Bei Verwendung des Lesezugriffs auf georedundante Speicher (RA-GRS) können Sie **LocationMode** verwenden, um zu bestimmen, dass Wiederholungsversuche nicht auf die sekundäre schreibgeschützte Kopie des Speichers zugreifen sollen, wenn der primäre Zugriff fehlschlägt. Bei Verwendung dieser Option müssen Sie jedoch sicherstellen, dass die Anwendung erfolgreich mit Daten arbeiten kann, die möglicherweise veraltet sind, wenn die Replikation vom primären Speicher noch nicht abgeschlossen wurde.

Erwägen Sie, mit den folgenden Einstellungen für Wiederholungsvorgänge zu beginnen. Hierbei handelt es sich um allgemeine Einstellungen und Sie sollten die Vorgänge überwachen und die Werte entsprechend Ihrem Szenario optimieren.  

| **Context** | **Beispiel-Ziel E2E<br />Maximale Wartezeit** | **Wiederholungsrichtlinie** | **Einstellungen** | **Werte** | **So funktioniert's** |
| --- | --- | --- | --- | --- | --- |
| Interaktiv, Benutzeroberfläche<br />oder Vordergrund |2 Sekunden |Linear |maxAttempt<br />deltaBackoff |3<br />500 ms |Versuch 1 – Verzögerung 500 ms<br />Versuch 2 – Verzögerung 500 ms<br />Versuch 3 – Verzögerung 500 ms |
| Hintergrund<br />oder Batch |30 Sekunden |Exponentiell |maxAttempt<br />deltaBackoff |5<br />4 Sekunden |Versuch 1 – Verzögerung ca. 3 Sek.<br />Versuch 2 – Verzögerung ca. 7 Sek.<br />Versuch 3 – Verzögerung ca. 15 Sek. |

### <a name="telemetry"></a>Telemetrie
Wiederholungsversuche werden in einer **TraceSource** protokolliert. Sie müssen einen **TraceListener** konfigurieren, um die Ereignisse zu erfassen und diese in ein geeignetes Zielprotokoll zu schreiben. Sie können **TextWriterTraceListener** oder **XmlWriterTraceListener** zum Schreiben der Daten in eine Protokolldatei, **EventLogTraceListener** zum Schreiben in ein Windows-Ereignisprotokoll oder **EventProviderTraceListener** zum Schreiben von Daten in das ETW-Subsystem verwenden. Sie können auch automatisches Leeren des Puffers und den Ausführlichkeitsgrad der Ereignisse konfigurieren, die protokolliert werden (z. B. Fehler, Warnung, Information und ausführlich). Weitere Informationen finden Sie unter [Clientseitige Protokollierung mit der .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).

Vorgänge können eine **OperationContext**-Instanz erhalten, die ein **Retrying**-Ereignis verfügbar macht, das zum Anfügen von benutzerdefinierter Telemetrielogik verwendet werden kann. Weitere Informationen finden Sie unter [OperationContext.Ereignis Erneuter Versuch](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).

### <a name="examples"></a>Beispiele
Im folgenden Codebeispiel wird veranschaulicht, wie Sie zwei **TableRequestOptions** -Instanzen mit verschiedenen Wiederholungseinstellungen erstellen; eine für interaktive Anforderungen und eine für Anforderungen im Hintergrund. Im Beispiel werden dann diese beiden Richtlinien auf dem Client festgelegt, sodass sie für alle Anforderungen gelten und außerdem wird die interaktive Strategie für eine bestimmte Anforderung festgelegt, damit sie die auf den Client angewandten Standardeinstellungen überschreibt.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a>Weitere Informationen
* [Empfehlungen zur Azure Storage-Clientbibliothek Wiederholungsrichtlinie](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [Storage Client Library 2.0 – Implementieren von Wiederholungsrichtlinien](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="sql-database-using-entity-framework-6-retry-guidelines"></a>SQL-Datenbank mit Entity Framework 6 Wiederholungsrichtlinien
SQL-Datenbank ist eine gehostete SQL-Datenbank, die in unterschiedlichen Größen und als Standard (freigegeben) und Premium (nicht freigegebenen)-Dienst verfügbar ist. Entity Framework ist eine objektrelationale Zuordnung, die .NET Entwicklern die  Arbeit mit relationalen Daten mithilfe von domänenspezifischen Objekten ermöglicht. Es entfällt die Notwendigkeit für den Großteil des Datenzugriffs-Codes, den Entwickler normalerweise schreiben müssen.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
Beim Zugriff auf SQL-Datenbank mit Entity Framework 6.0 und höher über einen Mechanismus namens [Verbindungsstabilität / Wiederholungslogik](http://msdn.microsoft.com/data/dn456835.aspx)wird Wiederholungsunterstützung geboten. Die wichtigsten Features des Wiederholungsmechanismus sind:

* Die primäre Abstraktion ist die **IDbExecutionStrategy** -Schnittstelle. Diese Benutzeroberfläche:
  * Definiert die synchrone und asynchrone Methoden zum **Ausführen***.
  * Definiert Klassen, die direkt verwendet oder in einem Datenbankkontext als eine Standardstrategie konfiguriert werden können, einem Anbieternamen oder einem Anbieternamen und Servernamen zugeordnet werden können. Wenn sie für einen Kontext konfiguriert ist, treten Wiederholungen auf der Ebene der einzelnen Datenbankvorgängen auf, von denen es möglicherweise mehrere für einen angegebenen Kontext gibt.
  * Definiert, wann und wie versucht wird, einen fehlgeschlagene Verbindung erneut herzustellen.
* Sie umfasst mehrere integrierte Implementierungen für die **IDbExecutionStrategy** -Schnittstelle:
  * Standard - keine Wiederholung.
  * Standard für SQL-Datenbank (automatisch) - kein erneuter Versuch, es werden jedoch Ausnahmen überprüft und in der SQL-Datenbank-Strategie mit der Empfehlung zur Verwendung eingebunden.
  * Standardeinstellung für SQL-Datenbank - exponentiell (von der Basisklasse geerbt) plus SQL-Datenbank Erkennungslogik.
* Sie implementiert eine exponentielle Backoff-Strategie, die einen Zufallsgenerator enthält.
* Die integrierten Wiederholungsklassen sind zustandsbehaftet und nicht threadsicher. Sie können jedoch wiederverwendet werden, nachdem der aktuelle Vorgang abgeschlossen ist.
* Wenn die angegebene Wiederholungsanzahl überschritten wird, werden die Ergebnisse in einer neue Ausnahme umschlossen. Sie bringt die aktuelle Ausnahme nicht zum Vorschein.

### <a name="policy-configuration"></a>Richtlinienkonfiguration
Beim Zugriff auf SQL-Datenbank mit Entity Framework 6.0 und höher wird Wiederholungsunterstützung  geboten. Wiederholungsrichtlinien werden programmgesteuert konfiguriert. Die Konfiguration kann nicht pro Vorgang geändert werden.

Wenn Sie eine Strategie im Kontext als Standard konfigurieren, geben Sie eine Funktion an, die bei Bedarf eine neue Strategie erstellt. Der folgende Code zeigt, wie Sie eine Wiederholungskonfigurationsklasse erstellen können, die die **DbConfiguration** -Basisklasse erweitert.

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

Sie können diese dann mit der **SetConfiguration**-Methode der **DbConfiguration**-Instanz als Standardwiederholungsstrategie für alle Vorgänge festlegen, wenn die Anwendung gestartet wird. In der Standardeinstellung erkennt EF die Konfigurationsklasse automatisch und verwenden diese.

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

Sie können die Wiederholungskonfigurationsklasse für einen Kontext angeben, indem Sie die Kontextklasse mit einem **DbConfigurationType** -Attribut kommentieren. Wenn Sie jedoch über nur eine Konfigurationsklasse verfügen, wird EF diese verwenden, ohne dass der Kontext kommentieren werden muss.

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

Wenn Sie unterschiedliche Wiederholungsstrategien für bestimmte Vorgänge verwenden oder Wiederholungen für bestimmte Vorgänge deaktivieren müssen, können Sie eine Konfigurationsklasse erstellen, mit der Sie Strategien anhalten oder  austauschen können, indem ein Flag in **CallContext**setzen. Die Konfigurationsklasse kann dieses Flag verwenden, um Strategien zu wechseln oder die Strategie zu deaktivieren, die Sie bereitstellen und eine Standardstrategie verwenden. Weitere Informationen finden Sie unter [Ausführungsstrategie anhalten](http://msdn.microsoft.com/dn307226#transactions_workarounds) auf der Seite Ausführungsstrategien wiederholen (EF6 oder höher).

Eine andere Technik für die Verwendung von bestimmten Wiederholungsstrategien für einzelne Vorgänge ist das Erstellen einer Instanz der erforderlichen Strategieklasse und das Liefern der gewünschten Einstellungen mithilfe von Parametern. Rufen Sie dann die **ExecuteAsync** -Methode auf.

    var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
    var blogs = await executionStrategy.ExecuteAsync(
        async () =>
        {
            using (var db = new BloggingContext("Blogs"))
            {
                // Acquire some values asynchronously and return them
            }
        },
        new CancellationToken()
    );

Die einfachste Möglichkeit zum Verwenden einer **DbConfiguration**-Klasse ist es, diese in der gleichen Assembly wie die **DbContext**-Klasse zu platzieren. Dies ist jedoch nicht geeignet, wenn der gleiche Kontext in verschiedenen Szenarios benötigt wird, wie z. B. bei anderen interaktiven Strategien und Hintergrundwiederholungsstrategien. Wenn die unterschiedlichen Kontexte in separaten AppDomains ausgeführt werden, können Sie die integrierte Unterstützung für das Angeben von Konfigurationsklassen in der Konfigurationsdatei verwenden oder explizit mithilfe von Code festlegen. Wenn die unterschiedlichen Kontexte in derselben AppDomain ausgeführt werden müssen, ist eine benutzerdefinierte Lösung erforderlich.

Weitere Informationen finden Sie unter [Codebasierte Konfiguration (EF6 oder höher)](http://msdn.microsoft.com/data/jj680699.aspx).

Die folgende Tabelle zeigt die Standardeinstellungen für die integrierte Wiederholungsrichtlinie bei Verwendung von EF6.

| Einstellung | Standardwert | Bedeutung |
|---------|---------------|---------|
| Richtlinie | Exponentiell | Exponentielles Backoff. |
| MaxRetryCount | 5 | Die maximale Anzahl von Warnungen. |
| MaxDelay | 30 Sekunden | Die maximale Verzögerung zwischen Wiederholungsversuchen. Dieser Wert hat keine Auswirkung auf die Berechnung der Verzögerungsreihen. Er definiert lediglich eine Obergrenze. |
| DefaultCoefficient | 1 Sekunde | Der Koeffizient für die Berechnung des exponentiellen Backoffs. Dieser Wert kann nicht geändert werden. |
| DefaultRandomFactor | 1.1 | Der Multiplikator zum Hinzufügen einer beliebigen Verzögerung für die einzelnen Einträge. Dieser Wert kann nicht geändert werden. |
| DefaultExponentialBase | 2 | Der Multiplikator für die Berechnung der nächsten Verzögerung. Dieser Wert kann nicht geändert werden. |

### <a name="retry-usage-guidance"></a>Gebrauchsanleitung Wiederholungen
Beachten Sie den Zugriff auf SQL-Datenbank mit EF6 die folgenden Richtlinien:

* Wählen Sie die entsprechende Dienstoption (freigegeben oder Premium). Bei einer freigegebenen Instanz kann es möglicherweise zu längeren Verbindungsverzögerungen als üblich und Drosselung durch die Verwendung des gemeinsam genutzten Servers durch andere Mandanten kommen. Wenn Vorgänge mit vorhersagbarer Leistung und zuverlässig geringe Latenz erforderlich sind, sollten Sie die Premium-Option wählen.
* Eine Strategie mit festgelegtem Intervall wird für die Verwendung mit Azure SQL-Datenbank nicht empfohlen. Verwenden Sie stattdessen eine exponentielle Backoff-Strategie, da der Dienst möglicherweise überlastet ist und längere Verzögerungen mehr Zeit für die Wiederherstellung einräumen.
* Wählen Sie einen geeigneten Wert für die Verbindungs- und Befehls-Timeouts bei der Definition von Verbindungen. Stützen Sie das Timeout auf Ihren Geschäftslogikentwurf und auf Tests. Sie müssen diesen Wert möglicherweise mit der Zeit ändern, wenn sich die Datenmengen oder Geschäftsprozesse ändern. Ein zu kurzes Timeout kann zu vorzeitigen Fehlern bei Verbindungen führen, wenn die Datenbank ausgelastet ist. Ein zu langes Timeout kann verhindern, dass die Wiederholungslogik ordnungsgemäß funktioniert, da sie vor der Erkennung einer fehlgeschlagenen Verbindung zu lange wartet. Der Wert für das Timeout ist eine Komponente der End-to-End-Latenz, auch es nicht leicht ist zu ermitteln, wie viele Befehle beim Speichern des Kontexts ausgeführt werden. Sie können das Standardtimeout ändern, indem Sie die Einstellung der **CommandTimeout**-Eigenschaft der **DbContext**-Instanz festlegen.
* Entity Framework unterstützt Wiederholungskonfigurationen, die in Konfigurationsdateien definiert sind. Für maximale Flexibilität in Azure sollten Sie allerdings die Konfiguration in der Anwendung  programmgesteuert erstellen. Die einzelnen Parameter für die Wiederholungsrichtlinien, wie z. B. die Anzahl der Wiederholungen und die Wiederholungsintervalle können in der Dienstkonfigurationsdatei gespeichert und zur Laufzeit verwendet werden, um die entsprechenden Richtlinien zu erstellen. Dadurch können die Einstellungen darin geändert werden, ohne dass die Anwendung neu gestartet werden muss.

Erwägen Sie, mit den folgenden Einstellungen für Wiederholungsvorgänge zu beginnen. Sie können die Verzögerung zwischen den Wiederholungen nicht angeben (sie ist als eine exponentielle Sequenz festgelegt und generiert). Sofern Sie keine benutzerdefinierte Wiederholungsstrategie erstellen, können Sie wie hier gezeigt nur die maximalen Werte angeben. Hierbei handelt es sich um allgemeine Einstellungen und Sie sollten die Vorgänge überwachen und die Werte entsprechend Ihrem Szenario optimieren.

| **Context** | **Beispiel-Ziel E2E<br />Maximale Wartezeit** | **Wiederholungsrichtlinie** | **Einstellungen** | **Werte** | **So funktioniert's** |
| --- | --- | --- | --- | --- | --- |
| Interaktiv, Benutzeroberfläche<br />oder Vordergrund |2 Sekunden |Exponentiell |MaxRetryCount<br />MaxDelay |3<br />750 ms |Versuch 1 – Verzögerung 0 Sek.<br />Versuch 2 – Verzögerung 750 ms<br />Versuch 3 - Verzögerung 750 ms |
| Hintergrund<br /> oder Batch |30 Sekunden |Exponentiell |MaxRetryCount<br />MaxDelay |5<br />12 Sekunden |Versuch 1 – Verzögerung 0 Sek.<br />Versuch 2 – Verzögerung ca. 1 Sek.<br />Versuch 3 – Verzögerung ca. 3 Sek.<br />Versuch 4 – Verzögerung ca. 7 Sek.<br />Versuch 5 – Verzögerung ca. 12 Sek. |

> [!NOTE]
> Die End-to-End-Latenzziele setzen das Standardtimeout für Verbindungen mit dem Dienst voraus. Wenn Sie längere Verbindungstimeouts angeben, wird die End-to-End-Latenz durch diese zusätzliche Zeit für jeden Wiederholungsversuch erweitert.
>
>

### <a name="examples"></a>Beispiele
Im folgenden Codebeispiel wird eine einfachen Datenzugriffslösung definiert, die Entity Framework verwendet. Eine bestimmte Wiederholungsstrategie wird durch die Definition einer Instanz einer Klasse mit dem Namen **BlogConfiguration** festgelegt, die **DbConfiguration** erweitert.

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

Weitere Beispiele zur Verwendung des Entity Framework Wiederholungsmechanismus finden Sie unter [Verbindungsstabilität / Wiederholungslogik](http://msdn.microsoft.com/data/dn456835.aspx).

### <a name="more-information"></a>Weitere Informationen
* [Anleitung zu Leistung und Flexibilität von Azure SQL-Datenbanken](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core-retry-guidelines"></a>SQL-Datenbank mit Entity Framework Core – Wiederholungsrichtlinien
[Entity Framework Core](/ef/core/) ist eine objektrelationale Zuordnung, die .NET Core-Entwicklern die Arbeit mit Daten mithilfe von domänenspezifischen Objekten ermöglicht. Es entfällt die Notwendigkeit für den Großteil des Datenzugriffs-Codes, den Entwickler normalerweise schreiben müssen. Diese Version von Entity Framework wurde von Grund auf neu geschrieben und erbt nicht automatisch alle Features von EF6.x.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
Beim Zugriff auf SQL-Datenbank mit Entity Framework Core wird über einen Mechanismus mit dem Namen [Verbindungsstabilität](/ef/core/miscellaneous/connection-resiliency) die Wiederholungsunterstützung ermöglicht. Verbindungsstabilität wurde mit EF Core 1.1.0 eingeführt.

Die primäre Abstraktion ist die `IExecutionStrategy`-Schnittstelle. Die Ausführungsstrategie für SQL Server, einschließlich SQL Azure, ist über die Ausnahmetypen informiert, die wiederholt werden können, und verfügt über sinnvolle Standardwerte für maximale Wiederholungsversuche, Verzögerung zwischen Wiederholungen usw.

### <a name="examples"></a>Beispiele

Der folgende Code ermöglicht automatische Wiederholungen beim Konfigurieren des DbContext-Objekts, das eine Sitzung mit der Datenbank repräsentiert. 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

Der folgende Code veranschaulicht, wie Sie eine Transaktion mit automatischen Wiederholungsversuchen ausführen, indem Sie eine Ausführungsstrategie verwenden. Die Transaktion wird in einem Delegaten definiert. Wenn ein vorübergehender Fehler auftritt, wird der Delegat von der Ausführungsstrategie erneut aufgerufen.

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

### <a name="more-information"></a>Weitere Informationen
* [Verbindungsstabilität](/ef/core/miscellaneous/connection-resiliency)
* [Data Points – EF Core 1.1](https://msdn.microsoft.com/en-us/magazine/mt745093.aspx)

## <a name="sql-database-using-adonet-retry-guidelines"></a>SQL-Datenbank mit ADO.NET Wiederholungsrichtlinien
SQL-Datenbank ist eine gehostete SQL-Datenbank, die in unterschiedlichen Größen und als Standard (freigegeben) und Premium (nicht freigegebenen)-Dienst verfügbar ist.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
SQL-Datenbank hat keine integrierte Unterstützung für Wiederholungen, wenn auf diese mit ADO.NET zugegriffen wird. Allerdings können die Rückgabecodes von Anforderungen verwendet werden, um zu bestimmen, warum eine Anforderung fehlgeschlagen ist. Weitere Informationen zur SQL-Datenbank-Drosselung finden Sie unter [Ressourceneinschränkungen für Azure SQL-Datenbank](/azure/sql-database/sql-database-resource-limits). Eine Liste mit relevanten Fehlercodes finden Sie unter [SQL-Fehlercodes für SQL-Datenbank-Clientanwendungen](/azure/sql-database/sql-database-develop-error-messages).

Sie können die Polly-Bibliothek verwenden, um Wiederholungen für SQL-Datenbank zu implementieren. Weitere Informationen finden Sie unter [Behandeln von vorübergehenden Fehlern mit Polly](#transient-fault-handling-with-polly).

### <a name="retry-usage-guidance"></a>Gebrauchsanleitung Wiederholungen
Beachten Sie den Zugriff auf SQL-Datenbank mit ADO.NET die folgenden Richtlinien:

* Wählen Sie die entsprechende Dienstoption (freigegeben oder Premium). Bei einer freigegebenen Instanz kann es möglicherweise zu längeren Verbindungsverzögerungen als üblich und Drosselung durch die Verwendung des gemeinsam genutzten Servers durch andere Mandanten kommen. Wenn Vorgänge mit vorhersagbarerer Leistung und zuverlässig geringe Latenz erforderlich sind, sollten Sie die Premium-Option wählen.
* Stellen Sie sicher, dass Sie Wiederholungen auf der entsprechenden Ebene oder im entsprechenden Bereich durchführen, um nicht-idempotente Operationen zu vermeiden, die Dateninkonsistenzen verursachen. Im Idealfall sollten alle Vorgänge idempotent sein, sodass sie ohne Inkonsistenzen wiederholt werden können. Wenn dies nicht der Fall ist, sollte die Wiederholung auf einer Ebene oder in einem Bereich durchgeführt werden, die ermöglichen, dass alle zugehörigen Änderungen rückgängig gemacht werden können, wenn ein Vorgang fehlschlägt; zum Beispiel im Transaktionsbereich. Weitere Informationen finden Sie unter [Clouddienst-Grundlagen Datenzugriffsebene - Handhabung vorübergehender Fehler](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).
* Eine Strategie mit festgelegtem Intervall wird für die Verwendung mit Azure SQL-Datenbank nicht empfohlen, mit Ausnahme von interaktiven Szenarien, wo nur wenige Wiederholungen in sehr kurzen Intervallen vorkommen. Ziehen Sie stattdessen eine exponentielle Backoff-Strategie für die meisten Szenarien in Betracht.
* Wählen Sie einen geeigneten Wert für die Verbindungs- und Befehls-Timeouts bei der Definition von Verbindungen. Ein zu kurzes Timeout kann zu vorzeitigen Fehlern bei Verbindungen führen, wenn die Datenbank ausgelastet ist. Ein zu langes Timeout kann verhindern, dass die Wiederholungslogik ordnungsgemäß funktioniert, da sie vor der Erkennung einer fehlgeschlagenen Verbindung zu lange wartet. Der Wert für das Timeout ist eine Komponente der End-to-End-Latenz. Sie wird effektiv der Wiederholungsverzögerung hinzugefügt, die in der Wiederholungsrichtlinie für jeden Wiederholungsversuch festgelegt ist.
* Schließen Sie die Verbindung nach einer bestimmten Anzahl von Wiederholungen, selbst wenn Sie eine exponentielle Backoff-Wiederholungslogik verwenden, und wiederholen Sie den Vorgang auf einer neuen Verbindung. Das mehrmalige Wiederholen des gleichen Vorgangs für dieselbe Verbindung kann ein Faktor sein, der zu Verbindungsproblemen beiträgt. Beispiele für dieses Verfahrens finden Sie unter [Clouddienst-Grundlagen Datenzugriffsebene - Handhabung vorübergehender Fehler](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).
* Wenn Verbindungspooling verwendet wird (Standardeinstellung), besteht die Möglichkeit, dass auch nach dem Schließen und erneuten Öffnen einer Verbindung dieselbe Verbindung aus dem Pool ausgewählt wird. Wenn dies der Fall ist, ist ein Verfahren zu Behebung das Aufrufen der **ClearPool**-Methode der **SqlConnection**-Klasse, um die Verbindung als nicht wiederverwendbar zu markieren. Allerdings sollten Sie dies erst nach mehreren fehlgeschlagenen Verbindungsversuchen tun und nur, wenn die spezifische Klasse des vorübergehende Fehler wie z. B. SQL-Timeouts (Fehlercode: -2) im Zusammenhang mit fehlerhaften Verbindungen auftritt.
* Wenn der Datenzugriffscode als **TransactionScope** -Instanzen initiierte Transaktionen verwendet, sollte die Wiederholungslogik erneut eine Verbindung herstellen und einen neuen Transaktionsbereich initiieren. Aus diesem Grund sollte der wiederholbare Codeblock den gesamten Bereich der Transaktion umfassen.

Erwägen Sie, mit den folgenden Einstellungen für Wiederholungsvorgänge zu beginnen. Hierbei handelt es sich um allgemeine Einstellungen und Sie sollten die Vorgänge überwachen und die Werte entsprechend Ihrem Szenario optimieren.

| **Context** | **Beispiel-Ziel E2E<br />Maximale Wartezeit** | **Wiederholungsstrategie** | **Einstellungen** | **Werte** | **So funktioniert's** |
| --- | --- | --- | --- | --- | --- |
| Interaktiv, Benutzeroberfläche<br />oder Vordergrund |2 Sek |FixedInterval |Anzahl der Wiederholungen<br />Wiederholungsintervall<br />Erster schneller Wiederholungsversuch |3<br />500 ms<br />true |Versuch 1 – Verzögerung 0 Sek.<br />Versuch 2 – Verzögerung 500 ms<br />Versuch 3 – Verzögerung 500 ms |
| Hintergrund<br />oder Batch |30 Sek |ExponentialBackoff |Anzahl der Wiederholungen<br />Min. Backoff<br />Max. Backoff<br />Delta-Backoff<br />Erster schneller Wiederholungsversuch |5<br />0 Sek.<br />60 Sekunden<br />2 Sek<br />false |Versuch 1 – Verzögerung 0 Sek.<br />Versuch 2 – Verzögerung ca. 2 Sek.<br />Versuch 3 – Verzögerung ca. 6 Sek.<br />Versuch 4 – Verzögerung ca. 14 Sek.<br />Versuch 5 – Verzögerung ca. 30 Sek. |

> [!NOTE]
> Die End-to-End-Latenzziele setzen das Standardtimeout für Verbindungen mit dem Dienst voraus. Wenn Sie längere Verbindungstimeouts angeben, wird die End-to-End-Latenz durch diese zusätzliche Zeit für jeden Wiederholungsversuch erweitert.
>
>

### <a name="examples"></a>Beispiele
In diesem Abschnitt wird veranschaulicht, wie Sie Polly zum Zugreifen auf Azure SQL-Datenbank verwenden können, indem Sie eine Reihe von Wiederholungsrichtlinien verwenden, die in der `Policy`-Klasse konfiguriert sind.

Der folgende Code enthält eine Erweiterungsmethode für die `SqlCommand`-Klasse, mit der `ExecuteAsync` mit exponentiell ansteigender Wartezeit aufgerufen wird.

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}

```

Diese asynchrone Erweiterungsmethode kann wie unten beschrieben verwendet werden.

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a>Weitere Informationen
* [Clouddienst-Grundlagen Datenzugriffsebene - Handhabung vorübergehender Fehler.](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

Eine allgemeine Anleitung dazu, wie Sie für SQL-Datenbank den größtmöglichen Nutzen erzielen, finden Sie unter [Anleitung zu Leistung und Flexibilität von Azure SQL-Datenbanken](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).


## <a name="service-bus-retry-guidelines"></a>Wiederholungsrichtlinien Service Bus
Service Bus ist ein Cloud  Messaging-Plattform, die lose gekoppelten Nachrichtenaustausch mit verbesserter Skalierbarkeit und Stabilität für die Komponenten einer Anwendung bietet, egal ob in der Cloud oder lokal gehostet.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
Service Bus implementiert Wiederholungen mithilfe von Implementierungen der [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) -Basisklasse. Alle Service Bus-Clients machen eine **RetryPolicy**-Eigenschaft verfügbar, die auf eine der Implementierungen der **RetryPolicy**-Basisklasse festgelegt werden kann. Die integrierten Implementierungen sind:

* Die [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx). Dies macht Eigenschaften verfügbar, die das Backoff-Intervall, die Anzahl der Wiederholungsversuche und die **TerminationTimeBuffer** -Eigenschaft steuern, die verwendet wird, um die Gesamtzeit für die Ausführung des Vorgangs zu beschränken.
* Die [NoRetry-Klasse](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx). Dies wird verwendet, wenn Wiederholungen auf der Ebene des Service Bus-API Levels  nicht erforderlich sind, z. B. wenn Wiederholungen von einem anderen Prozess als Teil eines Batches oder als Vorgang mit mehreren Schritten verwaltet werden.

Service Bus-Aktionen können einen Reihe von Ausnahmen zurückgeben, wie im [Anhang: Messagingausnahmen](http://msdn.microsoft.com/library/hh418082.aspx)aufgeführt. Die Liste enthält Informationen darüber, ob diese für die Wiederholung des Vorgangs geeignet ist. Angenommen, eine [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) gibt an, dass der Client für eine bestimmte Zeitspanne warten sollte, bevor der Vorgang wiederholt wird. Das Auftreten einer **ServerBusyException** bewirkt auch, dass Service Bus in einen anderen Modus wechselt, in dem den berechneten Wiederholungsverzögerungen zusätzliche 10 Sekunden hinzugefügt werden. Dieser Modus wird nach kurzer Zeit zurückgesetzt.

Die vom Service Bus zurückgegebenen Ausnahmen machen die **IsTransient**-Eigenschaft verfügbar, die angibt, ob der Client den Vorgang wiederholen soll. Die integrierte **RetryExponential**-Richtlinie stützt sich auf die **IsTransient**-Eigenschaft in der **MessagingException**-Klasse, die die Basisklasse für alle Service Bus-Ausnahmen ist. Bei der Erstellung benutzerdefinierter Implementierungen der **RetryPolicy**-Basisklasse können Sie eine Kombination aus Ausnahmetyp und **IsTransient**-Eigenschaft verwenden, um eine genauere Kontrolle über Wiederholungsaktionen zu ermöglichen. Sie können z.B. eine **QuotaExceededException** erkennen und Maßnahmen ergreifen, um die Warteschlange zu leeren, bevor erneut versucht wird, darüber eine Nachricht zu senden.

### <a name="policy-configuration"></a>Richtlinienkonfiguration
Wiederholungsrichtlinien werden programmgesteuert festgelegt und können als Standardrichtlinie für einen **NamespaceManager** und für eine **MessagingFactory** oder einzeln für jeden Messaging Client festgelegt werden. Um die Standard-Wiederholungsrichtlinie für die Messaging-Sitzung festzulegen, legen Sie die **RetryPolicy** von **NamespaceManager** fest.

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

Legen Sie zum Festlegen der Standard-Wiederholungsrichtlinie für alle Clients einer Messaging-Factory die **RetryPolicy** von **MessagingFactory** fest.

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

Um die Standard-Wiederholungsrichtlinie für einen Messaging Client festzulegen oder die Standardrichtlinie zu überschreiben, legen Sie seine **RetryPolicy** -Eigenschaft mit einer Instanz der erforderlichen Richtlinienklasse fest:

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

Die Wiederholungsrichtlinie kann auf der individuellen Vorgangsebene festgelegt werden. Sie gilt für alle Vorgänge des Messagin-Client.
Die folgende Tabelle zeigt die Standardeinstellungen für die integrierte Wiederholungsrichtlinie.

| Einstellung | Standardwert | Bedeutung |
|---------|---------------|---------|
| Richtlinie | Exponentiell | Exponentielles Backoff. |
| MinimalBackoff | 0 | Das minimale Backoff-Intervall. Dieser Wert wird zum Wiederholungsintervall addiert, das auf der Grundlage von „deltaBackoff“ berechnet wurde. |
| MaximumBackoff | 30 Sekunden | Das maximale Backoff-Intervall. „MaximumBackoff“ wird verwendet, wenn das berechnete Wiederholungsintervall größer als „MaxBackoff“ ist. |
| DeltaBackoff | 3 Sekunden | Backoff-Intervall zwischen den Wiederholungen. Ein Vielfaches dieser Zeitspanne wird für nachfolgende Wiederholungen verwendet. |
| TimeBuffer | 5 Sekunden | Der Beendigungszeitpuffer für die Wiederholung. Wiederholungsversuche werden abgebrochen, wenn die verbleibende Zeit kleiner als „TimeBuffer“ ist. |
| MaxRetryCount | 10 | Die maximale Anzahl von Warnungen. |
| ServerBusyBaseSleepTime | 10 Sekunden | Wenn es sich bei der zuletzt aufgetretenen Ausnahme um **ServerBusyException** handelt, wird dieser Wert zum berechneten Wiederholungsintervall addiert. Dieser Wert kann nicht geändert werden. |

### <a name="retry-usage-guidance"></a>Gebrauchsanleitung Wiederholungen
Berücksichtigen Sie bei Verwendung von Service Bus die folgenden Richtlinien:

* Implementieren Sie bei Verwendung der integrierten **RetryExponential** -Implementierung keinen Fallbackvorgang, da die Richtlinie auf Server Busy-Ausnahmen reagiert und automatisch in einen entsprechenden Wiederholungsmodus wechselt.
* Service Bus unterstützt eine Funktion namens Paired Namespace, mit der automatisches Failover zu einer Backup-Warteschlange in einem separaten Namespace implementiert wird, falls die Warteschlange im primären Namespace fehlschlägt. Nachrichten aus der sekundären Warteschlange können an die primäre Warteschlange gesendet werden, wenn diese wiederhergestellt wird. Mit dieser Funktion können vorübergehender Fehler behandelt werden. Weitere Informationen finden Sie unter [Asynchrone Nachrichtenmuster und Hochverfügbarkeit](http://msdn.microsoft.com/library/azure/dn292562.aspx).

Erwägen Sie, mit den folgenden Einstellungen für Wiederholungsvorgänge zu beginnen. Hierbei handelt es sich um allgemeine Einstellungen und Sie sollten die Vorgänge überwachen und die Werte entsprechend Ihrem Szenario optimieren.

| Kontext | Beispiel für maximale Wartezeit | Wiederholungsrichtlinie | Einstellungen | So funktioniert's |
|---------|---------|---------|---------|---------|
| Interaktiv, Benutzeroberfläche oder Vordergrund | 2 Sekunden*  | Exponentiell | MinimumBackoff = 0 <br/> MaximumBackoff = 30 s <br/> DeltaBackoff = 300 ms <br/> TimeBuffer = 300 ms <br/> MaxRetryCount = 2 | 1. Versuch: Verzögerung von 0 s <br/> 2. Versuch: Verzögerung von ~300 ms <br/> 3. Versuch: Verzögerung von ~900 ms |
| Hintergrund oder Batch | 30 Sekunden | Exponentiell | MinimumBackoff = 1 <br/> MaximumBackoff = 30 s <br/> DeltaBackoff = 1,75 s <br/> TimeBuffer = 5 s <br/> MaxRetryCount = 3 | 1. Versuch: Verzögerung von ~1 s <br/> 2. Versuch: Verzögerung von ~3 s <br/> 3. Versuch: Verzögerung von ~6 ms <br/> 4. Versuch: Verzögerung von ~13 ms |

\*Ohne Berücksichtigung der Verzögerung, die hinzukommt, wenn eine Antwort vom Typ „Server ausgelastet“ empfangen wird.

### <a name="telemetry"></a>Telemetrie
Service Bus protokollieren Wiederholungen als ETW-Ereignisse mit einer **EventSource**. Sie müssen einen **EventListener** an die Ereignisquelle anfügen, um die Ereignisse zu erfassen und in Performance-Viewer anzuzeigen oder diese in ein geeignetes Ziel-Protokoll schreiben. Dazu können Sie den [Anwendungsblock Semantic Logging](http://msdn.microsoft.com/library/dn775006.aspx) verwenden. Die Wiederholungsereignisse sind im folgenden Format:

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a>Beispiele
Das folgende Codebeispiel zeigt, wie Sie die Wiederholungsrichtlinie festlegen für:

* Einen Namespace-Manager. Die Richtlinie gilt für alle Vorgänge dieses Managers und kann nicht durch einzelne Vorgänge überschrieben werden.
* Eine Messaging-Factory. Die Richtlinie gilt für alle Clients, die von dieser Factory erstellt wurden, und kann bei der Erstellung einzelner Clients nicht überschrieben werden.
* Ein einzelner Messaging Client. Nachdem ein Client erstellt wurde, können Sie die Wiederholungsrichtlinie für diesen festlegen. Die Richtlinie gilt für alle Vorgänge auf dem Client.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }


            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }


            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a>Weitere Informationen
* [Asynchrone Nachrichtenmuster und Hochverfügbarkeit](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="azure-redis-cache-retry-guidelines"></a>Azure Redis Cache-Wiederholungsrichtlinien
Azure Redis Cache ist ein schneller Datenzugriff und ein Cache Service mit niedriger Latenz, der auf dem beliebten Open Source-Redis Cache basiert. Er ist sicher, von Microsoft verwaltet und ist von jeder Anwendung in Azure zugänglich.

Die Anleitung in diesem Abschnitt beruht auf der Verwendung des StackExchange.Redis-Clients für den Zugriff auf den Cache. Eine Liste der anderer geeigneter Clients finden Sie auf der [Redis-Website](http://redis.io/clients), diese haben möglicherweise unterschiedliche Wiederholungsmechanismen.

Beachten Sie, dass der StackExchange.Redis Client Multiplex über eine einzige Verbindung verwendet. Die empfohlene Verwendung ist das Erstellen einer Instanz des Clients beim Starten der Anwendung und die Verwendung dieser Instanz für alle Vorgänge im Cache. Aus diesem Grund wird die Verbindung mit dem Cache nur einmal hergestellt und die Anleitungen in diesem Abschnitt beziehen sich auf die Wiederholungsrichtlinie für die anfängliche Verbindung - und nicht für jeden Vorgang, der auf den Cache zugreift.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
Der StackExchange.Redis-Client verwendet eine Verbindungs-Manager-Klasse, die über eine Reihe von Optionen konfiguriert ist, z.B.:

- **ConnectRetry**: Die Anzahl, wie häufig erneut versucht wird, eine fehlgeschlagene Verbindung mit dem Cache herzustellen.
- **ReconnectRetryPolicy**: Die Wiederholungsstrategie, die verwendet werden soll.
- **ConnectTimeout**: Die maximale Wartezeit in Millisekunden.

### <a name="policy-configuration"></a>Richtlinienkonfiguration
Wiederholungsrichtlinien werden programmgesteuert durch Festlegen der Optionen für den Client konfiguriert, bevor eine Verbindung mit dem Cache hergestellt wird. Dies kann durch das Erstellen einer Instanz der **ConfigurationOptions**-Klasse, das Auffüllen ihrer Eigenschaften und die Übergabe an die **Connect**-Methode erreicht werden.

Die integrierten Klassen unterstützen die lineare (konstante) Verzögerung und exponentiell ansteigende Wartezeiten mit zufälligen Wiederholungsintervallen. Sie können auch eine benutzerdefinierte Wiederholungsrichtlinie erstellen, indem Sie die **IReconnectRetryPolicy**-Schnittstelle implementieren.

Im folgenden Beispiel wird eine Wiederholungsstrategie mit exponentiell ansteigender Wartezeit konfiguriert.

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Alternativ können Sie die Optionen als Zeichenfolge angeben und diese an die **Connect** Methode übergeben. Beachten Sie, dass die **ReconnectRetryPolicy**-Eigenschaft auf diese Weise nicht festgelegt werden kann, sondern nur per Code.

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Es ist auch möglich, Optionen direkt beim Herstellen der Verbindung mit dem Cache anzugeben.

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

Weitere Informationen finden Sie unter [Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) (Konfiguration) in der StackExchange.Redis-Dokumentation.

Die folgende Tabelle zeigt die Standardeinstellungen für die integrierte Wiederholungsrichtlinie.

| **Context** | **Einstellung** | **Standardwert**<br />(v 1.2.2) | **Bedeutung** |
| --- | --- | --- | --- |
| ConfigurationOptions |ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout<br /><br />ReconnectRetryPolicy |3<br /><br />Maximal 5000 ms plus SyncTimeout<br />1000<br /><br />LinearRetry 5000 ms |Die Anzahl der Wiederholungen von Verbindungsversuchen während des Erstverbindungsvorgangs.<br />Zeitlimit (ms) für Verbindungen. Keine Verzögerung zwischen den Wiederholungsversuchen.<br />Zeit (ms), um synchrone Vorgänge zu ermöglichen.<br /><br />Wiederholung jeweils nach 5000 ms.|

> [!NOTE]
> Für synchrone Vorgänge kann `SyncTimeout` einen Beitrag zur End-to-End-Wartezeit leisten, aber wenn der Wert zu niedrig festgelegt wird, kann es zu übermäßigen Zeitüberschreitungen kommen. Weitere Informationen finden Sie unter [Problembehandlung für Azure Redis Cache][redis-cache-troubleshoot]. Im Allgemeinen ist es ratsam, die Verwendung von synchronen Vorgängen zu vermeiden und stattdessen asynchrone Vorgänge zu verwenden. Weitere Informationen finden Sie unter [Pipelines und Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).
>
>

### <a name="retry-usage-guidance"></a>Gebrauchsanleitung Wiederholungen
Berücksichtigen Sie bei Verwendung von Azure Redis Cache die folgenden Richtlinien :

* Der StackExchange Redis-Client verwaltet seine eigene Wiederholungen, aber nur beim Herstellen einer Verbindung mit dem Cache beim ersten Starten der Anwendung. Sie können das Verbindungstimeout, die Anzahl von Wiederholungsversuchen und die Dauer zwischen den Wiederholungen zur Herstellung dieser Verbindung konfigurieren, aber die Wiederholungsrichtlinie gilt nicht für Vorgänge für den Cache.
* Anstatt eine große Anzahl von Wiederholungsversuchen zu verwenden, sollten Sie auf die ursprüngliche Datenquelle zurückgreifen.

### <a name="telemetry"></a>Telemetrie
Sie können Informationen zu Verbindungen (aber nicht für andere Vorgänge) mit einem **TextWriter**sammeln.

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Ein Beispiel für die dadurch generierte Ausgabe ist unten dargestellt.

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a>Beispiele
Im folgenden Codebeispiel wird eine konstante (lineare) Verzögerung zwischen Wiederholungsversuchen konfiguriert, wenn der StackExchange.Redis-Client initialisiert wird. In diesem Beispiel wird veranschaulicht, wie die Konfiguration mit einer **ConfigurationOptions**-Instanz festgelegt wird.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries
                    
                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

Im nächsten Beispiel wird die Konfiguration durch Angeben der Optionen als Zeichenfolge festgelegt. Das Verbindungstimeout ist der maximale Zeitraum für das Warten auf die Verbindungsherstellung mit dem Cache und nicht die Verzögerung zwischen Wiederholungsversuchen. Beachten Sie, dass die **ReconnectRetryPolicy**-Eigenschaft nur per Code festgelegt werden kann.

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

Weitere Beispiele finden Sie unter [Konfiguration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) auf der Projektwebsite.

### <a name="more-information"></a>Weitere Informationen
* [Redis-Website](http://redis.io/)

## <a name="documentdb-api-retry-guidelines"></a>DocumentDB-API-Wiederholungsrichtlinien

Cosmos DB ist eine vollständig verwaltete Datenbank mit Unterstützung mehrerer Modelle, die schemalose JSON-Daten per [DocumentDB-API][documentdb-api] unterstützt. Sie bietet konfigurierbare und zuverlässige Leistung, systemeigene JavaScript-Transaktionsverarbeitung und ist für die Cloud mit elastischer Skalierung konzipiert.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
Die `DocumentClient`-Klasse führt bei Fehlversuchen automatisch Wiederholungsversuche durch. Konfigurieren Sie zum Festlegen der Anzahl von Wiederholungsversuchen und der maximalen Wartezeit [ConnectionPolicy.RetryOptions]. Ausnahmen, die vom Client ausgelöst werden, unterliegen entweder nicht der Wiederholungsrichtlinie oder sind keine vorübergehenden Fehler.

Wenn Cosmos DB den Client einschränkt, wird ein HTTP 429-Fehler zurückgegeben. Überprüfen Sie den Statuscode unter `DocumentClientException`.

### <a name="policy-configuration"></a>Richtlinienkonfiguration
Die folgende Tabelle enthält die Standardeinstellungen für die `RetryOptions`-Klasse.

| Einstellung | Standardwert | BESCHREIBUNG |
| --- | --- | --- |
| MaxRetryAttemptsOnThrottledRequests |9 |Die maximale Anzahl von Wiederholungsversuchen bei einem Fehlschlagen der Anforderung, weil Cosmos DB die Rateneinschränkung auf den Client angewendet hat. |
| MaxRetryWaitTimeInSeconds |30 |Die maximale Wiederholungszeit in Sekunden. |

### <a name="example"></a>Beispiel
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a>Telemetrie
Wiederholungsversuche werden als unstrukturierte Ablaufverfolgungsmeldungen über eine .NET **TraceSource**protokolliert. Sie müssen einen **TraceListener** konfigurieren, um die Ereignisse zu erfassen und diese in ein geeignetes Zielprotokoll zu schreiben.

Wenn Sie Ihrer Datei „App.config“ beispielsweise Folgendes hinzugefügt haben, werden die Ablaufverfolgungen in einer Textdatei an demselben Speicherort wie die ausführbare Datei generiert:

```
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="DocumentDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```


## <a name="azure-search-retry-guidelines"></a>Azure Search-Wiederholungsrichtlinien
Azure Search kann dafür verwendet werden, nützliche und anspruchsvolle Suchfunktionen zu einer Website oder Anwendung hinzuzufügen, Suchergebnisse schnell und einfach zu optimieren und umfassende und fein abgestimmte Rangmodelle zu erstellen.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
Das Wiederholungsverhalten im Azure Search SDK wird mit der `SetRetryPolicy`-Methode in den Klassen [SearchServiceClient] und [SearchIndexClient] gesteuert. Bei der Standardrichtlinie werden Wiederholungsversuche mit exponentiellem Backoff (exponentiell ansteigende Wartezeiten) durchgeführt, wenn Azure Search eine 5xx- oder 408-Antwort (Anforderungstimeout) zurückgibt.

### <a name="telemetry"></a>Telemetrie
Nachverfolgung mit ETW oder per Registrierung eines benutzerdefinierten Ablaufverfolgungsanbieters. Weitere Informationen finden Sie in der [AutoRest-Dokumentation][autorest].

## <a name="azure-active-directory-retry-guidelines"></a>Wiederholungsrichtlinien für Azure Active Directory
Azure Active Directory (Azure AD) ist eine umfassende Cloud-Lösung für die Identitäts- und Zugriffsverwaltung, die zentrale Verzeichnisdienste, eine erweiterte Identitätsgovernance, Sicherheit und Zugriffsverwaltung von Anwendungen kombiniert. Azure AD bietet Entwicklern auf der Grundlage von zentralen Richtlinien und Regeln auch eine Plattform für die Identitätsverwaltung zur Bereitstellung einer Zugriffssteuerung für deren Anwendungen.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
In der Active Directory-Authentifizierungsbibliothek (ADAL) ist ein integrierter Wiederholungsmechanismus für Azure Active Directory enthalten. Zur Vermeidung von unerwarteten Sperrungen empfehlen wir, für Drittanbieterbibliotheken und -anwendungscode **keine** Wiederholungsversuche zum Herstellen von fehlgeschlagenen Verbindungen durchzuführen, sondern die Verarbeitung von Wiederholungsversuchen ADAL zu überlassen. 

### <a name="retry-usage-guidance"></a>Gebrauchsanleitung Wiederholungen
Berücksichtigen Sie Bei Verwendung von Azure Active Directory die folgenden Richtlinien:

* Verwenden Sie nach Möglichkeit die ADAL-Bibliothek und die integrierte Unterstützung für Wiederholungsversuche.
* Wenn Sie die REST-API für Azure Active Directory verwenden, sollten Sie den Vorgang nur wiederholen, wenn das Ergebnis ein Fehler im Bereich 5xx ist (z. B. 500 Interner Serverfehler, 502 ungültiges Gateway, 503 Dienst nicht verfügbar und 504 Gateway-Timeout). Bei anderen Fehlern den Vorgang nicht wiederholen.
* Eine exponentielle Backoff-Richtlinie wird für die Verwendung in Batch-Szenarien mit Azure Active Directory empfohlen.

Erwägen Sie, mit den folgenden Einstellungen für Wiederholungsvorgänge zu beginnen. Hierbei handelt es sich um allgemeine Einstellungen und Sie sollten die Vorgänge überwachen und die Werte entsprechend Ihrem Szenario optimieren.

| **Context** | **Beispiel-Ziel E2E<br />Maximale Wartezeit** | **Wiederholungsstrategie** | **Einstellungen** | **Werte** | **So funktioniert's** |
| --- | --- | --- | --- | --- | --- |
| Interaktiv, Benutzeroberfläche<br />oder Vordergrund |2 Sek |FixedInterval |Anzahl der Wiederholungen<br />Wiederholungsintervall<br />Erster schneller Wiederholungsversuch |3<br />500 ms<br />true |Versuch 1 – Verzögerung 0 Sek.<br />Versuch 2 – Verzögerung 500 ms<br />Versuch 3 – Verzögerung 500 ms |
| Hintergrund oder <br />Batch |60 Sekunden |ExponentialBackoff |Anzahl der Wiederholungen<br />Min. Backoff<br />Max. Backoff<br />Delta-Backoff<br />Erster schneller Wiederholungsversuch |5<br />0 Sek.<br />60 Sekunden<br />2 Sek<br />false |Versuch 1 – Verzögerung 0 Sek.<br />Versuch 2 – Verzögerung ca. 2 Sek.<br />Versuch 3 – Verzögerung ca. 6 Sek.<br />Versuch 4 – Verzögerung ca. 14 Sek.<br />Versuch 5 – Verzögerung ca. 30 Sek. |

### <a name="more-information"></a>Weitere Informationen
* [Azure Active Directory-Authentifizierungsbibliotheken][adal]

## <a name="service-fabric-retry-guidelines"></a>Service Fabric-Wiederholungsrichtlinien

Die Verteilung von Reliable Services in einem Service Fabric-Cluster dient als Schutz vor den meisten potenziellen vorübergehenden Fehlern, die in diesem Artikel beschrieben wurden. Einige vorübergehende Fehler sind aber weiterhin möglich. Für den Naming Service kann beispielsweise gerade eine Routingänderung durchgeführt werden, wenn er eine Anforderung erhält, sodass eine Ausnahme ausgelöst wird. Wenn dieselbe Anforderung 100 Millisekunden später eintrifft, ist der Vorgang wahrscheinlich erfolgreich.

Intern verwaltet Service Fabric diese Art von vorübergehenden Fehlern. Sie können einige Einstellungen konfigurieren, indem Sie beim Einrichten Ihrer Dienste die `OperationRetrySettings`-Klasse verwenden.  Der folgende Code enthält hierzu ein Beispiel. In den meisten Fällen sollte dies nicht erforderlich sein, sodass die Standardeinstellungen verwendet werden können.

```csharp
    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        OperationTimeout = TimeSpan.FromSeconds(30)
    };

    var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

    var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

    var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

    var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
        new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
        new ServicePartitionKey(0));
```

## <a name="more-information"></a>Weitere Informationen

* [Remote Exception Handling](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling) (Remotebehandlung von Ausnahmen)


## <a name="azure-event-hubs-retry-guidelines"></a>Wiederholungsrichtlinien für Azure Event Hubs

Azure Event Hubs ist ein hyperskalierter Dienst für die Erfassung von Telemetriedaten, der Millionen von Ereignissen sammelt, transformiert und speichert.

### <a name="retry-mechanism"></a>Wiederholungsmechanismus
Das Wiederholungsverhalten in der Azure Event Hubs-Clientbibliothek wird von der `RetryPolicy`-Eigenschaft in der `EventHubClient`-Klasse gesteuert. Die Standardrichtlinie führt Wiederholungsversuche mit exponentiell ansteigender Wartezeit durch, wenn Azure Event Hub eine vorübergehende Ausnahme vom Typ `EventHubsException` oder `OperationCanceledException` zurückgibt.

### <a name="example"></a>Beispiel
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a>Weitere Informationen
[.NET Standard client library for Azure Event Hubs](https://github.com/Azure/azure-event-hubs-dotnet) (.NET Standard-Clientbibliothek für Azure Event Hubs)

## <a name="general-rest-and-retry-guidelines"></a>Allgemeine REST-und Wiederholungsrichtlinien
Berücksichtigen Sie beim Zugriff auf die Dienste von Azure oder von Drittanbietern Folgendes:

* Verwenden Sie einen systematischen Ansatz zur Verwaltung von Wiederholungen, vielleicht als wiederverwendbaren Code, sodass Sie eine einheitliche Methodik für alle Clients und alle Lösungen anwenden können.
* Erwägen Sie die Verwendung eines Wiederholungsframeworks, wie z. B. den Anwendungsblock zur Handhabung vorübergehender Fehler, um Wiederholungen zu verwalten, wenn der Zieldienst oder der Client nicht über integrierte Wiederholungsmechanismen verfügt. Dies hilft Ihnen bei der Implementierung eines konsistenten Wiederholungsverhaltens, und kann eine geeignete Standardwiederholungsstrategie für den Zieldienst bereitstellen. Sie müssen aber unter Umständen einen benutzerdefinierten Wiederholungscode für Dienste erstellen, die kein Standardverhalten aufweisen und nicht auf Ausnahmen vertrauen, um vorübergehende Fehler anzuzeigen, oder wenn Sie eine **Retry-Response**-Antwort verwenden möchten, um das Wiederholungsverhalten zu verwalten.
* Die vorübergehende Erkennungslogik hängt von der tatsächlichen Client-API ab, die Sie verwenden, um die REST-Aufrufe aufzurufen. Einige Clients, z.B. die neuere **HttpClient**-Klasse, lösen keine Ausnahmen für abgeschlossene Anforderungen mit einem nicht erfolgreichen HTTP-Statuscode aus. Dies verbessert die Leistung, verhindert jedoch die Verwendung des Anwendungsblocks zur Handhabung vorübergehender Fehler. In diesem Fall können Sie den Aufruf der REST-API mit einem Code umschließen, der Ausnahmen für nicht erfolgreiche HTTP-Statuscodes erzeugt, die dann vom Block verarbeitet werden können. Alternativ können Sie einen anderen Mechanismus für die Durchführung der Wiederholungen verwenden.
* Der vom Dienst zurückgegebene HTTP-Statuscode kann dabei helfen, zu erkennen, ob der Fehler vorübergehend ist. Möglicherweise müssen Sie die Ausnahmen, die von einem Client oder dem Wiederholungsframework erzeugt wurden untersuchen, um auf den Statuscode zuzugreifen oder um den entsprechenden Ausnahmetyp bestimmen zu können. Die folgenden HTTP-Codes geben in der Regel an, dass eine Wiederholung geeignet ist:
  * 408 Anforderungstimeout
  * 500 Interner Serverfehler
  * 502 Ungültiges Gateway
  * 503 Dienst nicht verfügbar
  * 504 Gateway-Timeout
* Wenn Sie Ihre Wiederholungslogik auf Ausnahmen basieren, wird in der Regel durch Folgendes angezeigt, dass ein vorübergehender Fehler besteht, da keine Verbindung hergestellt werden konnte:
  * WebExceptionStatus.ConnectionClosed
  * WebExceptionStatus.ConnectFailure
  * WebExceptionStatus.Timeout
  * WebExceptionStatus.RequestCanceled
* Im Fall des Status „Dienst nicht verfügbar“ weist der Dienst unter Umständen auf die entsprechende Verzögerung vor der Wiederholung im **Retry-After** -Antwort-Header oder in einem anderen benutzerdefinierten Header hin. Dienste können auch zusätzliche Informationen als benutzerdefinierte Header oder in den Inhalt der Antwort eingebettet senden. Der Anwendungsblock zur Handhabung vorübergehender Fehler kann keine Standard- oder benutzerdefinierte „Retry-after“-Header verwenden.
* Führen Sie keine Wiederholung für Statuscodes durch, die Clientfehler (Fehler im Bereich 4xx) darstellen, mit Ausnahme eines Anforderungstimeouts 408.
* Testen Sie Ihre Wiederholungsstrategien und Mechanismen gründlich unter verschiedenen Bedingungen, wie z. B. verschiedenen Netzwerkzuständen und sich ändernden Systemlastverteilungen.

### <a name="retry-strategies"></a>Wiederholungsstrategien
Im Folgenden sind die typischen Arten von Wiederholungsstrategie-Intervallen dargestellt:

* **Exponentiell**: eine Wiederholungsrichtlinie, mit der eine angegebene Anzahl von Wiederholungsversuchen durchgeführt wird, bei der ein zufälliger exponentieller Backoff-Ansatz zur Ermittlung des Intervalls zwischen den Wiederholungsversuchen verwendet wird. Beispiel: 

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
* **Inkrementell**: eine Wiederholungsstrategie mit einer angegebenen Anzahl von Wiederholungsversuchen und ein inkrementelles Zeitintervall zwischen den Wiederholungen. Beispiel: 

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* **LinearRetry**Eine Wiederholungsstrategie, bei der eine angegebene Anzahl von Wiederholungen durchgeführt wird, mit einem angegebenen festen Zeitintervall zwischen den Wiederholungen. Beispiel: 

        retryInterval = this.deltaBackoff;

### <a name="more-information"></a>Weitere Informationen
* [Trennschalter-Strategien](http://msdn.microsoft.com/library/dn589784.aspx)

## <a name="transient-fault-handling-with-polly"></a>Behandeln von vorübergehenden Fehlern mit Polly
[Polly](http://www.thepollyproject.org) ist eine Bibliothek zur programmgesteuerten Behandlung von Wiederholungsversuchen und [Schutzschalter][circuit-breaker]-Strategien. Das Polly-Projekt ist ein Mitglied der [.NET Foundation][dotnet-foundation]. Für Dienste, bei denen der Client Wiederholungsversuche nicht nativ unterstützt, ist Polly eine zulässige Alternative. Es ist hierbei nicht erforderlich, benutzerdefinierten Code für die Wiederholungsversuche zu schreiben, dessen richtige Implementierung schwierig sein kann. Polly bietet auch eine Möglichkeit zum Überwachen des Auftretens von Fehlern, damit Sie Wiederholungsversuche protokollieren können.

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[documentdb-api]: /azure/documentdb/documentdb-introduction
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
