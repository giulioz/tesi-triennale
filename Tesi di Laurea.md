# Tesi di Laurea

## Refactoring di una pipeline di continuous integration per il build e il deploy automatico di plugin Liferay

Una pipeline Jenkins per il CI/CD (Continuous Integration/Continuous Delivery) di plugins Liferay utlizzando un cluster Kubernetes come ambiente di deploy automatico. Grazie ad essa è possibile eseguire una serie di operazioni automatiche come: analisi statica del codice, versionamento, building, backup dell'eseguibile, deploy di un container con il plugin appena rilasciato.





## Introduzione Devops



## Liferay

## Jenkins (Groovy)

## Docker

## Kubernetes

## Nexus

## Sonar

## Git



## Generic Release Pipeline

La prima parte dello stage è stata dedicata alla creazione della Generic Release Pipeline, ovvero una Jenkins pipeline dedicata al rilascio di uno o più plugin Liferay di un certo progetto.

L'obiettivo della pipeline consiste nel: 

- Eseguire un insieme di test *Sonar* che il codice deve superare in modo da garantire la qualità del codice
- Storing a lungo termine in una *Nexus* repository, in modo da avere tutti gli eseguibili delle varie versioni del plugin in storage, garantendo così un veloce rollback nel caso in cui la nuova versione del plugin presenti qualche problema.
- Commiting sul branch **release** della repository in modo da separare il codice funzionante dal codice in production.
- Uploading su un container in esecuzione nel caso in cui si voglia automaticamente esseguire un live test del nuovo plugin su un Docker container già istanziato.
- Inviare un report sul rilascio del plugin all'utente  

- Checkout
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

