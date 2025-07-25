---
lab:
  title: Ottimizzare il modello usando un set di dati sintetico
  description: Informazioni su come creare set di dati sintetici e usarli per migliorare le prestazioni e l'affidabilità del modello.
---

## Ottimizzare il modello usando un set di dati sintetico

L'ottimizzazione di un'applicazione di IA generativa comporta l'uso di set di dati per migliorare le prestazioni e l'affidabilità del modello. Usando dati sintetici, gli sviluppatori possono simulare un'ampia gamma di scenari e casi limite che potrebbero non essere presenti nei dati reali. Inoltre, la valutazione degli output del modello è fondamentale per ottenere applicazioni di IA affidabili e di alta qualità. L'intero processo di ottimizzazione e valutazione può essere gestito in modo efficiente tramite l'SDK di valutazione di Azure AI, che mette a disposizione strumenti e framework avanzati per semplificare e standardizzare queste operazioni.

Questo esercizio richiederà circa **30** minuti\*

> \* Questa stima non include l'attività facoltativa alla fine dell'esercizio.
## Scenario

Si supponga di voler creare un'app di guida intelligente basata su intelligenza artificiale per migliorare l'esperienza dei visitatori in un museo. L'applicazione ha lo scopo di rispondere a domande su personaggi storici. Per valutare le risposte dell'app, è necessario creare un set di dati sintetico completo di domande e risposte che copra vari aspetti di queste personalità e del loro lavoro.

È stato selezionato un modello GPT-4 per fornire risposte generative. È ora necessario sviluppare un simulatore in grado di generare interazioni contestualmente pertinenti, al fine di valutare le prestazioni dell'IA in una varietà di scenari.

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
    cd ~/mslearn-genaiops/Files/06/
     ```

1. Immettere i comandi seguenti per attivare un ambiente virtuale e installare le librerie necessarie:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-evaluation azure-ai-projects promptflow wikipedia aiohttp openai==1.77.0
    ```

1. Immettere il comando seguente per aprire il file di configurazione fornito:

    ```powershell
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice sostituire i segnaposto **your_azure_openai_service_endpoint** e **your_azure_openai_service_api_key** con i valori dell'endpoint e della chiave copiati in precedenza.
1. *Dopo* aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Generare dati sintetici

A questo punto si eseguirà uno script che genera un set di dati sintetico e lo usa per valutare la qualità del modello con training preliminare.

1. Eseguire il comando seguente per **modificare lo script** fornito:

    ```powershell
   code generate_synth_data.py
    ```

1. Nello script individuare **# Define callback function**.
1. Sotto questo commento, incollare il codice seguente:

    ```
    async def callback(
        messages: List[Dict],
        stream: bool = False,
        session_state: Any = None,  # noqa: ANN401
        context: Optional[Dict[str, Any]] = None,
    ) -> dict:
        messages_list = messages["messages"]
        # Get the last message
        latest_message = messages_list[-1]
        query = latest_message["content"]
        context = text
        # Call your endpoint or AI application here
        current_dir = os.getcwd()
        prompty_path = os.path.join(current_dir, "application.prompty")
        _flow = load_flow(source=prompty_path)
        response = _flow(query=query, context=context, conversation_history=messages_list)
        # Format the response to follow the OpenAI chat protocol
        formatted_response = {
            "content": response,
            "role": "assistant",
            "context": context,
        }
        messages["messages"].append(formatted_response)
        return {
            "messages": messages["messages"],
            "stream": stream,
            "session_state": session_state,
            "context": context
        }
    ```

    È possibile specificare qualsiasi endpoint dell'applicazione da usare per la simulazione specificando una funzione di richiamata di destinazione. In questo caso si userà un'applicazione che è un modello linguistico di grandi dimensioni con un file Prompty `application.prompty`. La funzione di richiamata precedente elabora ogni messaggio generato dal simulatore eseguendo le attività seguenti:
    * Recupera il messaggio utente più recente.
    * Carica un prompt flow da application.prompty.
    * Genera una risposta usando il prompt flow.
    * Formatta la risposta per rispettare il protocollo di chat OpenAI.
    * Aggiunge la risposta dell'assistente all'elenco dei messaggi.

    >**Nota**: Per altre informazioni sull'uso di Prompty, vedere la [documentazione di Prompty](https://www.prompty.ai/docs).

1. Individuare quindi **# Run the simulator**.
1. Sotto questo commento, incollare il codice seguente:

    ```
    model_config = {
        "azure_endpoint": os.getenv("AZURE_OPENAI_ENDPOINT"),
        "api_key": os.getenv("AZURE_OPENAI_API_KEY"),
        "azure_deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT"),
    }
    
    simulator = Simulator(model_config=model_config)
    
    outputs = asyncio.run(simulator(
        target=callback,
        text=text,
        num_queries=1,  # Minimal number of queries
    ))
    
    output_file = "simulation_output.jsonl"
    with open(output_file, "w") as file:
        for output in outputs:
            file.write(output.to_eval_qr_json_lines())
    ```

   Il codice riportato sopra inizializzerà il simulatore ed lo eseguirà per generare conversazioni sintetiche in base a un testo estratto in precedenza da Wikipedia.

1. Individuare quindi **# Evaluate the model**.
1. Sotto questo commento, incollare il codice seguente:

    ```
    groundedness_evaluator = GroundednessEvaluator(model_config=model_config)
    eval_output = evaluate(
        data=output_file,
        evaluators={
            "groundedness": groundedness_evaluator
        },
        output_path="groundedness_eval_output.json"
    )
    ```

    Ora che si ha un set di dati, è possibile valutare la qualità e l'efficacia dell'applicazione di IA generativa. Nel codice precedente si userà la fondatezza come metrica di qualità.

1. Salva le modifiche.
1. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, immettere il seguente comando per **eseguire l'applicazione**:

    ```
   python generate_synth_data.py
    ```

    Al termine dello script è possibile scaricare i file di output eseguendo `download simulation_output.jsonl` e `download groundedness_eval_output.json` ed esaminando il relativo contenuto. Se la metrica di base non è vicina a 3,0, è possibile modificare i parametri del modello linguistico di grandi dimensioni, ad esempio `temperature`, `top_p`, `presence_penalty` o `frequency_penalty` nel file `application.prompty` ed eseguire nuovamente lo script per generare un nuovo set di dati per la valutazione. È anche possibile modificare `wiki_search_term` per ottenere un set di dati sintetico basato su un contesto diverso.

## (FACOLTATIVO) Ottimizzare il modello

Se si ha più tempo, è possibile usare il set di dati generato per ottimizzare il modello in Fonderia Azure AI. L'ottimizzazione dipende dalle risorse dell'infrastruttura cloud, che possono richiedere una quantità variabile di tempo per il provisioning a seconda della capacità del data center e della domanda simultanea.

1. Aprire una nuova scheda del browser e passare al [Portale Fonderia Azure AI](https://ai.azure.com) all'indirizzo `https://ai.azure.com`, quindi accedere usando le credenziali di Azure.
1. Nella home page di Fonderia AI selezionare il progetto creato all'inizio dell'esercizio.
1. Passare alla pagina **Ottimizzazione** nella sezione **Crea e personalizza**, usando il menu a sinistra.
1. Selezionare il pulsante per aggiungere un nuovo modello di ottimizzazione, selezionare il modello **gpt-4o** e quindi **Avanti**.
1. **Ottimizzare** il modello usando la configurazione seguente:
    - **Versione del modello**: *selezionare la versione predefinita*
    - **Metodo di personalizzazione**: supervisione
    - **Suffisso del modello**: `ft-travel`
    - **Risorsa AI connessa**: *selezionare la connessione predefinita creata al momento della creazione dell'hub. Dovrebbe essere selezionata per impostazione predefinita.*
    - **Dati di training**: caricare i file

    <details>  
    <summary><b>Suggerimento per la risoluzione dei problemi</b>: errore di autorizzazioni</summary>
    <p>Se viene visualizzato un errore di autorizzazione, provare i seguenti passaggi per la risoluzione dei problemi:</p>
    <ul>
        <li>Nel portale di Azure, selezionare la risorsa Servizi di intelligenza artificiale.</li>
        <li>Nella scheda Identità, in Gestione risorse, confermare che si tratta dell'identità gestita assegnata dal sistema.</li>
        <li>Passare all'account di archiviazione associato. Nella pagina IAM, aggiungere l'assegnazione di ruolo <em>Proprietario dei dati del BLOB di archiviazione</em>.</li>
        <li>In <strong>Assegna accesso a</strong>, scegliere <strong>Identità gestita</strong>, <strong>+Seleziona membri</strong>, selezionare <strong>Tutte le identità gestite assegnate dal sistema</strong> e selezionare la risorsa dei Servizi di Azure AI.</li>
        <li>Rivedere e assegnare per salvare le nuove impostazioni e ripetere il passaggio precedente.</li>
    </ul>
    </details>

    - **Caricare il file**: selezionare il file JSONL scaricato nel passaggio precedente.
    - **Dati di convalida**: nessuno
    - **Parametri attività**: *mantenere le impostazioni predefinite*
1. Verrà avviata l'ottimizzazione, che potrebbe richiedere del tempo per il completamento.

    > **Nota**: l'ottimizzazione e la distribuzione possono richiedere una notevole quantità di tempo (30 minuti o più), quindi potrebbe essere necessario eseguire periodicamente il controllo. Per visualizzare altri dettagli sullo stato di avanzamento finora, selezionare il processo del modello di ottimizzazione e visualizzarne la scheda **Log**.

## (FACOLTATIVO) Distribuire il modello ottimizzato

Al termine dell'ottimizzazione, è possibile distribuire il modello ottimizzato.

1. Selezionare il collegamento al processo di ottimizzazione per aprire la relativa pagina dei dettagli. Selezionare la scheda **Metriche** ed esplorare le metriche ottimizzate.
1. Distribuire il modello ottimizzato con le configurazioni seguenti:
    - **Nome distribuzione**: *nome univoco per la distribuzione del modello*
    - **Tipo di distribuzione**: Standard
    - **Limite di velocità dei token al minuto (migliaia)**: 5.000 *(o il massimo disponibile nella sottoscrizione se inferiore a 5.000)
    - **Filtro contenuto**: Predefinito
1. Attendere il completamento della distribuzione prima di sottoporla a test. L'operazione potrebbe richiedere alcuni minuti. Controllare lo **stato provisioning** fino a quando non è stato completato (potrebbe essere necessario aggiornare il browser per visualizzare lo stato aggiornato).
1. Quando la distribuzione è pronta, passare al modello ottimizzato e selezionare **Apri nel playground**.

    Dopo aver distribuito il modello ottimizzato, è possibile testarlo nel playground di Chat come si farebbe con qualsiasi modello di base.

## Conclusione

In questo esercizio è stato creato un set di dati sintetico simulando una conversazione tra un utente e un'app di completamento della chat. Usando questo set di dati, è possibile valutare la qualità delle risposte dell'app e ottimizzarla per ottenere i risultati desiderati.

## Eseguire la pulizia

Al termine dell'esplorazione di Servizi di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com?azure-portal=true) in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
