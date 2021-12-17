## Pre-requisiti 
Installare docker e docker-compose come nella guida precedente.

## Comandi:
### Crea il docker-compose.yml
```
./vendor/bin/ece-docker build:compose --mode="developer" 
```

## Configurare docker-compose.yml
### Db
- aggiungere nel container 'db'
    ```
    ports:
      - '3307:3306'
    ```
- effettuare un dump dei dati (sulla propria macchina)
    ```
    magento-cloud db:dump
    ```
    mettere il risultato del dump nella cartella .docker/mysql/docker-entrypoint-initdb.d (se non funziona creare una nuova cartella e aggiungere il mapping dentro volumes nel docker-compose)
- lanciare manualmente il dump (dovrebbe farlo automaticamente):
    ```
    // entrare prima dentor il container e creare il database
    docker exec -it backend_db_1 bash -c 'mysql -u root -p magento_db < /docker-entrypoint-initdb.d/zgzvw2kr4mr5m--backend-cdqiuqi--mysql--main--dump.sql'
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
  In questo modo mappiamo la posta 8082 del nostro pc con la porta 80 del container. Assicurarsi che la porta 8082 sia libera. Nel caso non lo fosse usarne un'altra.
### Redis

- sostiuire nel file env.php nella parte di cache (HOSTNAME_REDIS è definito nel file docker-compose):
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
              'server' => 'HOSTNAME_REDIS',
              'port' => 6379,
              'database' => 1,
            ),
          ),
          'page_cache' =>
          array (
            'backend' => 'Cm_Cache_Backend_Redis',
            'backend_options' =>
            array (
              'server' => 'HOSTNAME_REDIS',
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
      'host' => 'HOSTNAME_REDIS',
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
Questo comando va lanciato dopo ogni riavvio. 
## Lanciare il container
```
docker-compose build (solo la prima volta)
docker-compose up -d
```

copiare il file app/etc/env.php in locale e modificare le opzioni sotto 'db' con le configurazioni del db in locale.


## Lanciare composer install 
```
docker run -it -v $(pwd):/app/:delegated -v ~/.composer/:/root/.composer/:delegated magento/magento-cloud-docker-php:7.3-cli-1.1 bash -c "composer install&&chown www. /app/"
```

## Accedere alla pagina
il link è composto da:
```
127.0.0.1:port/web_hostname
```
dove: 
- port è la porta mappata sotto il container "web"


## Deploy modifiche
In breve ci sono diversi container. ciascun container ha uno scopo specifico per maggiorin info(https://devdocs.magento.com/cloud/docker/docker-containers-cli.html):
Gli effettivi comandi che vengono lanciati (questi sono alias) sono visibili nella sezione "hooks" sotto il file .magento.app.yaml

### build  
prima di lanciare i comandi sotto fare un copia del file app/etc/env.php (il comando cloud-deploy sovrascrive il file nel caso di modifiche alle configurazioni). 
In alcuni casi è necessare anche riportare l'ownership del file indietro con:
```
sudo chown user:user app/etc/*
```
Build environment:
```
// build environment (avg. time: 2m 20s)
docker-compose run --rm build cloud-build
```
### deploy
```
// deploy environment (avg. time: 2m)
docker-compose run --rm deploy cloud-deploy
// run post-deploy hooks (avg. time: 13s)
docker-compose run --rm deploy cloud-post-deploy
```
## Lanicare comandi magento
usare il container deploy per lanciare comandi. Alcuni esempi:
```
docker-compose run --rm deploy magento-command setup:static-content:deploy
docker-compose run --rm deploy magento-command setup:upgrade
docker-compose run --rm deploy magento-command setup:store-config:set --base-url="http://localhost:8082/"
docker-compose run --rm deploy magento-command dev:tests:run <test>
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
Consiglio: usare sempre il comando docker-compose down per cancellare correttamente i network creati da docker e le loro interfacce relative
### regex vscode x trovare gli as nei referenceBlock
```
(?<=referenceBlock)(.*)(?=as)
```

### PHPUnit test
Per aggiungere la possibilità di eseguire i test (unit e integration) nel docker:
```
composer require --dev "phpunit/phpunit:^6.5.0" // anche una versione più bassa va bene
composer require --dev allure-framework/allure-phpunit:~1.2.0
```
aggiungere al file docker-compose.yml
```rabbitmq:
    hostname: rabbitmq.magento2.docker
    image: 'rabbitmq'
    ports:
      - '5672:5672'
    environment:
      RABBITMQ_ERLANG_COOKIE: RABBIT_COOKIE
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    networks:
      magento:
        aliases:
          - rabbitmq.magento2.docker
```
Aggiornare il file dev/tests/integration/etc/install-config-mysql.php (se è presente solo il file install-config-mysql.php.dist copiarlo e creare il file install-config-mysql.php).
```
return [
    'db-host' => 'db.magento2.docker',
    'db-user' => 'db_user',
    'db-password' => 'db_psw',
    'db-name' => 'db_name',
    'db-prefix' => '',
    'backend-frontname' => 'backend',
    'admin-user' => \Magento\TestFramework\Bootstrap::ADMIN_NAME,
    'admin-password' => \Magento\TestFramework\Bootstrap::ADMIN_PASSWORD,
    'admin-email' => \Magento\TestFramework\Bootstrap::ADMIN_EMAIL,
    'admin-firstname' => \Magento\TestFramework\Bootstrap::ADMIN_FIRSTNAME,
    'admin-lastname' => \Magento\TestFramework\Bootstrap::ADMIN_LASTNAME,
    'amqp-host' => 'rabbitmq.magento2.docker',
    'amqp-port' => '5672',
    'amqp-user' => 'RABBITMQ_DEFAULT_USER',
    'amqp-password' => 'RABBITMQ_DEFAULT_PASS',
];
```
Consiglio: per gli integration test settare nel file dev/tests/integration/phpunit.xml
```
<const name="TESTS_CLEANUP" value="disabled"/> // non ricrea l'installazione di magento da capo e non ripulisce il db al termine
```
Per lanciare i test
```
docker-compose run --rm deploy magento-command dev:tests:run <test>
```

## Problemi riscontrati
### adabra
Per ora il plugin di adabra da dei problemi con gli id impostati nel file vendor/adabra/adabra-magento2/etc/adminhtml/system.xml. Per ora basta rimuovere i dash '-' dagli id

### logs
Per controllare i log basta lanciare il comando
```
docker logs -f <container_name>
// i log di docker sono messi sullo stderr. ridirezioniamo l'output sul stdout in modo da poter effettuare il grep
docker logs -f <container_name> 2&1> | grep "ciao" 
```

### db
Connetersi al db
```
docker exec -it backend_db_1 mysql -u develop -p
```

### sistemare links tabella 
```
update core_config_data set value = 'https://localhost:8082' where path like '%/secure/base_url';
update core_config_data set value = 'http://localhost:8082' where path like '%/unsecure/base_url';

docker-compose run --rm deploy magento-command c:c
```

### webapi.xml
nelle webapi xml è importante che il tag <service> sia prima del tag <resources>

### DNS_PROBE_FINISHED_NXDOMAIN
In questo caso non riesce a trovare il domain name indicato nel file .docker/config.php, sotto la voce 'MAGENTO_CLOUD_ROUTES'. In questo caso aggiungere il domain all'interno del file /etc/hosts:
```
127.0.0.1 domain.name
```

### Pages on https
Il docker non è settato con il ssl. In questo caso il sito prova a connettersi di default con l'https (sopratutto nella parte di admin):
```
docker-compose run --rm deploy magento-command setup:store-config:set --base-url="http://<dominio>/" # <domnio> = <url>:<porta> (solo per il base-url)
docker-compose run --rm deploy magento-command setup:store-config:set --use-secure=0
docker-compose run --rm deploy magento-command setup:store-config:set --use-secure-admin=1 # Va messo a 1 per forza (altrimenti va in ERR_TOO_MANY_REDIRECT)
docker-compose run --rm deploy magento-command cache:clean
```

### ricreare un container cancellando anche i volumi e ricreare l'immagine:
```
docker-compose up -d --force-recreate --build -V <container>
```
dove <container> è il nome messo nel doker-compose (per esempio db)


### Grafica tema "rotta"
Se non si riesce a vedere la grafica corretta lanciare il comando di generazione del tema di magento, aggiungendo il flag -f per forzare la ricreazione e specificare il tema da creare e la lingua:

```
docker-compose run --rm deploy magento-command setup:static-content:deploy -f it_IT
```

Nel nostro caso it_IT è la lingua del tema associata alla store_view che ci interessa sistemare.

### Nuove credenziali db non riconosciute nel cloud-deploy (v. 2.4)
Nel caso si modificano le credenziali del container con all'interno del db, non è sufficiente aggiornare il fiel env.php a volte.

Se durante il comando 
```
docker-compose run --rm deploy cloud-deploy
```
non si riesce a connettere al db, bisogna modificare il file `.magento.env.yaml` con le nuove credenziali nella parte DATABASE_CONFIGURATION.

Per un esempio di come strutturare le informazioni consultare il file `.magento.env.yaml.dist`
    
 ### Admin Https ERR_SSL_PROTOCOL_ERROR
 Nel caso si vuole utilizzare l'admin senza che venga controllato l'HTTPS, bisogna lanciare il comando:
 ```
 docker-compose run --rm deploy magento-command setup:store-config:set --base-url-secure="http://<dominio>/"
 ```
 
 Non inserire la porta dentro l'url altrimenti andrà in errore ERR_SSL_PROTOCOL_ERROR.
