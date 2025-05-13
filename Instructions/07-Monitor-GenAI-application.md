---
lab:
  title: Monitorare l'applicazione di IA generativa
---

# Monitorare l'applicazione di IA generativa

Questo esercizio richiede circa **30** minuti.

> **Nota**: questo esercizio presuppone una certa familiarità con Fonderia Azure AI, motivo per cui alcune istruzioni sono intenzionalmente meno dettagliate per incoraggiare l'esplorazione più attiva e l'apprendimento pratico.

## Introduzione

In questo esercizio, viene abilitato il monitoraggio per un'app di completamento della chat e vengono visualizzate le prestazioni in Monitoraggio di Azure. È possibile interagire con il modello distribuito per generare dati, visualizzare i dati generati tramite le informazioni dettagliate della dashboard delle applicazioni di IA generativa e configurare avvisi per ottimizzare la distribuzione del modello.

## 1. Configurare l'ambiente

Per completare le attività di questo esercizio, è necessario:

- un hub di Fonderia Azure AI,
- un progetto Fonderia Azure AI
- un modello distribuito (come GPT-4o),
- una risorsa di Application Insights connessa.

### R. Creare un hub e un progetto Fonderia Azure AI

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

### B. Distribuire un modello

Per generare dati che è possibile monitorare, è prima necessario distribuire un modello e interagire con esso. Nelle istruzioni viene chiesto di distribuire un modello GPT-4o, ma **è possibile usare qualsiasi modello** dalla raccolta Servizio OpenAI di Azure disponibile.

1. Utilizzare il menu a sinistra, nella sezione **Risorse personali**, selezionare la pagina **Modelli + endpoint**.
1. Distribuire un **modello di base** e scegliere **gpt-4o**.
1. **Personalizzare i dettagli di distribuzione**.
1. Impostare la **capacità** su **5.000 token al minuto (TPM)**.

L'hub e il progetto sono pronti, con tutte le risorse di Azure necessarie di cui viene eseguito automaticamente il provisioning.

### C. Connetti Application Insights

Connettere Application Insights al progetto in Fonderia Azure AI per avviare il monitoraggio dei dati raccolti.

1. Aprire il progetto nel portale Fonderia Azure AI.
1. Usare il menu a sinistra e selezionare la pagina **Traccia**.
1. **Creare una nuova** risorsa di Application Insights per connettersi all'app.
1. Immettere un **nome della risorsa di Application Insights**.

Application Insights è ora connesso al progetto e verrà iniziata l'analisi dei dati raccolti.

## 2. Interagire con un modello distribuito

Interagire con il modello distribuito a livello di codice configurando una connessione al progetto Fonderia Azure AI usando Azure Cloud Shell. In questo modo sarà possibile inviare una richiesta al modello e generare i dati di monitoraggio.

### R. Connettersi con a un modello tramite Cloud Shell

Iniziare recuperando le informazioni necessarie da autenticare per interagire con il modello. Accedere quindi ad Azure Cloud Shell e aggiornare la configurazione per inviare le richieste fornite al modello distribuito.

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
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Questo comando clona il repository GitHub contenente i file di codice per questo esercizio.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-ai-foundry/Files/07
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

### B. Inviare richieste al modello distribuito

A questo punto è possibile eseguire più script che inviano richieste diverse al modello distribuito. Queste interazioni generano dati che è possibile osservare in un secondo momento in Monitoraggio di Azure.

1. Eseguire il comando seguente per **visualizzare il primo script** fornito:

    ```
   code start-prompt.py
    ```

1. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, immettere il seguente comando per **eseguire l'applicazione**:

    ```
   python start-prompt.py
    ```

    Il modello genererà una risposta, che verrà acquisita con Application Insights per un'ulteriore analisi. È possibile variare le richieste per esplorare i relativi effetti.

1. **Aprire ed esaminare lo script**, in cui la richiesta indica al modello di **rispondere solo con una frase e un elenco**:

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

## 4. Visualizzare il monitoraggio di dati in Monitoraggio di Azure

Per visualizzare i dati raccolti dalle interazioni con il modello, accedere al dashboard che collega a una cartella di lavoro in Monitoraggio di Azure.

### R. Passare a Monitoraggio di Azure dal portale Fonderia Azure AI

1. Passare alla scheda nel browser con il **portale Fonderia Azure AI** aperto.
1. Usare il menu a sinistra, selezionare **Traccia**.
1. Selezionare il collegamento nella parte superiore, che indica **Eseguire il checkout delle informazioni dettagliate per il dashboard delle applicazioni di IA generativa**. Il collegamento aprirà Monitoraggio di Azure in una nuova scheda.
1. Esaminare la **panoramica** fornendo dati riepilogati delle interazioni con il modello distribuito.

## 5. Interpretare le metriche di monitoraggio in Monitoraggio di Azure

Ora è il momento di analizzare i dati e iniziare a interpretare cosa dicono.

### R. Esaminare l'utilizzo dei token

Concentrarsi prima sulla sezione relativa **all'utilizzo dei token** ed esaminare le metriche seguenti:

- **Token di richiesta**: numero totale di token usati nell'input (le richieste inviate) in tutte le chiamate di modello.

> Pensare a questo come al *costo di porre* al modello una domanda.

- **Token di completamento**: numero di token restituiti dal modello come output, essenzialmente la lunghezza delle risposte.

> I token di completamento generati spesso rappresentano la maggior parte dell'utilizzo e dei costi dei token, soprattutto per le risposte lunghe o dettagliate.

- **Totale token**: i token di richiesta combinati e i token di completamento.

> Metrica più importante per la fatturazione e le prestazioni, perché determina la latenza e il costo.

- **Chiamate totali**: numero di richieste di inferenza separate, ovvero quante volte è stato chiamato il modello.

> Utile per analizzare la velocità effettiva e comprendere il costo medio per chiamata.

### B. Confrontare le singole richieste

Scorrere verso il basso per trovare gli **intervalli di IA generativa**, visualizzati come tabella in cui ogni richiesta viene rappresentata come una nuova riga di dati. Esaminare e confrontare il contenuto delle colonne seguenti:

- **Stato**: indica se una chiamata di modello è riuscita o meno.

> Usare questa opzione per identificare le richieste problematiche o gli errori di configurazione. L'ultima richiesta non è probabilmente riuscita perché era troppo lunga.

- **Durata**: indica quanto tempo il modello ha impiegato per rispondere, in millisecondi.

> Confrontare le righe per esplorare quali criteri di richiesta generano tempi di elaborazione più lunghi.

- **Input**: mostra il messaggio utente inviato al modello.

> Utilizzare questa colonna per valutare le formulazioni delle richieste efficienti o problematiche.

- **Sistema**: mostra il messaggio di sistema usato nella richiesta (se presente).

> Confrontare le voci per valutare l'impatto dell'uso o della modifica dei messaggi di sistema.

- **Output**: contiene la risposta del modello.

> Usarlo per valutare il livello di dettaglio, la pertinenza e la coerenza. In particolare in relazione ai conteggi dei token e alla durata.

## 6. (FACOLTATIVO) Creare un avviso

Se si dispone di tempo aggiuntivo, provare a configurare un avviso per notificare quando la latenza del modello supera una determinata soglia. Questo è un esercizio progettato per sfidare l'utente, il che significa che le istruzioni sono intenzionalmente meno dettagliate.

- In Monitoraggio di Azure, creare una **nuova regola di avviso** per il progetto e il modello Fonderia Azure AI.
- Scegliere una metrica, ad esempio **Durata della richiesta (ms)** e definire una soglia (ad esempio, maggiore di 4000 ms).
- Creare un **nuovo gruppo di azioni** per definire la modalità di notifica.

Gli avvisi consentono di prepararsi per la produzione stabilendo il monitoraggio proattivo. Gli avvisi configurati dipendono dalle priorità del progetto e dal modo in cui il team ha deciso di misurare e mitigare i rischi.

## Dove trovare altri lab

È possibile esplorare altri lab ed esercizi nel [portale di apprendimento di Fonderia Azure AI](https://ai.azure.com) oppure fare riferimento alla **sezione lab** del corso per altre attività disponibili.
