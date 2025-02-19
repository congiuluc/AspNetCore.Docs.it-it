---
title: Razor Pages con EF Core in ASP.NET Core - Migrazioni - 4 di 8
author: rick-anderson
description: In questa esercitazione si inizia a usare la funzionalità delle migrazioni EF Core per la gestione delle modifiche al modello di dati in un'app ASP.NET Core MVC.
ms.author: riande
ms.date: 07/22/2019
uid: data/ef-rp/migrations
ms.openlocfilehash: e6f44a32a473f8f3bacfa4d9608deba2f879b6b6
ms.sourcegitcommit: 7d3c6565dda6241eb13f9a8e1e1fd89b1cfe4d18
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/11/2019
ms.locfileid: "72259716"
---
# <a name="razor-pages-with-ef-core-in-aspnet-core---migrations---4-of-8"></a>Razor Pages con EF Core in ASP.NET Core - Migrazioni - 4 di 8

Di [Tom Dykstra](https://github.com/tdykstra), [Jon P Smith](https://twitter.com/thereformedprog) e [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE [about the series](~/includes/RP-EF/intro.md)]

::: moniker range=">= aspnetcore-3.0"

In questa esercitazione viene presentata la funzionalità delle migrazioni di EF Core per la gestione delle modifiche al modello di dati.

Quando viene sviluppata una nuova app, il modello di dati cambia spesso. Ogni volta che vengono apportate modifiche al modello, questo non è più sincronizzato con il database. Questa serie di esercitazioni è iniziata con la configurazione di Entity Framework per creare il database se non esiste già. Ogni volta che il modello di dati viene modificato, è necessario eliminare il database. Alla successiva esecuzione dell'app, la chiamata a `EnsureCreated` ricrea il database in modo che corrisponda al nuovo modello di dati. Viene quindi eseguita la classe `DbInitializer` per il seeding del nuovo database.

Questo approccio che consiste nel mantenere il database sincronizzato con il modello di dati funziona bene fino a quando non si distribuisce l'app nell'ambiente di produzione. Quando l'app è in esecuzione nell'ambiente di produzione, in genere esegue l'archiviazione di dati che devono essere mantenuti. L'app non può essere avviata con un database di test ogni volta che viene apportata una modifica, come l'aggiunta di una colonna. La funzionalità delle migrazioni di EF Core risolve questo problema consentendo a EF Core di aggiornare lo schema del database anziché crearne uno nuovo.

Anziché eliminare e ricreare il database quando il modello di dati viene modificato, le migrazioni aggiornano lo schema e mantengono i dati esistenti.

[!INCLUDE[](~/includes/sqlite-warn.md)]

## <a name="drop-the-database"></a>Eliminare il database

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Usare **Esplora oggetti di SQL Server** (SSOX) per eliminare il database oppure eseguire il comando seguente nella **console di Gestione pacchetti**:

```powershell
Drop-Database
```

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Eseguire il comando seguente al prompt dei comandi per installare gli strumenti dell'interfaccia della riga di comando EF:

  ```dotnetcli
  dotnet tool install --global dotnet-ef
  ```

* Al prompt dei comandi passare alla cartella del progetto. La cartella del progetto contiene il file *ContosoUniversity.csproj*.

* Eliminare il file *CU.db* oppure eseguire il comando seguente:

  ```dotnetcli
  dotnet ef database drop --force
  ```

---

## <a name="create-an-initial-migration"></a>Creare una migrazione iniziale

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Eseguire i comandi seguenti nella console di Gestione pacchetti:

```powershell
Add-Migration InitialCreate
Update-Database
```

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Assicurarsi che il prompt dei comandi si trovi nella cartella del progetto ed eseguire i comandi seguenti:

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

## <a name="up-and-down-methods"></a>Metodi Up e Down

Il comando `migrations add` di EF Core ha generato il codice per la creazione del database. Questo codice di migrazioni si trova nel file *Migrations\<timestamp>_InitialCreate.cs*. Il metodo `Up` della classe `InitialCreate` crea le tabelle di database che corrispondono al set di entità del modello di dati. Il metodo `Down` le elimina, come illustrato nell'esempio seguente:

[!code-csharp[](intro/samples/cu30/Migrations/20190731193522_InitialCreate.cs)]

Il codice precedente è per la migrazione iniziale. Il codice:

* È stato generato dal comando `migrations add InitialCreate`. 
* Viene eseguito dal comando `database update`.
* Crea un database per il modello di dati specificato dalla classe del contesto di database.

Il parametro del nome della migrazione, nell'esempio "InitialCreate", viene usato per il nome del file. Il nome della migrazione può essere qualsiasi nome di file valido. È consigliabile scegliere una parola o una frase che riepiloghi cosa viene fatto nella migrazione. Ad esempio, una migrazione che ha aggiunto una tabella di reparto potrebbe essere denominata "AddDepartmentTable".

## <a name="the-migrations-history-table"></a>Tabella di cronologia delle migrazioni

* Usare SSOX o lo strumento SQLite per esaminare il database.
* Si noti l'aggiunta di una tabella `__EFMigrationsHistory`. La tabella `__EFMigrationsHistory` tiene traccia di quali migrazioni sono state applicate al database.
* Visualizzare i dati nella tabella `__EFMigrationsHistory`. Viene visualizzata una riga per la prima migrazione.

## <a name="the-data-model-snapshot"></a>Snapshot del modello di dati

Le migrazioni creano uno *snapshot* del modello di dati corrente in *Migrations/SchoolContextModelSnapshot.cs*. Quando si aggiunge una migrazione, EF determina le modifiche apportate confrontando il modello di dati corrente con il file dello snapshot.

Poiché il file di snapshot tiene traccia dello stato del modello di dati, non è possibile eliminare una migrazione eliminando il file `<timestamp>_<migrationname>.cs`. Per eliminare la migrazione più recente, è necessario usare il comando `migrations remove`. Tale comando elimina la migrazione e garantisce che lo snapshot venga reimpostato correttamente. Per altre informazioni, vedere [dotnet ef migrations remove](/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove).

## <a name="remove-ensurecreated"></a>Rimuovere EnsureCreated

Questa serie di esercitazioni è iniziata usando `EnsureCreated`. `EnsureCreated` non crea una tabella di cronologia delle migrazioni e pertanto non può essere usato con le migrazioni. È progettato per il test o per la creazione rapida di prototipi in cui il database viene eliminato e ricreato frequentemente.

Da questo punto in poi, le esercitazioni useranno le migrazioni.

In *Data/DBInitializer.cs* impostare come commento la riga seguente:

```csharp
context.Database.EnsureCreated();
```
Eseguire l'app e verificare che il database sia sottoposto a seeding.

## <a name="applying-migrations-in-production"></a>Applicazione delle migrazioni nell'ambiente di produzione

È consigliabile fare in modo che le app nell'ambiente di produzione **non** chiamino [Database.Migrate](/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.migrate?view=efcore-2.0#Microsoft_EntityFrameworkCore_RelationalDatabaseFacadeExtensions_Migrate_Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_) all'avvio dell'applicazione. `Migrate` non deve essere chiamato da un'app distribuita in una server farm. Se l'app viene distribuita in più istanze del server, è difficile assicurarsi che gli aggiornamenti dello schema di database non vengano eseguiti da più server o che non siano in conflitto con l'accesso in lettura/scrittura.

La migrazione del database deve essere eseguita come parte della distribuzione e in modo controllato. Gli approcci alla migrazione di database in ambiente di produzione includono:

* Uso delle migrazioni per creare script SQL e uso degli script SQL nella distribuzione.
* Esecuzione di `dotnet ef database update` da un ambiente controllato.

## <a name="troubleshooting"></a>Risoluzione dei problemi

Se l'app usa SQL Server Local DB e visualizza l'eccezione seguente:

```text
SqlException: Cannot open database "ContosoUniversity" requested by the login.
The login failed.
Login failed for user 'user name'.
```

È possibile che la soluzione esegua `dotnet ef database update` al prompt dei comandi.

### <a name="additional-resources"></a>Risorse aggiuntive

* [Interfaccia della riga di comando EF Core](/ef/core/miscellaneous/cli/dotnet).
* [Console di Gestione pacchetti (Visual Studio)](/ef/core/miscellaneous/cli/powershell)

## <a name="next-steps"></a>Passaggi successivi

L'esercitazione successiva compila il modello di dati, aggiungendo le proprietà delle entità e le nuove entità.

> [!div class="step-by-step"]
> [Esercitazione precedente](xref:data/ef-rp/sort-filter-page)
> [Esercitazione successiva](xref:data/ef-rp/complex-data-model)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

In questa esercitazione viene usata la funzionalità delle migrazioni EF Core per la gestione delle modifiche al modello di dati.

Se si verificano problemi che non si è in grado di risolvere, scaricare l'[app completa](
https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples).

Quando viene sviluppata una nuova app, il modello di dati cambia spesso. Ogni volta che vengono apportate modifiche al modello, questo non è più sincronizzato con il database. L'esercitazione è iniziata con la configurazione di Entity Framework per creare il database se non esiste già. Ogni volta che il modello di dati cambia:

* Il database viene eliminato.
* EF ne crea uno nuovo che corrisponde al modello.
* L'app esegue il seeding del database con dati di test.

Questo approccio che consiste nel mantenere il database sincronizzato con il modello di dati funziona bene fino a quando non si distribuisce l'app nell'ambiente di produzione. Quando l'app è in esecuzione nell'ambiente di produzione, in genere esegue l'archiviazione di dati che devono essere mantenuti. L'app non può essere avviata con un database di test ogni volta che viene apportata una modifica, come ad esempio l'aggiunta di una colonna. La funzionalità delle migrazioni di EF Core risolve questo problema consentendo a EF Core di aggiornare lo schema del database anziché creare un nuovo database.

Anziché eliminare e ricreare il database quando il modello di dati viene modificato, le migrazioni aggiornano lo schema e mantengono i dati esistenti.

## <a name="drop-the-database"></a>Eliminare il database

Usare **Esplora oggetti di SQL Server** o il comando `database drop`:

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Nella **console di Gestione pacchetti** eseguire il comando seguente:

```PMC
Drop-Database
```

Eseguire `Get-Help about_EntityFrameworkCore` dalla console di Gestione pacchetti per ottenere informazioni.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Aprire una finestra di comando e passare alla cartella del progetto. La cartella del progetto contiene il file *Startup.cs*.

Digitare quanto segue nella finestra di comando:

 ```dotnetcli
 dotnet ef database drop
 ```

---

## <a name="create-an-initial-migration-and-update-the-db"></a>Creare una migrazione iniziale e aggiornare il database

Compilare il progetto e creare la prima migrazione.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

```PMC
Add-Migration InitialCreate
Update-Database
```

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

### <a name="examine-the-up-and-down-methods"></a>Esaminare i metodi Up e Down

Il comando `migrations add` di EF Core ha generato codice per la creazione del database. Questo codice di migrazioni si trova nel file *Migrations\<timestamp>_InitialCreate.cs*. Il metodo `Up` della classe `InitialCreate` crea le tabelle di database che corrispondono al set di entità del modello di dati. Il metodo `Down` le elimina, come illustrato nell'esempio seguente:

[!code-csharp[](intro/samples/cu21/Migrations/20180626224812_InitialCreate.cs?range=7-24,77-88)]

Le migrazioni chiamano il metodo `Up` per implementare le modifiche al modello di dati per una migrazione. Quando si immette un comando per annullare l'aggiornamento, le migrazioni chiamano il metodo `Down`.

Il codice precedente è per la migrazione iniziale. Tale codice è stato creato quando è stato eseguito il comando `migrations add InitialCreate`. Il parametro del nome della migrazione, nell'esempio "InitialCreate", viene usato per il nome del file. Il nome della migrazione può essere qualsiasi nome di file valido. È consigliabile scegliere una parola o una frase che riepiloghi cosa viene fatto nella migrazione. Ad esempio, una migrazione che ha aggiunto una tabella di reparto potrebbe essere denominata "AddDepartmentTable".

Se la migrazione iniziale viene creata e il database esiste:

* Viene generato il codice di creazione del database.
* Non è necessario che venga eseguito il codice di creazione del database perché il database corrisponde già al modello di dati. Se viene eseguito il codice di creazione del database, non apporta alcuna modifica perché il database corrisponde già al modello di dati.

Quando l'app viene distribuita in un nuovo ambiente, è necessario eseguire il codice di creazione del database per creare il database.

Il database infatti non esiste in quanto è stato precedentemente eliminato, pertanto la migrazione crea il nuovo database.

### <a name="the-data-model-snapshot"></a>Snapshot del modello di dati

Le migrazioni creano uno *snapshot* dello schema del database corrente in *Migrations/SchoolContextModelSnapshot.cs*. Quando si aggiunge una migrazione, EF determina le modifiche apportate confrontando il modello di dati con il file dello snapshot.

Per eliminare una migrazione, usare il comando seguente:

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Remove-Migration

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
dotnet ef migrations remove
```

Per altre informazioni, vedere [dotnet ef migrations remove](/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove).

---

Il comando di rimozione migrazioni elimina la migrazione e garantisce che lo snapshot venga reimpostato correttamente.

### <a name="remove-ensurecreated-and-test-the-app"></a>Rimuovere EnsureCreated e testare l'app

Per lo sviluppo iniziale è stato usato il comando `EnsureCreated`. In questa esercitazione vengono usate le migrazioni. `EnsureCreated` presenta le limitazioni seguenti:

* Ignora le migrazioni e crea il database e lo schema.
* Non crea una tabella delle migrazioni.
* *Non* può essere usato con le migrazioni.
* È progettato per il test o per la creazione rapida di prototipi in cui il database viene eliminato e ricreato frequentemente.

Rimuovere `EnsureCreated`:

```csharp
context.Database.EnsureCreated();
```

Eseguire l'app e verificare che il database venga inizializzato.

### <a name="inspect-the-database"></a>Esaminare il database

Per controllare il database, usare **Esplora oggetti di SQL Server**. Si noti l'aggiunta di una tabella `__EFMigrationsHistory`. La tabella `__EFMigrationsHistory` tiene traccia di quali migrazioni sono state applicate al database. Visualizzare i dati nella tabella `__EFMigrationsHistory`: viene visualizzata una riga per la prima migrazione. Nell'ultimo log nell'esempio di output della CLI precedente viene visualizzata l'istruzione INSERT che crea tale riga.

Eseguire l'app e verificare che tutto funzioni.

## <a name="applying-migrations-in-production"></a>Applicazione delle migrazioni nell'ambiente di produzione

È consigliabile fare in modo che le app nell'ambiente di produzione **non** chiamino [Database.Migrate](/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.migrate?view=efcore-2.0#Microsoft_EntityFrameworkCore_RelationalDatabaseFacadeExtensions_Migrate_Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_) all'avvio dell'applicazione. L'elemento `Migrate` non deve essere chiamato da un'app nella server farm. Ad esempio, se l'app è stata distribuita in un cloud con scale-out, ovvero se sono in esecuzione più istanze dell'app.

La migrazione del database deve essere eseguita come parte della distribuzione e in modo controllato. Gli approcci alla migrazione di database in ambiente di produzione includono:

* Uso delle migrazioni per creare script SQL e uso degli script SQL nella distribuzione.
* Esecuzione di `dotnet ef database update` da un ambiente controllato.

EF Core usa la tabella `__MigrationsHistory` per stabilire se è necessario eseguire migrazioni. Se il database è aggiornato, non viene eseguita alcuna migrazione.

## <a name="troubleshooting"></a>Risoluzione dei problemi

Scaricare l'[app completa](
https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/cu21snapshots/cu-part4-migrations).

L'app genera l'eccezione seguente:

```text
SqlException: Cannot open database "ContosoUniversity" requested by the login.
The login failed.
Login failed for user 'user name'.
```

Soluzione: Eseguire `dotnet ef database update`

### <a name="additional-resources"></a>Risorse aggiuntive

* [Versione YouTube dell'esercitazione](https://www.youtube.com/watch?v=OWSUuMLKTJo)
* [Interfaccia della riga di comando di .NET Core](/ef/core/miscellaneous/cli/dotnet)
* [Console di Gestione pacchetti (Visual Studio)](/ef/core/miscellaneous/cli/powershell)



> [!div class="step-by-step"]
> [Precedente](xref:data/ef-rp/sort-filter-page)
> [Successivo](xref:data/ef-rp/complex-data-model)

::: moniker-end

