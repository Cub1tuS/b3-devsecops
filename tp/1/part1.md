# Part 1 : DevOps init

## Index

- [Part 1 : DevOps init](#part-1--devops-init)
  - [Index](#index)
  - [1. Create code repo](#1-create-code-repo)
  - [2. Grab the code](#2-grab-the-code)
  - [3. Gitlab Runner](#3-gitlab-runner)
    - [A. Intro](#a-intro)
    - [B. Install and conf](#b-install-and-conf)
  - [4. First pipeline](#4-first-pipeline)
  - [Go next](#go-next)

## 1. Create code repo

➜ **Go sur [Gitlab](https://gitlab.com)**

- créer un compte et connectez-vous si c'est pas déjà fait
- créer un nouveau dépôt git vide et **public**
- **appelez le dépôt `broken_webapp`**

➜ **Get the repo**

- clonez le dépôt sur votre machine, votre PC, votre OS (pas une VM ou koa)
- utilisez la ligne de commande `git` depuis votre PC
- poussez un ptit `README.md` tout pourri de votre choix (`add`, `commit`, `push`)

> Vérifiez que le fichier `README.md` est visible sur la WebUI avant de continuer.

🌞 **Balancez l'URL de ce dépôt `broken_webapp` dans le compte-rendu Markdown**

> [Lien Repo](https://gitlab.com/Cub1tuS/broken_webapp/)
## 2. Grab the code

🌞 **Récupérez le code**

```bash
dorian@Air-de-Dorian broken_webapp % git clone https://gitlab.com/it4lik/b3-devsecops-2024.git
Cloning into 'b3-devsecops-2024'...
remote: Enumerating objects: 141, done.
remote: Counting objects: 100% (78/78), done.
remote: Compressing objects: 100% (76/76), done.
remote: Total 141 (delta 30), reused 0 (delta 0), pack-reused 63 (from 1)
Receiving objects: 100% (141/141), 2.58 MiB | 5.69 MiB/s, done.
Resolving deltas: 100% (41/41), done.
dorian@Air-de-Dorian broken_webapp % ls
b3-devsecops-2024	README.md
dorian@Air-de-Dorian broken_webapp % mv b3-devsecops-2024 ../.
dorian@Air-de-Dorian broken_webapp % cp -r ../b3-devsecops-2024/tp/1/app/* .
dorian@Air-de-Dorian broken_webapp % cp -r ../b3-devsecops-2024/tp/1/app/.* .

```

🌞 **Lancez le code, histoire de voir ce qu'il fait**

```bash 
docker compose up
curl localhost:8080
dorian@Air-de-Dorian app % curl localhost:8080
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Home</title>
  </head>
  <body>
    
      <h1>Welcome to My Flask App</h1>
      <p><a href="/login">Log In</a></p>
    
  </body>
</html>
```
> J'ai modifié le port par défaut car déjà prit de mon côté

🌞 **Connectez-vous sur la WebUI**

```bash
dorian@Air-de-Dorian db % cat seed.sql 
USE broken_webapp;
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(100) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL
);

INSERT INTO users (username, password)
VALUES ('alice', 'alice'), ('bob', 'bob'), ('meo', 'meo');
```

```bash
dorian@Air-de-Dorian db % curl -X POST -d "username=alice&password=alice" localhost:8080/login
<!doctype html>
<html lang=en>
<title>Redirecting...</title>
<h1>Redirecting...</h1>
<p>You should be redirected automatically to the target URL: <a href="/dashboard">/dashboard</a>. If not, click the link.
dorian@Air-de-Dorian db % curl -X POST -d "username=alice&password=alice" localhost:8080/login
<!doctype html>
<html lang=en>
<title>Redirecting...</title>
<h1>Redirecting...</h1>
<p>You should be redirected automatically to the target URL: <a href="/dashboard">/dashboard</a>. If not, click the link.
```

🌞 **`add`, `commit`, `push`**

```bash
dorian@Air-de-Dorian broken_webapp % git add Dockerfile docker-compose.yml src/
dorian@Air-de-Dorian broken_webapp % git commit -m "push app" 
[main 9a5a778] push app
 6 files changed, 177 insertions(+)
 create mode 100644 Dockerfile
 create mode 100644 docker-compose.yml
 create mode 100644 src/app.py
 create mode 100644 src/templates/dashboard.html
 create mode 100644 src/templates/home.html
 create mode 100644 src/templates/login.html

dorian@Air-de-Dorian broken_webapp % git push
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 8 threads
Compressing objects: 100% (10/10), done.
Writing objects: 100% (10/10), 2.74 KiB | 2.74 MiB/s, done.
Total 10 (delta 0), reused 0 (delta 0), pack-reused 0
To gitlab.com:Cub1tuS/broken_webapp.git
   c6f36e0..9a5a778  main -> main
```

➜ **A la racine du dépôt, vous devriez donc avoir :**

```bash
dorian@Air-de-Dorian broken_webapp % git add .env db/ requirements.txt 
dorian@Air-de-Dorian broken_webapp % git commit -m "push app second time"
[main fd07771] push app second time
 3 files changed, 17 insertions(+)
 create mode 100644 .env
 create mode 100644 db/seed.sql
 create mode 100644 requirements.txt
dorian@Air-de-Dorian broken_webapp % git push
```

```bash
dorian@Air-de-Dorian broken_webapp % ls -a
.			.git			Dockerfile		src
..			db			README.md
.env			docker-compose.yml	requirements.txt
```

## 3. Gitlab Runner

### A. Intro

Quand on fait de la CI/CD avec GitLab, on a besoin d'utiliser des *Runner*. Un *Runner* est une machine qui va exécuter les tests automatisés.

Concrètement :

- on fait un `git push` pour envoyer du code sur GitLab
- à la réception du `push`, Gitlab va contacter un *Runner* associé au dépôt
- il va demander au *Runner* d'exécuter les tests listés dans le fichier `.gitlab-ci.yml`

Avant de pouvoir lancer nos premiers tests automatisés, il faut donc qu'on installe et configure un *Runner GitLab*.

### B. Install and conf

➜ **Allumez une VM Rocky**

- elle doit avoir un accès internet, et vous l'administrez en SSH

🌞 **Installer Docker**

```
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```

🌞 **Installer un Runner Gitlab**

```
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
```

```
sudo dnf install gitlab-runner
```

```
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "$RUNNER_TOKEN" \
  --executor "docker" \
  --docker-image debian:latest \
  --description "docker-runner"
```

```
systemctl enable --now gitlab-runner.service
```

- [suivez la doc officielle](https://docs.gitlab.com/runner/install/)
- je vous recommande de l'installer directement sur la machine hôte (pas de l'installer avec Docker)
- `start` et `enable` le service `gitlab-runner.service`

⚠️⚠️⚠️ **Avant de continuer, assurez-vous que vous voyez votre *Runner* dans la WebUI de GitLab**

- depuis la page principale de votre repo `broken_webapp`
- dans le menu latéral, allez dans la section `Settings > CI/CD > Runners`
- vous devriez voir votre *Runner* qui s'est enregistré

## 4. First pipeline

Allez, notre première *pipeline*. C'est un mot générique pour désigner un truc à exécuter automatiquement dans un dépôt git.

On va donc créer notre première *pipeline* avec GitLab : **en créant un fichier `gitlab-ci.yml` à la racine du dépôt git.**

Une *pipeline* Gitlab est composée de plusieurs *stages*. Chaque *stage* consiste en une série de *job*. Chaque *job* est une commande ou une suite de commande à exécuter dans un environnement précis.

> Vous allez voir, ça va prendre du sens au fur et à mesure du TP, on va poncer ces concepts !

➜ **Ajoutez un fichier `gitlab-ci.yml` à la racine de votre dépôt `broken_webapp`**

- il doit contenir ça :

```yml
image: debian:latest

stages:
  - meow

meow-job:
  stage: meow
  before_script:
    - apt-get update -qq
    - apt-get install -y cowsay
  script:
    - /usr/games/cowsay "Meoooow"
```

🌞 **`add`, `commit`, `push`**

```
dorian@Air-de-Dorian broken_webapp % nano gitlab-ci.yml
dorian@Air-de-Dorian broken_webapp % git add gitlab-ci.yml 
dorian@Air-de-Dorian broken_webapp % git commit -m  "add gitlab"
[main 31f7a9e] add gitlab
 1 file changed, 13 insertions(+)
 create mode 100644 gitlab-ci.yml
dorian@Air-de-Dorian broken_webapp % git push
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 403 bytes | 403.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
To gitlab.com:Cub1tuS/broken_webapp.git
   fd07771..31f7a9e  main -> main
```

- pour ajouter le fichier `gitlab-ci.yml` au dépôt
- un message de commit psa tout pourri encore svp :ddd

➜ **RDV sur la WebUI de GitLab**

- allez sur la page principale de votre repo `broken_webapp`, et vérifiez que vous voyez bien le fichier `.gitlab-ci.yml`
- toujours depuis la page de votre repo `broken_webapp`, depuis le menu latéral, allez dans la section `Build`
- vous devriez voir votre *job* en cours d'exécution, vous pouvez avoir l'output du test

> Le *job* a été exécuté sur votre machine *Gitlab Runner*, qui a lancé un conteneur éphémère (avec Docker) pour exécuter le code demandé. Vous pouvez consulter l'output console du *job* depuis la WebUI, et vous pourrez voir que la première étape avant d'exécuter notre *job* c'est de cloner le dépôt git à l'intérieur du conteneur éphémère. Beh ui, comme ça il peut faire des tests dessus !

⚠️⚠️⚠️ **Ne continuez pas tant que vous n'avez pas vu votre *job* s'exécuter et vous devriez pouvoir voir l'output du test. Donc tu continues pas tant que t'as pas vu une vache miauler**

```
 _________
< Meoooow >
 ---------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

> JOB REUSSI !!
## Go next

👉 [**Gogogo partie 2 : Test then Build**](./part2.md)
