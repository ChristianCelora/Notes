## Pre-requisiti (??):
```
sudo apt install mysql-server
```

## Comandi:
### Crea il docker-compose.yml
```
./vendor/bin/ece-docker build:compose --mode="developer" 
```

## Configurare docker-compose.yml
### Db
- configurare il container 'db' con le variabili d'ambiente del mysql in locale
- aggiungere nel container 'db'
    ```
    ports:
      - '3307:3306'
    ```
- effettuare un dump dei dati (sulla propria macchina)
    ```
    magento-cloud db:dump
    ```
    mettere il risultato del dump nella cartella .docker/mysql/docker-entrypoint-initdb.d
- lanciare manualmente il dump (dovrebbe farlo automaticamente):
    ```
    docker exec -it backend_db_1 bash -c 'mysql -u root -p magento_db < /docker-entrypoint-initdb.d/zgzvw2kr4mr5m--   backend-cdqiuqi--mysql--main--dump.sql'
    ```
- testare la connessione al db con il comando docker (e verificare la presenza delle tabelle)
    ```
    docker exec -it backend_db_1 mysql -u root -p
    ```
- ricordarsi di aggiornare il file in app/etc/env.php dove si mettono i dati della connessione x il db. IMPORTANTE: l'hostname deve essere uguale al hostname specificato sotto il servizio "db" nel file docker-compose.yml
### Web
- configurare il container 'web' aggiungendo:
    ```
    ports:
      - '8082:80' 
    ```

### Redis (dopo aver lanciato il container)
- prendere l'ip gateway del docker di redis:
    ```
    docker ps # per prendere il id del container di docker
    docker inspect <container_id_redis>  # l'id è sotto NetworkSettings > Networks > "nome enviornment" > Gateway
    ```
- sostiuire nel file env.php nella parte di cache:
    ```
    'cache' =>
      array (
        'frontend' =>
        array (
          'default' =>
          array (
            'backend' => 'Cm_Cache_Backend_Redis',
            'backend_options' =>
            array (
              'server' => 'IP_GATEWAY_REDIS',
              'port' => 6379,
              'database' => 1,
            ),
          ),
          'page_cache' =>
          array (
            'backend' => 'Cm_Cache_Backend_Redis',
            'backend_options' =>
            array (
              'server' => 'IP_GATEWAY_REDIS',
              'port' => 6379,
              'database' => 2,
            ),
          ),
        ),
      )
    ```
- sostituire nel file env.php nella parte di redis:
    ```
    array (
    'save' => 'redis',
    'redis' =>
    array (
      'host' => 'IP_GATEWAY_REDIS',
      'port' => 6379,
      'database' => 0,
      'disable_locking' => 1,
    ),
  ),
    ```
- aggiungere nel file docker-compose.yml:
    ```
    redis:
    hostname: redis.magento2.docker
    image: 'redis:5.0'
    volumes:
      - 'mymagento-magento-sync:/app:delegated'
    ports:
      - 6379
    healthcheck:
      test: 'redis-cli ping || exit 1'
      interval: 30s
      timeout: 30s
      retries: 3
    networks:
      magento:
        aliases:
          - redis.magento2.docker
      magento-build:
        aliases:
          - redis.magento2.docker
    ```
nel caso non funzionasse provare con https://www.magemodule.com/all-things-magento/magento-2-tutorials/redis-for-local-magento-2-development/

### test connessione redis
```
docker-compose run --rm redis redis-cli -h redis.magento2.docker
```

## Elasticsearch: 
Guida -> https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
impostare sulla propria macchina (non all'interno del container):
```
sudo sysctl -w vm.max_map_count=262144
```

## Lanciare il container
```
docker-compose build (solo la prima volta)
docker-compose up -d
```

copiare il file app/etc/env.php in locale e modificare le opzioni sotto 'db' con le configurazioni del db in locale.


## Lanciare composer install 
```
docker run -it  -v $(pwd):/app/:delegated -v ~/.composer/:/root/.composer/:delegated magento/magento-cloud-docker-php:7.3-cli-1.1 bash -c "composer install&&chown www. /app/"
```

## Accedere alla pagina
il link è composto da:
```
127.0.0.1:port/web_hostname
```
dove: 
- port è la porta mappata sotto il container "web"
- hostname è il nome dell'host (hostname) sotto il container "web"


## Deploy modifiche
In breve ci sono diversi container. ciascun container ha uno scopo specifico per maggiorin info(https://devdocs.magento.com/cloud/docker/docker-containers-cli.html):
Gli effettivi comandi che vengono lanciati (questi sono alias) sono visibili nella sezione "hooks" sotto il file .magento.app.yaml

prima di lanciare i comandi sotto fare un copia del file app/etc/env.php (c'è qualche cosa che sovrascrive il file. Nel caso viene sovrascritto riportarlo alla versione originale)
### build  
```
// build environment
docker-compose run --rm build cloud-build
```
### deploy
```
// deploy environment
docker-compose run --rm deploy cloud-deploy
// run post-deploy hooks
docker-compose run --rm deploy cloud-post-deploy
```
## Lanicare comandi magento
usare il container deploy per lanciare comandi
```
docker-compose run --rm deploy magento-command setup:static-content:deploy
docker-compose run --rm deploy magento-command setup:upgrade
docker-compose run --rm deploy magento-command setup:store-config:set --base-url="http://localhost:8082"
```
### guide
Guida:
https://devdocs.magento.com/cloud/docker/docker-containers-cli.html

Quick reference:
https://devdocs.magento.com/cloud/docker/docker-quick-reference.html

## Altri comandi
### eliminare interfacce ip
```
ifconfig
ip link delete <id_interfaccia>
```
Se si elimina l'interfaccia di un network di docker quando va ricreato il progetto in docker-compose usare:
```
docker-compose up --force-recreate
```
### regex vscode x trovare gli as nei referenceBlock
```
(?<=referenceBlock)(.*)(?=as)
```

## Problemi riscontrati
### adabra
Per ora il plugin di adabra da dei problemi con gli id impostati nel file vendor/adabra/adabra-magento2/etc/adminhtml/system.xml. Per ora basta rimuovere i dash '-' dagli id

## logs
Per controllare i log basta lanciare il comando
```
docker logs -f <container_name>
// i log di docker sono messi sullo stderr. ridirezioniamo l'output sul stdout in modo da poter effettuare il grep
docker logs -f <container_name> 2&1> | grep "ciao" 
```

## webapi.xml
nelle webapi xml è importante che il tag <service> sia prima del tag <resources>
