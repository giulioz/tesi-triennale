# Tesi di Laurea

## Refactoring di una pipeline di continuous integration per il build e il deploy automatico di plugin Liferay

Una pipeline Jenkins per il CI/CD (Continuous Integration/Continuous Delivery) di plugins Liferay utlizzando un cluster Kubernetes come ambiente di deploy automatico. Grazie ad essa è possibile eseguire una serie di operazioni automatiche come: analisi statica del codice, versionamento, building, backup dell'eseguibile, deploy di un container con il plugin appena rilasciato.





## Introduzione Devops



## Liferay

## Docker

**Docker** è un progetto open-source che permette di automatizzare il deployment di applicazioni all'interno di **Container** software, che sono tra loro indipendenti anche se coesistono sulla stessa macchina.

### Cos'è un Container

Un Container è un pacchetto software che contiene un programma e tutte le dipendenze di cui quel programma ha bisogno, che è completamente (o quasi) isolato dal sistema sul quale è ospitato, in modo da risolvere problemi di versioni di dipendenze diverse e molto altro.

L'obiettivo principale dei containers è quello di eliminare il frequente problema del "Funziona sulla mia macchina", che si presenta durante il software developement, fornendo un ambiente di sviluppo comune a tutti gli sviluppatori del progettto a prescindere dal software o sistemi operativi che preferiscono.  

In particolare i **Docker Containers** sono:

- **Sicuri**: Ogni Docker Container provvede la migliore sicurezza di default rispetto ad ogni altra soluzione di virtualizzazione perchè è completamente isolato dagli altri e la comunicazione tra lui e l'host è permessa solo su alcune porte che bisogna esporre manualmente.
- **Standardizzati**: Docker ha creato lo standard per i containers in modo che possono essere il più portabili possibile (Ci sono altri manager di Container, ma Docker è il più famoso).

### Container vs Virtual Machines

Quando si parla di containers la prima cosa che salta in mente sono le Virtual Machines, in quanto condividono gli stessi obbiettivi di isolamento del software, tuttavia ci sono vari motivi per cui i container sono preferibili ad una VM:


- **Occupano meno spazio sul disco**: I Container condividono il kernel dell'host OS, quindi un immagine Docker occupa mediamente qualche centinaio di MB (Una VM occupa di solito qualche GB)
- **Boot-up Time**: Un container ci mette al massimo qualche secondo ad avviarsi (Una VM ci può mettere minuti)
- **Performance migliori**: Un container tende ad avere migliori e più consistenti performance. 
- **Facile da configurare e scalare**: Creare e scalare un container è semplice, mentre le VM sono lunghe da configurare e spesso complicate da scalare.
- **Più portabili**: Docker è compatibile con tutti i maggiori OS permettendo a tutti di usare lo stesso environment per sviluppare un applicazione e di usare gli strumenti che i sviluppatori preferiscono.
- **Condividono facilmente volumi di dati con l'host OS**: È possibile condividere facilmente una directory con un container. 


### Come si crea un Docker Container?

Per creare un Docker Container bisogna avere una **Docker Image**, cioè un pacchetto standalone che contiene tutto quello che ti serve per avviare l'applicazione: codice, dipendenze, impostazioni, tools e librerie di sistema.

Ci sono due modi per ottenere una Docker Image:

- **Scaricare un immagine già pronta da un Registry**: Per applicazioni comuni (Per esempio un web server apache già pronto)
- **Creare la propria immagine partendo da un altra**: Se servono più funzionalità rispetto all'immagine di base (Esporre una particolare porta del container, oppure installare qualche pacchetto in più nel container)


Infine per eseguire una Docker Image (quindi per creare un Container), è necessario usare un **Docker Engine**.

### Docker Engine

Docker Engine è un programma eseguibile su vari sistemi operativi Linux *(CentOS, Debian, Fedora, Oracle Linux, RHEL, SUSE e Ubuntu)** e *Windows Server** che permette una semplice gestione dei Containers. Docker Engine in particolare gestisce l'interfaccia tra l'host/utente con i vari Containers.

Dato che è stato sviluppato con la sicurezza delle applicazioni come obiettivo, dispone di sistemi di sicurezza come encryption e metodi per la certificazione delle immagini (per evitare di eseguire immagini contraffatte).


### Docker Registry

Un **Docker Registry** è una repository di immagini Docker che può essere pubblica o privata su cui si possono fare semplici operazioni di **push** (Pubblicazione di un immagine in un registry) e di **pull** (Download di un immagine da un registry):

`docker push publisher/imageName:tag`
`docker pull publisher/imageName:tag`

- *publisher*: Nome della compagnia o dell'utente che ha creato e mantiene l'immagine
- *tag*: Versione dell'immagine desiderata, di default è "latest"

Normalmente si sconsiglia l'uso del tag "latest" in quanto le immagini tendono ad essere aggiornate regolarmente e quindi se si utilizza "latest" per creare la propria immagine, quindi ogni volta che si costruisce un immagine si dovrà ogni volta riscaricare  la versione più aggiornata dell'immagine se disponibile, occupando inutilmente molto spazio sulla memoria dell'host che si utilizza.


**Docker Hub** é il più grande registry pubblico per trovare e condividere Docker Images attualmente disponibile, utilizzato da numerosi sviluppatori della community e da aziende.
Infatti sono presenti numerosissime immagini create e mantenute direttamente dagli sviluppatori di un certo servizio, queste immagini sono di solito certificate da Docker o dal pubblicatore stesso, il quale ne garantisce la qualitá.

Se si desidera creare un container per un servizio di cui si ha bisogno, di solito, la prima cosa da fare é cercare un immagine direttamente nell' Hub e se serve qualcosa di piú specifico partire da una di esse e costruirne una di nuova. Infatti sono presenti immagini per ogni necessitá come:


- **Web server** *(Node, Apache HTTP, Apache Tomcat, ...)*
- **Database** *(MySQL, Mongo DB, Redis, ...)*
- **Compilatori ed Interpreti** *(Java, GCC, Perl, Go, Php, Python, ...)*
- **Content Management System (CMS)** *(Wordpress, Liferay, Joomla ...)*
- **Sistemi Operativi** *(Ubuntu, CentOS, Alpine, Windows, ...)*
- **Docker** (È possibile usare Docker dentro un Container)

## Kubernetes

## Nexus

## Sonar

## Git

## Jenkins

- enviroment
- credentials
- steps
- Groovy



## Generic Release Pipeline

Il primo problema da risolvere è stato la creazione della Generic Release Pipeline, ovvero una Jenkins pipeline dedicata al rilascio di uno o più plugin Liferay di un qualsiasi progetto.

Questa pipeline è stata creata a partire da un altra pipeline pre-esistente con lo stesso obbiettivo finale, ma che è stata creata con altri  obbiettivi e aggiornata man mano con nuove features, rendendola molto lenta, soprattutto perchè non prevedeva nessuna forma di *multithreading*, e molto complicata, in quanto è stata creata da molte persone diverse che hanno ripetuto molte parti di essa inutilmente. 

Nelle prime versioni della Generic Release Pipeline, quando aveva raggiunto la quantità di features della vecchia versione, il codice era intorno alle 700 righe con codice in più per eseguire certi stadi in parallelo , mentre la controparte aveva più di 1400 righe non avendo nessun supporto per eseguire codice in parallelo.

L'obiettivo della pipeline consiste nel: 

- Eseguire un insieme di test *Sonar* che il codice deve superare in modo da garantire la qualità del codice
- Storing a lungo termine in una *Nexus* repository, in modo da avere tutti gli eseguibili delle varie versioni del plugin in storage, garantendo così un veloce rollback nel caso in cui la nuova versione del plugin presenti qualche problema.
- Commiting sul branch **release** della repository in modo da separare il codice funzionante dal codice in production.
- Uploading su un container in esecuzione nel caso in cui si voglia automaticamente esseguire un live test del nuovo plugin su un Docker container già istanziato.
- Inviare un report sul rilascio del plugin all'utente  

Per funzionare la pipeline ha bisogno di due cose:

- Un file `gradle.properties` (che è già presente nella root in un progetto Liferay) con le seguenti configurazioni:

  - **major.version**:

    La Major Version del progetto, da cambiare se il progetto cambia in maniera drastica (come per il supporto ad una versione di Liferay diversa da quella precedente, rendendo la nuova versione incompatibile con quella precedente).

  - **group.name**:

    Project group name usato da Sonatype Nexus per identificare e trovare il progetto.

  - **email.devops**:

    Indirizzo email nella quale si vuole inviare i risultati del rilascio dei plugins (successo/fallimento).

  - **node.version**:

    Versione di node richiesta per compilare i temi Liferay.

  - **liferay.version**:

    Versione di Liferay richiesta per eseguire il plugin in rilascio (Richiesto solo nel caso in cui si voglia usare Kubernetes)

  ### Esempio

```
...
major.version=1
group.name=it.smc.example.test
email.devops=example@gmail.com
node.version=6.10.3
liferay.version=7.0.6-ga7
```

- Input da inserire durante l'avvio della pipeline:

  - **PROJECT_REPOSITORY**:

    Un link alla repository *Git* del progetto

  ```
  https://git.smc.it/mario.rossi/example.git
  ```

  - **MASTER_BRANCH**:

    Nome del branch che contiene il plugin che si vuole rilasciare, ci sono 2 sitassi supportate:

  ```
  master
  master-major_version    // Utile per mantere plugin di vecchie versioni
  ```

  - **PLUGIN_NAMES**:

    Nomi delle cartelle che contengono i plugins che si intendono rilasciare (moduli, temi o layout). 

    È importante che i nomi siano i nomi delle **CARTELLE** dove i plugins risiedono e non il symbolic name del plugin.

    Ogni plugin nella cartella indicata verrà rilasciato, quindi se una cartella `search-modules` continene più moduli, verranno rilasciati tutti e tre.

    Per rilasciare più plugins si separano con un `;` nella seguente maniera:

```
test-module;test-theme;test-layout
```

- ​	**BUILD_TYPE** [*fix/release*]:

  ​	Specifica se il rilascio è di tipo *fix*: (versione X.X.(x+1)) o una release (versione X.(X + 1).0)

  ### 	Esempio

  ```
  FIX      1.2.5   -> 1.2.6
  RELEASE  1.2.5   -> 1.3.0
  ```

  - **USING_SONAR** [*true/false*]:

    If you want to use SONAR for code analysis. <br>
    **It's optional if you are only if you are not releasing modules**

  - **SENDING_EMAIL** [*true/false*]:

    If you want to receive an email in case of success/error of the pipeline (to EMAIL_DEVOPS address)

  - **CONTAINER_URL**:

    Specify the url of the active docker container that you want to upload the builded plugin into.

  - **TELNET_CHECK** [*true/false*]:

    If you want to check the status of the plugin that you just installed on the container



```

```



Ora inizieremo a descrivere ogni *step* della pipeline, spiegheremo che problema risolve e come lo risolve.

### Checkout

Il primo stadio della pipeline è il checkout del *master* e *release* branch, ovvero il download di tutto il sorgente dalla repository **Gitlab** proveniente dal branch *master*, che può essere settato con il campo dell'input `MASTER_BRANCH`, e dal branch *release*, che viene ottenuto nella seguente maniera:

```
MASTER_BRANCH   = master
RELEASE_BRANCH -> release

MASTER_BRANCH   = master-2
RELEASE_BRANCH -> release-2
```

In questo modo è possibile rilasciare plugins provenienti da branch diversi da master e release, permettendo una maggiore flessibilità nel caso si vogliano eseguire dei test più completi.

Questo stadio è necessario per avere il sorgente necessario da analizzare e compilare negli stadi successivi della pipeline.

La sintassi per eseguire un checkout è la seguente:

```groovy
checkout(
    [
      $class: 'GitSCM',
      branches: [[name: "*/${env.MASTER_BRANCH}"]],
      doGenerateSubmoduleConfigurations: false,
      extensions: [],
      submoduleCfg: [],
      userRemoteConfigs: [[credentialsId: env.CREDENTIALS_ID, url: env.PROJECT_REPOSITORY]]
    ]
  )
```



- Finding Plugins
- Copy Plugins
- Analyze with Sonar
- Quality Gates
- Versioning
- Releasing Plugins
- Commiting to Release Branch
- Uploading to Container
- Report and Cleanup

## Deploying to Cluster Pipeline

- Checkout
- Setup Enviroment
- Building Docker Image
- Uploading to Registry
- Deploying Container To Cluster
- Cleanup

## Conclusione (Sviluppi futuri)

## Souces

https://www.docker.com/