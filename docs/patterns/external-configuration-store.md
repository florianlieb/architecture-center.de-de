---
title: Externer Konfigurationsspeicher
description: Konfigurationsinformationen aus dem Anwendungsbereitstellungspaket an einen zentralen Speicherort verschieben
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="external-configuration-store-pattern"></a>Muster mit externem Konfigurationsspeicher

[!INCLUDE [header](../_includes/header.md)]

Konfigurationsinformationen aus dem Anwendungsbereitstellungspaket an einen zentralen Speicherort verschieben Dies kann das Verwalten und Steuern von Konfigurationsdaten und das gemeinsame Verwenden von Konfigurationsdaten in allen Anwendungen und Anwendungsinstanzen erleichtern.

## <a name="context-and-problem"></a>Kontext und Problem

Die meisten Anwendungslaufzeitumgebungen enthalten Konfigurationsinformationen, die in Dateien gespeichert sind, welche mit der Anwendung bereitgestellt werden. In einigen Fällen können diese Dateien bearbeitet werden, um das Verhalten der Anwendung zu ändern, nachdem sie bereitgestellt wurde. Nach Änderungen an der Konfiguration muss die Anwendung jedoch neu bereitgestellt werden, was häufig zu inakzeptablen Ausfallzeiten und sonstigem Verwaltungsaufwand führt.

Lokale Konfigurationsdateien beschränken ferner die Konfiguration auf eine einzelne Anwendung, gelegentlich wäre es jedoch sinnvoll, Konfigurationseinstellungen für mehrere Anwendungen freizugeben. Beispiele: Datenbankverbindungszeichenfolgen, UI-Designinformationen, die URLs von Warteschlangen und durch eine zusammengehörige Gruppe von Anwendungen verwendeter Speicher.

Es ist nicht einfach, Änderungen an lokalen Konfigurationen über mehrere ausgeführte Instanzen der Anwendung, insbesondere in einem cloudgehosteten Szenario, zu verwalten. Dies kann dazu führen, dass Instanzen andere Konfigurationseinstellungen verwenden, während das Update bereitgestellt wird.

Darüber hinaus erfordern Updates von Anwendungen und Komponenten möglicherweise Änderungen an den Konfigurationsschemas. Viele Konfigurationssysteme unterstützen keine verschiedenen Versionen von Konfigurationsinformationen.

## <a name="solution"></a>Lösung

Speichern Sie die Konfigurationsinformationen in einem externen Speicher, und stellen Sie eine Schnittstelle bereit, über die Konfigurationseinstellungen schnell und effizient gelesen und aktualisiert werden können. Der Typ des externen Speichers hängt von der Hosting- und Laufzeitumgebung der Anwendung ab. In einem cloudgehosteten Szenario handelt es sich in der Regel um einen cloudbasierten Speicherdienst, es könnte sich aber auch um eine gehostete Datenbank oder ein anderes System handeln.

Der Sicherungsspeicher, den Sie für Konfigurationsinformationen auswählen, sollte über eine Schnittstelle verfügen, die einen konsistenten, benutzerfreundlichen Zugriff erlaubt. Sie sollten die Informationen in einem ordnungsgemäß typisierten und strukturierten Format verfügbar machen. Die Implementierung muss möglicherweise auch den Benutzerzugriff autorisieren, um Konfigurationsdaten zu schützen, und so flexibel sein, das Speichern mehrerer Versionen der Konfiguration (wie Entwicklungs-, Staging- und Produktionsversionen, jeweils einschließlich mehrerer Releases) zu ermöglichen.

> Viele integrierte Konfigurationssysteme lesen die Daten beim Start der Anwendung und speichern die Daten im Speicher zwischen, um einen schnellen Zugriff zu ermöglichen und Auswirkungen auf die Anwendungsleistung zu minimieren. Je nach Art des verwendeten Sicherungsspeichers und der Latenz dieses Speichers kann es sinnvoll sein, einen Zwischenspeichermechanismus im externen Konfigurationsspeicher zu implementieren. Weitere Informationen finden Sie unter [Caching Guidance (Leitfaden für die Zwischenspeicherung)](https://msdn.microsoft.com/library/dn589802.aspx). Die Abbildung zeigt einen Überblick über das Muster des externen Konfigurationsspeichers mit einem optionalen lokalen Cache.

![Die Abbildung zeigt einen Überblick über das Muster des externen Konfigurationsspeichers mit einem optionalen lokalen Cache.](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:

Wählen Sie einen Sicherungsspeicher aus, der eine akzeptable Leistung und eine hohe Verfügbarkeit und Stabilität bietet und im Rahmen der Anwendungswartung und -verwaltung gesichert werden kann. In einer cloudgehosteten Anwendung ist die Verwendung eines Cloudspeichermechanismus in der Regel eine gute Wahl, um diese Anforderungen zu erfüllen.

Gestalten Sie das Schema des Sicherungsspeichers so, dass der Speicher hinsichtlich der Informationsarten, die er speichern kann, flexibel ist. Stellen Sie sicher, dass er für alle Konfigurationsanforderungen wie typisierte Daten, Sammlungen von Einstellungen, mehrere Versionen von Einstellungen und andere Funktionen geeignet ist, die die Anwendungen erfordern, die den Speicher verwenden. Das Schema sollte sich leicht erweitern lassen, um zusätzliche Einstellungen zu unterstützen, wenn sich die Anforderungen ändern.

Berücksichtigen Sie die physischen Möglichkeiten des Sicherungsspeichers, welchen Bezug sie zu der Art der Speicherung der Konfigurationsinformationen haben und welche Auswirkungen auf die Leistung sich hieraus ergeben. Für das Speichern eines XML-Dokuments, das Konfigurationsinformationen enthält, benötigen Sie beispielsweise entweder die Konfigurationsschnittstelle oder die Anwendung, um das Dokument zu analysieren und so einzelne Einstellungen zu lesen. Das Aktualisieren einer Einstellung wird dadurch komplizierter, wenngleich eine Zwischenspeicherung der Einstellungen dazu beitragen kann, eine langsamere Leseleistung zu kompensieren.

Berücksichtigen Sie, in welcher Weise die Konfigurationsschnittstelle eine Kontrolle über den Bereich und die Vererbung von Konfigurationseinstellungen zulässt. Möglicherweise kann es erforderlich sein, Konfigurationseinstellungen auf Organisations-, Anwendungs- oder Computerebene zu betrachten. Es kann erforderlich sein, eine Delegierung der Zugriffsteuerung auf verschiedene Bereiche zu unterstützen und zu verhindern bzw. zuzulassen, dass einzelne Anwendungen Einstellungen außer Kraft setzen.

Stellen Sie sicher, dass die Konfigurationsschnittstelle die Konfigurationsdaten in den erforderlichen Formaten wie typisierte Werte, Sammlungen, Schlüssel-/Wertpaare oder Eigenschaftensammlungen bereitstellen kann.

Berücksichtigen Sie das Verhalten der Konfigurationsschnittstelle, wenn Einstellungen Fehler enthalten oder nicht im Sicherungsspeicher vorhanden sind. Es kann zweckmäßig sein, Standardeinstellungen und Fehlerprotokolle zurückzugeben. Berücksichtigen Sie auch Aspekte wie die Groß-/Kleinschreibung von Konfigurationseinstellungsschlüsseln oder -namen, die Speicherung und Handhabung binärer Daten sowie die Handhabung von Nullwerten oder leeren Werten.

Überlegen Sie, wie die Konfigurationsdaten geschützt werden können, sodass nur angemessenen Benutzern und Anwendungen Zugriff gewährt wird. Hierbei handelt es sich wahrscheinlich um eine Funktion der Konfigurationsspeicherschnittstelle, aber es muss auch sichergestellt werden, dass direkt ohne entsprechende Berechtigung auf die Daten im Sicherungsspeicher zugegriffen werden kann. Stellen Sie eine strikte Trennung zwischen den Berechtigungen sicher, die für das Lesen und das Schreiben von Konfigurationsdaten erforderlich sind. Überlegen Sie auch, ob Sie einige oder alle Konfigurationseinstellungen verschlüsseln müssen und wie dies in der Konfigurationsspeicher-Schnittstelle implementiert werden muss.

Zentral gespeicherte Konfigurationen, die das Verhalten der Anwendung während der Laufzeit ändern, sind sehr wichtig und sollten mit denselben Mechanismen bereitgestellt, aktualisiert und verwaltet werden, die auch für das Bereitstellen von Anwendungscode verwendet werden. Änderungen, die beispielsweise mehrere Anwendungen betreffen können, müssen mithilfe eines umfassenden Test- und Stagingbereitstellungsansatzes durchgeführt werden, um sicherzustellen, dass sich die Änderung für alle Anwendungen eignet, die diese Konfiguration verwenden. Wenn ein Administrator eine Einstellung bearbeitet, sodass eine Anwendung aktualisiert wird, kann sich dies negativ auf die übrigen Anwendungen auswirken, die dieselbe Einstellung verwenden.

Wenn eine Anwendung die Konfigurationsinformationen zwischenspeichert, muss die Anwendung benachrichtigt werden, wenn sich die Konfiguration ändert. Unter Umständen ist es möglich, eine Ablaufrichtlinie über zwischengespeicherte Konfigurationsdaten zu implementieren, sodass diese Informationen automatisch in regelmäßigen Abständen aktualisiert werden und alle Änderungen übernommen (und verarbeitet) werden.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Dieses Muster ist hilfreich:

- Für Konfigurationseinstellungen, die von verschiedenen Anwendungen und Anwendungsinstanzen gemeinsam genutzt werden, oder bei denen eine Standardkonfiguration zwischen mehreren Anwendungen und Anwendungsinstanzen erzwungen werden muss.

- Für ein Standardkonfigurationssystem, das nicht alle erforderlichen Konfigurationseinstellungen wie die Speicherung von Bildern oder komplexen Datentypen unterstützt.

- Als ergänzender Speicher für einige der Anwendungseinstellungen, wobei möglicherweise Anwendungen gestattet wird, einige oder alle der zentral gespeicherten Einstellungen außer Kraft zu setzen.

- Als Methode, um die Verwaltung mehrerer Anwendungen zu vereinfachen und optional die Verwendung von Konfigurationseinstellungen zu überwachen, indem einige oder alle Zugriffsarten auf den Konfigurationsspeicher protokolliert werden.

## <a name="example"></a>Beispiel

In einer gehosteten Microsoft Azure-Anwendung besteht eine gängige Methode für die externe Speicherung von Konfigurationsdaten darin, Azure Storage zu verwenden. Azure Storage ist stabil, bietet eine hohe Leistung und wird für eine hohe Verfügbarkeit dreimal repliziert (mit automatischem Failover). Azure Table Storage bietet einen Schlüssel/Wert-Speicher mit der Möglichkeit, ein flexibles Schema für die Werte zu verwenden. Azure Blob Storage bietet einen hierarchisch strukturierten, containerbasierten Speicher, der beliebige Datentypen in einzeln benannten Blobs aufnehmen kann.

Das folgende Beispiel zeigt, wie ein Konfigurationsspeicher über Blob Storage implementiert werden kann, um Konfigurationsinformationen zu speichern und verfügbar zu machen. Die `BlobSettingsStore`-Klasse abstrahiert Blob Storage für das Aufnehmen von Konfigurationsinformationen und implementiert die `ISettingsStore`-Schnittstelle, die im folgenden Code gezeigt wird.

> Diesen Code finden Sie unter dem Projekt _ExternalConfigurationStore.Cloud_ in der Lösung _ExternalConfigurationStore_, die auf [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) verfügbar ist.

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

Diese Schnittstelle definiert Methoden zum Abrufen und Aktualisieren von Konfigurationseinstellungen, die im Konfigurationsspeicher gespeichert werden, und enthält eine Versionsnummer, die verwendet werden kann, um festzustellen, ob Konfigurationseinstellungen kürzlich geändert wurden. Die `BlobSettingsStore`-Klasse verwendet die `ETag`-Eigenschaft des Blobs zum Implementieren einer Versionsverwaltung. Die `ETag`-Eigenschaft wird automatisch jedes Mal aktualisiert, wenn in das Blob geschrieben wird.

> Standardmäßig macht diese einfache Lösung alle Konfigurationseinstellungen als Zeichenfolgenwerte anstelle typisierter Werten verfügbar.

Die `ExternalConfigurationManager`-Klasse fungiert als Wrapper um ein `BlobSettingsStore`-Objekt. Eine Anwendung kann diese Klasse zum Speichern und Abrufen von Konfigurationsinformationen verwenden. Diese Klasse verwendet die Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx)-Bibliothek, um alle Änderungen an der Konfiguration über eine Implementierung der `IObservable`-Schnittstelle verfügbar zu machen. Wenn eine Einstellung durch Aufrufen der `SetAppSetting`-Methode geändert wird, wird das `Changed`-Ereignis ausgelöst, und alle Abonnenten dieses Ereignisses werden benachrichtigt.

Beachten Sie, dass alle Einstellungen für den schnellen Zugriff auch in einem `Dictionary`-Objekt innerhalb der `ExternalConfigurationManager`-Klasse zwischengespeichert werden. Die `GetSetting`-Methode zum Abrufen einer Konfigurationseinstellung liest die Daten aus dem Cache. Wenn die Einstellung nicht im Cache gefunden wird, wird sie stattdessen aus dem `BlobSettingsStore`-Objekt abgerufen.

Die `GetSettings`-Methode ruft die `CheckForConfigurationChanges`-Methode ab, um festzustellen, ob sich die Konfigurationsinformationen in Blob Storage geändert haben. Hierzu wird die Versionsnummer untersucht und mit der aktuellen Versionsnummer verglichen, die im `ExternalConfigurationManager`-Objekt gespeichert ist. Wenn eine oder mehrere Änderungen aufgetreten sind, wird das `Changed`-Ereignis ausgelöst, und die im `Dictionary`-Objekt zwischengespeicherten Konfigurationseinstellungen werden aktualisiert. Dies ist eine Anwendung des [Cache-Aside-Musters](cache-aside.md).

Das folgende Codebeispiel zeigt, wie das `Changed`-Ereignis, die `GetSettings`-Methode und die `CheckForConfigurationChanges`-Methode implementiert werden:

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache. 
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> Die `ExternalConfigurationManager`-Klasse stellt außerdem eine Eigenschaft namens `Environment` bereit. Diese Eigenschaft unterstützt unterschiedliche Konfigurationen für eine Anwendung, die in verschiedenen Umgebungen wie Staging und Produktion ausgeführt wird.

Ein `ExternalConfigurationManager`-Objekt kann außerdem das `BlobSettingsStore`-Objekt in regelmäßigen Abständen nach Änderungen abfragen. Im folgenden Code ruft die `StartMonitor`-Methode `CheckForConfigurationChanges` in einem Intervall auf, um Änderungen zu erkennen und das `Changed`-Ereignis auszulösen, wie weiter oben beschrieben.

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

Die `ExternalConfigurationManager`-Klasse wird durch die unten gezeigte `ExternalConfiguration`-Klasse als Singletoninstanz instanziiert.

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

Der folgende Code entstammt der `WorkerRole`-Klasse im Projekt _ExternalConfigurationStore.Cloud_. Er zeigt, wie die Anwendung die `ExternalConfiguration`-Klasse zum Lesen einer Einstellung verwendet.

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

Der folgende Code, der ebenfalls aus der `WorkerRole`-Klasse stammt, zeigt, wie die Anwendung Konfigurationsereignisse abonniert.

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

- Ein Beispiel für dieses Muster steht auf [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).
