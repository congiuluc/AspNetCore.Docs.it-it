---
title: Provider di file in ASP.NET Core
author: guardrex
description: Informazioni su come ASP.NET Core astrae l'accesso al file system tramite l'utilizzo di provider di file.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/07/2019
uid: fundamentals/file-providers
ms.openlocfilehash: 3a92b44efc70d156596ee9fe80b4f6a65266e73d
ms.sourcegitcommit: 3d082bd46e9e00a3297ea0314582b1ed2abfa830
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/07/2019
ms.locfileid: "72007175"
---
# <a name="file-providers-in-aspnet-core"></a>Provider di file in ASP.NET Core

Di [Steve Smith](https://ardalis.com/) e [Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core astrae l'accesso al file system tramite l'utilizzo di provider di file. I provider di file vengono usati in tutto il framework di ASP.NET Core:

* `IWebHostEnvironment` espone la radice del [contenuto](xref:fundamentals/index#content-root) dell'app e la [radice web](xref:fundamentals/index#web-root) come tipi `IFileProvider`.
* Il [middleware dei file statici](xref:fundamentals/static-files) usa i provider di file per trovare i file statici.
* [Razor](xref:mvc/views/razor) usa i provider di file per individuare pagine e viste.
* Gli strumenti .NET Core usano i provider di file e i criteri GLOB per specificare i file da pubblicare.

[Visualizzare o scaricare il codice di esempio](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/file-providers/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="file-provider-interfaces"></a>Interfacce del provider di file

L'interfaccia primaria è <xref:Microsoft.Extensions.FileProviders.IFileProvider>. `IFileProvider` espone metodi per:

* Ottenere informazioni sui file (<xref:Microsoft.Extensions.FileProviders.IFileInfo>).
* Ottenere informazioni sulle directory (<xref:Microsoft.Extensions.FileProviders.IDirectoryContents>).
* Configurare le notifiche di modifica (usando un <xref:Microsoft.Extensions.Primitives.IChangeToken>).

`IFileInfo` fornisce metodi e proprietà per l'uso di file:

* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Exists>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.IsDirectory>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Name>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Length> (in byte)
* Data <xref:Microsoft.Extensions.FileProviders.IFileInfo.LastModified>

È possibile leggere dal file usando il metodo [IFileInfo.CreateReadStream](xref:Microsoft.Extensions.FileProviders.IFileInfo.CreateReadStream*).

L'app di esempio dimostra come configurare un provider di file in `Startup.ConfigureServices` da usare in tutta l'app tramite [inserimento delle dipendenze](xref:fundamentals/dependency-injection).

## <a name="file-provider-implementations"></a>Implementazioni dei provider di file

Sono disponibili tre implementazioni di `IFileProvider`.

| Implementazione | Descrizione |
| -------------- | ----------- |
| [PhysicalFileProvider](#physicalfileprovider) | Il provider fisico viene usato per accedere ai file fisici del sistema. |
| [ManifestEmbeddedFileProvider](#manifestembeddedfileprovider) | Il provider incorporato nel manifesto viene usato per accedere ai file incorporati negli assembly. |
| [CompositeFileProvider](#compositefileprovider) | Il provider composito consente di offrire un accesso combinato a file e directory da uno o più provider. |

### <a name="physicalfileprovider"></a>PhysicalFileProvider

<xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> consente di accedere al file system fisico. `PhysicalFileProvider` usa il tipo <xref:System.IO.File?displayProperty=fullName> (per il provider fisico) e definisce una directory e i relativi elementi figlio come ambito per tutti i percorsi. La definizione dell'ambito impedisce l'accesso al file system al di fuori della directory specificata e dei relativi elementi figlio. Lo scenario più comune per la creazione e l'uso di un `PhysicalFileProvider` consiste nel richiedere un oggetto `IFileProvider` in un costruttore tramite [inserimento delle dipendenze](xref:fundamentals/dependency-injection).

Per la creazione diretta di un'istanza di questo provider, è richiesto un percorso di directory che viene usato come percorso di base per tutte le richieste effettuate tramite il provider.

Il codice seguente illustra come creare un `PhysicalFileProvider` e usarlo per ottenere il contenuto della directory e informazioni sui file:

```csharp
var provider = new PhysicalFileProvider(applicationRoot);
var contents = provider.GetDirectoryContents(string.Empty);
var fileInfo = provider.GetFileInfo("wwwroot/js/site.js");
```

Tipi nell'esempio precedente:

* `provider` è `IFileProvider`.
* `contents` è `IDirectoryContents`.
* `fileInfo` è `IFileInfo`.

Il provider di file può essere usato per scorrere la directory specificata da `applicationRoot` oppure per chiamare `GetFileInfo` per ottenere informazioni su un file. Il provider di file non ha accesso all'esterno della directory `applicationRoot`.

L'app di esempio crea il provider nella classe `Startup.ConfigureServices` dell'app usando [IHostingEnvironment.ContentRootFileProvider](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootFileProvider):

```csharp
var physicalProvider = _env.ContentRootFileProvider;
```

### <a name="manifestembeddedfileprovider"></a>ManifestEmbeddedFileProvider

<xref:Microsoft.Extensions.FileProviders.ManifestEmbeddedFileProvider> consente di accedere ai file incorporati negli assembly. `ManifestEmbeddedFileProvider` usa un manifesto compilato nell'assembly per ricostruire i percorsi originali dei file incorporati.

Aggiungere un riferimento al pacchetto nel progetto per il pacchetto [Microsoft.Extensions.FileProviders.Embedded](https://www.nuget.org/packages/Microsoft.Extensions.FileProviders.Embedded).

Per generare un manifesto dei file incorporati, impostare la proprietà `<GenerateEmbeddedFilesManifest>` su `true`. Specificare i file da incorporare con [\<EmbeddedResource>](/dotnet/core/tools/csproj#default-compilation-includes-in-net-core-projects):

[!code-csharp[](file-providers/samples/3.x/FileProviderSample/FileProviderSample.csproj?highlight=6,14)]

Usare [modelli GLOB](#glob-patterns) per specificare uno o più file da incorporare nell'assembly.

L'app di esempio crea un `ManifestEmbeddedFileProvider` e passa l'assembly attualmente in esecuzione al rispettivo costruttore.

*Startup.cs*:

```csharp
var manifestEmbeddedProvider = 
    new ManifestEmbeddedFileProvider(Assembly.GetEntryAssembly());
```

Overload aggiuntivi consentono di:

* Specificare un percorso di file relativo.
* Definire la data dell'ultima modifica come ambito dei file.
* Assegnare un nome alla risorsa incorporata contenente il manifesto dei file incorporati.

| Overload | Descrizione |
| -------- | ----------- |
| `ManifestEmbeddedFileProvider(Assembly, String)` | Accetta un parametro di percorso relativo `root` facoltativo. Specificare `root` per definire come ambito delle chiamate a <xref:Microsoft.Extensions.FileProviders.IFileProvider.GetDirectoryContents*> le risorse incluse nel percorso specificato. |
| `ManifestEmbeddedFileProvider(Assembly, String, DateTimeOffset)` | Accetta un parametro di percorso relativo `root` facoltativo e un parametro di data `lastModified` (<xref:System.DateTimeOffset>). La data `lastModified` definisce la data dell'ultima modifica come ambito per le istanze di <xref:Microsoft.Extensions.FileProviders.IFileInfo> restituite da <xref:Microsoft.Extensions.FileProviders.IFileProvider>. |
| `ManifestEmbeddedFileProvider(Assembly, String, String, DateTimeOffset)` | Accetta un parametro di percorso relativo `root` facoltativo, un parametro di data `lastModified` e il parametro `manifestName`. `manifestName` rappresenta il nome della risorsa incorporata contenente il manifesto. |

### <a name="compositefileprovider"></a>CompositeFileProvider

<xref:Microsoft.Extensions.FileProviders.CompositeFileProvider> combina le istanze `IFileProvider` esponendo una sola interfaccia per l'uso dei file di più provider. Quando si crea `CompositeFileProvider`, passare una o più istanze `IFileProvider` al relativo costruttore.

Nell'app di esempio, un `PhysicalFileProvider` e un `ManifestEmbeddedFileProvider` forniscono i file a un `CompositeFileProvider` registrato nel contenitore dei servizi dell'app:

[!code-csharp[](file-providers/samples/3.x/FileProviderSample/Startup.cs?name=snippet1)]

## <a name="watch-for-changes"></a>Controllo delle modifiche

Il metodo [IFileProvider.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*) consente di controllare uno o più file o directory per il rilevamento delle modifiche. `Watch` accetta una stringa di percorso in cui è possibile usare [criteri GLOB](#glob-patterns) per specificare più file. `Watch` restituisce un oggetto <xref:Microsoft.Extensions.Primitives.IChangeToken>. Il token di modifica espone:

* <xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> &ndash; Una proprietà che può essere ispezionata per determinare se è stata apportata una modifica.
* <xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*> &ndash; Chiamato quando vengono rilevate modifiche alla stringa di percorso specificata. Ogni token di modifica chiama solo il relativo callback associato in risposta a una singola modifica. Per abilitare un monitoraggio continuo, usare <xref:System.Threading.Tasks.TaskCompletionSource`1> come illustrato di seguito oppure ricreare istanze di `IChangeToken` in risposta alle modifiche.

Nell'app di esempio, l'app console *WatchConsole* viene configurata per visualizzare un messaggio quando viene modificato un file di testo:

[!code-csharp[](file-providers/samples/3.x/WatchConsole/Program.cs?name=snippet1&highlight=1-2,16,19-20)]

È possibile che alcuni file system, ad esempio i contenitori Docker e le condivisioni di rete, non inviino sempre le notifiche di modifica. Impostare la variabile di ambiente `DOTNET_USE_POLLING_FILE_WATCHER` su `1` o `true` per eseguire il polling delle modifiche nel file system ogni quattro secondi (intervallo non configurabile).

## <a name="glob-patterns"></a>Modelli GLOB

Nei percorsi del file system vengono usati criteri con caratteri jolly chiamati *criteri GLOB (o globbing)* . Specificare gruppi di file con questi criteri. I due caratteri jolly sono `*` e `**`:

**`*`**  
Cerca qualsiasi elemento a livello della cartella corrente, qualsiasi nome di file o qualsiasi estensione di file. Le corrispondenze vengono terminate con i caratteri `/` e `.` nel percorso dei file.

**`**`**  
Cerca qualsiasi elemento in più livelli di directory. Può essere usato per cercare in modo ricorsivo numerosi file all'interno di una gerarchia di directory.

**Esempi di criteri GLOB**

**`directory/file.txt`**  
Cerca un file specifico in una directory specifica.

**`directory/*.txt`**  
Cerca tutti i file con estensione *txt* in una directory specifica.

**`directory/*/appsettings.json`**  
Cerca tutti i file `appsettings.json` nelle directory esattamente un livello sotto la cartella *directory*.

**`directory/**/*.txt`**  
Cerca tutti i file con estensione *txt* che si trovano in qualsiasi posizione nella cartella *directory*.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core astrae l'accesso al file system tramite l'utilizzo di provider di file. I provider di file vengono usati in tutto il framework di ASP.NET Core:

* <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> espone la radice del [contenuto](xref:fundamentals/index#content-root) dell'app e la [radice web](xref:fundamentals/index#web-root) come tipi `IFileProvider`.
* Il [middleware dei file statici](xref:fundamentals/static-files) usa i provider di file per trovare i file statici.
* [Razor](xref:mvc/views/razor) usa i provider di file per individuare pagine e viste.
* Gli strumenti .NET Core usano i provider di file e i criteri GLOB per specificare i file da pubblicare.

[Visualizzare o scaricare il codice di esempio](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/file-providers/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="file-provider-interfaces"></a>Interfacce del provider di file

L'interfaccia primaria è <xref:Microsoft.Extensions.FileProviders.IFileProvider>. `IFileProvider` espone metodi per:

* Ottenere informazioni sui file (<xref:Microsoft.Extensions.FileProviders.IFileInfo>).
* Ottenere informazioni sulle directory (<xref:Microsoft.Extensions.FileProviders.IDirectoryContents>).
* Configurare le notifiche di modifica (usando un <xref:Microsoft.Extensions.Primitives.IChangeToken>).

`IFileInfo` fornisce metodi e proprietà per l'uso di file:

* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Exists>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.IsDirectory>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Name>
* <xref:Microsoft.Extensions.FileProviders.IFileInfo.Length> (in byte)
* Data <xref:Microsoft.Extensions.FileProviders.IFileInfo.LastModified>

È possibile leggere dal file usando il metodo [IFileInfo.CreateReadStream](xref:Microsoft.Extensions.FileProviders.IFileInfo.CreateReadStream*).

L'app di esempio dimostra come configurare un provider di file in `Startup.ConfigureServices` da usare in tutta l'app tramite [inserimento delle dipendenze](xref:fundamentals/dependency-injection).

## <a name="file-provider-implementations"></a>Implementazioni dei provider di file

Sono disponibili tre implementazioni di `IFileProvider`.

| Implementazione | Descrizione |
| -------------- | ----------- |
| [PhysicalFileProvider](#physicalfileprovider) | Il provider fisico viene usato per accedere ai file fisici del sistema. |
| [ManifestEmbeddedFileProvider](#manifestembeddedfileprovider) | Il provider incorporato nel manifesto viene usato per accedere ai file incorporati negli assembly. |
| [CompositeFileProvider](#compositefileprovider) | Il provider composito consente di offrire un accesso combinato a file e directory da uno o più provider. |

### <a name="physicalfileprovider"></a>PhysicalFileProvider

<xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> consente di accedere al file system fisico. `PhysicalFileProvider` usa il tipo <xref:System.IO.File?displayProperty=fullName> (per il provider fisico) e definisce una directory e i relativi elementi figlio come ambito per tutti i percorsi. La definizione dell'ambito impedisce l'accesso al file system al di fuori della directory specificata e dei relativi elementi figlio. Lo scenario più comune per la creazione e l'uso di un `PhysicalFileProvider` consiste nel richiedere un oggetto `IFileProvider` in un costruttore tramite [inserimento delle dipendenze](xref:fundamentals/dependency-injection).

Per la creazione diretta di un'istanza di questo provider, è richiesto un percorso di directory che viene usato come percorso di base per tutte le richieste effettuate tramite il provider.

Il codice seguente illustra come creare un `PhysicalFileProvider` e usarlo per ottenere il contenuto della directory e informazioni sui file:

```csharp
var provider = new PhysicalFileProvider(applicationRoot);
var contents = provider.GetDirectoryContents(string.Empty);
var fileInfo = provider.GetFileInfo("wwwroot/js/site.js");
```

Tipi nell'esempio precedente:

* `provider` è `IFileProvider`.
* `contents` è `IDirectoryContents`.
* `fileInfo` è `IFileInfo`.

Il provider di file può essere usato per scorrere la directory specificata da `applicationRoot` oppure per chiamare `GetFileInfo` per ottenere informazioni su un file. Il provider di file non ha accesso all'esterno della directory `applicationRoot`.

L'app di esempio crea il provider nella classe `Startup.ConfigureServices` dell'app usando [IHostingEnvironment.ContentRootFileProvider](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootFileProvider):

```csharp
var physicalProvider = _env.ContentRootFileProvider;
```

### <a name="manifestembeddedfileprovider"></a>ManifestEmbeddedFileProvider

<xref:Microsoft.Extensions.FileProviders.ManifestEmbeddedFileProvider> consente di accedere ai file incorporati negli assembly. `ManifestEmbeddedFileProvider` usa un manifesto compilato nell'assembly per ricostruire i percorsi originali dei file incorporati.

Per generare un manifesto dei file incorporati, impostare la proprietà `<GenerateEmbeddedFilesManifest>` su `true`. Specificare i file da incorporare con [&lt;EmbeddedResource&gt;](/dotnet/core/tools/csproj#default-compilation-includes-in-net-core-projects):

[!code-csharp[](file-providers/samples/2.x/FileProviderSample/FileProviderSample.csproj?highlight=6,14)]

Usare [modelli GLOB](#glob-patterns) per specificare uno o più file da incorporare nell'assembly.

L'app di esempio crea un `ManifestEmbeddedFileProvider` e passa l'assembly attualmente in esecuzione al rispettivo costruttore.

*Startup.cs*:

```csharp
var manifestEmbeddedProvider = 
    new ManifestEmbeddedFileProvider(Assembly.GetEntryAssembly());
```

Overload aggiuntivi consentono di:

* Specificare un percorso di file relativo.
* Definire la data dell'ultima modifica come ambito dei file.
* Assegnare un nome alla risorsa incorporata contenente il manifesto dei file incorporati.

| Overload | Descrizione |
| -------- | ----------- |
| `ManifestEmbeddedFileProvider(Assembly, String)` | Accetta un parametro di percorso relativo `root` facoltativo. Specificare `root` per definire come ambito delle chiamate a <xref:Microsoft.Extensions.FileProviders.IFileProvider.GetDirectoryContents*> le risorse incluse nel percorso specificato. |
| `ManifestEmbeddedFileProvider(Assembly, String, DateTimeOffset)` | Accetta un parametro di percorso relativo `root` facoltativo e un parametro di data `lastModified` (<xref:System.DateTimeOffset>). La data `lastModified` definisce la data dell'ultima modifica come ambito per le istanze di <xref:Microsoft.Extensions.FileProviders.IFileInfo> restituite da <xref:Microsoft.Extensions.FileProviders.IFileProvider>. |
| `ManifestEmbeddedFileProvider(Assembly, String, String, DateTimeOffset)` | Accetta un parametro di percorso relativo `root` facoltativo, un parametro di data `lastModified` e il parametro `manifestName`. `manifestName` rappresenta il nome della risorsa incorporata contenente il manifesto. |

### <a name="compositefileprovider"></a>CompositeFileProvider

<xref:Microsoft.Extensions.FileProviders.CompositeFileProvider> combina le istanze `IFileProvider` esponendo una sola interfaccia per l'uso dei file di più provider. Quando si crea `CompositeFileProvider`, passare una o più istanze `IFileProvider` al relativo costruttore.

Nell'app di esempio, un `PhysicalFileProvider` e un `ManifestEmbeddedFileProvider` forniscono i file a un `CompositeFileProvider` registrato nel contenitore dei servizi dell'app:

[!code-csharp[](file-providers/samples/2.x/FileProviderSample/Startup.cs?name=snippet1)]

## <a name="watch-for-changes"></a>Controllo delle modifiche

Il metodo [IFileProvider.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*) consente di controllare uno o più file o directory per il rilevamento delle modifiche. `Watch` accetta una stringa di percorso in cui è possibile usare [criteri GLOB](#glob-patterns) per specificare più file. `Watch` restituisce un oggetto <xref:Microsoft.Extensions.Primitives.IChangeToken>. Il token di modifica espone:

* <xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> &ndash; Una proprietà che può essere ispezionata per determinare se è stata apportata una modifica.
* <xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*> &ndash; Chiamato quando vengono rilevate modifiche alla stringa di percorso specificata. Ogni token di modifica chiama solo il relativo callback associato in risposta a una singola modifica. Per abilitare un monitoraggio continuo, usare <xref:System.Threading.Tasks.TaskCompletionSource`1> come illustrato di seguito oppure ricreare istanze di `IChangeToken` in risposta alle modifiche.

Nell'app di esempio, l'app console *WatchConsole* viene configurata per visualizzare un messaggio quando viene modificato un file di testo:

[!code-csharp[](file-providers/samples/2.x/WatchConsole/Program.cs?name=snippet1&highlight=1-2,16,19-20)]

È possibile che alcuni file system, ad esempio i contenitori Docker e le condivisioni di rete, non inviino sempre le notifiche di modifica. Impostare la variabile di ambiente `DOTNET_USE_POLLING_FILE_WATCHER` su `1` o `true` per eseguire il polling delle modifiche nel file system ogni quattro secondi (intervallo non configurabile).

## <a name="glob-patterns"></a>Modelli GLOB

Nei percorsi del file system vengono usati criteri con caratteri jolly chiamati *criteri GLOB (o globbing)* . Specificare gruppi di file con questi criteri. I due caratteri jolly sono `*` e `**`:

**`*`**  
Cerca qualsiasi elemento a livello della cartella corrente, qualsiasi nome di file o qualsiasi estensione di file. Le corrispondenze vengono terminate con i caratteri `/` e `.` nel percorso dei file.

**`**`**  
Cerca qualsiasi elemento in più livelli di directory. Può essere usato per cercare in modo ricorsivo numerosi file all'interno di una gerarchia di directory.

**Esempi di criteri GLOB**

**`directory/file.txt`**  
Cerca un file specifico in una directory specifica.

**`directory/*.txt`**  
Cerca tutti i file con estensione *txt* in una directory specifica.

**`directory/*/appsettings.json`**  
Cerca tutti i file `appsettings.json` nelle directory esattamente un livello sotto la cartella *directory*.

**`directory/**/*.txt`**  
Cerca tutti i file con estensione *txt* che si trovano in qualsiasi posizione nella cartella *directory*.

::: moniker-end
