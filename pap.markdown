---
layout: page
title: PAP
alt_title: Programmation des Architectures parallèles
permalink: /pap/
---

### Liens 
- [Introduction](https://gforgeron.gitlab.io/pap/cours/introduction.pdf)
- [Premiers pas en OpenMP](https://gforgeron.gitlab.io/pap/cours/omp-parallel-for.pdf)
- [Les tâches en OpenMP](https://gforgeron.gitlab.io/pap/cours/taches.pdf)

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
#pragama omp paralell{ //pthread_create
    ...
    #pragma omp for schedule(dynamic){
        ...
    }
} //pthread_join(barrier)
```

Pour paralléliser un code, les itérations doivent être indépendantes les unes des autres. Ce n'est pas le cas de t[i] = t[i] + t[i-1]

### Comment paralléliser ce code ?
```c
for (etape = 0; etape < k; etape++){
    for(int i = 0; i<10; i++){
        out[i] = f(input,i);
    }
    memcpy(input,out,...);
}
```

#### Parallélisation :

```c
#pragma omp parallel // Ici car si on le place dans une boucle, on créerait des threads à chaque itérations
for (etape = 0; etape < k; etape++){
    #pragma omp parallel for
    for(int i = 0; i<10; i++){
        out[i] = f(input,i);
    } // Barrière implicite
    #pragma omp master // ou if (omp_get_thread_num() == 0)
    memcpy(input,out,...); // 1 seul thread fait memcpy
    #pragma omp barrier // on attend tous les threads
}
```

#pragma omp single fait comme #pragma omp master mais avec une barrière implicite.
```c
#pragma omp single
memcpy(input,out,...);
```

##### Pour enlever le memcpy :

```c
int * entree = input;
int * sortie = out;
#pragma omp parallel 
for (etape = 0; etape < k; etape++){
    #pragma omp parallel for
    for(int i = 0; i<10; i++){
        sortie[i] = f(entree,i);
    } // Barrière implicite
    #pragma omp master 
    swap(entree,sortie); 
    #pragma omp barrier 
}
```

##### Pour encore améliorer le code :

```c
int * entree = input;
int * sortie = out;
#pragma omp parallel firstprivate(entree,sortie) // Chaque thread à sa propre instance de variable et est initialisée avec la valeur de la variable
for (etape = 0; etape < k; etape++){
    #pragma omp parallel for // ou "pragma omp parallel for nowait" pour faire sauter la barrière implice plutôt que la dernière barrière 
    for(int i = 0; i<10; i++){
        sortie[i] = f(entree,i);
    } // Barrière implicite
    swap(entree,sortie); 
}
```

### Comment paralléliser ce code ?

```c
for (int i =0; i<N; i++)
    s += f(i);
```

#### Parallélisation :

```c
#pragma omp parallel for reduction (+:s)
for (int i =0; i<N; i++)
    s += f(i);
```

### Différents cas de compléxités :

- constant -> static
- linéaire -> static,1
- exponentielle -> dynamic
- aléatoire -> static,1 (sauf si périodique) ou dynamic si les variations sont grandes

### Tâches

#pragma omp task permet d'initialiser une tâche avec un contexte d'éxecution + structure de données.


#pragma omp task firstprivate(data1) shared (data2) congèle data1 en créant une copie et utilise un pointeur sur data2.

#### Fibonacci version sequentielle :

```c
int fib(int n){
    if (n<2) return n;
    return fib(n-1) + fib(n-2);
}
```

#### Fibonacci version parallèle :

```c
int fib_par(int n){
    if(n<2) return n;
    int x,y;
    #pragma omp task firstprivate(n) shared(x)
    x = fib(n-1);
    #pragma omp task shared(n,y)
    y = fib(n-2);
    #pragma omp taskwait
    return x + y;
}
```

```c
int main(){
    #pragma omp parallel //sinon pas de parallélisme pour fib(4)
    #þragma omp master // 1 seul thread genère toutes les tâches, les autres sont bloqués sur la barrière implicite
    int f = fib(4);
    printf("fib(4)= %d",f);
}
```

#### Dépendence des tâches :

```c
#pragma omp task
x = f()
#pragma omp task
y = g(x)
```
La seconde tâche peut être executée avant la première et donc causer des problèmes. POur éviter cela :
```c
#pragma omp task depend(out:x)
x = f()
#pragma omp task depend(in:x)
y = g(x)
```

### Correction DS 2016

#### Question 1

##### Version sequentielle 

```c
for(int s = 0; s < N ; s++){
    for(int i = seq[s].debut; i<seq[s].fin; i++){
        moyennes[s]+= elements[i];
        }
    moyennes[s]= moyennes[s]/seq[s].fin - seq[s].debut
}
```

##### Version parallèle

```c
#pragma omp parallel for
for(int s = 0; s < N ; s++){
    #pragma omp parallel for reduction (+:moyennes[s])
    for(int i = seq[s].debut; i<seq[s].fin; i++){
        moyennes[s]+= elements[i];
        }
    moyennes[s]= moyennes[s]/seq[s].fin - seq[s].debut
}
```
#### Question 3

|       | 10^7 seq 4 | 4 seq de 10^7 | melange |
|       |:--------   |:-------------:|--------:|
|externe| 100%       | 4%            | ~50%    |
|interne| 10^-4%   | 100%            | 10^-4%  |
|       |======================================|

#### Question 4 

```c
#pragma omp parallel for
for(int s = 0; s < N ; s++){
    if(seq[i].fin - seq[i].debut > SEUIL){
        #pragma omp critical
        gros_cas[SIZE] = s;
    else{
        for(int i = seq[s].debut; i<seq[s].fin; i++){
            moyennes[s]+= elements[i];
            }
        moyennes[s]= moyennes[s]/seq[s].fin - seq[s].debut
    }
}
```
