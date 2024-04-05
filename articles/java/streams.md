[Retour](../../index.md)

# Les streams
## Manipuler vos collections plus facilement

Sorti depuis la version 8, il est devenu beaucoup plus facile de manipuler les collections avec l'API stream de Java. Le code est devenu plus concis et plus lisible, sans avoir besoin de boucles imbriqués, de variable de condition et d'index numériques.

Pour vous donner un exemple rapidement, on veux récupérer tout les nombres pairs de cette liste:
```java
List<Integer> numbers = new ArrayList<>();
numbers.add(1);
numbers.add(2);
numbers.add(3);
numbers.add(4);
numbers.add(5);
```

Une implémentation simple pour les récupérer sans l'API stream:
```java
List<Integer> evenNumbers = new ArrayList<>();
for (int i = 0 ; i < numbers.size() ; i++) {
    if (numbers.get(i) % 2 == 0) {
        eventNumbers.add(numbers.get(i));
    }
}
```

Maintenant avec l'API stream:
```java
List<Integer> evenNumbers = numbers.stream()
    .filter(number -> number % 2 == 0)
    .toList();
```

Dans cette article, je vous montrerais les opérations qui me paraissent ls plus importantes à connaître. Une petite FAQ accompagnera chaque description de fonction, qui sont des cas questions que je me suis déjà posé.

## Comment récupérer un objet stream ?

De base, tout les objets de type `Collection` du package `java.util` (donc les `List`, les `Set` et les `Map` etc.) ont accès à la fonction `stream()` pour les transformer en objet de type `Stream`.

Il suffit d'instancier la variable et d'appeler cette fonction:
```java
List<Integer> numbers = new ArrayList<>();
Stream<Integer> stream = numbers.stream();
```

Mais il est possible d'instancier un `Stream` sans passer par une variable existante:
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
```

### FAQ
- Est-ce que je peux utiliser une même variable `Stream` plusieurs fois ?

Non, une fois instancié et utilisé pour être transformer, on ne peux plus l'utiliser. Il faut re-instancié une variable `Stream`. On ne peux pas faire:
```java
Stream<Integer> stream = numbers.stream();
List<Integer> numbers2 = stream.toList();
List<Integer> numbers3 = stream.toList(); // Une exception est lancé
```
Une exception sera lancé à l'exécution:
```
java.lang.IllegalStateException: stream has already been operated upon or closed
```

## Filtrer des éléments avec la fonction filter()

Pour filtrer des éléments, il faut faire appel à la fonction `filter` sur un objet `Stream`:
```java
List<Integer> numbers = new ArrayList<>();
Stream<Integer> stream = numbers.stream().filter(/* (argument) */);
```
Elle prend en paramètre une fonction pour trier les éléments. Il faut que cette fonction retourne un booléen. Par exemple pour récupérer tout les éléments inférieurs à 4:
```java
List<Integer> numbers = new ArrayList<>();
Stream<Integer> stream = numbers.stream().filter(number -> number < 4);
```

### FAQ
- Est-ce qu'il est plus performant d'avoir un seul `filter` avec beaucoup de condition ou plusieurs `filters` avec des petites conditions ?

Il est mieux d'avoir un seul `filter`. Un benchmark a été fais sur [Baeldung.com](https://www.baeldung.com/java-streams-multiple-filters-vs-condition). Je vous invite à le lire pour plus de détail.

## Transformer vos éléments dans un autre type avec map()

Pour transformer des éléments, il faut faire appel à la fonction `map` sur un objet `Stream`:
```java
List<Integer> numbers = new ArrayList<>();
numbers.stream().map(/* (argument) */)
```
Elle prends en argument une fonction qui réalise la transformation. Cela peut-être un changement de type, une valeur calculée etc. Le type de retour dépend de votre fonction. Si par exemple vous faîtes:
```java
List<String> names = new ArrayList<>();
names.stream().map(name -> name.toUpperCase())
```
Ici, comme le type de retour de `toUpperCase` est String. Vous êtes passé d'un `Stream<String>` à un `Stream<String>` après la fonction `map`, ce qui ne change rien dans le type de l'objet.

Par contre, si nous faisons:
```java
List<String> numbers = new ArrayList<>();
numbers.add("0");
numbers.add("1");
numbers.add("2");
numbers.stream() // Stream<String>
    .map(number -> Integer.valueOf(number)) // Stream<Integer>
```
Ici, on transforme des nombres en chaine de caractère, en nombre de type `Integer`. Comme le type de retour de `Integer.valueOf` est `Integer`, on est passé d'un `Stream<String>` à un `Stream<Integer>` après la fonction `map`. On ne pourra pas appeler de méthode de la classe `String` dans la suite du `Stream` vu que le type des éléments a changé. Exemple:
```java
numbers.stream() // Stream<String>
    .map(number -> Integer.valueOf(number)) // Stream<Integer>
    .map(number -> number.toUpperCase()) // Erreur de compilation
```
On ne peux pas appeler la fonction `toUpperCase` car l'élément n'est plus de type `String` mais de type `Integer`.

### FAQ
- Est-ce que je peux utiliser `map` pour appliquer un traitement sur mes éléments sans les transformer ?

Il ne vaut mieux pas, cette fonction n'est pas faite pour. Cela va rendre la lecture du code plus difficile pour d'autre personne qui vont le lire. Elles ne vont pas comprendre pourquoi vous avez utilisé cette fonction. 

Il vaut mieux passer par une boucle `forEach` dans ce cas-là. La fonction `peek` existe aussi, mais selon la [documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#peek-java.util.function.Consumer-), elle est destiné seulement à du débuggage.