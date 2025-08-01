---
lab:
  title: Analizzare ed eseguire il debug dell'applicazione di intelligenza artificiale generativa con la traccia
  description: Informazioni su come eseguire il debug dell'applicazione di IA generativa tracciandone il flusso di lavoro dall'input dell'utente alla risposta del modello e alla fase di post-elaborazione.
---

# Analizzare ed eseguire il debug dell'applicazione di intelligenza artificiale generativa con la traccia

Questo esercizio richiede circa **30** minuti.

> **Nota**: questo esercizio presuppone una certa familiarità con Fonderia Azure AI, motivo per cui alcune istruzioni sono intenzionalmente meno dettagliate per incoraggiare l'esplorazione più attiva e l'apprendimento pratico.

## Introduzione

In questo esercizio verrà eseguito un assistente di IA generativa a più passaggi, che suggerisce escursioni a piedi e consiglia attrezzature per l'outdoor. Verrà usata la funzione di traccia dell'SDK di inferenza di Intelligenza artificiale di Azure per analizzare l'esecuzione dell'applicazione e identificare i punti chiave decisionali presi dal modello e dalla logica circostante.

Verrà eseguita l'interazione con un modello distribuito per simulare un percorso reale dell'utente, tracciando ogni fase dell'applicazione, dall'input dell'utente alla risposta del modello fino alla post-elaborazione, visualizzando i dati di analisi in Fonderia Azure AI. Ciò consentirà di comprendere come la traccia migliora l'osservabilità, semplifica il debug e supporta l'ottimizzazione delle prestazioni delle applicazioni di IA generativa.

## Configurare l'ambiente

Per completare le attività di questo esercizio, è necessario:

- un hub di Fonderia Azure AI,
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

Connettere Application Insights al progetto in Fonderia Azure AI per iniziare a raccogliere i dati da analizzare.

1. Usare il menu a sinistra e selezionare la pagina **Traccia**.
1. **Creare una nuova** risorsa di Application Insights per connettersi all'app.
1. Immettere un nome di risorsa di Application Insights e selezionare **Crea**.

Application Insights è ora connesso al progetto e verrà iniziata l'analisi dei dati raccolti.

## Eseguire un'app di IA generativa con Cloud Shell

Verrà effettuata la connessione al progetto Fonderia Azure AI da Azure Cloud Shell e verrà eseguita l'interazione a livello di codice con un modello distribuito come parte di un'app di IA generativa.

### Interagire con un modello distribuito

Iniziare recuperando le informazioni necessarie da autenticare per interagire con il modello distribuito. Quindi, sarà possibile accedere ad Azure Cloud Shell e aggiornare il codice dell'app di IA generativa.

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

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-genaiops/Files/08
    ```

1. Nel riquadro della riga di comando di Cloud Shell, immettere il comando seguente per installare le librerie che verranno utilizzate:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
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

### Aggiornare il codice per l'app di IA generativa

Ora che l'ambiente e il file .env sono stati configurati, è il momento di preparare lo script dell'assistente IA per l'esecuzione. Oltre a connettersi con un progetto di IA e ad abilitare Application Insights, è necessario:

- Interagire con il modello distribuito.
- Definire la funzione per specificare il prompt.
- Definire il flusso principale che chiama tutte le funzioni.

Queste tre parti verranno aggiunte a uno script iniziale.

1. Eseguire il comando seguente per **aprire lo script** fornito:

    ```
   code start-prompt.py
    ```

    Sarà possibile notare che diverse righe chiave sono state lasciate vuote o contrassegnate da # Commenti vuoti. Il compito consiste nel completare lo script copiando e incollando le righe corrette di seguito nelle posizioni appropriate.

1. Nello script, individuare **# Function to call the model and handle tracing**.
1. Sotto questo commento, incollare il codice seguente:

    ```
   # Function to call the model and handle tracing
   def call_model(system_prompt, user_prompt, span_name):
       with tracer.start_as_current_span(span_name) as span:
           span.set_attribute("session.id", SESSION_ID)
           span.set_attribute("prompt.user", user_prompt)
           start_time = time.time()
    
           response = chat_client.chat.completions.create(
               model=model_deployment,
               messages=[
                   { 
                       "role": "system", 
                       "content": system_prompt 
                   },
                   { 
                       "role": "user", 
                       "content": user_prompt
                   }
               ]
           )
    
           duration = time.time() - start_time
           output = response.choices[0].message.content
           span.set_attribute("response.time", duration)
           span.set_attribute("response.tokens", len(output.split()))
           return output
    ```

1. Nello script, individuare **# Function to recommend a hike based on user preferences**.
1. Sotto questo commento, incollare il codice seguente:

    ```
   # Function to recommend a hike based on user preferences 
   def recommend_hike(preferences):
        with tracer.start_as_current_span("recommend_hike") as span:
            prompt = f"""
            Recommend a named hiking trail based on the following user preferences.
            Provide only the name of the trail and a one-sentence summary.
            Preferences: {preferences}
            """
            response = call_model(
                "You are an expert hiking trail recommender.",
                prompt,
                "recommend_model_call"
            )
            span.set_attribute("hike_recommendation", response.strip())
            return response.strip()
    ```

1. Nello script, individuare **# ---- Main Flow ----**.
1. Sotto questo commento, incollare il codice seguente:

    ```
   if __name__ == "__main__":
       with tracer.start_as_current_span("trail_guide_session") as session_span:
           session_span.set_attribute("session.id", SESSION_ID)
           print("\n--- Trail Guide AI Assistant ---")
           preferences = input("Tell me what kind of hike you're looking for (location, difficulty, scenery):\n> ")

           hike = recommend_hike(preferences)
           print(f"\n✅ Recommended Hike: {hike}")

           # Run profile function


           # Run match product function


           print(f"\n🔍 Trace ID available in Application Insights for session: {SESSION_ID}")
    ```

1. **Salvare le modifiche** apportate nello script.
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

1. Fornire una descrizione del tipo di escursione che si sta cercando, ad esempio:

    ```
   A one-day hike in the mountains
    ```

    Il modello genererà una risposta, che verrà acquisita con Application Insights. È possibile visualizzare le tracce nel **portale Fonderia Azure AI**.

> **Nota**: potrebbero essere necessari alcuni minuti affinché i dati di monitoraggio vengano mostrati in Monitoraggio di Azure.

## Visualizzare i dati delle tracce nel portale Fonderia Azure AI

Dopo aver eseguito lo script, è stata acquisita una traccia dell'esecuzione dell'applicazione di IA. A questo punto, verrà esaminata usando Application Insights in Fonderia Azure AI.

> **Nota:** in seguito, il codice verrà eseguito nuovamente e le tracce verranno visualizzate di nuovo nel portale Fonderia Azure AI. Per prima cosa, è necessario capire dove trovare le tracce per visualizzarle.

### Passare al portale Fonderia Azure AI

1. **Tenere aperto Cloud Shell**. Sarà necessario tornare a questa sezione per aggiornare il codice ed eseguirlo di nuovo.
1. Passare alla scheda nel browser con il **portale Fonderia Azure AI** aperto.
1. Usare il menu a sinistra, selezionare **Traccia**.
1. *Se* non vengono visualizzati dati, **aggiornare** la vista.
1. Selezionare la traccia **train_guide_session** per aprire una nuova finestra che mostra altri dettagli.

### Esaminare la traccia

Questa vista mostra la traccia di una sessione completa di Trail Guide AI Assistant.

- **Intervallo di primo livello**: trail_guide_session è l'intervallo padre. Rappresenta l'intera esecuzione dell'assistente dall'inizio alla fine.

- **Intervalli figlio annidati**: ogni riga rientrata rappresenta un'operazione annidata. Informazioni:

    - **recommend_hike** acquisisce la logica con cui viene decisa un'escursione.
    - **recommend_model_call** è l'intervallo creato da call_model() all'interno di recommend_hike.
    - **chat gpt-4o** viene automaticamente instrumentato dall'SDK di inferenza di Azure per intelligenza artificiale per mostrare l'effettiva interazione con LLM.

1. È possibile fare clic su qualsiasi intervallo per visualizzare:

    1. La durata.
    1. I relativi attributi, ad esempio il prompt dell'utente, i token usati, il tempo di risposta.
    1. Eventuali errori o dati personalizzati associati a **span.set_attribute(...)**.

## Aggiungere altre funzioni al codice

1. Passare alla scheda nel browser con il **portale di Azure** aperto.
1. Eseguire il seguente comando seguente per **aprire nuovamente lo script:**

    ```
   code start-prompt.py
    ```

1. Nello script, individuare **# Function to generate a trip profile for the recommended hike**.
1. Sotto questo commento, incollare il codice seguente:

    ```
   def generate_trip_profile(hike_name):
       with tracer.start_as_current_span("trip_profile_generation") as span:
           prompt = f"""
           Hike: {hike_name}
           Respond ONLY with a valid JSON object and nothing else.
           Do not include any intro text, commentary, or markdown formatting.
           Format: {{ "trailType": ..., "typicalWeather": ..., "recommendedGear": [ ... ] }}
           """
           response = call_model(
               "You are an AI assistant that returns structured hiking trip data in JSON format.",
               prompt,
               "trip_profile_model_call"
           )
           print("🔍 Raw model response:", response)
           try:
               profile = json.loads(response)
               span.set_attribute("profile.success", True)
               return profile
           except json.JSONDecodeError as e:
               print("❌ JSON decode error:", e)
               span.set_attribute("profile.success", False)
               return {}
    ```

1. Nello script, individuare **# Function to match recommended gear with products in the catalog**.
1. Sotto questo commento, incollare il codice seguente:

    ```
   def match_products(recommended_gear):
       with tracer.start_as_current_span("product_matching") as span:
           matched = []
           for gear_item in recommended_gear:
               for product in mock_product_catalog:
                   if any(word in product.lower() for word in gear_item.lower().split()):
                       matched.append(product)
                       break
           span.set_attribute("matched.count", len(matched))
           return matched
    ```

1. Nello script, individuare **# Run profile function**.
1. Sotto e **allineato a** questo commento, incollare il seguente codice:

    ```
           profile = generate_trip_profile(hike)
           if not profile:
               print("Failed to generate trip profile. Please check Application Insights for trace.")
               exit(1)

           print(f"\n📋 Trip Profile for {hike}:")
           print(json.dumps(profile, indent=2))
    ```

1. Nello script, individuare **# Run match product function**.
1. Sotto e **allineato a** questo commento, incollare il seguente codice:

    ```
           matched = match_products(profile.get("recommendedGear", []))
           print("\n🛒 Recommended Products from Lakeshore Retail:")
           print("\n".join(matched))
    ```

1. **Salvare le modifiche** apportate nello script.
1. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, immettere il seguente comando per **eseguire l'applicazione**:

    ```
   python start-prompt.py
    ```

1. Fornire una descrizione del tipo di escursione che si sta cercando, ad esempio:

    ```
   I want to go for a multi-day adventure along the beach
    ```

<br>
<details>
<summary><b>Script della soluzione</b>: Nel caso in cui il codice non funzioni.</summary><br>
<p>Esaminando l'analisi del modello LLM per la funzione generate_trip_profile, è possibile notare che la risposta dell'assistente include i backtick e il testo JSON per formattare l'output come blocco di codice.

Sebbene questo sia utile per la visualizzazione, può causare problemi nel codice, poiché l'output non risulta più essere un JSON valido. Ciò comporta un errore di analisi durante l'ulteriore elaborazione.

L'errore è probabilmente dovuto al modo in cui l'LLM viene istruito a seguire un formato specifico per il proprio output. Includere le istruzioni direttamente nel prompt utente risulta più efficace rispetto a inserirle nel menu vocale del sistema.</p>
</details>


> **Nota**: potrebbero essere necessari alcuni minuti affinché i dati di monitoraggio vengano mostrati in Monitoraggio di Azure.

### Visualizzare le nuove tracce nel portale Fonderia Azure AI

1. Passare al portale Fonderia Azure AI.
1. Verrà visualizzata una nuova traccia con lo stesso nome **trail_guide_session**. Aggiornare la vista, se necessario.
1. Selezionare la nuova traccia per aprire la vista più dettagliata.
1. Esaminare i nuovi intervalli figlio annidati **trip_profile_generation** e **product_matching**.
1. Selezionare **product_matching** ed esaminare i metadati visualizzati.

    Nella funzione product_matching, è stato incluso **span.set_attribute(“matched.count”, len(matched))**. Impostando l'attributo con la coppia chiave-valore **matched.count** e la lunghezza della variabile matched, questa informazione è stata aggiunta alla traccia **product_matching**. È possibile trovare questa coppia chiave-valore in **attributi** nei metadati.

## (Facoltativo) Tracciare un errore

Se si dispone di tempo aggiuntivo, è possibile rivedere come usare le tracce quando si verifica un errore. Verrà fornito uno script che probabilmente genererà un errore. Eseguirlo ed esaminare le tracce.

Questo è un esercizio progettato per sfidare l'utente, il che significa che le istruzioni sono intenzionalmente meno dettagliate.

1. In Cloud Shell, aprire lo script **error-prompt.py**. Questo script è situato nella stessa directory dello script **start-prompt.py**. Rivedere il contenuto.
1. Eseguire lo script **error-prompt.py**. Fornire una risposta nella riga di comando quando richiesto.
1. *Idealmente*, il messaggio di output includerà **Failed to generate trip profile** (Impossibile generare un profilo di viaggio). Controllare Application Insights per la traccia.
1. Passare alla traccia per **trip_profile_generation** e controllare il motivo per cui si è verificato un errore.

<br>
<details>
<summary><b>Ottenere la risposta su</b>: il motivo per cui si è verificato un errore...</summary><br>
<p>Esaminando l'analisi del modello LLM per la funzione generate_trip_profile, è possibile notare che la risposta dell'assistente include i backtick e il testo JSON per formattare l'output come blocco di codice.

Sebbene questo sia utile per la visualizzazione, può causare problemi nel codice, poiché l'output non risulta più essere un JSON valido. Ciò comporta un errore di analisi durante l'ulteriore elaborazione.

L'errore è probabilmente dovuto al modo in cui l'LLM viene istruito a seguire un formato specifico per il proprio output. Includere le istruzioni direttamente nel prompt utente risulta più efficace rispetto a inserirle nel menu vocale del sistema.</p>
</details>

## Dove trovare altri lab

È possibile esplorare altri lab ed esercizi nel [portale di apprendimento di Fonderia Azure AI](https://ai.azure.com) oppure fare riferimento alla **sezione lab** del corso per altre attività disponibili.
