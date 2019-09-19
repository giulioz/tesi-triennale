# Tesi di Laurea Baccega Sandro

## Refactoring di una pipeline di continuous integration per il build e il deploy automatico di plugin Liferay

### Abstract

Delle pipeline Jenkins per il CI/CD (Continuous Integration/Continuous Delivery) di plugins Liferay utlizzando un cluster Kubernetes come ambiente di deploy automatico. Grazie ad esse è possibile eseguire una serie di operazioni automatiche come: analisi statica del codice, versionamento, building, backup dell'eseguibile, deploy di un Docker container con i plugins appena rilasciati.



## Introduzione

### L'Azienda



### Il Progetto





## Strumenti Utilizzati

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

Docker Engine è un programma eseguibile su vari sistemi operativi Linux *(CentOS, Debian, Fedora, Oracle Linux, RHEL, SUSE e Ubuntu)* e *Windows Server* che permette una semplice gestione dei Containers. Docker Engine in particolare gestisce l'interfaccia tra l'host/utente con i vari Containers.

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

- group names

## Sonar

## Git ?

- repository
- branch

## Jenkins

- enviroment
- credentials
- steps
- Groovy



## Generic Release Pipeline

La prima parte del progetto riguardava il miglioramento di una Jenkins pipeline dedicata al rilascio di un plugin Liferay di un qualsiasi progetto per la quale era stata configurata. Tutto questo si è tradotto nella creazione della Generic Release Pipeline.

### Confronto Generic Release Pipeline e vecchia Pipeline

In particolare, la vecchia pipeline permetteva di: 

- Eseguire un insieme di test con *Sonar* che il codice deve superare in modo da garantirne la qualità
- Versionare automaticamente il modulo
- Storing a lungo termine in una *Nexus* repository, in modo da avere tutti gli eseguibili delle varie versioni del plugin in storage, garantendo così un veloce rollback nel caso in cui la nuova versione del plugin presenti qualche problema.
- Commiting sul branch **release** della repository in modo da separare il codice funzionante dal codice in production.
- Inviare un report sul rilascio del plugin all'utente 

Nelle prime versioni della Generic Release Pipeline, quando aveva raggiunto la quantità di features della vecchia versione, il codice era intorno alle 700 righe con codice in più per eseguire certi stadi in parallelo , mentre la controparte aveva più di 1400 righe non avendo nessun supporto per eseguire codice in parallelo.

Oltre alle features della vecchia pipeline ne sono state aggiunte molte altre:

- Rilascio di Temi e Layout (Prima erano rilasciabili solo i moduli)
- Avere una pipeline unica per tutti i progetti invece di avere pipeline tutte uguali per ogni progetto
- Possibilità di rilasciare multipli plugins contemporaneamente
- Risoluzione di problemi di dipendenze dovuti al rilascio contemporaneo
- Uploading del plugin rilasciato in un Docker Container già istanziato con la corretta versione di Liferay
- Test via Telnet dello stato di installazione del plugin
- Error reporting per facilitarne il debug

### Features

#### Rilascio di Temi e Layout

La più grande limitazione della vecchia Pipeline era il poter rilasciare solo moduli Liferay, non permettendo quindi il rilascio di Temi e Layout.

Il motivo principale di questa grave mancanza era la sua difficoltà, infatti il rilascio di temi è profondamente diverso dal rilascio di moduli, in quanto i temi vanno compilati utilizzando **NodeJS** al posto di **Gradle**.

Infatti per distinguerli è stato necessario creare una superclasse *Plugin* e delle sottoclassi *Module*, *Theme* e *Layout*.

Grazie a Groovy infatti è possibile utlizzare il **dynamic dispatching** per creare una semplice struttura dati che contiene Plugins ed inserirci delle sottoclassi in modo da rendere il codice molto più semplice e chiaro.

Per esempio: 

```groovy
// Data entry
def dataStructure = new DataStruction<Plugin>()
dataStructure.add(new Module())
dataStructure.add(new Theme())
// Pipeline logic
for(plugin in dataStruction){
  plugin.compile()
}
// Output: 
// Il modulo verrà compilato con Gradle
// Il tema verrà compilato con NodeJS
```

Questo semplice esempio dimostra come diventa semplice la logica con una struttura dati di questo tipo, anche se ci possono esseri molti tipi diversi di plugin. 

 

Per il compile dei temi è stato necessario fare un analisi sugli strumenti da utilizzare.

Il problema principale da risolvere era che i temi Liferay variano la versione di NodeJS in base al progetto (ovviamente un progetto più vecchio avrà una versione di Node meno recente), quindi lo strumento scelto deve andar bene con tutte le versioni di Node.

Alla fine c'erano 3 scelte:

- Utilizzare un Docker Container con NodeJS
- Utilizzare NVM (Node Version Manager)
- Utlizzare il NodeJS Plugin di Jenkins

Utilizzare un Docker Container con un immagine di NodeJS:

|                             PRO                              |                            CONTRO                            |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|   Si può utilizzare qualunque versione di Node si desideri   | Avviare un container per ogni tema che si rilascia è un dispendio di risorse non indifferente |
| Facilmente parallelizzabile indipendentemente da quante pipeline sono in esecuzione |                                                              |

Utilizzare NVM (Node Version Manager):

|      PRO       |                            CONTRO                            |
| :------------: | :----------------------------------------------------------: |
|  Molto veloce  | Potrebbe causare problemi se si avviano più pipeline in contemporaneo, <br />dato che NVM cambia la versione globale di NodeJS. |
| Molto semplice |                                                              |

Utilizzare NodeJS Plugin for Jenkins:

|       PRO        |                            CONTRO                            |
| :--------------: | :----------------------------------------------------------: |
|      Veloce      |      Molto difficile trovare documentazione al riguardo      |
| Parallelizzabile | Necessario installare a mano ogni versione nuova di cui si ha bisogno |

Alla fine quest'ultima si è rivelata l'opzione migliore.

#### Avere una pipeline unica per tutti i progetti invece di avere pipeline tutte uguali per ogni progetto

Per poter utilizzare la vecchia Pipeline occorreva andare sull'interfaccia di Jenkins, copiarla e modificare le configurazioni adatte, ogni volta che la si voleva utlizzare su un progetto.

Uno degli obbiettivi del progetto era creare una Pipeline unica che potesse essere utilizzato per qualsiasi progetto, dividendo le configurazioni necessarie in: 

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

  ### Esempio

  ```
  FIX      1.2.5   -> 1.2.6
  RELEASE  1.2.5   -> 1.3.0
  ```

  - **USING_SONAR** [*true/false*]:

    Se vuoi usare *Sonar* per l'analisi del codice.

  - **SENDING_EMAIL** [*true/false*]:

    Se vuoi ricevere un email di successo/fallimento della pipeline.

  - **CONTAINER_URL**:

    Specifica l'url del Docker Container nella quale uploadare il plugin una volta rilasciato. 

  - **TELNET_CHECK** [*true/false*]:

    Se vuoi controllare lo stato di installazione del plugin all'interno del Docker Container, tramite telnet.

    

  Così facendo si è potuto organizzare meglio l'interfaccia di Jenkins, che si stava intasando di Pipeline, e semplificare l'integrazione DevOps per ogni progetto.

#### Possibilità di rilasciare multipli plugins contemporaneamente

Uno delle comodità d'uso che ha portato la nuova Pipeline è il rilascio di multipli plugins contemporaneamente.

Con la nuova pipeline è possibile specificare il nome della cartella da rilasciare, sia che essa contenga un plugin, sia che ne contenga più di uno.

Per fare questo utilizza un algoritmo ricorsivo che: 

- Visita le cartelle del progetto
- Identifica quanti plugins si intende rilasciare
- Identifica il tipo di ogni plugin da rilasciare (Modulo, Tema o Layout)

 

#### Risoluzione di problemi di dipendenze dovuti al rilascio contemporaneo

Uno dei problemi più grandi del progetto si è rivelato la risoluzione dei problemi di dipendenze dovuti al rilascio contemporaneo di plugins.

In particolare il problema si riscontrava quando si intendeva rilasciare dei plugins con delle dipendenze del tipo:

```
Plugin_1;Plugin_2
(dove Plugin_1 dipende da Plugin_2)
```

Questo generava un output del tipo:

```
Compilazione di Plugin_1 -> Errore
Compilazione di Plugin_2 -> Successo
```

E per rilasciarli entrambi bisognava rilanciare la Pipeline una seconda volta, ora che Plugin_2 è stato compilato.

Questo era un grosso problema se si intendeva rilasciare un progetto intero (per esempio rilasciando tutta la cartella `modules`), in quanto era necessario rilanciare più di una volta la stessa Pipeline, sprecando risorse e tempo.

La soluzione che si è adottato è l'aggiunta di un campo `priority` dentro la classe Plugin che stà ad indicare se il plugin dipende da altri plugin con il quale sta per essere rilasciato, per esempio:

```
priority = 1		->		Plugin non dipende da nessun altro
priority = 2		->		Plugin dipende da un plugin
priority = n		->		Plugin dipende da n plugins
```

E poi si procedeva a compilare tutti i plugins in sequenza a seconda della priorità (Naturalmente tutti i plugin con la stessa priorità vengono compilati in parallelo).

Per ottenere il campo `priority` è stato necessario cercare manualmente nelle dipendenze contenute nel file `build.gradle` di ogni modulo e confrontarle con la lista dei plugin in rilascio.  

#### Uploading del plugin rilasciato in un Docker Container già istanziato con la corretta versione di Liferay



- Test via Telnet dello stato di installazione del plugin

- Error reporting per facilitarne il debug

  

### Funzionamento

- - 

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