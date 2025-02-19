---
title: Configurazione di ASP.NET Core SignalR
author: bradygaster
description: Informazioni su come configurare ASP.NET Core le app SignalR.
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 08/05/2019
uid: signalr/configuration
ms.openlocfilehash: 66f274fcda27392091de6b4be8c7221bc87b7585
ms.sourcegitcommit: c452e6af92e130413106c4863193f377cde4cd9c
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/10/2019
ms.locfileid: "72246489"
---
# <a name="aspnet-core-signalr-configuration"></a>Configurazione di ASP.NET Core SignalR

## <a name="jsonmessagepack-serialization-options"></a>Opzioni di serializzazione JSON/MessagePack

ASP.NET Core SignalR supporta due protocolli per la codifica dei messaggi: [JSON](https://www.json.org/) e [MessagePack](https://msgpack.org/index.html). Ogni protocollo dispone di opzioni di configurazione della serializzazione.

::: moniker range=">= aspnetcore-3.0"

La serializzazione JSON può essere configurata nel server usando il metodo di estensione [AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol) . è possibile aggiungere `AddJsonProtocol` dopo [AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr) in `Startup.ConfigureServices`. Il metodo `AddJsonProtocol` accetta un delegato che riceve un oggetto `options`. La proprietà [PayloadSerializerOptions](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializeroptions) su tale oggetto è un oggetto `System.Text.Json` <xref:System.Text.Json.JsonSerializerOptions> che può essere utilizzato per configurare la serializzazione di argomenti e valori restituiti. Per ulteriori informazioni, vedere la [documentazione di System. Text. JSON](/dotnet/api/system.text.json).

Ad esempio, per configurare il serializzatore in modo da non modificare la combinazione di maiuscole e minuscole dei nomi di proprietà, anziché i nomi predefiniti "camelCase", usare il codice seguente in `Startup.ConfigureServices`:

```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerOptions.PropertyNamingPolicy = null
    });
```

Nel client .NET è presente lo stesso metodo di estensione `AddJsonProtocol` in [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder). Per risolvere il metodo di estensione, è necessario importare lo spazio dei nomi `Microsoft.Extensions.DependencyInjection`:

```csharp
// At the top of the file:
using Microsoft.Extensions.DependencyInjection;

// When constructing your connection:
var connection = new HubConnectionBuilder()
    .AddJsonProtocol(options => {
        options.PayloadSerializerOptions.PropertyNamingPolicy = null;
    })
    .Build();
```

### <a name="switch-to-newtonsoftjson"></a>Passa a Newtonsoft. JSON

Se sono necessarie funzionalità di `Newtonsoft.Json` che non sono supportate in `System.Text.Json`, vedere [passare a Newtonsoft. JSON](xref:migration/22-to-30#switch-to-newtonsoftjson).

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

La serializzazione JSON può essere configurata nel server usando il metodo di estensione [AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol) , che può essere aggiunto dopo [AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr) nel metodo `Startup.ConfigureServices`. Il metodo `AddJsonProtocol` accetta un delegato che riceve un oggetto `options`. La proprietà [PayloadSerializerSettings](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializersettings) su tale oggetto è un oggetto JSON.NET `JsonSerializerSettings` che può essere utilizzato per configurare la serializzazione di argomenti e valori restituiti. Per ulteriori informazioni, vedere la [documentazione di JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm).
 
Per configurare, ad esempio, il serializzatore per l'utilizzo di nomi di proprietà "PascalCase", anziché i nomi predefiniti "camelCase", utilizzare il codice seguente in `Startup.ConfigureServices`:
 
```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver =
            new DefaultContractResolver();
    });
```

Nel client .NET è presente lo stesso metodo di estensione `AddJsonProtocol` in [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder). Per risolvere il metodo di estensione, è necessario importare lo spazio dei nomi `Microsoft.Extensions.DependencyInjection`:

```csharp
// At the top of the file:
using Microsoft.Extensions.DependencyInjection;
 
// When constructing your connection:
var connection = new HubConnectionBuilder()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver =
            new DefaultContractResolver();
    })
    .Build();
```

::: moniker-end

> [!NOTE]
> Al momento non è possibile configurare la serializzazione JSON nel client JavaScript.

### <a name="messagepack-serialization-options"></a>Opzioni di serializzazione MessagePack

La serializzazione MessagePack può essere configurata fornendo un delegato alla chiamata [AddMessagePackProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.msgpackprotocoldependencyinjectionextensions.addmessagepackprotocol) . Per ulteriori informazioni, vedere [MessagePack in SignalR](xref:signalr/messagepackhubprotocol) .

> [!NOTE]
> Al momento non è possibile configurare la serializzazione MessagePack nel client JavaScript.

## <a name="configure-server-options"></a>Configurare le opzioni del server

La tabella seguente descrive le opzioni per la configurazione degli hub SignalR:

::: moniker range=">= aspnetcore-3.0"

| Opzione | Default Value | DESCRIZIONE |
| ------ | ------------- | ----------- |
| `ClientTimeoutInterval` | 30 secondi | Il server prenderà in considerazione il client disconnesso se non ha ricevuto un messaggio (incluso Keep-Alive) in questo intervallo. Potrebbe essere necessario più tempo di questo intervallo di timeout affinché il client venga effettivamente contrassegnato come disconnesso, a causa della modalità di implementazione. Il valore consigliato è Double il valore `KeepAliveInterval`.|
| `HandshakeTimeout` | 15 secondi | Se il client non invia un messaggio di handshake iniziale entro questo intervallo di tempo, la connessione viene chiusa. Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano errori di timeout dell'handshake a causa di una latenza di rete grave. Per informazioni dettagliate sul processo di handshake, vedere la [specifica del protocollo dell'hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |
| `KeepAliveInterval` | 15 secondi | Se il server non ha inviato un messaggio entro questo intervallo, viene inviato automaticamente un messaggio di ping per lasciare aperta la connessione. Quando si modifica `KeepAliveInterval`, modificare l'impostazione `ServerTimeout` @ no__t-2 @ no__t-3 nel client. Il valore `ServerTimeout` @ no__t-1 @ no__t-2 consigliato è il doppio del valore `KeepAliveInterval`.  |
| `SupportedProtocols` | Tutti i protocolli installati | Protocolli supportati da questo hub. Per impostazione predefinita, tutti i protocolli registrati nel server sono consentiti, ma i protocolli possono essere rimossi da questo elenco per disabilitare protocolli specifici per i singoli Hub. |
| `EnableDetailedErrors` | `false` | Se `true`, i messaggi di eccezione dettagliati vengono restituiti ai client quando viene generata un'eccezione in un metodo dell'hub. Il valore predefinito è `false`, in quanto questi messaggi di eccezione possono contenere informazioni riservate. |
| `StreamBufferCapacity` | `10` | Numero massimo di elementi che possono essere memorizzati nel buffer per i flussi di caricamento client. Se viene raggiunto questo limite, l'elaborazione delle chiamate viene bloccata fino a quando il server non elabora gli elementi del flusso.|
| `MaximumReceiveMessageSize` | 32 KB | Dimensione massima di un singolo messaggio dell'hub in ingresso. |

::: moniker-end

::: moniker range="= aspnetcore-2.2"

| Opzione | Default Value | DESCRIZIONE |
| ------ | ------------- | ----------- |
| `ClientTimeoutInterval` | 30 secondi | Il server prenderà in considerazione il client disconnesso se non ha ricevuto un messaggio (incluso Keep-Alive) in questo intervallo. Potrebbe essere necessario più tempo di questo intervallo di timeout affinché il client venga effettivamente contrassegnato come disconnesso, a causa della modalità di implementazione. Il valore consigliato è Double il valore `KeepAliveInterval`.|
| `HandshakeTimeout` | 15 secondi | Se il client non invia un messaggio di handshake iniziale entro questo intervallo di tempo, la connessione viene chiusa. Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano errori di timeout dell'handshake a causa di una latenza di rete grave. Per informazioni dettagliate sul processo di handshake, vedere la [specifica del protocollo dell'hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |
| `KeepAliveInterval` | 15 secondi | Se il server non ha inviato un messaggio entro questo intervallo, viene inviato automaticamente un messaggio di ping per lasciare aperta la connessione. Quando si modifica `KeepAliveInterval`, modificare l'impostazione `ServerTimeout` @ no__t-2 @ no__t-3 nel client. Il valore `ServerTimeout` @ no__t-1 @ no__t-2 consigliato è il doppio del valore `KeepAliveInterval`.  |
| `SupportedProtocols` | Tutti i protocolli installati | Protocolli supportati da questo hub. Per impostazione predefinita, tutti i protocolli registrati nel server sono consentiti, ma i protocolli possono essere rimossi da questo elenco per disabilitare protocolli specifici per i singoli Hub. |
| `EnableDetailedErrors` | `false` | Se `true`, i messaggi di eccezione dettagliati vengono restituiti ai client quando viene generata un'eccezione in un metodo dell'hub. Il valore predefinito è `false`, in quanto questi messaggi di eccezione possono contenere informazioni riservate. |

::: moniker-end

::: moniker range="= aspnetcore-2.1"

| Opzione | Default Value | Descrizione |
| ------ | ------------- | ----------- |
| `HandshakeTimeout` | 15 secondi | Se il client non invia un messaggio di handshake iniziale entro questo intervallo di tempo, la connessione viene chiusa. Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano errori di timeout dell'handshake a causa di una latenza di rete grave. Per informazioni dettagliate sul processo di handshake, vedere la [specifica del protocollo dell'hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |
| `KeepAliveInterval` | 15 secondi | Se il server non ha inviato un messaggio entro questo intervallo, viene inviato automaticamente un messaggio di ping per lasciare aperta la connessione. Quando si modifica `KeepAliveInterval`, modificare l'impostazione `ServerTimeout` @ no__t-2 @ no__t-3 nel client. Il valore `ServerTimeout` @ no__t-1 @ no__t-2 consigliato è il doppio del valore `KeepAliveInterval`.  |
| `SupportedProtocols` | Tutti i protocolli installati | Protocolli supportati da questo hub. Per impostazione predefinita, tutti i protocolli registrati nel server sono consentiti, ma i protocolli possono essere rimossi da questo elenco per disabilitare protocolli specifici per i singoli Hub. |
| `EnableDetailedErrors` | `false` | Se `true`, i messaggi di eccezione dettagliati vengono restituiti ai client quando viene generata un'eccezione in un metodo dell'hub. Il valore predefinito è `false`, in quanto questi messaggi di eccezione possono contenere informazioni riservate. |

::: moniker-end

È possibile configurare le opzioni per tutti gli hub fornendo un delegato Options per la chiamata `AddSignalR` in `Startup.ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR(hubOptions =>
    {
        hubOptions.EnableDetailedErrors = true;
        hubOptions.KeepAliveInterval = TimeSpan.FromMinutes(1);
    });
}
```

Le opzioni per un singolo Hub eseguono l'override delle opzioni globali fornite in `AddSignalR` e possono essere configurate con <xref:Microsoft.Extensions.DependencyInjection.SignalRDependencyInjectionExtensions.AddHubOptions*>:

```csharp
services.AddSignalR().AddHubOptions<MyHub>(options =>
{
    options.EnableDetailedErrors = true;
});
```

### <a name="advanced-http-configuration-options"></a>Opzioni di configurazione HTTP avanzate

::: moniker range=">= aspnetcore-3.0"

Utilizzare `HttpConnectionDispatcherOptions` per configurare le impostazioni avanzate relative ai trasporti e alla gestione dei buffer di memoria. Queste opzioni vengono configurate passando un delegato a [MapHub @ no__t-1T >](/dotnet/api/microsoft.aspnetcore.builder.hubendpointroutebuilderextensions.maphub) in `Startup.Configure`.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<MyHub>("/myhub", options =>
        {
            options.Transports =
                HttpTransportType.WebSockets |
                HttpTransportType.LongPolling;
        });
    });
}
```
::: moniker-end

::: moniker range="<= aspnetcore-2.2"

Utilizzare `HttpConnectionDispatcherOptions` per configurare le impostazioni avanzate relative ai trasporti e alla gestione dei buffer di memoria. Queste opzioni vengono configurate passando un delegato a [MapHub @ no__t-1T >](/dotnet/api/microsoft.aspnetcore.signalr.hubroutebuilder.maphub) in `Startup.Configure`.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseSignalR((configure) =>
    {
        var desiredTransports =
            HttpTransportType.WebSockets |
            HttpTransportType.LongPolling;

        configure.MapHub<MyHub>("/myhub", (options) =>
        {
            options.Transports = desiredTransports;
        });
    });
}
```

::: moniker-end

La tabella seguente descrive le opzioni per la configurazione delle opzioni HTTP avanzate di ASP.NET Core SignalR:

::: moniker range=">= aspnetcore-3.0"

| Opzione | Default Value | DESCRIZIONE |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | Il numero massimo di byte ricevuti dal client che il server memorizza nel buffer prima di applicare la contropressione. L'aumento di questo valore consente al server di ricevere più rapidamente messaggi di dimensioni maggiori senza applicare la sovrapressione, ma può aumentare l'utilizzo della memoria. |
| `AuthorizationData` | Dati raccolti automaticamente dagli attributi `Authorize` applicati alla classe Hub. | Elenco di oggetti [IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata) usati per determinare se un client è autorizzato a connettersi all'hub. |
| `TransportMaxBufferSize` | 32 KB | Numero massimo di byte inviati dall'app che il server memorizza nel buffer prima di osservare la sovrapressione. L'aumento di questo valore consente al server di memorizzare nel buffer i messaggi di dimensioni maggiori più rapidamente senza attendere la sovrapressione, ma può aumentare l'utilizzo della memoria. |
| `Transports` | Tutti i trasporti sono abilitati. | Enumerazione di flag di bit di valori `HttpTransportType` che possono limitare i trasporti che un client può utilizzare per la connessione. |
| `LongPolling` | come illustrato più avanti. | Opzioni aggiuntive specifiche per il trasporto di polling lungo. |
| `WebSockets` | come illustrato più avanti. | Opzioni aggiuntive specifiche per il trasporto WebSocket. |

::: moniker-end

::: moniker range="< aspnetcore-3.0"

| Opzione | Default Value | DESCRIZIONE |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | Numero massimo di byte ricevuti dal client che il server memorizza nel buffer. L'aumento di questo valore consente al server di ricevere messaggi di dimensioni maggiori, ma può influire negativamente sull'utilizzo della memoria. |
| `AuthorizationData` | Dati raccolti automaticamente dagli attributi `Authorize` applicati alla classe Hub. | Elenco di oggetti [IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata) usati per determinare se un client è autorizzato a connettersi all'hub. |
| `TransportMaxBufferSize` | 32 KB | Numero massimo di byte inviati dall'app che il server memorizza nel buffer. L'aumento di questo valore consente al server di inviare messaggi di dimensioni maggiori, ma può influire negativamente sull'utilizzo della memoria. |
| `Transports` | Tutti i trasporti sono abilitati. | Enumerazione di flag di bit di valori `HttpTransportType` che possono limitare i trasporti che un client può utilizzare per la connessione. |
| `LongPolling` | come illustrato più avanti. | Opzioni aggiuntive specifiche per il trasporto di polling lungo. |
| `WebSockets` | come illustrato più avanti. | Opzioni aggiuntive specifiche per il trasporto WebSocket. |

::: moniker-end

Il trasporto di polling lungo dispone di opzioni aggiuntive che possono essere configurate utilizzando la proprietà `LongPolling`:

| Opzione | Default Value | Descrizione |
| ------ | ------------- | ----------- |
| `PollTimeout` | 90 secondi | Quantità massima di tempo di attesa del server per l'invio di un messaggio al client prima che venga terminata una singola richiesta di polling. La riduzione di questo valore determina la frequenza con cui il client rilascia più richieste di polling. |

Il trasporto WebSocket dispone di opzioni aggiuntive che possono essere configurate con la proprietà `WebSockets`:

| Opzione | Default Value | DESCRIZIONE |
| ------ | ------------- | ----------- |
| `CloseTimeout` | 5 secondi | Dopo la chiusura del server, se il client non riesce a chiudersi entro questo intervallo di tempo, la connessione viene terminata. |
| `SubProtocolSelector` | `null` | Delegato che può essere utilizzato per impostare l'intestazione `Sec-WebSocket-Protocol` su un valore personalizzato. Il delegato riceve i valori richiesti dal client come input ed è previsto che restituisca il valore desiderato. |

## <a name="configure-client-options"></a>Configurare le opzioni client

Le opzioni client possono essere configurate nel tipo `HubConnectionBuilder`, disponibile nei client .NET e JavaScript. È anche disponibile nel client Java, ma la sottoclasse `HttpHubConnectionBuilder` contiene le opzioni di configurazione del compilatore, oltre a `HubConnection`.

### <a name="configure-logging"></a>Configurare la registrazione

La registrazione viene configurata nel client .NET utilizzando il metodo `ConfigureLogging`. I provider di registrazione e i filtri possono essere registrati nello stesso modo in cui si trovano nel server. Per ulteriori informazioni, vedere la documentazione relativa all' [accesso ASP.NET Core](xref:fundamentals/logging/index) .

> [!NOTE]
> Per registrare i provider di registrazione, è necessario installare i pacchetti necessari. Per un elenco completo, vedere la sezione [provider di registrazione incorporati](xref:fundamentals/logging/index#built-in-logging-providers) di docs.

Ad esempio, per abilitare la registrazione della console, installare il pacchetto NuGet `Microsoft.Extensions.Logging.Console`. Chiamare il metodo di estensione `AddConsole`:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub")
    .ConfigureLogging(logging => {
        logging.SetMinimumLevel(LogLevel.Information);
        logging.AddConsole();
    })
    .Build();
```

Nel client JavaScript esiste un metodo `configureLogging` simile. Fornire un valore `LogLevel` che indica il livello minimo di messaggi di log da produrre. I log vengono scritti nella finestra della console del browser.

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub")
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

::: moniker range=">= aspnetcore-3.0"

Anziché un valore `LogLevel`, è anche possibile fornire un valore `string` che rappresenta il nome di un livello di log. Questa operazione è utile quando si configura la registrazione di SignalR in ambienti in cui non si ha accesso alle costanti `LogLevel`.

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub")
    .configureLogging("warn")
    .build();
```

Nella tabella seguente sono elencati i livelli di log disponibili. Il valore fornito per `configureLogging` imposta il livello di registrazione **minimo** che verrà registrato. Verranno registrati i messaggi registrati a questo livello **o i livelli elencati dopo tale operazione nella tabella**.

| String                      | LogLevel               |
| --------------------------- | ---------------------- |
| `trace`                     | `LogLevel.Trace`       |
| `debug`                     | `LogLevel.Debug`       |
| `info` **o** `information` | `LogLevel.Information` |
| `warn` **o** `warning`     | `LogLevel.Warning`     |
| `error`                     | `LogLevel.Error`       |
| `critical`                  | `LogLevel.Critical`    |
| `none`                      | `LogLevel.None`        |

::: moniker-end

> [!NOTE]
> Per disabilitare completamente la registrazione, specificare `signalR.LogLevel.None` nel metodo `configureLogging`.

Per ulteriori informazioni sulla registrazione, vedere la [documentazione di diagnostica di SignalR](xref:signalr/diagnostics).

Il client Java SignalR usa la libreria [SLF4J](https://www.slf4j.org/) per la registrazione. Si tratta di un'API di registrazione di alto livello che consente agli utenti della libreria di scegliere una propria implementazione di registrazione specifica inserendo una dipendenza di registrazione specifica. Il frammento di codice seguente illustra come usare `java.util.logging` con il client Java SignalR.

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

Se non si configura la registrazione nelle dipendenze, SLF4J carica un logger senza operazioni predefinito con il seguente messaggio di avviso:

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

Questa operazione può essere ignorata in modo sicuro.

### <a name="configure-allowed-transports"></a>Configurare i trasporti consentiti

I trasporti usati da SignalR possono essere configurati nella chiamata `WithUrl` (`withUrl` in JavaScript). È possibile utilizzare un operatore OR bit per bit dei valori di `HttpTransportType` per limitare il client in modo che utilizzi solo i trasporti specificati. Tutti i trasporti sono abilitati per impostazione predefinita.

Ad esempio, per disabilitare il trasporto degli eventi inviati dal server, ma consentire i WebSocket e le connessioni di polling lungo:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", HttpTransportType.WebSockets | HttpTransportType.LongPolling)
    .Build();
```

Nel client JavaScript, i trasporti vengono configurati impostando il campo `transport` nell'oggetto options fornito a `withUrl`:

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub", { transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling })
    .build();
```

::: moniker range=">= aspnetcore-2.2"

In questa versione dei WebSocket client Java è l'unico trasporto disponibile.

::: moniker-end

::: moniker range="= aspnetcore-3.0"

Nel client Java il trasporto viene selezionato con il metodo `withTransport` sul `HttpHubConnectionBuilder`. Per impostazione predefinita, il client Java usa il trasporto WebSocket.

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/myhub")
    .withTransport(TransportEnum.WEBSOCKETS)
    .build();
```

> [!NOTE]
> Il client Java SignalR non supporta ancora il fallback del trasporto.

::: moniker-end

### <a name="configure-bearer-authentication"></a>Configurare l'autenticazione della porta

Per fornire i dati di autenticazione insieme alle richieste SignalR, usare l'opzione `AccessTokenProvider` (`accessTokenFactory` in JavaScript) per specificare una funzione che restituisce il token di accesso desiderato. Nel client .NET questo token di accesso viene passato come token "autenticazione di connessione" HTTP (usando l'intestazione `Authorization` con un tipo di `Bearer`). Nel client JavaScript, il token di accesso viene usato come token di connessione, **ad eccezione** dei casi in cui le API del browser limitano la possibilità di applicare intestazioni (in particolare, nelle richieste di eventi inviati dal server e WebSocket). In questi casi, il token di accesso viene fornito come valore stringa di query `access_token`.

Nel client .NET è possibile specificare l'opzione `AccessTokenProvider` usando il delegato options in `WithUrl`:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", options => {
        options.AccessTokenProvider = async () => {
            // Get and return the access token.
        };
    })
    .Build();
```

Nel client JavaScript, il token di accesso viene configurato impostando il campo `accessTokenFactory` nell'oggetto opzioni in `withUrl`:

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub", {
        accessTokenFactory: () => {
            // Get and return the access token.
            // This function can return a JavaScript Promise if asynchronous
            // logic is required to retrieve the access token.
        }
    })
    .build();
```

Nel client Java SignalR è possibile configurare un bearer token da usare per l'autenticazione fornendo una factory del token di accesso a [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr._http_hub_connection_builder?view=aspnet-signalr-java). Usare [withAccessTokenFactory](/java/api/com.microsoft.signalr._http_hub_connection_builder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__) per fornire una [RxJava](https://github.com/ReactiveX/RxJava) [stringa a\<> singola](https://reactivex.io/documentation/single.html). Con una chiamata a [Single. rinvia](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-), è possibile scrivere la logica per produrre i token di accesso per il client.

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/myhub")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

### <a name="configure-timeout-and-keep-alive-options"></a>Configurare le opzioni di timeout e Keep-Alive

Sono disponibili opzioni aggiuntive per la configurazione del comportamento di timeout e Keep-Alive nell'oggetto `HubConnection`:


::: moniker range=">= aspnetcore-2.2"

# <a name="nettabdotnet"></a>[.NET](#tab/dotnet)

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30 secondi (30.000 millisecondi) | Timeout per l'attività del server. Se il server non ha inviato un messaggio in questo intervallo, il client considera il server disconnesso e attiva l'evento `Closed` (`onclose` in JavaScript). Questo valore deve essere sufficientemente grande da consentire l'invio del messaggio ping dal server **e** la ricezione da parte del client entro l'intervallo di timeout. Il valore consigliato è un numero almeno il doppio del valore `KeepAliveInterval` del server per consentire l'arrivo dei ping. |
| `HandshakeTimeout` | 15 secondi | Timeout per l'handshake iniziale del server. Se il server non invia una risposta di handshake in questo intervallo, il client Annulla l'handshake e attiva l'evento `Closed` (`onclose` in JavaScript). Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano errori di timeout dell'handshake a causa di una latenza di rete grave. Per informazioni dettagliate sul processo di handshake, vedere la [specifica del protocollo dell'hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |
| `KeepAliveInterval` | 15 secondi | Determina l'intervallo in corrispondenza del quale il client invia messaggi ping. L'invio di messaggi dal client Reimposta il timer all'inizio dell'intervallo. Se il client non ha inviato un messaggio nel `ClientTimeoutInterval` impostato sul server, il server considera il client disconnesso. |

Nel client .NET i valori di timeout vengono specificati come valori `TimeSpan`.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30 secondi (30.000 millisecondi) | Timeout per l'attività del server. Se il server non ha inviato un messaggio in questo intervallo, il client considera il server disconnesso e attiva l'evento `onclose`. Questo valore deve essere sufficientemente grande da consentire l'invio del messaggio ping dal server **e** la ricezione da parte del client entro l'intervallo di timeout. Il valore consigliato è un numero almeno il doppio del valore `KeepAliveInterval` del server per consentire l'arrivo dei ping. |
| `keepAliveIntervalInMilliseconds` | 15 secondi (15.000 millisecondi) | Determina l'intervallo in corrispondenza del quale il client invia messaggi ping. L'invio di messaggi dal client Reimposta il timer all'inizio dell'intervallo. Se il client non ha inviato un messaggio nel `ClientTimeoutInterval` impostato sul server, il server considera il client disconnesso. |

# <a name="javatabjava"></a>[Java](#tab/java)

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30 secondi (30.000 millisecondi) | Timeout per l'attività del server. Se il server non ha inviato un messaggio in questo intervallo, il client considera il server disconnesso e attiva l'evento `onClose`. Questo valore deve essere sufficientemente grande da consentire l'invio del messaggio ping dal server **e** la ricezione da parte del client entro l'intervallo di timeout. Il valore consigliato è un numero almeno il doppio del valore `KeepAliveInterval` del server per consentire l'arrivo dei ping. |
| `withHandshakeResponseTimeout` | 15 secondi | Timeout per l'handshake iniziale del server. Se il server non invia una risposta di handshake in questo intervallo, il client Annulla l'handshake e attiva l'evento `onClose`. Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano errori di timeout dell'handshake a causa di una latenza di rete grave. Per informazioni dettagliate sul processo di handshake, vedere la [specifica del protocollo dell'hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |
| `getKeepAliveInterval` / `setKeepAliveInterval` | 15 secondi (15.000 millisecondi) | Determina l'intervallo in corrispondenza del quale il client invia messaggi ping. L'invio di messaggi dal client Reimposta il timer all'inizio dell'intervallo. Se il client non ha inviato un messaggio nel `ClientTimeoutInterval` impostato sul server, il server considera il client disconnesso. |

::: moniker-end

::: moniker range="< aspnetcore-2.2"

# <a name="nettabdotnet"></a>[.NET](#tab/dotnet)

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30 secondi (30.000 millisecondi) | Timeout per l'attività del server. Se il server non ha inviato un messaggio in questo intervallo, il client considera il server disconnesso e attiva l'evento `Closed` (`onclose` in JavaScript). Questo valore deve essere sufficientemente grande da consentire l'invio del messaggio ping dal server **e** la ricezione da parte del client entro l'intervallo di timeout. Il valore consigliato è un numero almeno il doppio del valore `KeepAliveInterval` del server per consentire l'arrivo dei ping. |
| `HandshakeTimeout` | 15 secondi | Timeout per l'handshake iniziale del server. Se il server non invia una risposta di handshake in questo intervallo, il client Annulla l'handshake e attiva l'evento `Closed` (`onclose` in JavaScript). Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano errori di timeout dell'handshake a causa di una latenza di rete grave. Per informazioni dettagliate sul processo di handshake, vedere la [specifica del protocollo dell'hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |

Nel client .NET i valori di timeout vengono specificati come valori `TimeSpan`.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30 secondi (30.000 millisecondi) | Timeout per l'attività del server. Se il server non ha inviato un messaggio in questo intervallo, il client considera il server disconnesso e attiva l'evento `onclose`. Questo valore deve essere sufficientemente grande da consentire l'invio del messaggio ping dal server **e** la ricezione da parte del client entro l'intervallo di timeout. Il valore consigliato è un numero almeno il doppio del valore `KeepAliveInterval` del server per consentire l'arrivo dei ping. |

# <a name="javatabjava"></a>[Java](#tab/java)

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30 secondi (30.000 millisecondi) | Timeout per l'attività del server. Se il server non ha inviato un messaggio in questo intervallo, il client considera il server disconnesso e attiva l'evento `onClose`. Questo valore deve essere sufficientemente grande da consentire l'invio del messaggio ping dal server **e** la ricezione da parte del client entro l'intervallo di timeout. Il valore consigliato è un numero almeno il doppio del valore `KeepAliveInterval` del server, per consentire l'arrivo dei ping. |
| `withHandshakeResponseTimeout` | 15 secondi | Timeout per l'handshake iniziale del server. Se il server non invia una risposta di handshake in questo intervallo, il client Annulla l'handshake e attiva l'evento `onClose`. Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano errori di timeout dell'handshake a causa di una latenza di rete grave. Per informazioni dettagliate sul processo di handshake, vedere la [specifica del protocollo dell'hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |

::: moniker-end

---

### <a name="configure-additional-options"></a>Configurare opzioni aggiuntive

È possibile configurare opzioni aggiuntive nel metodo `WithUrl` (`withUrl` in JavaScript) in `HubConnectionBuilder` o nelle varie API di configurazione di `HttpHubConnectionBuilder` nel client Java:

# <a name="nettabdotnet"></a>[.NET](#tab/dotnet)

| Opzione .NET |  Valore predefinito | DESCRIZIONE |
| ----------- | -------------- | ----------- |
| `AccessTokenProvider` | `null` | Funzione che restituisce una stringa fornita come token di autenticazione di un portatore nelle richieste HTTP. |
| `SkipNegotiation` | `false` | Impostare questa impostazione su `true` per ignorare il passaggio di negoziazione. **Supportato solo quando il trasporto WebSocket è l'unico trasporto abilitato**. Questa impostazione non può essere abilitata quando si usa il servizio Azure SignalR. |
| `ClientCertificates` | Empty | Raccolta di certificati TLS da inviare per autenticare le richieste. |
| `Cookies` | Empty | Raccolta di cookie HTTP da inviare con ogni richiesta HTTP. |
| `Credentials` | Empty | Credenziali da inviare con ogni richiesta HTTP. |
| `CloseTimeout` | 5 secondi | Solo WebSocket. Quantità massima di tempo di attesa del client dopo la chiusura del server per confermare la richiesta di chiusura. Se il server non riconosce la chiusura entro questo intervallo di tempo, il client si disconnette. |
| `Headers` | Empty | Mappa di intestazioni HTTP aggiuntive da inviare con ogni richiesta HTTP. |
| `HttpMessageHandlerFactory` | `null` | Delegato che può essere utilizzato per configurare o sostituire il `HttpMessageHandler` utilizzato per inviare le richieste HTTP. Non usato per le connessioni WebSocket. Questo delegato deve restituire un valore non null e riceve il valore predefinito come parametro. Modificare le impostazioni del valore predefinito e restituirlo oppure restituire una nuova istanza di `HttpMessageHandler`. **Quando si sostituisce il gestore, assicurarsi di copiare le impostazioni che si desidera gestire dal gestore fornito. in caso contrario, le opzioni configurate (ad esempio cookie e intestazioni) non verranno applicate al nuovo gestore.** |
| `Proxy` | `null` | Proxy HTTP da usare per l'invio di richieste HTTP. |
| `UseDefaultCredentials` | `false` | Impostare questo valore booleano per inviare le credenziali predefinite per le richieste HTTP e WebSocket. Questo consente l'uso dell'autenticazione di Windows. |
| `WebSocketConfiguration` | `null` | Delegato che può essere usato per configurare opzioni WebSocket aggiuntive. Riceve un'istanza di [Metodo ClientWebSocketOptions](/dotnet/api/system.net.websockets.clientwebsocketoptions) che può essere utilizzata per configurare le opzioni. |

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

| Opzione JavaScript | Default Value | DESCRIZIONE |
| ----------------- | ------------- | ----------- |
| `accessTokenFactory` | `null` | Funzione che restituisce una stringa fornita come token di autenticazione di un portatore nelle richieste HTTP. |
| `skipNegotiation` | `false` | Impostare questa impostazione su `true` per ignorare il passaggio di negoziazione. **Supportato solo quando il trasporto WebSocket è l'unico trasporto abilitato**. Questa impostazione non può essere abilitata quando si usa il servizio Azure SignalR. |

# <a name="javatabjava"></a>[Java](#tab/java)

| Opzione Java | Default Value | DESCRIZIONE |
| ----------- | ------------- | ----------- |
| `withAccessTokenProvider` | `null` | Funzione che restituisce una stringa fornita come token di autenticazione di un portatore nelle richieste HTTP. |
| `shouldSkipNegotiate` | `false` | Impostare questa impostazione su `true` per ignorare il passaggio di negoziazione. **Supportato solo quando il trasporto WebSocket è l'unico trasporto abilitato**. Questa impostazione non può essere abilitata quando si usa il servizio Azure SignalR. |
| `withHeader` `withHeaders` | Empty | Mappa di intestazioni HTTP aggiuntive da inviare con ogni richiesta HTTP. |

---

Nel client .NET queste opzioni possono essere modificate dal delegato options fornito a `WithUrl`:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", options => {
        options.Headers["Foo"] = "Bar";
        options.Cookies.Add(new Cookie(/* ... */);
        options.ClientCertificates.Add(/* ... */);
    })
    .Build();
```

Nel client JavaScript queste opzioni possono essere fornite in un oggetto JavaScript fornito a `withUrl`:

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub", {
        skipNegotiation: true,
        transport: signalR.HttpTransportType.WebSockets
    })
    .build();
```

Nel client Java queste opzioni possono essere configurate con i metodi del `HttpHubConnectionBuilder` restituiti dal `HubConnectionBuilder.create("HUB URL")`

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/myhub")
        .withHeader("Foo", "Bar")
        .shouldSkipNegotiate(true)
        .withHandshakeResponseTimeout(30*1000)
        .build();
```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:tutorials/signalr>
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
* <xref:signalr/messagepackhubprotocol>
* <xref:signalr/supported-platforms>
