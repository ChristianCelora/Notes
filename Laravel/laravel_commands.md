# Comandi Laravel (7.x)

## compilare scss - js 
Per compilare i vari scss e js (sotto la cartella resources/js resources/sass)  lanciare il comando:
``` 
npm run dev
``` 
Per non lanciare il comando ogni volta si può lanciare il comando che compilerà i file ad ogni modifica:
``` 
npm run watch
``` 

## Installare Bootstrap in laravel 7
Installare Laravel UI:
``` 
composer require laravel/ui
``` 

Aggiungere Bootstrap come dipendenza (aggiunge anche JQuery):
``` 
php artisan ui bootstrap 
php artisan ui bootstrap --auth  # Generate login / registration scaffolding...
``` 
Con questo comando crea i file resource/js/bootstrap.js e resource/sass/_variables.scss

Installa dipendenze con:
``` 
npm install 
``` 
Inserire nel file app.scss il link per usare bootstrap (come da esempio sotto).

### Esempio app.scss
Esempio file app.scss per importare bootstrap e font da google
``` 
// Fonts
@import url('https://fonts.googleapis.com/css?family=Nunito');
// Variables
@import 'variables';
// Bootstrap
@import '~bootstrap/scss/bootstrap';
``` 

## Installare Font Awesome
``` 
npm install font-awesome --save             # font awesome
npm install @fortawesome/fontawesome-free   # font awesome 5 
``` 

Inserire nel file app.scss:
``` 
@import '~@fortawesome/fontawesome-free/scss/fontawesome';
@import '~@fortawesome/fontawesome-free/scss/regular';
@import '~@fortawesome/fontawesome-free/scss/solid';
@import '~@fortawesome/fontawesome-free/scss/brands';
``` 
