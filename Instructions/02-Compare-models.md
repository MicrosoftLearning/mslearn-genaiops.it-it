---
lab:
  title: Confrontare i modelli linguistici dal catalogo dei modelli
  description: Informazioni su come confrontare e selezionare i modelli appropriati per il progetto di IA generativa.
---

## Confrontare i modelli linguistici dal catalogo dei modelli

Dopo aver definito il caso d'uso, è possibile usare il catalogo dei modelli per verificare se un modello di intelligenza artificiale risolve il problema. È possibile usare il catalogo dei modelli per selezionare i modelli da distribuire, che è quindi possibile confrontare per verificare quale sia il modello più adatto alle proprie esigenze.

In questo esercizio vengono confrontati due modelli linguistici tramite il catalogo dei modelli nel Portale Fonderia Azure AI.

Questo esercizio richiederà circa **30** minuti.

## Scenario

Si supponga di voler creare un'app per aiutare gli studenti a imparare a scrivere codice in Python. Nell'app si vuole un tutor automatizzato che possa aiutare gli studenti a scrivere e valutare il codice. In un esercizio, gli studenti devono trovare il codice Python necessario per tracciare un grafico a torta, basato sull'immagine di esempio seguente:

![Grafico a torta che mostra i voti ottenuti in un esame con le sezioni di matematica (34,9%), fisica (28,6%), chimica (20,6%) e inglese (15,9%)](./images/demo.png)

È necessario selezionare un modello linguistico che accetti immagini come input e che sia in grado di generare codice accurato. I modelli disponibili che soddisfano questi criteri sono GPT-4 Turbo, GPT-4o e GPT-4o mini.

Per iniziare, distribuire le risorse necessarie per lavorare con questi modelli nel Portale Fonderia Azure AI.

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

## Confrontare i modelli

Si sa che esistono tre modelli che accettano immagini come input la cui infrastruttura di inferenza è completamente gestita da Azure. A questo scopo, è necessario confrontarli per decidere quale sia l'ideale per il proprio caso d'uso.

1. In una nuova scheda del browser aprire il [Portale Fonderia Azure AI](https://ai.azure.com) all'indirizzo `https://ai.azure.com` e accedere usando le credenziali di Azure.
1. Se richiesto, selezionare il progetto di intelligenza artificiale creato in precedenza.
1. Usando il menu a sinistra, passare alla pagina **Catalogo modelli**.
1. Selezionare **Confronta modelli** (trovare il pulsante accanto ai filtri nel riquadro di ricerca).
1. Rimuovere i modelli selezionati.
1. Uno per uno, aggiungere i tre modelli da confrontare: **gpt-4**, **gpt-4o** e **gpt-4o-mini**. Per **gpt-4**, assicurarsi che la versione selezionata sia **turbo-2024-04-09**, perché è l'unica versione che accetta immagini come input.
1. Modificare l'asse x in **Accuratezza**.
1. Verificare che l'asse y sia impostato su **Costo**.

Esaminare il tracciato e provare a rispondere alle domande seguenti:

- *Qual è il modello più accurato?*
- *Qual è il modello più economico da usare?*

L'accuratezza della metrica del benchmark viene calcolata in base ai set di dati generici disponibili pubblicamente. Dal tracciato è già possibile filtrare uno dei modelli, in quanto ha il costo più alto per token, ma non l'accuratezza più elevata. Prima di prendere una decisione, si esaminerà la qualità degli output dei due modelli rimanenti specificamente per il proprio caso d'uso.

## Configurare l'ambiente di sviluppo in Cloud Shell

Per sperimentare ed eseguire rapidamente l'iterazione, si userà un set di script Python in Cloud Shell.

1. Tornare alla scheda Portale di Azure e passare al gruppo di risorse creato in precedenza dallo script di distribuzione, quindi selezionare la risorsa **Fonderia Azure AI**.
1. Nella pagina **Panoramica **della risorsa selezionare **Fare clic qui per visualizzare gli endpoint** e copiare l'endpoint dell'API Fonderia AI.
1. Salvare l'endpoint in un Blocco note. Verrà usato per connettersi al progetto in un'applicazione client.
1. Tornare nella scheda Portale di Azure e aprire Cloud Shell se è stato chiuso prima, quindi eseguire il comando seguente per passare alla cartella con i file di codice usati in questo esercizio:

     ```powershell
    cd ~/mslearn-genaiops/Files/02/
     ```

1. Nel riquadro della riga di comando di Cloud Shell, immettere il comando seguente per installare le librerie che verranno utilizzate:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai matplotlib
    ```

1. Immettere il comando seguente per aprire il file di configurazione fornito:

    ```powershell
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice sostituire il segnaposto **your_project_endpoint** con l'endpoint del progetto copiato in precedenza. Come si può notare, il primo e il secondo modello usati nell'esercizio sono rispettivamente **gpt-4o** e **gpt-4o-mini**.
1. *Dopo* aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Inviare richieste ai modelli distribuiti

A questo punto è possibile eseguire più script che inviano richieste diverse ai modelli distribuiti. Queste interazioni generano dati che è possibile osservare in un secondo momento in Monitoraggio di Azure.

1. Eseguire il comando seguente per **visualizzare il primo script** fornito:

    ```powershell
   code model1.py
    ```

Lo script codificherà l'immagine usata in questo esercizio in un URL dati. Questo URL verrà usato per incorporare l'immagine direttamente nella richiesta di completamento della chat insieme alla prima richiesta di testo. Lo script restituirà successivamente la risposta del modello e la aggiungerà alla cronologia della chat, quindi invierà una seconda richiesta. La seconda richiesta viene inviata e archiviata allo scopo di rendere più significative le metriche osservate più avanti, ma è possibile rimuovere il commento dalla sezione facoltativa del codice per avere anche la seconda risposta come output.

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per accedere ad Azure.

    ```
   az login
    ```

    **<font color="red">È necessario accedere ad Azure, anche se la sessione di Cloud Shell è già autenticata.</font>**

    > **Nota**: nella maggior parte degli scenari, il semplice uso di *az login* sarà sufficiente. Tuttavia, in caso di sottoscrizioni in più tenant, potrebbe essere necessario specificare il tenant usando il parametro *--tenant*. Per dettagli, visualizzare [Accedere ad Azure in modo interattivo usando l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Quando richiesto, seguire le istruzioni per aprire la pagina di accesso in una nuova scheda e immettere il codice di autenticazione fornito e le credenziali di Azure. Completare quindi il processo di accesso nella riga di comando, selezionando la sottoscrizione contenente l'hub di Fonderia Azure AI, se richiesto.
1. Dopo aver eseguito l'accesso, immettere il comando seguente per eseguire l'applicazione:

    ```powershell
   python model1.py
    ```

    Il modello genererà una risposta, che verrà acquisita con Application Insights per un'ulteriore analisi. Si userà ora il secondo modello per esplorare le differenze.

1. Nel riquadro della riga di comando di Cloud Shell sotto l'editor di codice immettere il comando seguente per eseguire il **secondo** script:

    ```powershell
   python model2.py
    ```

    Ora che sono stati generati output da entrambi i modelli, si nota qualche differenza?

    > **Nota**: Facoltativamente, è possibile testare gli script forniti come risposte copiando i blocchi di codice, eseguendo il comando `code your_filename.py`, incollando il codice nell'editor, salvando il file e quindi eseguendo il comando `python your_filename.py`. Se lo script è stato eseguito correttamente, si dovrebbe avere un'immagine salvata che può essere scaricata con `download imgs/gpt-4o.jpg` o `download imgs/gpt-4o-mini.jpg`.

## Confrontare l'utilizzo dei token dei modelli

Si eseguirà infine un terzo script che traccia il numero di token elaborati nel tempo per ogni modello. Questi dati vengono ottenuti da Monitoraggio di Azure.

1. Prima di eseguire l'ultimo script, è necessario copiare l'ID della risorsa per la risorsa Fonderia Azure AI dal portale di Azure. Passare alla pagina di panoramica della risorsa Fonderia Azure AI e selezionare **Visualizzazione JSON**. Copiare l'ID della risorsa e sostituire il segnaposto `your_resource_id` nel file di codice:

    ```powershell
   code plot.py
    ```

1. Salva le modifiche.

1. Nel riquadro della riga di comando di Cloud Shell sotto l'editor di codice immettere il comando seguente per eseguire il **terzo** script:

    ```powershell
   python plot.py
    ```

1. Al termine dello script, immettere il comando seguente per scaricare il tracciato delle metriche:

    ```powershell
   download imgs/plot.png
    ```

## Conclusione

Dopo aver esaminato il tracciato e tenendo presenti i valori di benchmark osservati in precedenza nel grafico di confronto tra accuratezza e costo, è possibile determinare qual è il modello migliore per il caso d'uso specifico? La differenza nell'accuratezza degli output ha un peso maggiore rispetto alla differenza nei token generati e quindi nei costi?

## Eseguire la pulizia

Al termine dell'esplorazione di Servizi di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com?azure-portal=true) in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
