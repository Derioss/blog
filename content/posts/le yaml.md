---
title: "Le YAML: c'est bon mangez en!"
description: "Les basiques du devops"
date: 2020-01-28T20:21:20+01:00
publishDate: 2020-01-28T20:21:20+01:00
author: "Derios"
images: []
draft: false
tags: ["devops", "automatisation", "déploiement", "ansible", "sécurité","docker","gitlab-ci"]
---

# C'est quoi?

Ce n'est pas du XML.. on est bien avancé!

C'est une language fait par des humains pour des humains qui fait le taff avec beaucoup de languages de programmation moderne.

Quand on fait du devops, on manipule du YAML absolument PARTOUT (docker,elasticstack,docker-compose,ansible,prometheus,python,gitlab-ci,k8s,openshift..)

# Rentrons dans le sujet.

Le YAML permet nombre de chose .. la [bible](https://yaml.org/spec/1.2/spec.html).

Je ne vais faire au final que le survoler.

Mais pour être franc à moins de besoin spécifique, je n'ai pas eu le besoin au jour le jour de vraiment pousser jusqu'à la maîtrise.

## Les différent type de valeur

```YAML
sting: 'un'
bolean: true
bolean: false
null: null
int: 1
date: 2002-12-14
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

.template_script: &template_script
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

# Conclusion

Un article court, mais qui met le pied à l'étrier pour réaliser pipeline, compose file sans trop de duplication de code !