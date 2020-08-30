# Nuovo progetto Laravel
Installare laravel linux. Ci sono diversi modi per creare un nuovo progetto in laravel.

## Laravel Installer
Scaricare l'installer tramite Composer:
``` 
composer global require laravel/installer
``` 

Includere composer all'interno della variabile *$PATH* 

Creare un nuovo progetto con il comando:
``` 
laravel new project_name
``` 

## Composer
Laravel si puiò installare anche tramite Composer:
``` 
composer create-project --prefer-dist laravel/laravel project_name
``` 

# Permessi laravel
Dopo aver creato un progetto laravel richiede di impostare i permessi ai file e alle cartelle generate in modo che il web-server (www-data) possa accedervi:

* cambiare ownership file:
    ``` 
    sudo chown -R www-data:www-data /path/to/root
    ``` 
* cambiare permessi ai file (www-data deve poter leggere-scrivere):
    ``` 
    sudo find /path/to/root -type f -exec chmod 644 {} \;
    ``` 
* cambiare permessi alle directory (www-data deve poter leggere-scrivere-eseguire):
    sudo find /path/to/root -type d -exec chmod 755 {} \;
* dare i permessi per le cartelle di storage e bootstrap/cache per scrivere ed eseguire file:
    ``` 
    sudo chgrp -R www-data storage bootstrap/cache
    sudo chmod -R ug+rwx storage bootstrap/cache
    ``` 

# Configurare Apache

## Configurare virtualhost
Esempio configurazione virtualhost:
``` 
<Directory /var/www/ghostpdf/GhostPDF/public>
        AllowOverride All
        Options None
        Order allow,deny
        allow from all

        Options +FollowSymLinks -Indexes
</Directory>
Alias /ghostpdf /var/www/ghostpdf/GhostPDF/public
``` 