## Create project
Per creare un nuovo progetto con heroku lanciare il comando
```
heroku create <project_name>
```
Si può impostare la regione con il flag
```
heroku create --region eu <project_name>
```
Per scegliere una buildpath specifica usare il flag
```
heroku create --buildpack <git_repo_link> <project_name>
```

## Heroku Aptfile
One is the experimental heroku-apt-buildpack https://github.com/heroku/heroku-buildpack-apt. You can use this by including any APT package in an Aptfile in your application. The buildpack will then install these packages on the dyno when you deploy your application.

## Heroku dyno operations
### Accedere ai log
```
heroku logs
heroku logs --tail
```
### Accedere alla cli
```
heroku run bash
heroku run bash -a <dyno_name>
```
