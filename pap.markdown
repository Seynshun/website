---
layout: page
title: PAP
alt_title: Programmation des Architectures parallèles
permalink: /pap/
---

### Liens 
- [Introduction](https://gforgeron.gitlab.io/pap/cours/introduction.pdf)
- [OpenMP](http://dept-info.labri.fr/ENSEIGNEMENT/pmc/transparents/openmp.pdf)

### Gdb 
Options d'optimisation pour gagner du temps :
- -O3 ~ facteur 6-7
- -O0 ~ facteur 10

### SIMD (Single Instruction on Multiple Data) : Parallélisme avec vecteurs
 
Problème : Tous les threads ne sont pas solicités de la même manière.
#### Solution : 
- Option "dynamic" : Utilise un "guichet" (mutex)
- Option "static" : Zone traitée par un coeur donné prédterminée

### Grain de parallélisation

Découper en petite taches = beaucoup de parallélisme mais coûte cher. Il faut trouver un équilibre sur la taille du découpage 

### Parallélisme Fork-Join 

```c
void *fun(void * id){
    printf("Bonjour\n");
    return NULL;
}

void main(){
    
    pthread_t tids[N];
    int id[N]; id[0] = 0;

    for (int i = 1; i< N; i++){
        id[i] = i;
        pthread_create(&tids[i], NULL , fun , &id[i]);
    }

    fun(&id[0]);

    for (int i = 1; i < N ; i++){
        pthread_join(tids[i], NULL);
    }
}
```
### Calcul en parallèle : distribution d'indices
Fonctionnement d'un omp parallel for (static) : 
```c
void * omp_fun(void * id){
    int me = * (int*) id;
    int taille = NBELEM / N;
    int debut = me * taille;
    int fin = (me + 1) * taille;
    if (me == N-1){
        fin = NBELEM;
    }
    for (int n = debut; n<fin ; n++){
        printf("Bonjour omp\n");
    }
    return NULL;
}
```

### Changer le comportement du programme :

```c
#pragama omp paralell{
    ...
    #pragma omp for schedule(dynamic){
        ...
    }
}
```