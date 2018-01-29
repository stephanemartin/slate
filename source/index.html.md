---
title: Tutorial Gradle

language_tabs: # must be one of https://git.io/vQNgJ
  - code

toc_footers:
  - <a href='https://docs.gradle.org/current/userguide/userguide.html'>Doc. Gradle</a>
  - <a href='https://docs.gradle.org/current/dsl/'>DSL Gradle </a>
  - <a href='https://docs.gradle.org/current/javadoc/'>Javadoc Gradle </a>

search: true
---

# Introduction

L'objectif est d'apprendre les concepts de base de Gradle (explications minimalistes) et ensuite de voir son intégration dans NeoLoad Web.

<aside class="warning">Notes à propos de cette doc. <br/>

Cette doc. est "responsive", ce qui suit ne concerne que ceux qui l'affichent en plein écran (et pas sur leur tel.): <br/>

- Le sommaire est à gauche.<br/>
- Le contenu - hors code - est la partie au milieu.<br/>
  &nbsp;&nbsp;-> Ce que l'on est en train de lire.<br/>
  - Et à droite se trouve le code (<b>Groovy</b> ou <b>commandes shell</b>).
</aside>

Le TP est assez procédurial, il consiste à suivre un ensemble de tâches à effectuer selon le fil conducteur suivant (correspond au sommaire):

* **2. Environnement général** 

On va voir comment exécuter Gradle en ligne de commande pour comprendre quelques concepts de base (vue haut niveau en réduisant le scope à l'échelle de l'exécutable).

-> l'idée est d'avoir une idée de l'environnement Gradle (exécutable / vue d'ensemble d'un build avec fichiers correspondants).

* **3. Système de build**

On va voir comment un "build" Gradle fonctionne. Comme précedemment, on joue avec le terminal pour s'affranchir au maximum du reste de l'écosystème (on ne parle pas de Java et on n'utilise pas d'IDE non plus).

* **4. Gradle et Java**

Ici on ajoute l'écosystème Java avec le build Gradle. Cela permet d'introduire la façon dont Java est supporté avec Gradle. On reste en mode terminal pour éviter l'intégration de l'IDE qui gère trop de trucs pour nous.

* **5. Gradle et IntelliJ**

Globalement, on va se familiariser avec le support de Gradle par IntelliJ. 

<aside class="success">
Arrivé ici, on a potentiellement une idée de notre façon de travailler avec Gradle dans notre environnement de développement.
</aside> 

* **6.Gradle et Neoload Web**

C'est la plus grosse partie du TP. Il s'agit maintenant de comprendre comment utiliser concrètement Gradle appliqué à notre environnement de développement.

<aside class="success">
Arrivé ici, on a potentiellement les infos. suffisantes pour développer dans NeoLoad Web avec Gradle.
</aside> 

# Installation

> MacOS : la commande d'install via le package manager "brew" :

```shell
# Installer la dernière version de Gradle via 'Brew' :
brew install gradle
```

> Tout OS : vérifier que Gradle est bien installé :

```shell
gradle -v
```

Gradle requiert la JDK et Groovy. Pour Groovy c'est optionel ; les distributions de Gradle embarquent Groovy. <br/><br/> 
Cliquez sur le lien pour lire la doc. officielle : [Lien officiel d'installation](https://gradle.org/install/).

# Gradle "from scratch"

## Introduction
Gradle est un outils de build générique basé sur des tâches. Contrairement à
Ant, il repose sur modèle de dépendances
([DAG](https://fr.wikipedia.org/wiki/Graphe_orient%C3%A9_acyclique)) entre ces
tâches.  Le tout contenu dans un projet. Pour définir ces tâche gradle s'appuis
sur un [DSL](https://fr.wikipedia.org/wiki/Langage_d%C3%A9di%C3%A9) reposant
sur le langage groovy. Cela rend le code beaucoup plus compact et moins
redondant.  Depuis 2016 nous pouvons aussi utiliser un autre DSL basé sur
kotlin. Pour notre introduction à gradle nous allons rester sur le DSL basé sur
groovy.

Le concept de gradle est générique, il peut compiler des projet Java mais aussi
des projets C/C++ ou javascript.

Gradle est un outil totalement compatible avec Maven, Ant et Ivy.
Cela va nous permettre de faire une migration douce.


On va maintenant - au fur et à mesure - exécuter des opérations basiques
permettant d'introduire les concepts de base de Gradle.

Ce qui suit est une version condensée et donc moins explicite du tutoriel officiel de Gradle.

Pour les sources, ci après le lien : [Gradle builds](https://guides.gradle.org/creating-new-gradle-builds/).



## "Projet"

On va créer dans un répertoire spécifique un fichier `build.gradle` (**code A.**)
> A. Pour les OS de type Unix :

```shell
mkdir mon-projet-gradle
cd mon-projet-gradle
touch build.gradle
```

Le fichier `build.gradle` est en charge de créer et configurer un **projet**.
Ce projet est le point d'entrée du système de build : on a accès via le projet
à toutes les fonctionnalités de Gradle. Par curiosité on peut aller voir la
[javadoc](https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html)
de `Project`.

## "Tâches"

Ce projet contient des sous-éléments appelés "tâches" (**tasks**). 

Par défault un projet Gradle vide contient déjà des tâches pré-définies. Pour lister les tâches d'un projet (voir **code B.**).

> B. Tout OS :

```shell
# Exécuter la tâche "tasks" du projet :
gradle tasks
```

A l'exécution de cette commande (qui lance la tâche **tasks**), l'on peut voir que, de façon générale  :

* les tâches sont groupées par "catégories" ("Build Setup tasks" et "Help tasks" pour notre projet vide)
* les tâches ont un id (un nom) et une description (pas obligatoire).

Le système de build de Gradle est basé sur ces tâches -> on crée nos proches
tâches, on les lie entre elles (dans un ordre spécifique), ce qui finalement
correspond à notre build.

<aside class="notice"> 

Pour info., il y a une tâche par défaut appellé <code>wrapper</code>.<br/> On
ne va pas en parler en detail ici. Par contre, pour culture générale, il s'agit
d'une tâche permettant de générer des scripts (pour OS Windows et Unix).
<br/>Ces scripts générés permettent d'exécuter toutes les tâches du projet sur
n'importe quelle machine, qu'elle ait Gradle d'installé ou pas (Ces scripts
embarquent en interne la version de Gradle de la machine sur laquelle ont été
généré ces scripts). <br/> Pour conclure sur ce <code>wrapper</code>,
l'objectif est de fournir des distribuables du build Gradle "cross-platform" et
"Gradle version independent". 

</aside>


## Ecrire une "tâche" - les pré-requis

Avant d'écrire notre 1ère tâche (et donc modifier `build.gradle`), il faut savoir certaines choses à propos du fichier de build :

* **Groovy**

  Dans ce fichier, on écrit en Groovy
  <br/>-> pour être plus précis, l'on écrit dans un language de type DSL basé sur Groovy (avec quelques "ajouts" pour faciliter la description du build).

* **UTF-8**

  Le fichier doit être encodé en **UTF-8**    

* **API Project**

  Dans le fichier Gradle on a accès directement à l'API [Project](https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html).
  <br/>-> Par exemple, on peut directement appeler la méthode `apply` dans le fichier.

* **API script**
  
  Le fichier `build.gradle` est converti par Gradle en une classe qui implémente [Script](https://docs.gradle.org/current/javadoc/org/gradle/api/Script.html).
  
  Cela signifie que l'on peut appeler toutes les méthodes venant de cette interface (dont le logging).
  
* **Groovy : les bases (méthodes et variables)**  
  
  L'on peut définir nos propres méthodes, variables et tâches.
  <br/>-> Une bonne pratique est d'éviter de donner des noms qui existent déjà dans l'API ([Plus de details](https://docs.gradle.org/4.5/userguide/writing_build_scripts.html#sec:project_api)).

  -Example de variable définis en Groovy : 
  
  `def maVariable = 12`
  
  -Example de méthode en Groovy :
  
  `def maMethode(param1) {println param1}`

  -> [Bases de Groovy DSL de Gradle](https://docs.gradle.org/4.5/userguide/writing_build_scripts.html#groovy-dsl-basics) 

* **Imports implicites**

Dans le fichier `build.gradle`, de nombreux imports sont implicites, permettant d'appeler l'API de Gradle sans 'full qualifier' les appels de classes, de méthodes ou de champs statiques ([Liste des imports](https://docs.gradle.org/4.5/userguide/writing_build_scripts.html#script-default-imports)).


## Ecrire une "tâche"

>Ecrivons notre première tâche dans build.gradle

```groovy
task hello {
	println 'configuration'
	doLast{
		println 'Hello world!'
	}
	doFirst{
		println 'first !'
	}
}
```

>Executer une tâche

```
gradle -q hello
```

>Liste des tâches

```
gradle tasks --all
```

Voilà c'est tout simple.

## Ecrire une "tâche" et des dépendances entre elles

> On peut ajouter des dépendances à ces tâches.


```groovy
task hello{
	doLast{
		println 'Hello world!'
	}
}
task intro(dependsOn: hello){
	doLast{
		println "I'm Gradle"
	}
}
```

> Gradle permet aussi de générer des taches.

```groovy
4.times { counter ->
    task "task$counter" {
	    println "I'm task number $counter"
    }
}
```

## Continuer une tâche ou ajouter une dépendance.

> Il est possible d'ajouter des action à la fin d'une tache ou au début.
```groovy
task hello.doLast{
	print 'Hello'
}
hello.doLast {
	println 'world!'	
}
hello.doFirst {
	println 'First !'
}
```

> Il est aussi possible d'ajouter une dépéndance après avoir définit la tâche.

```groovy
task hello {
	doLast{
		println 'Hello world!'
	}
}
task intro {
	doLast{
		println "I'm Gradle"
	}
}
intro.dependsOn hello
```

> Du coup dans la génération des tâches on peut faire comme suis:

```groovy
4.times { counter ->
    task "task$counter" {
	doLast{
            println "I'm task number $counter"
	}
    }
    if (counter > 1){
	tasks["task$counter"].dependsOn "task${counter-1}"
    }
}
```

## Rentre une tâche visible de tasks


  Maintenant que l'on connait le contexte d'écriture du fichier de build, créons des tâches sous différentes formes (voir **code D.**) :
> D. Nouveau contenu de notre 'build.gradle' :

```groovy
    description = 'Description de mon build'
    version = '1.0'
     
    def maVariableGlobale = 'valeur de ma variable globale'
    
    def maFonctionQuiAfficheParam1(param1) {

        println param1
    }
    
    task maTache1 { 

        maFonctionQuiAfficheParam1 'mon param'
    }

    task maTache2 { 

        logger.error(maVariableGlobale);
    }

    task copy(type: Copy) {
        from 'src'
        into 'dest'
    }

    maTache2.group= 'Ma catégorie de tâches'
```

Dans notre nouveau fichier `build.gradle`, nous avons défini 3 tâches :

* `maTache1`

Il s'agit d'une **tâche interne** qui appelle la fonction `maFonctionQuiAfficheParam1` avec comme paramètre en entrée la chaîne de caractères 'mon param'.

* `maTache2`

Il s'agit d'une **tâche exposée** (publique) associée à la catégorie de tâches `Ma catégorie de tâches`. Elle est exposée parce qu'elle appartient à un groupe / une catégorie.

Cette tâche log en erreur la variable globale `maVariableGlobale`. Comme dit précédemment, ce logger vient de l'interface **Script**.
([Details au sujet du logging](https://docs.gradle.org/4.5/userguide/logging.html))

* `copy`

Il s'agit d'une tâche se basant sur une tâche standard fournie par Gradle (**Copy**).
Cette tâche copie le contenu du repertoire 'src' (relatif au fichier `build.gradle`) dans le répertoire 'dest' ([Javadoc](https://docs.gradle.org/current/javadoc/org/gradle/api/tasks/Copy.html), [DSL](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Copy.html)).

Maintenant on va lister en ligne de commande ces tâches (voir **code E.**):

> E. Lister toutes les tâches incluant celles qui sont internes

```
gradle tasks --all
```

L'option `--all`permet d'afficher les tâches internes (sans groupe). L'on voit alors :

-'copy' et 'maTache1' dans 'Other tasks'

-'maTache2' dans 'Ma catégorie de tâches tasks'

On peut exécuter directement `maTache1`et `maTache2` (voir **code F.**):

> F. Exécuter 'maTache1' et 'maTache2'

```
gradle -q maTache1
gradle -q maTache2
```
Pour la tâche `copy`, créons le répertoire 'src' et un fichier arbitraire dedans.

Ensuite lançons la tâche 'copy' et vérifions que le répertoire 'dest' a été créé et le fichier provenant de 'src' copié.





<aside class="notice">
Les autres tâches déjà fournies sont disponibles via l' <a href="https://docs.gradle.org/current/javadoc/org/gradle/api/Task.html">API officielle</a>. Certaines nécessitent l'application/activation d'un <b>plugin</b> (on verra ça plus tard).
</aside>

<aside class="notice">
Il existe d'autres façons d'écrire des tâches :
<ul>
    <li><a href="https://guides.gradle.org/writing-gradle-tasks/#make_the_message_configurable">Tâche via déclaration d'une classe</a></li>
    <li><a href="https://docs.gradle.org/4.5/userguide/more_about_tasks.html#sec:defining_tasks">Syntaxes alternatives</a></li>
</ul>
</aside>

### "Propriétés"

Un projet contient des propriétés (**properties**) qui contiennent de
nombreuses informations - dont la plupart ont des valeurs par défaut - pour
permettre le build. Ces propriétés vont du nom du projet au répertoire de
destination de la génération du build.

Pour lister l'ensemble des propriétés : voir **code C.**.

> C. Tout OS :

```shell
# Exécuter la tâche "properties" du projet :
gradle properties
```

L'on peut changer la plupart de ces propriétés directement dans le fichier `build.gradle`.

Si dans le fichier `gradle.properties`, l'on ajoutait

`description = 'Description de mon projet'`<br/>
`version = '1.0'`

En lançant à nouveau la tâche **properties**, on aurait alors les propriétés `description` et `version` modifiées.


### Conclusion

On s'est familiarisé de façon minimaliste avec les notions de :

* Projet
* Tâche
* Propriété
* Groovy
* Points d'entrée de la doc avec :
  - <a href='https://docs.gradle.org/current/userguide/userguide.html'>Doc. Gradle</a>
  - <a href='https://docs.gradle.org/current/dsl/'>DSL Gradle </a>
  - <a href='https://docs.gradle.org/current/javadoc/'>Javadoc Gradle </a>


Il nous faut maintenant comprendre le système de build.

# Système de build

TODO

https://docs.gradle.org/4.4.1/userguide/build_lifecycle.html

Pour doc. exhaustive à propos des tâches + ordering des tâches
https://docs.gradle.org/current/dsl/org.gradle.api.Task.html



https://docs.gradle.org/current/userguide/plugins.html
https://docs.gradle.org/current/userguide/standard_plugins.html#sec:base_plugins

depuis la communauté:

https://plugins.gradle.org/



# Gradle et Java

https://docs.gradle.org/4.4.1/userguide/tutorial_java_projects.html
https://guides.gradle.org/building-java-applications/

# Gradle et IntelliJ

# Gradle et NeoLoad Web





























> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")


Welcome to the Kittn API! You can use our API to access Kittn API endpoints, which can get information on various cats, kittens, and breeds in our database.

We have language bindings in Shell, Ruby, and Python! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

This example API documentation page was created with [Slate](https://github.com/lord/slate). Feel free to edit it and use it as a base for your own API's documentation.

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```


## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete
