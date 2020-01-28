---
title: "Le YAML: c'est bon mangez en!"
description: "Les basiques du devops"
date: 2020-01-28T20:21:20+01:00
publishDate: 2020-01-28T20:21:20+01:00
author: "Derios"
images: []
draft: true
tags: ["devops", "automatisation", "déploiement", "ansible", "sécurité","docker","gitlab-ci"]
---

# C'est quoi?

Ce n'est pas du XML.. on est bien avancé!

C'est une language fait par des humains pour des humains qui fait le taff avec beaucoup de languages de programmation moderne.

Quand on fait du devops, on manipule du YAML absolument PARTOUT (docker,elasticstack,docker-compose,ansible,prometheus,python,gitlab-ci,k8s,openshift..)

# Rentrons dans le sujet.

## Une valeur

```YAML
liste: 'un'
```

## Une liste

Deux syntaxes.

```YAML
liste: ['un','deux','trois']
autre_liste:
    - 1
    - 2
    - 3
```

## Entiers

```YAML
canonique:      1.23015e+3
exponentielle:  12.3015e+02
sexagesimal:    20:30.15
fixe:           1_230.15
infini negatif: -.inf
pas un nombre:  .NaN
```

## Les ancres
`*anchor` : set la valeur 
`<<: *anchor` : permet d'extend
### Utilisation dans un pipeline gitlab

Ex: dans une pipeline Gitlab (ce qui sert à faire des templates), le `.` sert à commenter un job pour qu'il ne s'execute pas.

```YAML
# Mes trois scripts sont strictement équivalent.
.template: &template
  stage: test
  script:
    - |
      if [ -n $TEST ]; then
        echo 'hello world'
      fi
    - >
      if [ -n $TEST ]; then
        echo 'hello world';
      fi 
    -  if [ -n $TEST ]; then echo 'hello world'; fi

&template_script :
    - |
        if [ -n $TEST ]; then
        echo 'hello world'
        fi
    - >
        if [ -n $TEST ]; then
        echo 'hello world';
        fi 
    -  if [ -n $TEST ]; then echo 'hello world'; fi

job1:
  <<: *template
  rules:
    - if $CI_COMMIT_REF_NAME == "master"
  when: always

job2:
  <<: *template
  rules:
    - if: $CI_COMMIT_REF_NAME == "master"
      when: manual

job3:
  stage: test
  script: *template_script
  rules:
    - if: $CI_COMMIT_REF_NAME == "master"
      when: manual
```

