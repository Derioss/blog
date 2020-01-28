---
title: "Déploiement d'un blog avec Hugo"
description: "Je décris le montage de mon blog"
date: 2020-01-25T22:21:42+01:00
publishDate: 2020-01-25T22:21:42+01:00
author: "Derios"
images: []
draft: false
tags: ["blog", "web"]
---

La décision de création du blog est venu d'une volonté de transmettre de la documentation à des confrères.

J'aurais pu leurs envoyer une doc généré par [Joplin](https://joplinapp.org/) (alternative open-source à evernote), puis quitte à faire le boulot, pourquoi pas le publier sur la toile, c'est pas comme ci je passais mes journées à profiter du savoir librement diffusé par mes confrères sur le web...

Comment le diffuser ?? 

Je vais faire le site moi-même ! cool, monté en compétence sur Flask, sur du js que j'ai utilisé que pour le scrapping, ce qui implique location d'un vps.

Allez c'est partit, une bonne Centos, selinux, gestion de droit aux petits oignons, https, un ptit nginx en frontal, monitoring qui dit monitoring dit prometheus et elk.. mmm overkill, vu que je n'ai pas l'intention à court terme de générer du trafic.

C'est la qu'hugo, découvert au sein de mon entreprise intervient.

 Une soirée pour publier un site fonctionnel en comptant le peu de monté en compétence nécessaire et gratuit !

# Installation d'hugo

Plusieurs [choix](https://gohugo.io/getting-started/installing/) s'offre à nous pour l'installation, je vous déconseille juste l'install apt fournissant une vieille version d'Hugo.

Je vais aborder ici, l'installation sous linux.

Pour ma part un sous-système ubuntu sous windows (oui je sais..windows mais bon je suis gamer) pas que j'adore Ubuntu pour bosser mais pour l'instant le choix est plus que limité sous windows.. enfin dans 5 ans, ca sera une distribution linux plus qu'a attendre.

## installation d'hugo server

```bash
brew install hugo
```

## Création du site

On se place donc dans le dossier dans lequel on voudra mettre notre projet.

```bash
hugo new site blog && cd blog #vous être libre du nom.
```

On choisi notre [thème](https://themes.gohugo.io/)

```bash
# installation du thème choisi
git submodule add https://github.com/danielkvist/hugo-terrassa-theme.git themes/terrassa
echo 'theme = "terrassa"' >> config.toml
# on démarre le site
hugo server
```
Et c'est tout, vous avez maintenant votre site accessible en local!
![](/posts/images/hugo.PNG)

# Déploiement

Plusieurs solutions s'offre à nous vps, self-hosted, il suffit donc de coller un frontal et de créer un service hugo.

Mais au vu de la qualité de ma connection et de ma propension à déployer l'artillerie lourde pour exposer le moindre port sur le net..

J'ai choisi une solution simple et gratuite dans un premier temps : [github.io](https://pages.github.com/)

Pour se faciliter la vie le bon procéder est de créer deux repo :

- un en name.github.io qu'on initialise avec README.md, qui contiendra le site.
- un autre qui contiendra notre code.

Comme vous pouvez le voir sur mes repos [site](https://github.com/Derioss/Derioss.github.io), [code](https://github.com/Derioss/blog)

## git de notre code

```bash
# On se place à la racine de notre site
git init
git add .
git commit -m "first commit"
git remote add origin <votrerepo>
git push -u origin master
```

## génération et publication de notre site statique

```bash
# A la racine de notre site
rm -rf public
git submodule add -b master <votrerepo>.github.io.git public
# génération des pages statiques
hugo
# un ptit commit de notre opération
git add .
git commit -m "first commit" --amend
git push -f
cd <name>.github.io/
# vérification qu'on est bien sur le bon repo git
git remove -v
git add .
git commit -m "first commit"
git push -f
# voila votre site est publié à https://<name>.github.io/
```

Il nous restera plus qu'a créer nos pages dans `content` en suivant les instructions du thème pour la personnalisation et la structure.

On peut voir le rendu en local grâce à la commande `hugo server`, pour publier, il suffira de renouveler l'opération avec la commande `hugo` + commit du code.