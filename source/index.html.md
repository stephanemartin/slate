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

Ce qui suit est une version condensée et donc moins explicite du tutoriel
officiel de Gradle.

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

Par défault un projet Gradle vide contient déjà des tâches pré-définies. Pour
lister les tâches d'un projet (voir **code B.**).

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

Dans le fichier `build.gradle`, de nombreux imports sont implicites, permettant
d'appeler l'API de Gradle sans 'full qualifier' les appels de classes, de
méthodes ou de champs statiques 
([Liste des imports](https://docs.gradle.org/4.5/userguide/writing_build_scripts.html#script-default-imports)).


## Ecrire une "tâche"

>Ecrivons notre première tâche dans build.gradle

```groovy
task hello {
	println '==> configuration'
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

On constante que le mot configuration est affiché même si on n'appelle pas la
tâche hello. Même lorsqu'on demande la liste des tâches.
D'où l'importance du doFirst ou doLast

<aside class="notice">

Les autres tâches déjà fournies sont disponibles via l' <a
href="https://docs.gradle.org/current/javadoc/org/gradle/api/Task.html">API
officielle</a>. Certaines nécessitent l'application/activation d'un
<b>plugin</b> (on verra ça plus tard).

</aside>


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
task hello{
        doLast{
                print 'Hello '
        }
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

## Logging de gradle

Gradle possède un logger. Ce logger vient de l'interface **Script**.  ([Details au sujet du
logging](https://docs.gradle.org/4.5/userguide/logging.html))

```groovy
task run {
	doFirst{
		logger.quiet('An info log message which is always logged.')
		logger.error('An error log message.')
		logger.warn('A warning log message.')
		logger.lifecycle('A lifecycle info log message.')
		logger.info('An info log message.')
		logger.debug('A debug log message.')
		logger.trace('A trace log message.')
	}
}
```

Vous pouvez le régler avec différents flags en ligne de commandes : 
no logging options

Option | Description
--------- | -----------
no logging options | LIFECYCLE and higher
-q or --quiet | QUIET and higher
-w or --warn | WARN and higher
-i or --info | INFO and higher
-d or --debug | DEBUG and higher (that is, all log messages)

## Rentre une tâche visible de tasks


  Maintenant que l'on connait le contexte d'écriture du fichier de build, créons des tâches sous différentes formes (voir **code D.**) :
> Nouveau contenu de notre 'build.gradle' :

```groovy
task maTache1 { 
	doLast{
		logger.warn 'je suis caché'
	}	
}

task maTache2 { 
	doLast{
		logger.warn 'je suis visible'
	}
	group= 'Ma catégorie de tâches'
}
```

Dans notre nouveau fichier `build.gradle`, nous avons défini 2 tâches :

* `maTache1`

Il s'agit d'une **tâche interne** qui appelle la fonction
`maFonctionQuiAfficheParam1` avec comme paramètre en entrée la chaîne de
caractères 'mon param'.

* `maTache2`

Il s'agit d'une **tâche exposée** (publique) associée à la catégorie de tâches
`Ma catégorie de tâches`. Elle est exposée parce qu'elle appartient à un groupe
/ une catégorie.


## Differentes types de tâches


* `copy`

Il s'agit d'une tâche d'un type de tâche facilitant la Copie de fichiers.  
Cette tâche copie le contenu du repertoire 'src' (relatif au
fichier `build.gradle`) dans le répertoire 'dest'
***exclude*** est optionnel et permet de filtrer les fichers indésirable.
Il y a aussi ***include*** ***expand*** etc...
[Pour plus d'info](https://docs.gradle.org/current/userguide/working_with_files.html)

([Javadoc](https://docs.gradle.org/current/javadoc/org/gradle/api/tasks/Copy.html),
[DSL](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Copy.html)).


> copy task
```groovy
task copyTask(type: Copy) {
    from 'src'
    into 'dst'
    exclude '**/*staging*'
}
```
* `Zip`

Il s'agit d'une tâche d'un type de tâche facilitant la fabrication d'archive zip.
Même principe que copy. Il expose différentes méthodes permetant de faire une archive zip.

[DSL](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.bundling.Zip.html)



> Zip task

```groovy
apply plugin: 'java'

task zip(type: Zip) {
    from 'src/dist'
    into('libs') {
        from configurations.runtime
    }
}
```

```groovy
task myZip(type: Zip) {
    from 'somedir'
}

println myZip.archiveName
println relativePath(myZip.destinationDir)
println relativePath(myZip.archivePath)
```


* `Jar`

Il s'agit d'une tâche d'un type de tâche facilitant la fabrication de jar.
Dans notre exemple nous faisons un fatjar. Mais il est possible de faire autre chose.

Dans cet exemple nous ajoutons le classifier all
Puis on ajoute dans le manifest les informations nécessaires au manifest du jar.
***from*** nous permet de definir les fichier qu'on va inclure dans le jar.
***with*** indique qu'on sera un jar.

[DSL](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.bundling.Jar.html)


> Jar task

```groovy
task fatJar(type: Jar) {
    classifier 'all'
    manifest {
        attributes 'Implementation-Title': 'Kerberos Tester',
                'Implementation-Version': version,
                'Main-Class': 'com.neotys.tester.kerberos.KerberosConfigurationTester'
    }
    from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } }
    with jar
}
```




<aside class="notice">
Il existe d'autres façons d'écrire des tâches :
<ul>
    <li><a href="https://guides.gradle.org/writing-gradle-tasks/#make_the_message_configurable">Tâche via déclaration d'une classe</a></li>
    <li><a href="https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:defining_tasks">Syntaxes alternatives</a></li>
</ul>
</aside>

## "Propriétés"

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

> exemple d'un gradle.properties 

```
group=com.neotys.web
version=51.1.7-SNAPSHOT
name=neotys-web-foundation
description=description (optionel)
```

En lançant à nouveau la tâche **properties**, on aurait alors les propriétés `description` et `version` modifiées.

## Sous projects (modules/reactor en Maven)

### Les sous modules

Gradle support les modules comme maven. C'est à dire une configuration à la
racine du projet et inclure différents projets dans des sous répertoires.
Pour cela il faut juste les déclarer comme dans maven (***<module>***).
La déclaration dans gradle se fait dans settings.gradle

> Exemple de declaration

> settings.gradle

```groovy
include 'otherModules'
```

> build.gradle vide


> otherModules/build.gradle

```groovy
task hello{
        doLast{
                logger.warn "I'm submodule"
        }
}
```

Faire un gradle hello à la racine. Le nom du module est le nom du répertoire.
On peut changer le nom  du module dans le settings.gradle.

> Pour inclure tout les sous répertoires de premier niveau settings.gradle

```groovy
rootDir.eachDir {
	File subGradle = new File(it, "build.gradle")
	if (subGradle.exists()) {
		include it.name
	}
}
```

> settings.gradle

```groovy
include ':awesomeModule'
project(':awesomeModule').projectDir = "$rootDir/otherModule" as File
```

Relancer ***gradle hello*** vous verez un autre nom de module.

Essayer d'adapter la bouble for pour mettre un prefix à chaque nom de sous modules.


### Modifier tout les projet ou sous modules
Il est possible d'affecter toutes taches de tout les modules et de tout les projets.
Par exemple si on début on crée les tâches hello. Du coup il faut enlever le mot ***task*** dans le otherProject.

> build.gradle à la racine

```groovy
allprojects {
    task hello {
        doLast { task ->
            logger.warn "I'm $task.path"
        }
    }
}
subprojects {
    hello {
        doLast {
            logger.warn "- I depend on root"
        }
    }
}

hello{
        doFirst{
                logger.warn "I'm root"
        }
}
```




## Conclusion

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

## Lifecycle
Vous avez pu entrevoir qu'il y avait plusieurs étapes durant un lancement.

<ul>
<li>Initialization : Recherche des projet qui seront compilé</li>
<li>Configuration : instanciation des objets et génère les taches</li>
<li>Execution : executes les différentes tâches</li>
</ul>

in settings.gradle

```groovy
println 'This is executed during the initialization phase.'
```

in build.gradle

```groovy
println 'This is executed during the configuration phase.'

task configured {
    println 'This is also executed during the configuration phase.'
}

task test {
    doLast {
        println 'This is executed during the execution phase.'
    }
}

task testBoth {
    doFirst {
      println 'This is executed first during the execution phase.'
    }
    doLast {
      println 'This is executed last during the execution phase.'
    }
    println 'This is executed during the configuration phase as well.'
}
```

```shell
> gradle test testBoth
This is executed during the initialization phase.

> Configure project :
This is executed during the configuration phase.
This is also executed during the configuration phase.
This is executed during the configuration phase as well.

> Task :test
This is executed during the execution phase.

> Task :testBoth
This is executed first during the execution phase.
This is executed last during the execution phase.

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

## Plugins

Il existe beaucoup de plugins pour gradle. Vous pouvez en ajouter.
Souvent ces plugin sont écrit en gradle ou java.

Un petit tutoriel ce trouve ici.
https://blog.betomorrow.com/mon-premier-plugin-gradle-5bce748558b5
 
Un peut de doc:
https://docs.gradle.org/current/userguide/plugins.html
https://docs.gradle.org/current/userguide/standard_plugins.html#sec:base_plugins

https://plugins.gradle.org/


TODO

https://docs.gradle.org/current/userguide/build_lifecycle.html

Pour doc. exhaustive à propos des tâches + ordering des tâches
https://docs.gradle.org/current/dsl/org.gradle.api.Task.html



https://docs.gradle.org/current/userguide/plugins.html
https://docs.gradle.org/current/userguide/standard_plugins.html#sec:base_plugins

depuis la communauté:

https://plugins.gradle.org/



# Gradle et Java
Gradle possède un plugin java qui se calque sur la structure maven.
Se plugin se veut compatible avec les projets mavent. C'est ce plugin qu'on va utiliser pour nos projet.

Pour l'utiliser il suffit de mettre au debut de votre build.gradle :

```groovy
apply plugin: 'java'
```

des nouvelles taches apparaissent.

```shell
> gradle build
> Task :compileJava
> Task :processResources
> Task :classes
> Task :jar
> Task :assemble
> Task :compileTestJava
> Task :processTestResources
> Task :testClasses
> Task :test
> Task :check
> Task :build

BUILD SUCCESSFUL in 0s
6 actionable tasks: 6 executed
```

Nous pouvons utiliser mavenCentral

```groovy
repositories {
    mavenCentral()
}
```

Nous pouvons utiliser les dépendences:

```groovy
dependencies {
    compile group: 'commons-collections', name: 'commons-collections', version: '3.2.2'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

Essayez de faire votre propre projet avec IntelliJ.

Vous verez que le code gradle est beaucoup plus simple et court.


https://docs.gradle.org/current/userguide/tutorial_java_projects.htm
https://guides.gradle.org/building-java-applications/

## Gradle et IDE
Gradle est compatible avec IntelliJ et Eclipse


https://www.jetbrains.com/help/idea/gradle.html
https://www.jhipster.tech/configuring-ide-eclipse-gradle/


# Gradle et NeoLoad Web
Nous avons configuré un maximum de choses pour vous.
Nous avons aussi un projet type : neotys-web-foundation-gradle.


Pour cela nous nous somme calqué sur notre structure de projet maven. 
Ainsi les artefacts sont déclaré dans le build.gradle dans une map.

Nous avons fait un super gradle qui sont chargé comme suit:

```groovy
buildscript {scriptHandler->
	apply from: 'http://webstorage/gradle/super-gradle/develop/buildscript.gradle', to: scriptHandler
}
apply from: 'http://webstorage/gradle/super-gradle/develop/build.gradle'
```

Qui sera peut être transféré dans nexus.
Vous pouvez regarder ces fichier cela vous donnera une idée de gradle.


Les modules du projet sont décrit settings.gradle.
Nous pouvons faire en sorte de faire découvrir à gradle tout les sous projets.
Ce qui rend la configuration plus facile.

```groovy
rootDir.eachDir {
    File subGradle = new File(it, "build.gradle")
    if (subGradle.exists()) {
        String subName = ":neotys-web-foundation-${it.name}"
        include subName
        project(subName).projectDir = it as File
    }
}
```

Dans le fichier gradle.properties dans le projet il y a les coordonnée du projet.

```groovy
group=com.neotys.web
version=51.1.7-SNAPSHOT
name=neotys-web-foundation
```


## Configuration Nexus
<aside class="notice"> 
Pour configurer les credentials de nexus utiliser le fichier gradle.properties dans ~/.gradle ou C:\Users\userName\.gradle
</aside>

```groovy
nexusUrl=http://nexus.intranet.neotys.com
nexusUsername=XXX
nexusPassword=YYY
```


































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
