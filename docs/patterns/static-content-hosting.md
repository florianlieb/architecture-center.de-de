---
title: Hosten von statischen Inhalten
description: "Stellen Sie statische Inhalte in einem cloudbasierten Speicherdienst bereit, der die Inhalte direkt an den Client übermitteln kann."
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: deb15001bea2598d56a2793be78bbc3e7473bdf3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="static-content-hosting-pattern"></a>Muster für das Hosten von statischen Inhalten

[!INCLUDE [header](../_includes/header.md)]

Stellen Sie statische Inhalte in einem cloudbasierten Speicherdienst bereit, der die Inhalte direkt an den Client übermitteln kann. Dies kann den Bedarf an potenziell kostspieligen Compute-Instanzen reduzieren.

## <a name="context-and-problem"></a>Kontext und Problem

Webanwendungen enthalten i.d.R. auch statische Inhalte. Diese statischen Inhalte umfassen möglicherweise HTML-Seiten und andere Ressourcen, z.B. Bilder und Dokumente, die für den Client verfügbar sind, entweder als Teil einer HTML-Seite (z.B. Inlinebilder, Formatvorlagen und clientseitige JavaScript-Dateien) oder als eigene Downloads (z.B. PDF-Dokumente).

Obwohl Webserver Anforderungen durch die effiziente Ausführung von Code für dynamische Seiten und die Zwischenspeicherung der Ausgabe optimieren können, müssen sie Anforderungen zum Herunterladen von statischen Inhalten verarbeiten. Dies beansprucht Verarbeitungszyklen, die häufig besser genutzt werden können.

## <a name="solution"></a>Lösung

In den meisten Cloudhostingumgebungen kann der Bedarf an Compute-Instanzen minimiert werden (z.B. Verwendung einer kleineren Instanz oder einer geringeren Anzahl von Instanzen), indem einige der Ressourcen und statischen Seiten einer Anwendung in einer Speicherinstanz untergebracht werden. Die Kosten für in der Cloud gehosteten Speicher sind in der Regel weitaus geringer als die Kosten für Compute-Instanzen.

Wenn einige Teile einer Anwendung in einem Speicherdienst gehostet werden, beziehen sich die wichtigsten Überlegungen auf die Bereitstellung der Anwendung und das Sichern der Ressourcen, die nicht für anonyme Benutzer verfügbar sein sollen.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:

- Der gehostete Speicherdienst muss einen HTTP-Endpunkt verfügbar machen, auf den Benutzer zugreifen können, um die statischen Ressourcen herunterzuladen. Einige Speicherdienste unterstützen auch HTTPS, daher können Ressourcen in Speicherdiensten gehostet werden, die SSL erfordern.

- Für maximale Leistung und Verfügbarkeit empfiehlt es sich, ein Content Delivery Network (CDN) zu verwenden, um die Inhalte des Speichercontainers in mehreren Datencentern auf der ganzen Welt zwischenzuspeichern. Wahrscheinlich ist CDN jedoch für Sie kostenpflichtig.

- Speicherkonten werden häufig standardmäßig georepliziert, um Resilienz im Fall von Ereignissen zu bieten, die sich auf das Datencenter auswirken können. Dies bedeutet, dass sich die IP-Adresse ändern kann, die URL bleibt aber die gleiche.

- Wenn sich einige Inhalte in einem Speicherkonto und andere Inhalte in einer gehosteten Compute-Instanz befinden, wird es schwieriger, eine Anwendung bereitzustellen und zu aktualisieren. Möglicherweise müssen Sie getrennte Bereitstellungen durchführen und Versionen der Anwendung und Inhalte erstellen, um die Verwaltung zu vereinfachen – insbesondere, wenn die statischen Inhalte Skriptdateien oder UI-Komponenten umfassen. Wenn jedoch nur statische Ressourcen aktualisiert werden müssen, können sie einfach in das Speicherkonto hochgeladen werden, ohne das Anwendungspaket erneut bereitzustellen.

- Speicherdienste unterstützen möglicherweise nicht die Verwendung von benutzerdefinierten Domänennamen. In diesem Fall muss in Links die vollständige URL der Ressourcen angegeben werden, da sie sich in einer anderen Domäne als der dynamisch generierte Inhalt befinden, der die Links enthält.

- Die Speichercontainer müssen für öffentlichen Lesezugriff konfiguriert werden. Um das Hochladen von Inhalten durch Benutzer zu verhindern, muss jedoch unbedingt sichergestellt werden, dass sie nicht für den öffentlichen Schreibzugriff konfiguriert sind. Verwenden Sie einen Valet-Schlüssel oder ein Token zum Steuern des Zugriffs auf Ressourcen, die nicht anonym verfügbar sein sollten. Weitere Informationen finden Sie unter [Valet-Schlüssel-Muster](valet-key.md).

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Dieses Muster ist hilfreich:

- Zum Minimieren der Hostingkosten für Websites und Anwendungen, die statische Ressourcen enthalten.

- Zum Minimieren der Hostingkosten für Websites, die nur aus statischen Inhalten und Ressourcen bestehen. Abhängig von den Funktionen des Speichersystems des Hostinganbieters kann möglicherweise eine gesamte vollständig statische Website in einem Speicherkonto gehostet werden.

- Zum Verfügbarmachen statischer Ressourcen und Inhalte für Anwendungen, die in anderen Hostumgebungen oder auf lokalen Servern ausgeführt werden.

- Zum Suchen von Inhalten in mehreren geografischen Regionen mithilfe eines Content Delivery Network, das die Inhalte des Speicherkontos in mehreren Datencentern auf der ganzen Welt zwischenspeichert.

- Zum Überwachen von Kosten und Bandbreitennutzung. Durch die Verwendung eines eigenen Speicherkontos für einige oder alle statischen Inhalte lassen sich die Hosting- und Laufzeitkosten leichter trennen.

Dieses Muster ist in den folgenden Situationen eventuell nicht hilfreich:

- Die Anwendung muss statische Inhalte vor der Übermittlung an den Client verarbeiten. Beispielsweise kann es erforderlich, einem Dokument einen Zeitstempel hinzuzufügen.

- Die Menge statischer Inhalte ist sehr gering. Der Mehraufwand für das Abrufen dieser Inhalte aus einem eigenen Speicherkonto überwiegt möglicherweise den Kostenvorteil durch die Trennung von der Compute-Ressource.

## <a name="example"></a>Beispiel

Auf statische Inhalte in Azure Blob Storage kann von einem Webbrowser direkt zugegriffen werden. Azure stellt eine HTTP-basierte Schnittstelle für Speicher bereit, die für Clients öffentlich verfügbar gemacht werden kann. Beispielsweise können Inhalte in einem Azure Blob Storage-Container über eine URL im folgenden Format verfügbar gemacht werden:

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


Beim Hochladen der Inhalte müssen einer oder mehrere BLOB-Container für die Dateien und Dokumente erstellt werden. Beachten Sie, dass die Standardberechtigung für einen neuen Container „Privat“ lautet und in „Öffentlich“ geändert werden muss, um Clients Zugriff auf die Inhalte zu gewähren. Wenn die Inhalte vor anonymem Zugriff geschützt werden müssen, können Sie das [Valet-Schlüssel-Muster](valet-key.md) implementieren, damit Benutzer ein gültiges Token vorlegen müssen, um die Ressourcen herunterzuladen.

> In [Konzepte des Blob-Diensts](https://msdn.microsoft.com/library/azure/dd179376.aspx) finden Sie Informationen zu Blob Storage und den Verfahren, mit denen Sie auf diesen Dienst zugreifen und ihn verwenden können.

Die Links auf den einzelnen Seiten geben die URL der Ressource an, und der Client greift direkt aus dem Speicherdienst auf sie zu. Die Abbildung veranschaulicht die direkte Übermittlung von statischen Teilen einer Anwendung aus einem Speicherdienst.

![Abbildung 1 – Direkte Übermittlung von statischen Teilen einer Anwendung aus einem Speicherdienst](./_images/static-content-hosting-pattern.png)


Die Links auf den Seiten, die an den Client übermittelt werden, müssen die vollständige URL des BLOB-Containers und der Ressource angeben. Beispielsweise kann eine Seite, die einen Link zu einem Bild in einem öffentlichen Container enthält, den folgenden HTML-Code enthalten.

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> Wenn die Ressourcen durch einen Valet-Schlüssel, z.B. eine Azure SAS, geschützt sind, muss diese Signatur in den URLs der Links enthalten sein.

In [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) ist eine Lösung mit dem Namen „StaticContentHosting“ verfügbar, die das Verwenden von externem Speicher für statische Ressourcen veranschaulicht. Das Projekt „StaticContentHosting.Cloud“ enthält Konfigurationsdateien, die das Speicherkonto und den Container angeben, in dem sich die statischen Inhalte befinden.

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

Die `Settings`-Klasse in der Datei „Settings.cs“ des Projekts „StaticContentHosting.Web“ enthält Methoden zum Extrahieren dieser Werte und zum Erstellen eines Zeichenfolgenwerts, der die URL des Cloudspeicherkonto-Containers enthält.

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

Die `StaticContentUrlHtmlHelper`-Klasse in der Datei „StaticContentUrlHtmlHelper.cs“ macht die Methode `StaticContentUrl` verfügbar. Diese generiert eine URL, die den Pfad zu dem Cloudspeicherkonto enthält, wenn die an sie übergebene URL mit dem ASP.NET-Zeichen für das Stammverzeichnis (~) beginnt.

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

Die Datei „Index.cshtml“ im Ordner „Views\Home“ enthält ein image-Element, das mit der `StaticContentUrl`-Methode die URL für das `src`-Attribut erstellt.

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

- Ein Beispiel für dieses Muster steht auf [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).
- [Valet-Schlüssel-Muster](valet-key.md). Wenn die Zielressourcen nicht für anonyme Benutzer verfügbar sein sollen, muss Sicherheit für den Speicher implementiert werden, in dem sich die statischen Inhalte befinden. In diesem Artikel wird beschrieben, wie ein Token oder Schlüssel verwendet wird, das bzw. der Clients eingeschränkten Direktzugriff auf eine bestimmte Ressource oder einen bestimmten Dienst, z.B. einen in der Cloud gehosteten Speicherdienst, bietet.
- [An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) (Eine effiziente Möglichkeit zum Bereitstellen einer statischen Website in Azure, in englischer Sprache) im Infosys-Blog.
- [Konzepte des Blob-Diensts](https://msdn.microsoft.com/library/azure/dd179376.aspx)
