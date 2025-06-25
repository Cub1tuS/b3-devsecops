# Part 2 : Test then Build

On continue avec **les premiers morceaux d'une chaîne d'automatisation clean**. Une partie dédiée à **la CI**.

> *CI* pour *Continuous Integration* (ou *Intégration Continue*) : le fait d'effectuer des actions automatiques à chaque push sur un dépôt git.

On effectue des **tests automatiquement** sur le code, et on **refuse le push si les tests ne sont pas validés**.

Si les tests sont validés, on déclenche un **build automatique du code**. Ici, on déclenchera le build d'une image Docker.

## Index

- [Part 2 : Test then Build](#part-2--test-then-build)
  - [Index](#index)
  - [1. Linting](#1-linting)
    - [A. Intro](#a-intro)
    - [B. Lint me baby](#b-lint-me-baby)
    - [C. Format me baby](#c-format-me-baby)
  - [2. Building](#2-building)
  - [3. Publishing](#3-publishing)
  - [3. Some manual tests](#3-some-manual-tests)
  - [Next next next](#next-next-next)

## 1. Linting

### A. Intro

Le *linting* c'est le fait de vérifier qu'un code donné est conforme aux bonnes pratiques. Le *linting* est donc spécifique au langage qui est utilisé.

Généralement, les outils de *linting* se contentent de lever des alertes, en nous indiquant en quoi le code analysé n'est pas conforme aux bonnes pratiques.

Ici on parle uniquement de bonnes pratiques d'écriture de code, il n'y aucune notion de sécurité.

Ici, on va appliquer du *linting* sur les fichiers `.py` : le code Python. Pour Python il existe le standard PEP qui répertorie une tonne de bonnes pratiques.

> Vérifier que le code Python est conforme avec PEP c'est l'minimum pour du code Python !
²
### B. Lint me baby

➜ **Choisissez un *linter* Python**

- y'en a plein, faites vos ptites recherches
- vous devriez tomber par exemple sur `flake`, `pylint` ou encore `ruff`
- choisissez-en un !

> `ruff` est tout moderne tout stylé, mais attention il a besoin d'un fichier de conf. `pylint` est un peu une des références, mais plus limité et moins perf que `ruff`. Pour la simplicité je recommande `pylint`.

🌞 **Installez le *linter* sur votre poste**

```
pip3 install pylint
```

```
dorian@Air-de-Dorian src % pylint app.py 
************* Module app
app.py:80:0: C0305: Trailing newlines (trailing-newlines)
app.py:1:0: C0114: Missing module docstring (missing-module-docstring)
app.py:3:0: E0401: Unable to import 'flask' (import-error)
app.py:4:0: E0401: Unable to import 'mysql.connector' (import-error)
app.py:24:0: C0116: Missing function or method docstring (missing-function-docstring)
app.py:25:4: R1705: Unnecessary "else" after "return", remove the "else" and de-indent the code inside it (no-else-return)
app.py:34:0: C0116: Missing function or method docstring (missing-function-docstring)
app.py:52:12: R1705: Unnecessary "else" after "return", remove the "else" and de-indent the code inside it (no-else-return)
app.py:64:0: C0116: Missing function or method docstring (missing-function-docstring)
app.py:70:0: C0116: Missing function or method docstring (missing-function-docstring)

-----------------------------------
Your code has been rated at 5.50/10
```

- essayez de le lancer en local depuis votre terminal sur le code de `broken_webapp`
- pour voir un peu comment s'utilise l'outil !

> Normalement il devrait crier assez fort, mon code est volontairement tout pourri pour avoir un bon cas d'utilisation pour ce TP :)

🌞 **Ajoutez l'exécution automatique du *linter* à chaque `git push`**

```
dorian@Air-de-Dorian broken_webapp % cat .gitlab-ci.yml
image: python:3.9

stages:
  - lint

cache:
  key: pip-cache
  paths:
    - ~/.cache/pip

lint-job:
  stage: lint
  before_script:
    - pip install pylint
  script:
    - pylint src/app.py
```

- modifiez votre `gitlab-ci.yml`
- ajoutez un *stage* `lint`
  - il utilise une image Docker de votre choix
    - qui contient déjà votre *linter*
    - OU qui l'installe au début du *stage* (forcément plus lent)
  - il exécute une commande pour *lint* le code
- faites un `git push` et vérifier que :
  - votre job dans le *stage* `lint` a bien été effectué
  - il a refusé le *push* car le code est pourri

➜ **HINT :** n'hésitez pas à commenter des jobs dans votre `.gitlab-ci.yml` pour éviter d'attendre la full pipeline à chaque fois. [On peut aussi "cacher" le job, en le préfixant par un `.`.](https://docs.gitlab.com/ci/jobs/#hide-jobs)

### C. Format me baby

Bon on va pas se taper toutes les erreurs de *lint* à la main hein.

➜ **Choisissez un *formatter* Python**

- pareil y'en a plusieurs, faites vos recherches
- `black` est une référence, mais y'a aussi `ruff` et d'autres

🌞 **Installez le *formatter* sur votre poste**

```
pip install black
```

```
dorian@Air-de-Dorian broken_webapp % black src/app.py
```

- exécutez le *formatter* sur le code de `broken_webapp`
- normalement le *linter* ne devrait ~presque~ plus crier

🌞 **Armés de votre *formatter* et votre gro cervo faites un push qui passe**

[#10427282086: lint-job](https://gitlab.com/Cub1tuS/broken_webapp/-/jobs/10427282086)

- donc vous utilisez le *formatter*
- ptet fix quelques lignes de code à la mano pour que ça match ce qui est demandé par le *linter*
- faites un `git push` qui passe les tests du stage `lint`

## 2. Building

Okayyyy donc on a notre premier vrai test sur le code. On passe à l'étape suivante qu'on retrouve dans toute *pipeline CI/CD* digne de ce nom : le build automatisé !

On va donc ajouter un *stage* `build` à notre fichier  `gitlab-ci.yml` qui déclenche un `docker build` automatiquement.

🌞 **Modifiez votre `.gitlab-ci.yml`**

```
image: python:3.9

stages:
  - lint
  - build

cache:
  key: pip-cache
  paths:
    - ~/.cache/pip

lint-job:
  stage: lint
  before_script:
    - pip install -r requirements.txt
  script:
    - pylint src/app.py

build-docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t webapp:latest .
```

[#10427282086: lint-job](https://gitlab.com/Cub1tuS/broken_webapp/-/jobs/10427344628)

- ajoutez un nouveau stage `build`
- il doit exécuter un `docker build` pour déclencher automatiquement un `build` du `Dockerfile`

➜ **HINT :** n'hésitez pas à commenter des jobs dans votre `.gitlab-ci.yml` pour éviter d'attendre la full pipeline à chaque fois. [On peut aussi "cacher" le job, en le préfixant par un `.`.](https://docs.gitlab.com/ci/jobs/#hide-jobs)

## 3. Publishing

Okay c'est cool de build, mais ça serait bien que ça serve à quelque chose et pas juste pour le plaisir de build un machin :d

On va donc finir par automatiquement `push` l'image Docker sur un *registre Docker*, afin de pouvoir ensuite la récupérer sur une autre machine, et la lancer.

Gitlab embarque un *registre Docker* qui est privé, pour chaque dépôt git que l'on crée. Pour avoir les droits de `push` dessus, il faut y être autorisé. Vous devrez donc utiliser une commande `docker login` dans votre pipeline.

🌞 **Modifiez votre `.gitlab-ci.yml`**

- ajoutez un nouveau stage `publish`
- il exécute une commande `docker push`
- l'image doit être nommée `broken_webapp`
- je vous laisse faire un peu de recherches ou de Gepetto pour build/push depuis votre pipeline, c'est un grand classique, vous trouverez beaucoup de doc :)


➜ **C'est un classique de build/push une image Docker dans une pipeline de CI**

- avec Gitlab, la logique est la suivante :
- Gitlab propose un registre pour y pousser nos images
  - il faut être authentifié pour pousser des images
  - dans le  `.gitlab-ci.yml` on a accès à des variables que Gitlab nous fournit
  - il y a des variables qui vont nous permettre de `login` et `push` de façon authentifiée
- les *jobs*/*stages* sont indépendants
  - ils ne partagent pas les fichiers
  - comment on fait pour build un truc dans un *job*, et s'en reservir dans un autre *job* ?
  - réponse : **les Gitlab Artifacts**
  - le job qui produit un fichier, on indique qu'il crée un `artifact`
  - les autres *jobs* pourront réutiliser ce fichier

Alleeez je vous donne un code typique qui *build* votre `Dockerfile` puis *push* l'image résultante dans le registre Gitlab.

```yml
stages:
  - build
  - publish

# quelques variables qu'on va reuse ensuite
variables:
  IMAGE_NAME: $CI_REGISTRY_IMAGE/app
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA

# le stage qui build l'image
build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind # dind pour docker in docker : parce qu'on va utiliser des commandes `docker` depuis un conteneur !
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    # le build est opéré ici
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    # on génère un .tar qui contient notre image
    - docker save $IMAGE_NAME:$IMAGE_TAG -o image.tar
  artifacts:
    paths:
      - image.tar # on déclare l'image sous format .tar comme un artifact

# le stage publish qui push notre image sur le registre Gitlab
publish:
  stage: publish
  image: docker:24
  services:
    - docker:24-dind
  dependencies:
    - build
  script:
    # on s'authentifie auprès du registre Gitlab # on s'authentifie auprès du registre Gitlab
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    # on charge l'image format .tar : elle est dispo car elle a été passée à ce job en artifact
    - docker load -i image.tar
    # on push l'image qui contient le hash de commit comme tag (unique à chaque push)
    - docker push $IMAGE_NAME:$IMAGE_TAG
    # retag vitefé de l'image avec le tag latest
    - docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
    # on push aussi l'image latest, qui sera mise à jour à chaque push
    - docker push $IMAGE_NAME:latest
```

➜ **Une fois que l'image a été *push*, elle est visible sur la WebUI de GitLab**

- rendez-vous sur la page principale de votre dépôt
- dans le menu latéral : `Deploy > Container Registry`

## 4. Some manual tests

Maintenant que l'image est publiée sur le registre Gitlab, elle est récupérable depuis n'importe quelle machine qui a un accès internet.

> Vous pouvez le faire sur votre poste, une VM, peu importe. On veut juste vérifier qu'on peut pull/run l'image correctement.

🌞 **Récupérer l'image**

- faites un `docker pull <IMAGE_NAME>` : récupération de l'image

> Vous pourrez voir le nom de l'image depuis la Webui, toujours au même endroit : `Deploy > Container Registry`

🌞 **Modifier le `docker-compose.yml`**

- l'image qu'il utilise pour `broken_webapp` doit être celle du registre Gitlab

> Faites un `git push` après ça, on a besoin que le `docker-compose.yml` sur le dépôt `broken_webapp` soit à jour et fasse référence à l'image du registre Gitlab.

🌞 **Lancer l'application**

- un `docker compose up`

🌞 **Does it work ?**

- un `curl localhost` : vérifier que l'app fonctionne

![CI/CD IRL](./img/cicd_irl.jpg)

## Next next next

👉 Ca commence à ressembler à un truc ! On continue avec [**du déploiement continue dans la partie 3**](./part3.md).
