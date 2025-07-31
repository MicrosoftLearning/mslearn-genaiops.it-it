---
lab:
  title: Esplorare la progettazione dei prompt con Prompty
  description: Informazioni su come usare Prompty per testare e migliorare rapidamente diverse richieste con il modello linguistico e assicurarsi che siano costruite e orchestrate per ottenere risultati ottimali.
---

## Esplorare la progettazione dei prompt con Prompty

Questo esercizio richiede circa **45** minuti.

> **Nota**: questo esercizio presuppone una certa familiarità con Fonderia Azure AI, motivo per cui alcune istruzioni sono intenzionalmente meno dettagliate per incoraggiare l'esplorazione più attiva e l'apprendimento pratico.

## Introduzione

Durante l'ideazione, si vuole testare e migliorare rapidamente su prompt diversi con il modello linguistico. Esistono diversi modi per affrontare la progettazione dei prompt, tramite il playground nel Portale Fonderia Azure AI o usando Prompty per un approccio più incentrato sul codice.

In questo esercizio si esplora la progettazione delle richieste con Prompty in Azure Cloud Shell usando un modello distribuito tramite Fonderia Azure AI.

## Configurare l'ambiente

Per completare le attività di questo esercizio, è necessario:

- un hub di Fonderia Azure AI,
- un progetto Fonderia Azure AI
- Un modello distribuito (come GPT-4o).

### Creare un progetto e un hub di Azure per intelligenza artificiale

> **Nota**: Se si ha già un progetto di Azure AI, è possibile ignorare questa procedura e usare il progetto esistente.

È possibile creare un progetto di Azure AI manualmente tramite il Portale Fonderia Azure AI, nonché distribuire il modello usato nell'esercizio. Tuttavia, è possibile automatizzare questo processo usando un'applicazione modello con [Azure Developer CLI (azd)](https://aka.ms/azd).

1. In un browser web, aprire [Portale di Azure](https://portal.azure.com) all'indirizzo `https://portal.azure.com` e accedere usando le credenziali Azure.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Per altre informazioni sull'uso di Azure Cloud Shell, vedere la [documentazione su Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nel riquadro PowerShell, immettere i comandi seguenti per clonare il repository di questo esercizio:

     ```powershell
    rm -r mslearn-genaiops -f
    git clone https://github.com/MicrosoftLearning/mslearn-genaiops
     ```

1. Dopo aver clonato il repository, immettere i comandi seguenti per inizializzare il modello Starter.

     ```powershell
    cd ./mslearn-genaiops/Starter
    azd init
     ```

1. Una volta richiesto, assegnare un nome al nuovo ambiente, che verrà usato come base per assegnare nomi univoci a tutte le risorse fornite.

1. Successivamente, immettere il seguente comando per eseguire il modello Starter. Verrà eseguito il provisioning di un Hub AI completo delle relative risorse dipendenti, inclusi progetto AI, servizi AI associati e un endpoint online.

     ```powershell
    azd up
     ```

1. Quando richiesto, scegliere l'abbonamento che si desidera usare e quindi scegliere una delle seguenti posizioni per la fornitura delle risorse:
   - Stati Uniti orientali
   - Stati Uniti orientali 2
   - Stati Uniti centro-settentrionali
   - Stati Uniti centro-meridionali
   - Svezia centrale
   - Stati Uniti occidentali
   - Stati Uniti occidentali 3

1. Attendere il completamento dello script, che in genere richiede circa 10 minuti, ma in alcuni casi può richiedere più tempo.

    > **Nota**: le risorse Azure OpenAI sono limitate a livello di tenant da quote regionali. Le regioni elencate sopra comprendono le quote predefinite per i tipi di modello usati in questo esercizio. La scelta casuale di un'area riduce il rischio che una singola area raggiunga il limite di quota. Nel caso in cui venga raggiunto un limite di quota, è possibile che sia necessario creare un altro gruppo di risorse in una regione diversa. Altre informazioni sulla [disponibilità di modelli per area](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=standard%2Cstandard-chat-completions#global-standard-model-availability)

    <details>
      <summary><b>Suggerimento per la risoluzione dei problemi</b>: nessuna quota disponibile in una determinata regione</summary>
        <p>Se viene visualizzato un errore di distribuzione per uno dei modelli a causa di alcuna quota disponibile nella regione scelta, provare a eseguire i comandi seguenti:</p>
        <ul>
          <pre><code>azd env set AZURE_ENV_NAME new_env_name
   azd env set AZURE_RESOURCE_GROUP new_rg_name
   azd env set AZURE_LOCATION new_location
   azd up</code></pre>
        Sostituzione di <code>new_env_name</code>, <code>new_rg_name</code> e <code>new_location</code> con nuovi valori. La nuova posizione deve essere una delle regioni elencate all'inizio dell'esercizio, ad esempio <code>eastus2</code>, <code>northcentralus</code>, ecc.
        </ul>
    </details>

1. Dopo aver effettuato il provisioning di tutte le risorse, usare i seguenti comandi per ottenere l'endpoint e la chiave di accesso alla risorsa di servizi di Azure AI. Notare che è necessario sostituire `<rg-env_name>` e `<aoai-xxxxxxxxxx>` con i nomi del gruppo di risorse e della risorsa di servizi di Azure AI. Entrambi vengono stampati nell'output della distribuzione.

     ```powershell
    Get-AzCognitiveServicesAccount -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property endpoint
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copiare questi valori poiché verranno usati in seguito.

### Configurare l'ambiente virtuale in Cloud Shell

Per sperimentare ed eseguire rapidamente l'iterazione, si userà un set di script Python in Cloud Shell.

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per passare alla cartella con i file di codice usati in questo esercizio:

     ```powershell
    cd ~/mslearn-genaiops/Files/03/
     ```

1. Immettere i comandi seguenti per attivare un ambiente virtuale e installare le librerie necessarie:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai tiktoken azure-ai-projects prompty[azure]
    ```

1. Immettere il comando seguente per aprire il file di configurazione fornito:

    ```powershell
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice sostituire i segnaposto **ENDPOINTNAME** e **APIKEY** con i valori di endpoint e chiave copiati in precedenza.
1. *Dopo* aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Ottimizzare la richiesta di sistema

Ridurre al minimo la lunghezza delle richieste di sistema mantenendo al tempo stesso le funzionalità di IA generativa è fondamentale per le distribuzioni su larga scala. Richieste più brevi possono comportare tempi di risposta più rapidi, poiché il modello di intelligenza artificiale elabora meno token e usa anche meno risorse di calcolo.

1. Immettere il comando seguente per aprire il file dell'applicazione fornito:

    ```powershell
   code optimize-prompt.py
    ```

    Esaminare il codice e notare che lo script esegue il file modello `start.prompty` che ha già una richiesta di sistema predefinita.

1. Eseguire `code start.prompty` per esaminare la richiesta di sistema. Provare a pensare a come abbreviarla mantenendo la chiarezza e l'efficacia della finalità. Ad esempio:

   ```python
   original_prompt = "You are a helpful assistant. Your job is to answer questions and provide information to users in a concise and accurate manner."
   optimized_prompt = "You are a helpful assistant. Answer questions concisely and accurately."
   ```

   Rimuovere le parole ridondanti e concentrarsi sulle istruzioni essenziali. Salvare la richiesta ottimizzata nel file.

### Testare e convalidare l'ottimizzazione

Il test delle modifiche alle richieste è importante per garantire la riduzione dell'utilizzo dei token senza perdita di qualità.

1. Eseguire `code token-count.py` per aprire ed esaminare l'app contatore token fornita nell'esercizio. Se è stata usata una richiesta ottimizzata diversa da quella fornito nell'esempio precedente, è possibile usarla anche in questa app.

1. Eseguire lo script con `python token-count.py` e osservare la differenza nel conteggio dei token. Assicurarsi che la richiesta ottimizzata produca comunque risposte di alta qualità.

## Analizzare le interazioni degli utenti

Comprendere in che modo gli utenti interagiscono con l'app consente di identificare i modelli che aumentano l'utilizzo dei token.

1. Esaminare un set di dati di esempio di richieste utente:

    - **"Riepiloga la trama di *Guerra e pace*".**
    - **"Quali sono alcune curiosità sui gatti?"**
    - **"Scrivi un piano aziendale dettagliato per una startup che usa l'intelligenza artificiale per ottimizzare le catene di approvvigionamento."**
    - **"Traduci "Ciao, come stai?" in francese".**
    - **"Spiega l'entanglement quantistico a un bambino di 10 anni".**
    - **"Dammi 10 idee creative per una storia breve di fantascienza".**

    Per ogni richiesta, identificare se è probabile che si ottenga una risposta **breve**, **media** o **lunga/complessa** dall'intelligenza artificiale.

1. Esaminare le categorizzazioni. Quali criteri si notano? Tenere in considerazione:

    - Il **livello di astrazione**, ad esempio, creativo rispetto a oggettivo, influisce sulla lunghezza?
    - Le **richieste aperte** risultano tendenzialmente più lunghe?
    - In che modo la **complessità delle istruzioni**, ad esempio "Spiega come se io avessi 10 anni", influisce sulla risposta?

1. Immettere il comando seguente per eseguire l'applicazione **optimize-prompt**:

    ```
   python optimize-prompt.py
    ```

1. Usare alcuni degli esempi forniti in precedenza per verificare l'analisi.
1. Usare ora la richiesta in formato lungo seguente ed esaminarne l'output:

    ```
   Write a comprehensive overview of the history of artificial intelligence, including key milestones, major contributors, and the evolution of machine learning techniques from the 1950s to today.
    ```

1. Riscrivere questa richiesta per:

    - Limitare l'ambito
    - Definire le aspettative per la brevità
    - Usare la formattazione o la struttura per guidare la risposta

1. Confrontare le risposte per verificare che sia stata ottenuta una risposta più concisa.

> **NOTA**: È possibile usare `token-count.py` per confrontare l'utilizzo dei token in entrambe le risposte.
<br>
<details>
<summary><b>Esempio di richiesta riscritta:</b></summary><br>
<p>"Fornisci un riepilogo con punti elenco di 5 attività cardine chiave nella storia dell'intelligenza artificiale".</p>
</details>

## [**FACOLTATIVO**] Applicare le ottimizzazioni in uno scenario reale

1. Si supponga di creare un chatbot del servizio clienti che deve fornire risposte rapide e accurate.
1. Integrare la richiesta di sistema ottimizzata e il modello nel codice del chatbot. *È possibile usare `optimize-prompt.py` come punto*di partenza.
1. Testare il chatbot con varie domande utente per assicurarsi che risponda in modo efficiente ed efficace.

## Conclusione

L'ottimizzazione delle richieste è una competenza chiave per ridurre i costi e migliorare le prestazioni nelle applicazioni di IA generativa. Abbreviando le richieste, usando i modelli e analizzando le interazioni degli utenti, è possibile creare soluzioni più efficienti e scalabili.

## Eseguire la pulizia

Al termine dell'esplorazione di Servizi di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com?azure-portal=true) in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
