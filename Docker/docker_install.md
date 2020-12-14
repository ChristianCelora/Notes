# Installare Docker
Installare docker da repository

## Rimuovere precendenti versioni di Docker
La prima cosa da fare è assicurarsi che non ci siano altre versioni di Docker installate sul sistema. 
Per rimuovere eventuali versioni installate utilizzare il comando:
``` 
sudo apt remove docker docker-engine docker.io containerd runc
```

## Preparare la macchina
Aggiornare la cache APT
``` 
sudo apt update
``` 

E installare i vari pacchetti necessari a Docker
``` 
sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
gnupg-agent \
software-properties-common
``` 

## Aggiungere la chiave di Docker
Per abilitare il repository di Docker bisogna aggiungere la chiave GPG ufficiale
``` 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
``` 

Verificare che la chiave sia stata correttamente inserita
``` 
sudo apt-key fingerprint 0EBFCD88
``` 
Dovrebbe uscire un output simile
```
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

## Installare docker
Aggiungere il repository di Docker. Docker ha 3 repository:
* stable
* nightly
* test

In questo caso aggiungeremo quello stable:
``` 
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" // se non si conosce la distribuzione
``` 

Una volta aggiunto il repository bisogna effettuare il refresh della cache APT con il solito comando
``` 
sudo apt update
```

Infine, lanciare il comando per installare l'ultima versione di docker
```
sudo apt install docker-ce docker-ce-cli containerd.io
```

source https://linuxhint.com/install_docker_linux_mint/

### Installare docker-compose
Oltre a Docker avremo bisogno anche del docker-compose. Scaricare l'eseguibile con il comando (verrà scaricata l'ultima versione stabile compatibile con il sistema)
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
dare i permessi di esecuzione all'eseguibile
```
sudo chmod +x /usr/local/bin/docker-compose
```

Se il comando docker-compose dovrebbe fallire dopo l'installazione è necessario creare un symbolic link alla cartella /usr/bin
```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

## Use Docker senza privilegi root
Per lanciare i comandi di docker senza i privilegi di root, creare il gruppo "docker" se ancora non esiste
```
sudo groupadd docker
```
Aggiungere l'utenza dalla quale si vuole usare docker al gruppo "docker"
```
sudo usermod -aG docker $USER
```
Effettuare un logout / login per applicare le modifiche oppure lanciare il comando 
```
newgrp docker 
```
per applicare i cambiamenti al gruppo.

### Se ho già lanciato un comando docker con sudo
In tal caso uscirà un errore perchè sono stati cambiati i permessi della cartella .docker 
>WARNING: Error loading config file: /home/user/.docker/config.json -
>stat /home/user/.docker/config.json: permission denied
Per cambiare i permessi della cartella lanciare i comandi
```
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
```

### Test installazione docker
```
docker run hello-world
```
Output desiderato:
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:1a523af650137b8accdaed439c17d684df61ee4d74feac151b5b337bd29e7eec
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
