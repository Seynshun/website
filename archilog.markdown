---
layout: page
title: Architecture logicielle
permalink: /archilog/
---
[Cours](https://www.labri.fr/perso/auber/ALM1GL/cours/AL_C12.pdf)

- Identité d'un objet = son adresse
- Classification : Si 2 classes ont la même structure, on peut les transformer en 1 classe avec 2 instances.
- Héritage : Une classe fille hérite des méthodes de la classe mère. Héritage multiple interdit en Java (une classe ne peut pas hériter de 2 classes sinon il y aurait un problème dans la classification)
mais possible en C++
- Polymorphisme : Au lieu d'hériter une méthode, on peut la raffiner le long de l'arbre d'héritage en la redéfinissant dans les classes filles -> @Override

Une classe ne devrait pas avoir de méthodes ou données statiques car statique = partagé par toute les instances.
```java
public class A{
    public static int y;
    public int b;
}
```
```java
A a1 = new A();
A a2 = new A();
a1.y = 4;
a1.b = 7;
a2.y = 6;
a2.b = 8;
```

L'objet a1 à la fin possède y = 6 et b = 7. Lo'bjet a2 à la fin possède y = 6 et b = 8

#### Visibilité :
- public : si on change une méthode, cela impacte tous ceux l'utilisant
- protected : si on change une méthode, cela impacte ceux héritant de cette méthode (<=> public)
- private : les problèmes restent internes à la classe -> Maintenable


