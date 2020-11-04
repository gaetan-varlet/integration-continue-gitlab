# Gitlab CI/CD

Gitlab permet de gérer l'intégration et le déploiement continue (CI/CD) du projet en ajoutant le fichier de configuration **.gitlab-ci.yml** au répertoire racine du dépôt Git
- **Continuous Integration** : vérifier à chaque modification de code source que le résultat des modifications ne produit pas de régression dans l’application développée (tester de manière automatisée)
- **Continuous Delivery** : livrer chaque modification apporté au logiciel directement aux utilisateurs finaux sans intervention humaine

## Configuration de .gitlab-ci.yml

Ce fichier s'écrit en **YAML**. Il configure le projet pour utiliser un **Runner** pour que chaque *push* déclenche l'exécution des pipelines d'intégration continue.

Le fichier YAML définit un ensemble de **jobs** (*tâches*), qui doivent obligatoirement être exécutés dans un **stage** (*étape*) et de toujours contenir l'instruction **script** qui contient les commandes à exécuter par le runner. Los jobs sont exécutés dans l'ordre des étapes renseignées dans la clause **stages**.

```yml
image: python:3.6-slim

before_script: # Commandes à exécuter avant de lancer les jobs
  - python -V # Print out python version for debugging
  - pip install pyminifier

stages:
  - test
  - build
  - deploy

job_test:
  stage: test
  script:
    - echo "Running tests"
    - python -m unittest discover -s "./src/modele/formats/" -p "*Test.py"

only:
  - branches
except: - master

job_run:
  stage: build
  script:
    - echo "Building the app"
    - mkdir -p ./appminifier/modele/formats
    - pyminifier --destdir=appminifier/ src/*.py > appminifier/main.py
    - pyminifier --destdir=appminifier/modele/formats src/modele/formats/*.py
    - python appminifier/main.py

  artifacts:
    paths:
      - appminifier
      expire_in: 1 week

  only:
    - Dev


job_deployment_staging:
  stage: deploy
  script:
    - echo "deploy the app into staging"
  environment:
    name: staging
  only:
    - master


job_deploy_prod:
  stage: deploy
  script:
    - echo "Deploy into the production server"
  environment:
    name: production
  when: manual
  only:
    - master
```


## Pipeline

Un pipeline est un groupe de jobs exécutés par étapes. Les jobs d'une même étape sont exécutés en parralèle s'il y a suffisamment de Runners simultanés disponibles. Le pipeline passe à l'étape suivante lorsque les jobs en cours terminent leur exécution sans erreurs. Si un job échoue, le pipeline s'arrête.


## Gitlab CI Runner

Un Runner est une machine virtuelle, par exemple un container Docker, qui permet de personnaliser l'environment d'exécution du job.
