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

> **Nota**: se si dispone già di un hub di Azure AI e di un progetto, è possibile ignorare questa procedura e usare il progetto esistente.

È possibile creare un hub di Azure AI e un progetto manualmente tramite il portale Fonderia Azure AI, nonché distribuire il modello usato nell'esercizio. Tuttavia, è possibile automatizzare questo processo usando un'applicazione modello con [Azure Developer CLI (azd)](https://aka.ms/azd).

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
   
### Configurare un ambiente di sviluppo locale

Per sperimentare e iterare rapidamente, verrà usato Prompty in Visual Studio (VS) Code. Preparare VS Code per l'ideazione locale.

1. Aprire VS Code e **clonare** il repository Git seguente: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Archiviare il clone in un'unità locale e aprire la cartella dopo la clonazione.
1. Nel riquadro estensioni in VS Code, cercare e installare l'estensione **Prompty** .
1. In Esplora VS Code (riquadro sinistro), fare clic con il pulsante destro del mouse sulla cartella **Files/03**.
1. Nel menu a discesa, selezionare **Nuovo Prompty**.
1. Aprire il file appena creato denominato **basic.prompty**.
1. Eseguire il file Prompty selezionando il pulsante **riproduci** nell'angolo superiore destro (o premere F5).
1. Alla richiesta di accesso, selezionare **Consenti**.
1. Selezionare l'account di Azure e accedere.
1. Tornare a VS Code, in cui si aprirà un riquadro **Output** con un messaggio di errore. Il messaggio di errore indica che il modello distribuito non è specificato o non può essere trovato.

Per risolvere l'errore, è necessario configurare un modello per l'uso con Prompty.

## Aggiornare i metadati del prompt

Per eseguire il file Prompty, è necessario specificare il modello linguistico da usare per generare la risposta. I metadati vengono definiti in *frontmatter* del file Prompty. Aggiornare i metadati con la configurazione del modello e altre informazioni.

1. Aprire il riquadro del terminale di Visual Studio Code.
1. Copiare il file **basic.prompty** (nella stessa cartella) e rinominare la copia in `chat-1.prompty`.
1. Aprire **chat-1.prompty** e aggiornare i campi seguenti per modificare alcune informazioni di base:

    - **Nome:**

        ```yaml
        name: Python Tutor Prompt
        ```

    - **Descrizione:**

        ```yaml
        description: A teaching assistant for students wanting to learn how to write and edit Python code.
        ```

    - **Modello distribuito**:

        ```yaml
        azure_deployment: ${env:AZURE_OPENAI_CHAT_DEPLOYMENT}
        ```

1. Aggiungere quindi il segnaposto seguente per la chiave API nel parametro **azure_deployment** .

    - **Chiave dell'endpoint**:

        ```yaml
        api_key: ${env:AZURE_OPENAI_API_KEY}
        ```

1. Salvare il file Prompty aggiornato.

Il file Prompty contiene ora tutti i parametri necessari, ma alcuni parametri usano dei segnaposto per ottenere i valori richiesti. I segnaposti sono memorizzati nel file **.env** nella stessa cartella.

## Aggiornare la configurazione del modello

Per specificare il modello usato da Prompty, è necessario fornire le informazioni sul modello nel file .env.

1. Aprire il file **.env** nella cartella **Files/03**.
1. Aggiornare ciascuno dei segnaposti con i valori copiati in precedenza dall'output dell'installazione del modello nel portale di Azure:

    ```yaml
    - AZURE_OPENAI_CHAT_DEPLOYMENT="gpt-4"
    - AZURE_OPENAI_ENDPOINT="<Your endpoint target URI>"
    - AZURE_OPENAI_API_KEY="<Your endpoint key>"
    ```

1. Salvare il file con estensione env.
1. Eseguire di nuovo il file **chat-1.prompty**.

Ora sarà possibile ottenere una risposta generata dall'IA, anche se non correlata allo scenario, in quanto usa solo l'input di esempio. Aggiornare il modello per renderlo un assistente docente basato su IA.

## Modificare la sezione di esempio

La sezione di esempio specifica gli input di Prompty e fornisce valori predefiniti da usare se non vengono forniti input.

1. Modificare i campi dei parametri seguenti:

    - **firstName**: scegliere qualsiasi altro nome.
    - **context**: rimuovere l'intera sezione.
    - **question**: sostituire il testo fornito con:

    ```yaml
    What is the difference between 'for' loops and 'while' loops?
    ```

    La sezione di **esempio** dovrebbe essere simile all'esempio seguente:
    
    ```yaml
    sample:
    firstName: Daniel
    question: What is the difference between 'for' loops and 'while' loops?
    ```

    1. Eseguire il file Prompty aggiornato ed esaminare l'output.

