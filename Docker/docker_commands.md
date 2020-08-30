# Comandi docker

## Login Docker Hub
Effettuare il login a docker con il comando 
``` 
docker login --username=yourhubusername --email=youremail@company.com
``` 
Inserire la password una volta lanciato il comando. Se tutto va a buon fine dovrebbe uscire il messaggio 
>WARNING: login credentials saved in /home/username/.docker/config.json
>Login Succeeded

## Push e Pull su Docker Hub
Il Docker Hub è dove vengono salvate le vostre immagini. Una volta connessi al docker hub per visualizzare le immagini presenti sul proprio dispositivo lanciare il comando
``` 
docker images
```
### Push 
```
docker push <image>:<tag>
```
### Pull
per effettuare il pull dell'ultima versione
```
docker pull <image>
```
Altrimenti, per scegliere una versione utilizzare il comando
```
docker pull <image>:<tag>
```

## Creare un repository
Andare sul sito di [DockerHub](https://hub.docker.com/) e cliccare su "Create Repository" 

## Connettere Docker Repository a GitHub
Per connettere il repository di Docker a GitHub bisogna innaniztutto connettere Docker a Github:
* Da DockerHub andare in Account Settings -> Linked Accounts. 
* Cliccare sul link di GitHub
* Selezionare le opzioni pubblica e privata 
Una volta connesso l'account basta andare sul repository di Docker, che si vuole collegare ad un repository di GitHub:
* Cliccare su "Manage Repository"
* Dalla pagina di gestione del repository cliccare su "Builds"
* Schiacciare su "Link to GitHub"
* Specificare l'account e il repository alla quale si vuole collegare il repository di Docker.

## Aggiungere un container
Per aggiungere un container utilizzare il comando
```
docker create <image>:<tag>
```
Docker cercherà il container sulla macchina locale. Se non lo trova proverà a scaricare il container da Docker Hub.

## Vedere container sulla macchina
Visualizza solo i contenitori in esecuzione
```
docker ps
```
Visualizza tutti i contenitori (in esecuzione e fermati)
```
docker ps -a
```
Per visualizzare i contenitori con il relativo spazio utilizzato utilizzare il flag *-s*
```
docker ps -s
```
Questo flag aggiunge 2 colonne:
* la colonna "size" indica la quantità di disco che è utilizzata nel writable layer di ogni container
* la colonna "virtual size" indica lo spazio totale utilizzato dall'immagine in read-only per il contenitore e il writable layer

Il *writable layer* è un livello creato da Docker quando creiamo un nuovo container. In questo caso viene aggiunto questo livello sopra lo stack dei livelli presenti sopra l'immagine di base. Il nuovo livello conterrà tutti i cambiamenti effettuati al container in esecuzione, come la creazione di nuovi file o la modifica / cancellazione di file esistenti.

Per vedere le immagini presenti sulla macchina:
```
docker images -a
```


# Docker files
## Docker-compose.yml
E' un file di tipo YAML dove vengono definiti i servizi, la configurazione della rete e la configurazione del disco fisso. In questo file specificheremo la struttura di un file di versione 3.x. La versione è specificata all'inizio del file con il tag *version*. Prendiamo come esempio la realizzazione di un contenitore per un server mysql-apache con all'interno Laravel installato:
```
version: '3'

networks:
  laravel:

services:
  apache:
    ...
    networks:
      - laravel

  mysql:
    ...
    networks:
      - laravel
  php:
    ...
    networks:
      - laravel
```
Il tag *services* contiene la configurazione che viene applicata quanodo un container viene eseguito.

Ora, prendiamo per esempio la configurazione del servizio apache all'interno del Laravel
```
apache:
    image: httpd:latest
    container_name: apache
    ports: 
      - "8081:80"
    volumes:
      - ./src:/var/www/html
    depends_on: 
      - php
      - mysql
    networks:
      - laravel
```
Sotto i servizi troviamo il nome dei vari servizi. All'interno di un servizio troviamo le sue configurazioni.
Il tag *image* indica quale immagine deve essere eseguita del servizio. L'immagine si trova su DockerHub e viene specificata nel formato <image>:<tag>
Il tag *container_name* serve per sovrascrivere il nome del container che docker assegna al contenitore (nel caso di apache il container_name di default è httpd)
*Ports* indica la configurazione delle porte tra il nostro pc e il contenitore. 
Le porte hanno il formato
```
<computer_port>:<container_port>
```
Il comando ports effettua il binding tra le due porte.
Il comando *volumes* effettua un simlink tra una cartella sul nostro dispositivo e una cartella all'interno del contenitore.
*IMPORTANTE:* il comando volumes effettua un link tra i file presenti sulla macchina e il container. I permessi dei file vengono presi da quelli in locale.
Il comando *depends_on* specifica quali servizi devono essere eseguiti prima di eseguire il nostro servizio.

Prendiamo ora per esempio la configurazione del servizio mysql:
```
mysql:
    image: mysql:5.7.30
    restart: unless-stopped
    tty: true
    ports:
      - "3307:3306"
    volumes:
      -./mysql:/var/lib/mysql # necessario se il restart è uguale ad unless-stopped
    environment:
      - MYSQL_DATABASE: db_name
      - MYSQL_user: mysql_user
      - MYSQL_PASSWORD: secret
      - MYSQL_ROOT_PASSWORD: secret
    networks:
      - laravel
```
Di default un container non si riavvia mai. Se si necessita un riavvio si può specificare nel comando *restart* (si può anche specificare unless-stopped, il quale effettuerà un restart riavvio se il container viene arrestato).
Per abilitare l'interfaccia di shell si può abilitare il flag *tty*, settandolo a true.
All'interno del container è possibilie settare delle variabili d'ambiente all'interno del comando *environment*.

Nel caso non si volesse utilizzare per forza un'immagine dal DockerHub, si può utilizzare al posto del comando image il comando *build*. Prendiamo per esempio questa configurazione di un servizio PHP, dove vogliamo installare, oltre all'immagine, anche le varie estensioni PHP necessario per un contenitore Laravel.
```
php:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./src:/usr/local/apache2/htdocs # httpd container document root
    ports:
      - "9000:9000"
    networks:
      - laravel
```
all'interno del comando *build* troviamo i due comandi. *context* indica la cartella dove cercare il dockerfile. il comando *dockerfile* specifica il nome del dockerfile. Il dockerfile è un file contente istruzioni per effettuare il build dell'immagine. Il dockerfile viene utilizzato durante il build.
In questo caso il nostro dockerfile risulterà come riportato sotto:
```
FROM php:7.3.18-zts-alpine3.11

RUN docker-php-ext-install pdo pdo_mysql
```
Con il comando *FROM* viene specificata l'immagine da recuperare e con il comando *RUN* vengono installate le estensioni di php necessarie (in questo caso saranno pdo e pdo_mysql).

# Comandi docker
## Eseguire il container
Per eseguire il container lanciare i comandi 
```
docker-compose build
docker-compose up -d
```
Con il comando build viene create l'immagine dal Dockerfile e un "context".
il flag *-d* nel comando up indica detached-mode, ovvero il container viene eseguito in background.

Dopo il primo build è sufficiente lanciare il comando *docker-compose up* per ricreare l'ambiente.

Per fermare un container usare il comando
```
docker stop <container_id>
```
Per fermare tutti i container di docker in un singolo comando usare 
```
docker stop $(docker ps -a -q)
```

## Logs
Per vedere i log del container in esecuzione eseguire:
```
docker logs -f --details <container_name>
```
Nel caso si lancia il comando su un container con php e mysql mostrerà i corrispettivi error logs dei due.
Per effettuare un grep di una keyword nei log
```
docker logs -f <container_name> 2>&1 | grep <keyword>
```


## Eliminare un container
Per eliminare un container lanciare il comando 
```
docker rm <container>
```

## Altri comandi
Se si vuole lanciare un comando all'interno di un contenitore già in esecuzione si può utilizzare il comando *docker exec*
```
docker exec <container> <command>
```

Per stampare le informazioni su un container
```
docker inspect <container>
```

