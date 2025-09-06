---
lab:
  title: Monitorare l'applicazione di IA generativa
  description: Informazioni su come monitorare le interazioni con il modello distribuito e ottenere informazioni dettagliate su come ottimizzarne l'utilizzo con l'applicazione di IA generativa.
---

# Monitorare l'applicazione di IA generativa

Questo esercizio richiede circa **30** minuti.

> **Nota**: questo esercizio presuppone una certa familiarità con Fonderia Azure AI, motivo per cui alcune istruzioni sono intenzionalmente meno dettagliate per incoraggiare l'esplorazione più attiva e l'apprendimento pratico.

## Introduzione

In questo esercizio, viene abilitato il monitoraggio per un'app di completamento della chat e vengono visualizzate le prestazioni in Monitoraggio di Azure. È possibile interagire con il modello distribuito per generare dati, visualizzare i dati generati tramite le informazioni dettagliate della dashboard delle applicazioni di IA generativa e configurare avvisi per ottimizzare la distribuzione del modello.

## Configurare l'ambiente

Per completare le attività di questo esercizio, è necessario:

- un progetto Fonderia Azure AI
- un modello distribuito (come GPT-4o),
- una risorsa di Application Insights connessa.

### Distribuire un modello nel progetto Fonderia Azure AI

Per configurare rapidamente un progetto di Fonderia Azure AI, di seguito sono disponibili istruzioni semplici per usare l'interfaccia utente del Portale Fonderia Azure AI.

1. In un Web browser, aprire il [Portale Fonderia Azure AI](https://ai.azure.com) su `https://ai.azure.com` e accedere usando le credenziali di Azure.
1. Nella home page, nella sezione **Esplora modelli e funzionalità**, cercare il modello `gpt-4o`, che verrà usato nel progetto.
1. Nei risultati della ricerca, selezionare il modello **gpt-4o** per visualizzarne i dettagli e quindi nella parte superiore della pagina selezionare **Usa questo modello**.
1. Quando viene richiesto di creare un progetto, immettere un nome valido per il progetto ed espandere **Opzioni avanzate**.
1. Selezionare **Personalizza** e specificare le impostazioni seguenti per il progetto:
    - **Risorsa di Fonderia Azure AI**: *nome valido per la risorsa di Fonderia Azure AI*
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Area geografica**: *selezionare qualsiasi **località supportata per i servizi di intelligenza artificiale**\*

    > \* Alcune risorse Azure AI sono limitate da quote di modelli regionali. In caso di superamento di un limite di quota più avanti nell'esercizio, potrebbe essere necessario creare un'altra risorsa in un'area diversa.

1. Selezionare **Crea** e attendere la creazione del progetto, inclusa la distribuzione del modello gpt-4 selezionato.
1. Nel riquadro di spostamento a sinistra selezionare **Panoramica** per visualizzare la pagina principale del progetto,.
1. Nell'area **Endpoint e chiavi** assicurarsi che sia selezionata la libreria di **Fonderia Azure AI** e visualizzare l'**Endpoint del progetto di Fonderia Azure AI**.
1. **Salvare** l'endpoint in un Blocco note. Questo endpoint verrà usato per connettersi al progetto in un'applicazione client.

### Connetti Application Insights

Connettere Application Insights al progetto in Fonderia Azure AI per iniziare a raccogliere i dati per il monitoraggio.

1. Usare il menu a sinistra e selezionare la pagina **Traccia**.
1. **Creare una nuova** risorsa di Application Insights per connettersi all'app.
1. Immettere un nome di risorsa di Application Insights e selezionare **Crea**.

Application Insights è ora connesso al progetto e verrà iniziata l'analisi dei dati raccolti.

## Interagire con un modello distribuito

Interagire con il modello distribuito a livello di codice configurando una connessione al progetto Fonderia Azure AI usando Azure Cloud Shell. In questo modo sarà possibile inviare una richiesta al modello e generare i dati di monitoraggio.

### Connettersi con a un modello tramite Cloud Shell

Iniziare recuperando le informazioni necessarie da autenticare per interagire con il modello. Accedere quindi ad Azure Cloud Shell e aggiornare la configurazione per inviare le richieste fornite al modello distribuito.

1. Aprire una nuova scheda del browser (mantenendo aperto il Portale Fonderia Azure AI nella scheda esistente).
1. In una nuova scheda, passare al [portale di Azure](https://portal.azure.com) su `https://portal.azure.com`, accedendo con le credenziali di Azure se richiesto.
1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell*** senza archiviazione nell'abbonamento.
1. Nella barra degli strumenti di Cloud Shell, nel menu **Impostazioni**, selezionare **Vai alla versione classica**.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro Cloud Shell immettere ed eseguire il comando seguente:

    ```
    rm -r mslearn-genaiops -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Questo comando clona il repository GitHub contenente i file di codice per questo esercizio.

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-genaiops/Files/07
    ```

1. Nel riquadro della riga di comando di Cloud Shell, immettere il comando seguente per installare le librerie che verranno utilizzate:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai azure-identity azure-ai-projects opentelemetry-instrumentation-openai-v2 azure-monitor-opentelemetry
    ```

1. Immettere il comando seguente per aprire il file di configurazione fornito:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice:

    1. Nel file di codice sostituire il segnaposto **your_project_endpoint** con l'endpoint del progetto, copiato dalla pagina **Panoramica** del progetto nel Portale Fonderia Azure AI.
    1. Sostituire il segnaposto **your_model_deployment** con il nome assegnato alla distribuzione modello GPT-4o (per impostazione predefinita `gpt-4o`).

1. *Dopo* aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** aver sostituito i segnaposto con l'editor di codice, usare il comando **Fare clic con il pulsante destro del mouse > Salva** per **salvare le modifiche** e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Inviare richieste al modello distribuito

A questo punto è possibile eseguire più script che inviano richieste diverse al modello distribuito. Queste interazioni generano dati che è possibile osservare in un secondo momento in Monitoraggio di Azure.

1. Eseguire il comando seguente per **visualizzare il primo script** fornito:

    ```
   code start-prompt.py
    ```

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per accedere ad Azure.

    ```
   az login
    ```

    **<font color="red">È necessario accedere ad Azure, anche se la sessione di Cloud Shell è già autenticata.</font>**

    > **Nota**: nella maggior parte degli scenari, il semplice uso di *az login* sarà sufficiente. Tuttavia, in caso di sottoscrizioni in più tenant, potrebbe essere necessario specificare il tenant usando il parametro *--tenant*. Per dettagli, visualizzare [Accedere ad Azure in modo interattivo usando l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Quando richiesto, seguire le istruzioni per aprire la pagina di accesso in una nuova scheda e immettere il codice di autenticazione fornito e le credenziali di Azure. Completare quindi il processo di accesso nella riga di comando, selezionando la sottoscrizione contenente l'hub di Fonderia Azure AI, se richiesto.
1. Dopo aver eseguito l'accesso, immettere il comando seguente per eseguire l'applicazione:

    ```
   python start-prompt.py
    ```

    Il modello genererà una risposta, che verrà acquisita con Application Insights per un'ulteriore analisi. È possibile variare le richieste per esplorare i relativi effetti.

1. **Aprire ed esaminare lo script** in cui la richiesta indica al modello di **rispondere solo con una frase e un elenco**:

    ```
   code short-prompt.py
    ```

1. **Eseguire lo script** immettendo il comando seguente nella riga di comando:

    ```
   python short-prompt.py
    ```

1. Lo script successivo ha un obiettivo simile, ma include le istruzioni per l'output nel **messaggio di sistema** anziché nel messaggio dell'utente:

    ```
   code system-prompt.py
    ```

1. **Eseguire lo script** immettendo il comando seguente nella riga di comando:

    ```
   python system-prompt.py
    ```

1. Infine, provare ad attivare un errore eseguendo una richiesta con **troppi token**:

    ```
   code error-prompt.py
    ```

1. **Eseguire lo script** immettendo il comando seguente nella riga di comando. Notare che è molto **probabile che si verifichi un errore!**

    ```
   python error-prompt.py
    ```

Ora, dopo aver interagito con il modello, è possibile esaminare i dati in Monitoraggio di Azure.

> **Nota**: potrebbero essere necessari alcuni minuti affinché i dati di monitoraggio vengano mostrati in Monitoraggio di Azure.

## Visualizzare i dati di monitoraggio in Monitoraggio di Azure

Per visualizzare i dati raccolti dalle interazioni con il modello, accedere al dashboard che collega a una cartella di lavoro in Monitoraggio di Azure.

### Passare a Monitoraggio di Azure dal portale Fonderia Azure AI

1. Passare alla scheda nel browser con il **portale Fonderia Azure AI** aperto.
1. Usare il menu a sinistra e selezionare **Monitoraggio**.
1. Selezionare l'opzione **Utilizzo risorse** ed esaminare i dati riepilogati delle interazioni con il modello distribuito.

> **Nota**: È anche possibile selezionare **Esplora metriche di Monitoraggio di Azure** nella parte inferiore della pagina Monitoraggio per una visualizzazione completa di tutte le metriche disponibili. Il collegamento aprirà Monitoraggio di Azure in una nuova scheda.

## Interpretare le metriche di monitoraggio

Ora è il momento di analizzare i dati e iniziare a interpretare cosa dicono.

### Esaminare l'utilizzo dei token

Concentrarsi prima sulla sezione relativa **all'utilizzo dei token** ed esaminare le metriche seguenti:

- **Richieste totali**: Numero di richieste di inferenza separate, ovvero quante volte è stato chiamato il modello.

> Utile per analizzare la velocità effettiva e comprendere il costo medio per chiamata.

- **Numero totale di token**: Totale dei token di richiesta e i token di completamento combinati.

> Metrica più importante per la fatturazione e le prestazioni, perché determina la latenza e il costo.

- **Numero di token di richiesta**: Numero totale di token usati nell'input (le richieste inviate) in tutte le chiamate di modello.

> Pensare a questo come al *costo di porre* al modello una domanda.

- **Numero di token di completamento**: Numero di token restituiti dal modello come output, essenzialmente la lunghezza delle risposte.

> I token di completamento generati spesso rappresentano la maggior parte dell'utilizzo e dei costi dei token, soprattutto per le risposte lunghe o dettagliate.

### Confrontare le singole richieste

1. Usare il menu a sinistra, selezionare **Traccia**. Espandere ogni chiamata **generate_completion** per IA generativa per visualizzare le chiamate figlio. Ogni richiesta è rappresentata come una nuova riga di dati. Esaminare e confrontare il contenuto delle colonne seguenti:

- **Input**: mostra il messaggio utente inviato al modello.

> Utilizzare questa colonna per valutare le formulazioni delle richieste efficienti o problematiche.

- **Output**: contiene la risposta del modello.

> Usarlo per valutare il livello di dettaglio, la pertinenza e la coerenza. In particolare in relazione ai conteggi dei token e alla durata.

- **Durata**: indica quanto tempo il modello ha impiegato per rispondere, in millisecondi.

> Confrontare le righe per esplorare quali criteri di richiesta generano tempi di elaborazione più lunghi.

- **Operazione riuscita**: Indica se una chiamata di modello è riuscita o meno.

> Usare questa opzione per identificare le richieste problematiche o gli errori di configurazione. L'ultima richiesta non è probabilmente riuscita perché era troppo lunga.

## (FACOLTATIVO) Creare un avviso

Se si dispone di tempo aggiuntivo, provare a configurare un avviso per notificare quando la latenza del modello supera una determinata soglia. Questo è un esercizio progettato per sfidare l'utente, il che significa che le istruzioni sono intenzionalmente meno dettagliate.

- In Monitoraggio di Azure, creare una **nuova regola di avviso** per il progetto e il modello Fonderia Azure AI.
- Scegliere una metrica, ad esempio **Durata della richiesta (ms)** e definire una soglia (ad esempio, maggiore di 4000 ms).
- Creare un **nuovo gruppo di azioni** per definire la modalità di notifica.

Gli avvisi consentono di prepararsi per la produzione stabilendo il monitoraggio proattivo. Gli avvisi configurati dipendono dalle priorità del progetto e dal modo in cui il team ha deciso di misurare e mitigare i rischi.

## Dove trovare altri lab

È possibile esplorare altri lab ed esercizi nel [portale di apprendimento di Fonderia Azure AI](https://ai.azure.com) oppure fare riferimento alla **sezione lab** del corso per altre attività disponibili.
