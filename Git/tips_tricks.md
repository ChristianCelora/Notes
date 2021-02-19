# Git - Tips $ tricks

## git reset

Git reset viene utilizzato per spostare l'HEAD al commit specifico.

### Hard reset vs Soft Reset

Il flag --hard è utilizzato per resttare i file dell'indice (o della staging area) e della working directory. 
Al contrario il flag --soft non modifica la workgin directory e l'indice.
In questo caso lascia tutte le modifiche ai file come "da committare". 
Usare git status per vedere le varie modifiche.


### Utilizzo
```
$ git reset --hard HEAD		(riotrno all'HEAD)

$ git reset --hard HEAD^	(ritorno al commit prima dell'HEAD)
$ git reset --hard HEAD~1	(identico a sopra)

$ git reset --hard HEAD~2	(ritorno a 2 commit prima dell'HEAD)
```

### Esempio

```
$ git log --oneline --graph

  * 802a2ab (HEAD -> feature, origin/feature) feature commit
  * 7a9ad7f (origin/master, master) version 2 commit
  * 98a14be Version 2 commit
  * 53a7dcf Version 1.0 commit
  * 0a9e448 added files
  * bd6903f first commit

$ git reset --soft HEAD^             (oppure HEAD~1)

(in questo modo la staging area viene riempita con le modifiche fatte nei commit tra 7a9ad7f e 802a2ab)

$ git status

  On branch feature
  Your branch is behind 'origin/feature' by 1 commit, and can be fast-forwarded.
    (use "git pull" to update your local branch)

  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

          new file:   file-feature

```

### Muoversi in avanti

Per muoversi in avanti nella history di git usare il comando 

```
git reset 'HEAD@{1}'		(sposta l'HEAD al commit successivo)
```

Usare questa operazione nel caso si volesse ritornare ad un commit nel futuro dopo aver usato un comando 
di git reset su un commit precedente.

Per sicurezza consultare sempre i log di git dei reference updates, 
in cui ci sono tutti i commit in cui è stata spostata la HEAD (checkout,reset,commit,merge):

```
git reflog
git reflog show <branch>	(mostra solo i commit della branch specifica)
```



