---
lab:
  title: Ottimizzare il modello usando un set di dati sintetico
---

## Ottimizzare il modello usando un set di dati sintetico

L'ottimizzazione di un'applicazione di IA generativa comporta l'uso di set di dati per migliorare le prestazioni e l'affidabilità del modello. Usando dati sintetici, gli sviluppatori possono simulare un'ampia gamma di scenari e casi limite che potrebbero non essere presenti nei dati reali. Inoltre, la valutazione degli output del modello è fondamentale per ottenere applicazioni di IA affidabili e di alta qualità. L'intero processo di ottimizzazione e valutazione può essere gestito in modo efficiente tramite l'SDK di valutazione di Azure AI, che mette a disposizione strumenti e framework avanzati per semplificare e standardizzare queste operazioni.

## Scenario

Si supponga di voler creare un'app di guida intelligente basata su intelligenza artificiale per migliorare l'esperienza dei visitatori in un museo. L'applicazione ha lo scopo di rispondere a domande su personaggi storici. Per valutare le risposte dell'app, è necessario creare un set di dati sintetico completo di domande e risposte che copra vari aspetti di queste personalità e del loro lavoro.

È stato selezionato un modello GPT-4 per fornire risposte generative. È ora necessario sviluppare un simulatore in grado di generare interazioni contestualmente pertinenti, al fine di valutare le prestazioni dell'IA in una varietà di scenari.

Per iniziare, distribuire le risorse necessarie per creare l'applicazione.

## Creare un progetto e un hub di Azure per intelligenza artificiale

È possibile creare un hub e un progetto di Azure AI manualmente tramite il portale Fonderia Azure AI, nonché distribuire i modelli usati nell'esercizio. Tuttavia, è possibile automatizzare questo processo usando un'applicazione modello con [Azure Developer CLI (azd)](https://aka.ms/azd).

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copiare questi valori poiché verranno usati in seguito.

## Configurare un ambiente di sviluppo locale

Per favorire una sperimentazione rapida e iterativa, verrà usato un notebook Python all'interno di Visual Studio Code (VS Code). Preparare VS Code per l'ideazione locale.

1. Aprire VS Code e **clonare** il repository Git seguente: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Archiviare il clone in un'unità locale e aprire la cartella dopo la clonazione.
1. Nel riquadro sinistro di Esplora VS Code, aprire il notebook **06-Optimize-your-model.ipynb** nella cartella **File/06**.
1. Eseguire tutte le celle nel notebook.

## Eseguire la pulizia

Al termine dell'esplorazione di Servizi di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com?azure-portal=true) in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
