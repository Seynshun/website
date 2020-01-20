---
layout: post
title:  Commandes Git
date:   2020-01-07 11:37:29 +0100
categories: jekyll update
---

# Quelques commandes git utiles 

- Réinitialiser à l'état du dernier commit : 
```html
git reset --hard HEAD 
```
- Créer une branche :
```html
git branch nom-branche
```
- Changer de branche : 
```html
git checkout nom-branche
```
- Renommer fichier : 
```html
git mv old_filename new_filename
```

- Sauvegarder modifications : 
```html
git stash
```

- Restaurer modifications : 
```html
git stash apply
```

- Consulter la liste des stashs : 
```html
git stash list
```

- Fusionner des dépôts en gardant l'historique : [Lien](https://www.alexkras.com/git-merg-two-or-more-repositories-and-keeping-history/)