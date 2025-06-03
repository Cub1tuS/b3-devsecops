# TP1 : DevSecOps or something

## Sommaire

- [TP1 : DevSecOps or something](#tp1--devsecops-or-something)
  - [Sommaire](#sommaire)
  - [Prérequis](#prérequis)
  - [Part 1 : DevOps init](#part-1--devops-init)
  - [Part 2 : Test then Build](#part-2--test-then-build)
  - [Part 3 : Deploy](#part-3--deploy)
  - [Part 4 : Shift Left](#part-4--shift-left)
  - [Part 5 : U said SecOps](#part-5--u-said-secops)

## [Part 1 : DevOps init](./part1.md)

Dans cette première partie, on met en place le **socle d'outils DevOps** pour commencer à bosser sur la suite.

Au menu : on va utiliser **Gitlab** pour **lancer des tâches automatiquement** à chaque push sur un repo.

> On pourrait installer notre propre Gitlab, mais ça bouffe pas mal de ressources, et se taper l'install/conf, c'est chiant sur 2 jours si on peut l'éviter.

Pour bosser, **je vous file un bout de code que j'ai écrit qui lance un ptite appli web**.

[**Part 1 : DevOps init**](./part1.md)

---

🔨 **Main tools** : Gitlab, Gitlab Runner


## [Part 2 : Test then Build](./part2.md)

On continue avec **les premiers morceaux d'une chaîne d'automatisation clean**. Une partie dédiée à **la CI**.

> *CI* pour *Continuous Integration* (ou *Intégration Continue*) : le fait d'effectuer des actions automatiques à chaque push sur un dépôt git.

On effectue des **tests automatiquement** sur le code, et on **refuse le push si les tests ne sont pas validés**.

Si les tests sont validés, on déclenche un **build automatique du code**. Ici, on déclenchera le build d'une image Docker.

[**Part 2 : Test then Build**](./part2.md)

---

🔨 **Main tools** : Gitlab, Gitlab Registry, Python linter, Python formatter, Docker, Docker registry

## [Part 3 : Deploy the world](./part3.md)

Troisième partie : **on met en place la CD.**

> *CD* pour *Continuous Deployment* (ou *Déploiement Continu*) : le fait de déployer le code automatiquement sur une machine à la fin des tests automatisés.

Dans le TP, on va réutiliser votre ~magnifique~ compte Azure pour pop une VM avec une IP publique. **Ce sera notre environnement de *"production"*.**

> Je vous recommande d'utiliser le client shell `az` pour pop/depop rapidement des VMs au besoin depuis votre terminal, plutôt que la WebUI.

[**Part 3 : Deploy the world**](./part3.md)

---

🔨 **Main tools** : Gitlab, SSH, Azure, Docker

## [Part 4 : Shift Left](./part4.md)

**C kan mem chian d'attendre plusieurs minutes pour savoir que notre push est refusé à cause d'une erreur de syntaxe nulle.**

Dans cette partie on va ***shift left* le *linting*** : plutôt que d'avoir le *linting* uniquement en CI, qui ralentit le dév inutilement, on va ajouter un *hook* git directement dans le dépôt git des dévs (nous).

Précisément, on va ajouter **un *hook* *precommit*** : un ptit script qui s'exécute en local sur la machine du dév à chaque fois qu'un `git commit` est effectué. Si le script renvoie une erreur, le `commit` n'est pas validé.

> L'avantage : le *feedback* est instantané, et on a pas besoin d'attendre que la *pipeline* déroule. Aussi, on déclenchera moins de pipelines inutilement.

Concrètement, on va coder **un ptit script shell propre** qui va exécuter le *linting* en local, avant que le push soit effectué.

[**Part 4 : Shift Left**](./part4.md)

---

🔨 **Main tools** : Git, Git Hooks, Bash, Python Linter, Python formatter

## [Part 5 : U said SecOps](./part5.md)

On y vient enfin ? Sans une base CI/CD solide, DevSecOps n'a aucun sens, obligés donc de passer par les parties précédentes.

Dans cette partie, on va se concentrer sur **l'automatisation de tests liés directement à la sécurité**. Au menu, on va automatiser le déroulement de :

- recherche de secrets
- analyse statique de code
- analyse dynamique de code
- scan de vulnérabilités image Docker

[**Part 5 : U said SecOps**](./part5.md)

---

🔨 **Main tools** : Gitlab, Gitlab Variables, Vulnerability Scanner (Trivy), Python Static Analysis (Bandit), Dynamic Analysis (Nuclei), Secrets Digging (detect-secrets)