---
lab:
  title: Orchestrare un sistema RAG
  description: Informazioni su come implementare sistemi RAG (Retrieval-Augmented Generation) nelle app per migliorare l'accuratezza e la pertinenza delle risposte generate.
---

## Orchestrare un sistema RAG

I sistemi di Generazione aumentata da recupero (RAG) integrano la capacità di modelli linguistici di grandi dimensioni con meccanismi di recupero delle informazioni ad alta efficienza, al fine di migliorare l'accuratezza e la pertinenza delle risposte generate. Sfruttando LangChain per l'orchestrazione e Fonderia Azure AI per le funzionalità di IA, è possibile creare una pipeline robusta che recupera informazioni rilevanti da un set di dati e genera risposte coerenti. In questo esercizio verranno illustrate le fasi di configurazione dell'ambiente, pre-elaborazione dei dati, generazione degli incorporamenti e creazione di un indice, che costituiscono i passaggi fondamentali per implementare in modo efficace un sistema RAG.

Questo esercizio richiederà circa **30** minuti.

## Scenario

Si supponga di voler creare un'applicazione che fornisca consigli sugli hotel a Londra. All'interno dell'app, si intende integrare un agente in grado non solo di fornire consigli sugli hotel, ma anche di rispondere in modo accurato alle domande che gli utenti potrebbero porre a riguardo.

È stato selezionato un modello GPT-4 per fornire risposte generative. Ora si intende sviluppare un sistema RAG in grado di fornire al modello dati contestuali basati sulle recensioni degli altri utenti, orientando così il comportamento della chat nella formulazione di consigli personalizzati.

Per iniziare, distribuire le risorse necessarie per creare l'applicazione.

## Creare un progetto e un hub di Azure per intelligenza artificiale

È possibile creare un hub e un progetto di Azure AI manualmente tramite il portale Fonderia Azure AI, nonché distribuire i modelli usati nell'esercizio. Tuttavia, è possibile automatizzare questo processo usando un'applicazione modello con [Azure Developer CLI (azd)](https://aka.ms/azd).

1. In un browser web, aprire [Portale di Azure](https://portal.azure.com) all'indirizzo `https://portal.azure.com` e accedere usando le credenziali Azure.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Per altre informazioni sull'uso di Azure Cloud Shell, vedere la [documentazione su Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell, nel menu **Impostazioni**, selezionare **Vai alla versione classica**.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

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
        
1. Successivamente, immettere il seguente comando per eseguire il modello Starter. Verrà eseguito il provisioning di un Hub AI completo delle relative risorse dipendenti, inclusi progetto AI, servizi AI associati e un endpoint online. Verranno inoltre distribuiti i modelli GPT-4 Turbo, GPT-4o e GPT-4o mini.

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
     ```

     ```powershell
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copiare questi valori poiché verranno usati in seguito.

## Configurare l'ambiente di sviluppo in Cloud Shell

Per sperimentare ed eseguire rapidamente l'iterazione, si userà un set di script Python in Cloud Shell.

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per passare alla cartella con i file di codice usati in questo esercizio:

     ```powershell
    cd ~/mslearn-genaiops/Files/04/
     ```

1. Immettere i comandi seguenti per attivare un ambiente virtuale e installare le librerie necessarie:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv langchain-text-splitters langchain-community langchain-openai
    ```

1. Immettere il comando seguente per aprire il file di configurazione fornito:

    ```powershell
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice sostituire i segnaposto **your_azure_openai_service_endpoint** e **your_azure_openai_service_api_key** con i valori dell'endpoint e della chiave copiati in precedenza.
1. *Dopo* aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Implementare RAG

A questo punto si eseguirà uno script che inserisce e pre-elabora i dati, crea incorporamenti e compila un archivio vettoriale e un indice, consentendo infine di implementare un sistema RAG in modo efficace.

1. Eseguire il comando seguente per **visualizzare lo script** fornito:

    ```powershell
   code RAG.py
    ```

1. Esaminare lo script e notare che usa un file CSV con recensioni di hotel come dati di base. È possibile visualizzare il contenuto di questo file eseguendo il comando `download app_hotel_reviews.csv` e aprendo il file.
1. **Eseguire lo script** immettendo il comando seguente nella riga di comando:

    ```
   python RAG.py
    ```

1. Quando l'applicazione è in esecuzione, è possibile iniziare a porre domande, ad esempio `Where can I stay in London?`, e quindi procedere con richieste più specifiche.

## Conclusione

In questo esercizio è stato creato un sistema RAG tipico con i relativi componenti principali. Usando i propri documenti per informare le risposte di un modello, si forniscono dati di grounding usati dal modello linguistico di grandi dimensioni durante la formulazione di una risposta. Per una soluzione aziendale, ciò significa che è possibile vincolare l'IA generativa ai contenuti aziendali.

## Eseguire la pulizia

Al termine dell'esplorazione di Servizi di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com?azure-portal=true) in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
