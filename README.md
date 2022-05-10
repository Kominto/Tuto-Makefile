# LES MAKEFILES
**Merci à Emily Decher AKA Codinget pour toutes les corrections qu'elle a apporté, elle est géniale :D**

Alors, pour commencer, un makefile c'est un fichier qui contient toutes les "règles de compilation".
C'est quoi ces règles? Les différentes commandes `g++` (ou autre compilateur selon le langage) à effectuer, ainsi que quelques règles pour savoir dans quel ordre les faire etc...
Après, ya juste à faire la commande `make` pour (re)compiler votre programme, ce qui est un sacré gain de temps.
Ici on va voir comment faire un makefile complet, étapes par étapes.
Dans ce tutoriel on mentionnera plusieurs fois les headers, je pars donc du principe que vous comprennez tous·tes le principe des classes et des headers.

## LES BASES

Pour compiler `test.cpp` par exemple, une commande serait `g++ test.cpp -o TEST`.
Remarquez le paramètre `-o` suivit de `TEST`, le paramètre `-o` indique la sortie, et le `TEST` après sera donc le nom de l’exécutable. Cela permet de choisir le nom et l'extension de l'exécutable, sans ce paramètre il sera simplement nommé `a.out`.
En makefile, ça donnerait:
```make
all:
    g++ app.cpp -o TEST
```

`all:` est une règle, une sorte de recette à suivre, on peut la passer en argument de la commande `make` pour qu'elle effectue cette règle spécifiquement. Les commandes à l’intérieur doivent toujours être identées.
Si on effectue la commande `make` et `make all`, on obtiendra le même résultat car par défaut make invoque la première règle (ici all).

## LES DEPENDANCES

Imaginons maintenant qu'on a `main.cpp`, `thing.cpp` et `thing.h`, avec `thing` une classe incluse dans `main.cpp`. Le makefile deviendrait:
```make
all: main.o thing.o
    g++ main.o thing.o -o APP
main.o: main.cpp thing.h
    g++ -c main.cpp -o main.o
thing.o: thing.cpp thing.h
    g++ -c thing.cpp -o thing.o
```
Juste après avoir déclaré une règle, on peut déclarer les fichiers qu'elle va utiliser. Les `.cpp` et `.h` sont des fichiers existent, mais les `.o` sont créé pendant la compilation justement, il faut donc déclarer leur propre règle pour dire comment les construire. C'est ce qu'on fait avec `thing.o` (composé du `.cpp` et du header de la classe `thing`) et `main.o` (composé de `main.cpp`  et de la classe `thing`).

## LES VARIABLES 

Maintenant qu'on a un makefile fonctionnel, et si on l'embellissait?
```make
CC := g++
CCFLAGS := -Wall
BIN := ./TEST

all: $(BIN)

$(BIN): main.o thing.o
  $(CC) $(CCFLAGS) main.o thing.o -o $(BIN)

main.o: main.cpp thing.h
  $(CC) $(CCFLAGS) -c main.cpp -o main.o

thing.o: thing.cpp thing.h
  $(CC) $(CCFLAGS) -c thing.cpp -o thing.o
```
On a utilisé des variables pour ajouter un peu de lisibilité et de flexibilité:
- `CC` désigne le suffixe pour compiler (ici `g++`)

- `CCFLAGS` désigne les paramètres `g++` pour la compilation (ici `-Wall` permet d'avoir tout les warnings)

- `BIN` désigne le nom/le chemin vers l'executable
Notez qu'on utilise `:=` au lieu de `=` devant les variables, pourquoi? En fait, `:=` a la même signification que `const`, hors on ne va pas modifier ces variables plus tard donc on peut mettre des `:=` partout (même si des `=` fonctionnerait aussi).

On a aussi ajouté une règle intermédiaire, on dit que `all` nécessite l'exécutable puis on déclare une règle à part pour `BIN`. Pourquoi? Un projet peut avoir plusieurs exécutables, on peut donc tous les compiler chacun de leur côté et choisir lequels on veut créer specifiquement avec `all`. C'est encore plus utile quand on sait qu'on peut avoir d'autre paramètres que `all`, différents paramètres pouvant créer différents exécutables.

## LES PARAMETRES/REGLES

Plusieurs paramètres vous dites? Ça tombe bien, on va en rajouter un:
```make 
.PHONY: veryclean clean all
CC := g++
CCFLAGS := -Wall
BIN := ./TEST

all: $(BIN)

$(BIN): main.o thing.o
  $(CC) $(CCFLAGS) main.o thing.o -o $(BIN)

main.o: main.cpp thing.h
  $(CC) $(CCFLAGS) -c main.cpp -o main.o

thing.o: thing.cpp thing.h
  $(CC) $(CCFLAGS) -c thing.cpp -o thing.o
  
veryclean: clean
  rm -f ./$(BIN)

clean:
  rm -f *.o
```
On a crée les règles `clean` et `veryclean`, qui ne créent aucun exécutable et ne compilent rien du tout. Ce qu'elles font, c'est supprimer tout les fichiers objet (`.o`) ainsi que l’exécutable (`clean` ne supprime que les objets, `veryclean` fait les deux).
Au lieu d'executer la commande `make all`, on peut executer `make clean` ou `make veryclean`, ce qui nettoyeras tout les résidus des anciennes compilations, le prochain `make all` recompileras tout depuis le début. 

Evidemment, Vous pouvez créer autant de règles du genre que vous voulez et les adapter à vos besoins.
Notez le `.PHONY: veryclean clean all` au tout début. Cela permet de désigner les règles qui n'ont pas de fichiers associés (`clean` et `veryclean` ne font que détruire des fichiers, et `all` se contente d'appeler une autre règle). Comme ça, si jamais vous avez un fichier nommé `clean`, make ne refusera pas d'exécuter la règle parce que le fichier n'a pas changé.

## EXERCICE

Pour vérifier que c'est clair, imaginons maintenant qu'on ajoute les fichiers `vector.cpp` et `vector.h` correspondant à la classe `vector`, et que `thing.cpp` utilise cette classe. Que devriez vous rajouter à votre makefile? Pas d'triche hein >:c

### Solution:

<details>

  <summary>Cliquez ici</summary>

```make
.PHONY: veryclean clean all
CC := g++
CCFLAGS := -Wall
BIN := ./TEST

all: $(BIN)

$(BIN): main.o thing.o vector.o
  $(CC) $(CCFLAGS) main.o thing.o vector.o -o $(BIN)

main.o: main.cpp thing.h
  $(CC) $(CCFLAGS) -c main.cpp -o main.o

thing.o: thing.cpp thing.h vector.h       
  $(CC) $(CCFLAGS) -c thing.cpp -o thing.o

vector.o: vector.cpp vector.h
  $(CC) $(CCFLAGS) -c vector.cpp -o vector.o 

veryclean: clean
  rm -f $(BIN)

clean:
  rm -f *.o
```
On crée l'objet `vector.o` de la même manière qu'on a crée `thing.o`, et on ajoute `vector.h` dans les dépendances de `thing.o` 
</details>

## ALLER PLUS LOIN 

Pratique hein? Par contre, si on a beaucoup d'objets, ça risque d'être relou d'écrire une règle pour chacun d'entre eux, non? Ça tombe bien, voici comment y remédier pour ceux qui veulent aller plus loin:
```make
.PHONY: veryclean clean all

CC := g++
CCFLAGS := -Wall
BIN := App
SRCS := $(wildcard src/*.cpp)
OBJS := $(foreach i, $(SRCS), $(patsubst src/%.cpp, build/%.o, $(i)))

all: $(BIN)

$(BIN): $(OBJS)
    $(CC) $(CCFLAGS) @^ -o $(BIN)

$(OBJS): $(foreach i, $(OBJS), $(patsubst build/%.o, src/%.cpp, $(i)))
    $(CC) $(CCFLAGS) -c $< -o $@

veryclean: clean
  rm -f $(BIN)

clean:
  rm -f *.o
```
Vous n'y comprenez rien? C'est normal, on vas y aller par étape.
Les 4 premières lignes ne changent pas, mais on a deux nouvelles variables:
- `SRCS` désigne la liste de tout les `.cpp`. Notez qu'on ne les liste pas manuellement, à la place on les trouve automatiquement comme ceci:
```make
$(wildcard src/*.cpp)
```

Cette ligne signifie en gros "la valeur de la liste de tout les .cpp dans src/".
`wildcard` indique l'utilisation d'un "joker", ici c'est le `*` qui signifie qu'il peut y avoir n'importe quoi avant `.cpp`. 
Bien qu'on puisse utiliser `*` tel quel, dans les déclaration de  variables ils faut toujours le préceder de `wildcard`.
Vous remarquerez qu'on a aussi mis les fichiers `.cpp` et `.h` dans un nouveau dossier, `src/`. C'est pour les différentier des autres fichiers, dans un grand projet il est recommandé de créer des dossier différents pour les sources (`.cpp` et `.h`), les objets (`.o`, qui dans notre exemple irons dans `build/`), les différentes ressources comme des textures ou des sons, etc... Si vos fichiers ne sont pas rangés de la même façon, pensez évidemment à modifier le chemin pour qu'il corresponde.

- `OBJS`, c'est la même chose mais pour les objets...Quoi que, il y a une subtilité:

En effet les objets sont crée à la compilation, on ne peut donc pas dire au programme de lister tout les `.o` qu'il trouve car il n'en trouvera aucun à ce stade.
A la place, on utilise `patsubst` pour créer un pattern: Si on trouve un `.cpp` précédé de n'importe quel chaîne de caractères alors on le remplaceras par `.o` précédé de la même chaîne de caractère (c'est ce que signifie le `%`). On met le tout dans un `foreach` qui parcours `SRCS` et applique le pattern pour chaque source.

Maintenant qu'on a vu les variables, qu'est-ce qui change au niveau des règles?
Vous avez sûrement dû remarquer celle ci:
```make
$(OBJS): $(foreach i, $(OBJS), $(patsubst build/%.o, src/%.cpp, $(i)))
    $(CC) $(CCFLAGS) -c $^ -o $@
```

Regardez les `%`, si vous n'aviez pas compris leur signification avant elle est plus claire ici:
Pour n'importe quel `.o`, on nécessite le `.cpp` éponyme (ainsi que *tous* les headers avec `src/*.h`, car un code peut dépendre de plusieurs headers alors pour être sûr on les inclus tous).

Et qu'est ce que ça veut dire `$<` et `$@`?
Ce sont des tags spécifiques à make, ils y en a d'autres d'ailleurs, mais on utilise principalement ceux là:
- `$<` désigne la première source de la règle dans laquelle on est après matching (ici, `src/%.cpp`)
- `$@` désigne la règle elle même (`build/%.o`)
- `$^` désigne toutes les sources de la règle dans laquelle on est (utilisé pour `BIN` parce qu'on veut prendre en compte tous les objets)

Que fait ce makefile au final? Il trouve tout les `.cpp` et `.h` de votre projet automatiquement, construit les objets correspondants pour enfin créer un exécutable qui
contient tout les objets. Notez que c'est une façon un peu bête et méchante de faire les choses, par exemple on inclu *tout* les headers pour chaque objet alors qu'ils ne sont pas forcément tous utilisés. Il y a des méthodes pour calculer des dépendances, mais elles sont plus complexes et on ne les vera pas aujourd'hui.
