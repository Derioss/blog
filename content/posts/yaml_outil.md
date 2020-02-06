---
title: "Des petits outils sur le YAML"
description: "Présentation de yamllint et gitlabworkflow"
date: 2020-02-06T19:20:42+01:00
publishDate: 2020-02-06T19:20:42+01:00
author: "Derios"
images: []
draft: false
tags: ["devops", "automatisation", "gitlab", "ansible"]
---

Un petit article pour vous présenter deux outils pour des pipelines plus lisible et plus propre :

- une extension de vscode, gitlabworkflow
- yamllint

# Yamllint

C'est un linter pour yaml, c'est donc un outil d'analyse statique de code qui sert en l'occurence essentiellement pour des problèmes de styles.

C'est rapide en mettre en place, à utiliser et permet à vos confrères et à vous mêmes d'avoir une meilleur lisibilité, c'est donc indispensable.

## Installation

Il existe sur la majorité des distribution linux, peut être intégré facilement dans un pipeline (donc).

Sur debian :

```bash
sudo apt install yamllint
```

Et c'est tout..

## Utilisation

Pour exemple, je vais l'utiliser sur mes codes présenté sur blog.

```YAML
- hosts: basicansible
  gather_facts: yes
  tasks:
    - name: echo hello world
      include_tasks: "echo-{{ ansible_distribution | lower }}.yml"
    - name: display ansible_distribution
      debug:
        msg: "{{ ansible_distribution | lower }}"    
    - name: echo hello
      debug:
        msg: "hello"
      when: ( ansible_distribution  == "CentOS")

```
Voici le résultat

![](/posts/images/yaml_outil.md/yamllint.PNG)

Il nous indique donc de manière très simple ou se situe les erreurs de style.

## Configuration

Comme tout linter, vous pouvez aisément le configurer.

Pour ma part, j'enlève le warning de document start et le line-lenght.

On créer donc le yaml suivant par ex dans `/etc/yamllint/yaml_conf.yaml`.

```YAML
extends: default
rules:
  line-lenght: disable
  document-start: disable
```

Pour l'utiliser

```bash
yamllint -d /etc/yamllint/yaml_conf.yaml <filename>
```
Bon c'est un peu long, un alias vite...!

Donc on va le mettre dans le `.bash_aliases`, si il n'est pas présent..

Ce n'est pas grave, on va le créer.

```bash
vim ~/.bashrc
```
La ligne à rajouter
```bash
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

On créer notre alias.

```bash
vim ~/.bash_aliases
```
la ligne à rajouter
```bash
alias yl='yamllint -d /etc/yamllint/yaml_conf.yaml'
```

On l'active avec cette commande ou on relog au choix.

```bash
source  ~/.bash_aliases
```
![](/posts/images/yaml_outil.md/yamllint2.PNG)

Voila c'est déja beaucoup mieux, comme vous pouvez le constater la configuration est bien prise en compte.

## L'extension de visualcode gitlabworkflow.

Bon un jour on m'a dit qu'un bon adminsys, est fainéant.

Vu que je vis pour automatiser tout ce qui bouge.. je suppose que si l'adminsys est le rois des fainéant, moi en qualité de devops je suis le pape.

Alors, vous allez me dire.. ou il veut en venir..

En gros, j'ai découvert cet extension que ce soir, oui que ce soir.. c'est fatiguant de parcourir le web.

Ce qui est génial avec cet outil c'est que :

-  j'ai besoin de faire un clic de moins sur mon navigateur pour contrôler mes issues, mes pipelines.. 
- pas besoin de passer par le linter web pour voir si mon code ne vas pas me pourrir un commit (car critical error).
Ce qui va m'obliger derrière à faire un :
```bash
git commit -a -m 'entrer votre message de commit' --amend  && git push --force
```

Fatiguant tout ca !

Et encore plus fatiguant, de monter un gitlab sur une vm, monter un pipeline pour vous montrer comment utiliser et configurer cet outil !

Alors qu' [ici](https://marketplace.visualstudio.com/items?itemName=fatihacet.gitlab-workflow) tout est très bien expliqué!

# Conclusion

Et comme on est très faineant, autant mettre ce genre de config dans un pipeline de déploiement client comme ca.. ben.. uniformisation des postes clients destinés au devops.

Et cela permet de boire son café en cas de reception d'un nouveau ordi.. en supervisant le déploiement de votre client par un projet ansible!