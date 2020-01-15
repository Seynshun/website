---
layout: page
title: PAP
alt_title: Programmation des Architectures parallèles
permalink: /pap/
---

[Introduction](http://dept-info.labri.fr/ENSEIGNEMENT/pmc/transparents/introduction.pdf)

[OpenMP](http://dept-info.labri.fr/ENSEIGNEMENT/pmc/transparents/openmp.pdf)

 - Gdb : Options d'optimisation pour gagner du temps :

 -O3 ~ facteur 6-7

 -O0 ~ facteur 10

 - SIMD (Single Instruction on Multiple Data) : Parallélisme avec vecteurs

 
 Problème : Tous les threads ne sont pas solicités de la même manière.
 
 Solution : 
 - Option "dynamic" : Utilise un "guichet" (mutex)
 - Option "static" : Zone traitée par un coeur donné prédterminée

# Grain de parallélisation

Découper en petite taches : beaucoup de parallélisme mais coûte cher

Il faut trouver un équilibre sur la taille du découpage 

# Parallélisme Fork-Join 

```c
void *fun(void * id){
    printf("Bonjour\n");
    return NULL;
}
```

```C
void *fun(void * id){
    printf("Bonjour\n");
    return NULL;
}
```