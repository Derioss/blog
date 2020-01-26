---
title: "Les variables dans ansible"
description: "Première prise en main"
date: 2026-01-24T22:21:42+01:00
publishDate: 2020-01-19T22:21:42+01:00
author: "Derios"
images: []
draft: false
tags: ["devops", "automatisation", "déploiement", "ansible"]
---
![](/posts/images/ansible.png)
Dans cet article, j'aborderais l'utilisation des [variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html) dans ansible.

Il y a globalement deux méthodes pour utilisé nos variables dans l'automatisation :

- utilisé le système de variables de ansible.
- injecté des variables dans l'environnement de l'hôte.

On préfèrera l'une ou l'autre méthode suivant les cas d'utilisation.

# Les `ansible_facts`

## Description

Il s'agit de variables par défaults, des informations sur l'hôte qui sont récupérés par ansible.

Elles peuvent nous fournir diverses informations utiles sur l'hôte, réseau, système, comme `ansible_distribution` qui peut-être utile pour modifier le comportement de son playbook en fonction de la distribution de l'hôte.

## Bonne pratique

La bonne pratique quant on a besoin d'une information sur l'hôte est de privilégier l'utilisation de ces variables plutôt que d'éxecuter une commande comme `cat /etc/os-release` afin de récupérer la valeur dans une variable.

## activation/désactivation

```YAML
- hosts: basicansible
  gather_facts: yes # or false
```

## utilisation

On continue avec le hello world..

Dans le même répertoire: 

- fichier playbook.yml
```YAML
- hosts: basicansible
  gather_facts: yes
  tasks:
    - name: echo hello world
      include_tasks: "echo-{{ ansible_distribution | lower }}.yml" #permet d'inclure une task provenant d'un autre playbook.
    - name: display ansible_distribution
      debug:
        msg: "{{ ansible_distribution | lower }}"
    
    - name: echo hello
      debug:
        msg: "hello"
      when: ( ansible_distribution  == "CentOS")

    - name: echo world
      debug:
        msg: "world"
      when: ( ansible_distribution  == "Debian")
```
- un fichier hosts
- un fichier echo-centos.yml
```YAML
- name: echo hello
  shell: echo "hello"
  register: stdout

- name: Display stdout
  debug:
    msg: "{{ stdout }}"
```
- un fichier echo-debian.yml
```YAML
- name: echo world
  shell: echo "world"
  register: stdout

- name: Display stdout
  debug:
    msg: "{{ stdout }}"
```

Si on lance le résultat sur un centos.
![](/posts/images/ansible_variable/ansible_fact.PNG)

On viens de modifier le comportement d'un playbook grâce aux ansible_facts.

# Nos variables à nous.

Bien entendu nous pouvons utiliser des variables dans nos playbook.

## La convention de nommage

Le nommage peut se faire avec des chiffres et des lettres, mais doit commencer par des lettres mais l'underscore est permis..ouf!

valid :
- var
- var_var
- var_t0t0

non_valid:
- var-t0t0
- 0var

## La définition des variables
On peut les inclures dans :

- le fichier hosts
```INI
[group]
hostname var=toto
```
- dans les playbooks
```YAML
- hosts: group
  vars:
    http_port: 80
```
- dans un fichiers vars et l'inclure dans un playbook.

```YAML
# vars.yml
vars1: value
vars2: value
```

```YAML
# playbook.yml
- hosts: group
  vars_files:
    - vars.yml
```
- dans une task
```YAML
- name: echo world
  shell: echo "world"
  register: stdout
```
- les utiliser en ligne de commande
```BASH
ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"
```

- dans les rôle (qui feront l'objet d'un prochain article)
```YAML
# fichier playbook.yml
- hosts: group
  vars_files:
    - vars.yml
  gather_facts: yes
  roles:
  - {role: docker, var: value tags: [always]}
```

Sachant qu'il y a un phénomène de surcharge, si la même variable est set à différents niveau, se référer à la [doc](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)

## L'utilisation des variables

Ansible utilise Jinja2 pour les variables, on peut donc les transformer si besoin.

Elles peuvent s'utiliser à tous les niveaux des tasks, dans les templates...

```YAML
- hosts: basicansible
  gather_facts: yes
  vars:
    echo: echo world
  tasks:
    - name: echo "{{ echo world }}"
      include_tasks: "echo-{{ ansible_distribution | lower }}.yml" # la syntaxe "{{ var | filters}}" montre l'utilisation d'un filtre jinja2.
```

# Manipulation de l'environnement de l'hôte

On peut injecter très facilement avec ansible des variables d'environnement dans l'hôte, avec le mot clé `environment` ([doc officielle](https://docs.ansible.com/ansible/latest/user_guide/playbooks_environment.html)).

On peux l'utiliser à différents niveau, dans les tasks, au niveau du playbook comme le montre l'exemple, dans un block de task.

```YAML
- hosts: testhost

  roles:
     - php
     - nginx

  environment:
    http_proxy: http://proxy.example.com:8080
```