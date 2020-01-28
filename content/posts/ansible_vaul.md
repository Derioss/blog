---
title: "Ansible: un peu de sécurité avec Ansible vault"
description: "Première prise en main d'ansible"
date: 2020-01-28T20:21:20+01:00
publishDate: 2020-01-28T20:21:20+01:00
author: "Derios"
images: []
draft: false
tags: ["devops", "automatisation", "déploiement", "ansible", "sécurité"]
---
![](/posts/images/ansible.png)
Aujourd'hui je vais vous présenter une fonctionnalité d'ansible, **Ansible-vault**, un mini article donc mais qui à son importance.

Cette Feature existe dans toutes les versions d'Ansible, je suis assez fan, c'est simple et efficace.

Elle permet de chiffrer un fichier complet ou une valeur rapidement afin de l'utiliser dans nos playbook grâce à un Masterkey.
Il peut être contenu dans une variable (demande un ptit tricks) ou dans un fichier comme nous allons le voir plus loin ou même en prompt (mais c'est pas très automatisable un prompt...)

# Fournir le masterkey.

## La base.
```bash
--ask-vault-pass #prompt
--vault-password-file ~/.vault_pass.txt # le fichier .txt se doit de contenir un seul string sur une seule ligne.
--vault-password-file ~/.vault_pass.py # le script python doit fournir un string en sortie standard.
```
## La "mini" combine.

Par exemple dans un pipeline Gitlab-CI (je ferais prochainement un article complet), on aura auparavant fournit une variable protéger au repo `ANSIBLE_VAULT_PASSWORD`.

- build-vault construit le fichier contenant le password.
- syntax-check execute le playbook en utilisant le fichier.

Le masterkey est donc protégé, et nous pouvons ainsi protéger toutes nos informations sensible contenu dans nos variables.

Je note que la version graphique de ansible, permet de stocker de manière sécurisé les masterkeys, afin de les réutiliser dans nos jobs.. mais bon c'est moins marrant !

### Le contenu du pipeline

Un extrait de pipeline.

```YAML
build-vault:
  stage: vault
  script:
   - echo $ANSIBLE_VAULT_PASSWORD > .vault_pass.txt
  artifacts:
   paths:
   - .vault_password.txt
   expire_in: '10 mins'
syntax-check:
  stage: check
  dependencies:
   - build-vault
  script:
   - ansible-playbook -i hosts playbook.yml --vault-password-file .vault_pass.txt --syntax-check
```

### contenu du script.
```bash
#vault_pass.sh
#!/usr/bin/env bash
echo "${ANSIBLE_VAULT_PASSWORD}" > .vault_pass.txt
```

# Chiffrer

On peut donc chiffrer un fichier complet ou valeur.

## fichier

```bash
ansible-vault encrypt file # si on ne lui précise pas de masterkey, il nous la demande en prompt.
```

## valeur

```bash
ansible-vault encrypt_string "ma-valeur-secrete"
```
Le résultat avec comme masterkey.. masterkey

![](/posts/images/ansible_vault/vault.PNG)

On l'incorpore par exemple dans un fichier `vars.yml`.
```YAML
path: '/home/exploit'
var_vault: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66656666663434663231373834613061313136636633323463643965393162656331373639616363
          3463653861366636623035353531636331653533646131330a366138356634656438356632643963
          34653633646536613434323734663135386561353865656137376636363866363663646139653738
          6564343531376361630a313437663462666437393731616463613839383061623638393366363230
          30653165656330303331326261396664623065363737643731613934303862663563
another_vars: 'value'
```

Ainsi à l'execution du playbook, on lui précise la masterkey comme on l'a abordé plus haut .. et c'est tout!

# Lire

## fichier

```bash
ansible-vault edit file
```

## Valeur

```bash
echo '$ANSIBLE_VAULT;1.1;AES256
66656666663434663231373834613061313136636633323463643965393162656331373639616363
3463653861366636623035353531636331653533646131330a366138356634656438356632643963
34653633646536613434323734663135386561353865656137376636363866363663646139653738
6564343531376361630a313437663462666437393731616463613839383061623638393366363230
30653165656330303331326261396664623065363737643731613934303862663563' | ansible-vault decrypt
```
![](/posts/images/ansible_vault/vault_result.PNG)

p.s : le warning qui s'affiche sur les screens, c'est la non-présence du fichier INI de configuration ansible.cfg, ou une execution dans un répertoire ultra permissif (ce qui est ici le cas.. oui c'est mal).

Si malgrès tout ca se justifie pour X  mauvaises raison.

Le workaround c'est passer ANSIBLE_CONFIG=ansible.cfg dans l'environnement qui execute le playbook.

# Conclusion

J'ai aborder ici les principales fonctionnalitées qui couvre bien des périmètres d'utilisation, mais sachez qu'il y en a d'autre sur la [doc officielle](https://docs.ansible.com/ansible/latest/user_guide/vault.html)