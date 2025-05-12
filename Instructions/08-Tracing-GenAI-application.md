---
lab:
  title: Analizzare ed eseguire il debug dell'applicazione di intelligenza artificiale generativa con la traccia
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

### Creare un hub e un progetto Fonderia Azure AI

Per configurare rapidamente un hub e un progetto, di seguito sono disponibili istruzioni semplici per usare l'interfaccia utente del portale Fonderia Azure AI.

1. Passare al portale Fonderia Azure AI: aprire [https://ai.azure.com](https://ai.azure.com).
1. Accedere con le credenziali di Azure.
1. Creare un progetto:
    1. Passare a **Tutti gli hub e progetti**.
    1. Selezionare **+ Nuovo progetto**.
    1. Immettere un **nome di progetto**.
    1. Quando richiesto, **creare un nuovo hub**.
    1. Personalizzare l'hub:
        1. Scegliere **sottoscrizione**, **gruppo di risorse**, **posizione**, ecc.
        1. Connettere una nuova risorsa **Servizi di Azure AI** (ignorare AI Search).
    1. Rivedere e selezionare **Crea**.
1. **Attendere il completamento della distribuzione** (~ 1-2 minuti).

### Distribuire un modello

Per generare dati che è possibile monitorare, è prima necessario distribuire un modello e interagire con esso. Nelle istruzioni viene chiesto di distribuire un modello GPT-4o, ma **è possibile usare qualsiasi modello** dalla raccolta Servizio OpenAI di Azure disponibile.

1. Utilizzare il menu a sinistra, nella sezione **Risorse personali**, selezionare la pagina **Modelli + endpoint**.
1. Distribuire un **modello di base** e scegliere **gpt-4o**.
1. **Personalizzare i dettagli di distribuzione**.
1. Impostare la **capacità** su **5.000 token al minuto (TPM)**.

L'hub e il progetto sono pronti, con tutte le risorse di Azure necessarie di cui viene eseguito automaticamente il provisioning.

### Connetti Application Insights

Connettere Application Insights al progetto in Fonderia Azure AI per iniziare a raccogliere i dati da analizzare.

1. Aprire il progetto nel portale Fonderia Azure AI.
1. Usare il menu a sinistra e selezionare la pagina **Traccia**.
1. **Creare una nuova** risorsa di Application Insights per connettersi all'app.
1. Immettere un **nome della risorsa di Application Insights**.

Application Insights è ora connesso al progetto e verrà iniziata l'analisi dei dati raccolti.

## Eseguire un'app di IA generativa con Cloud Shell

Verrà effettuata la connessione al progetto Fonderia Azure AI da Azure Cloud Shell e verrà eseguita l'interazione a livello di codice con un modello distribuito come parte di un'app di IA generativa.

### Interagire con un modello distribuito

Iniziare recuperando le informazioni necessarie da autenticare per interagire con il modello distribuito. Quindi, sarà possibile accedere ad Azure Cloud Shell e aggiornare il codice dell'app di IA generativa.

1. Nel Portale Fonderia Azure AI visualizzare la pagina **Panoramica** per il progetto.
1. Nell'area **Dettagli di progetto** prendere nota della **stringa di connessione del progetto**.
1. **Salvare** la stringa in un Blocco note. Questa stringa di connessione verrà usata per connettersi al progetto in un'applicazione client.
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
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
    ```

1. Immettere il comando seguente per aprire il file di configurazione fornito:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice:

    1. Sostituire il segnaposto **your_project_connection_string** con la stringa di connessione del progetto (copiata dalla pagina **Panoramica** del progetto nel portale Fonderia Azure AI).
    1. Sostituire il segnaposto **your_model_deployment** con il nome assegnato alla distribuzione modello GPT-4o (per impostazione predefinita `gpt-4o`).

1. *Dopo* aver sostituito i segnaposto, nell'editor di codice usare il comando **CTRL+S** o **fare clic con il pulsante destro del mouse > Salva** per **salvare le modifiche**.

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
   def call_model(system_prompt, user_prompt, span_name):
        with tracer.start_as_current_span(span_name) as span:
            span.set_attribute("session.id", SESSION_ID)
            span.set_attribute("prompt.user", user_prompt)
            start_time = time.time()
    
            response = chat_client.complete(
                model=model_name,
                messages=[SystemMessage(system_prompt), UserMessage(user_prompt)]
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
1. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, immettere il seguente comando per **eseguire l'applicazione**:

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
