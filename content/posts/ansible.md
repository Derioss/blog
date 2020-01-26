---
title: "basic ansible"
description: "Première prise en main d'ansible"
date: 2020-01-19T22:21:42+01:00
publishDate: 2020-01-19T22:21:42+01:00
author: "Derios"
images: []
draft: false
tags: ["devops", "automatisation", "déploiement", "ansible"]
---

![](/posts/images/ansible.png)

Une courte introduction à ansible.

J'aborderais la version non-graphique.

# Description

Ansible  est un des outils d'automation phare, simple de prise en main et puissant.
Il existe sous plusieurs formes :
- la version console  [documentation officielle](https://docs.ansible.com/ansible/latest/index.html)
- La version graphique entreprise [site officiel](https://www.ansible.com/products/tower)
- La version graphique en community edition [github officiel](https://github.com/ansible/awx) uniquement sur docker

La version graphique apporte une interface de monitoring des jobs qui peuvent être 'schedule', et surtout des profils utilisateurs.

Ce qui peut répondre à des besoins de sécurité, de travail en équipe avec le support de RedHat.

Il existe également une API fonctionnelle pour vos pipelines.

Elle n'apporte pas par-contre, à l'heure ou j'écris cette article, de possibilités d'automatisation plus poussées.

Pour avoir utilisé les deux, la vitesse de développement des projets est en faveur de la version 'classique'.

# Utilisation

## Premier pas

### installation

```bash
yum install ansible # centos/redhat
apt install ansible # debian's distri
dnf install ansible # fedora
```

### Création de notre premier playbook

L'hôte est une machine linux, un centos7.

On va commencer par créer deux fichiers :

- hosts contenant un ou plusieurs groupes, qui contient eux-même un ou plusieurs hôtes.

On peut mettre également des vars à ce niveau comme `ansible_user`, qui est l'identifiant de connection ssh de l'hôte.
```INI
[nomdugroupe]
hostname1|ip ansible_user=test
hostname2|ip

```

- playbook.yml, qui contiendra les tâches d'automatisation.

Nous allons afficher un world sur l'hôte en capturant le stdout dans une variable afin de l'afficher dans ansible.

```yaml
- hosts: nomdugroupe
  tasks:
    - name: echo world
      shell: echo "world"
      register: stdout

    - name: Display stdout
      debug:
        msg: "{{ stdout }}"
```

On a donc l'hôte qui est renseigné et les commandes a effectué, ansible réalisant une connection de type ssh, il nous manquera soit un mot de passe, soit une clé.
Pour le password, il y a différente manière de lui fournir :

- Avec une variable `ansible_ssh_pass=valeur`, on peut lui passer en ligne de commande, ou dans les différents fichier de notre projet (j'y reviendrai plus tard)

```bash
--extra-vars "ansible_ssh_pass=valeur"
# on peut également lui demandé en prompt
--ask-pass
```

```bash
ansible-playbook -i hosts playbook.yml --ask-pass
```

Ce qui donne le résultat :
![](/posts/images/premier_playbook_resultat.PNG)
