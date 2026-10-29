> [!NOTE]
> **Guide participant**
> Guide à suivre pendant la séance ou à rejouer seul ensuite. Il présente les mécanismes dans l'ordre de la formation, porte les manipulations et donne le corrigé de chaque TP à la fin de son chapitre.

# Avant de commencer

À lire une fois, avant le premier TP : ce qu'il faut sur le poste et comment s'organisent les projets de la formation.

## Ce qu'il faut sur le poste

- Java 17 ou supérieur. Vérifier :

```bash
java -version
```

- `curl` et un terminal Bash ; sous Windows, Git Bash ou WSL. Dans Windows PowerShell, `curl` désigne une autre commande, `Invoke-WebRequest`, qui n'accepte pas les mêmes options : taper `curl.exe`, ou utiliser Git Bash ([curl on Windows](https://learn.microsoft.com/en-us/windows/curl/)).
- Un IDE Java, pour lancer les programmes qui ont un `main` et les tests.
- Docker, seulement à partir du TP « MongoDB réactif ».

Un client HTTP graphique est facultatif. Il permet de suivre sans terminal les réponses qui arrivent au fil de l'eau, comme un `Flux` ou des Server-Sent Events. Les commandes du Guide restent écrites avec `curl`.

| Client | Réponses en flux (Server-Sent Events) | Documentation |
|---|---|---|
| HTTP Client d'IntelliJ IDEA, fichiers `.http` | événements affichés à mesure qu'ils arrivent ; HTTP/2 pris en charge | [HTTP Client](https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html) |
| Postman | une requête HTTP ordinaire suffit | [Support for server-sent events](https://blog.postman.com/support-for-server-sent-events/) |
| Insomnia | requête de type event stream | [Requests in Insomnia](https://developer.konghq.com/insomnia/requests/) |

Maven n'est pas à installer : chaque projet a son lanceur `mvnw`, qui télécharge Maven au premier lancement. Dans tout le Guide, une commande écrite `mvn …` se lance `./mvnw …` depuis le dossier du projet concerné, ou `mvnw.cmd …` sous Windows.

## Les projets

Chaque exemple de la formation est un projet Maven, dans son propre dossier. Tous se rangent côte à côte dans un même dossier de travail.

Tous les noms de projet commencent par `demo-`, quel que soit le TP qui les utilise : le préfixe signale seulement un exemple de la formation. Chaque chapitre « TP » du Guide est une manipulation à faire soi-même : certains font écrire le code à partir d'un énoncé, d'autres font lancer un code fourni pour observer un mécanisme ciblé.

Il y a deux sortes de projets :

- **un projet progressif**, `demo-application-reactive` : créé au TP « Projet Spring Boot 4 avec WebFlux », il s'enrichit au fil des sept TP qui construisent l'application. Chacun part de l'état laissé par le précédent, d'où l'ordre à respecter ;
- **un projet autonome par TP autonome** : il ne dépend d'aucun autre projet et sert à un seul TP.

Deux TP autonomes utilisent aussi le projet progressif : « Serveur bloquant et non bloquant » compare `demo-serveur-bloquant`, en Spring MVC, avec `demo-application-reactive`, en WebFlux ; « Multiplexage HTTP2 » active HTTP2 dans `demo-application-reactive` et réutilise son contrôleur `AttenteReactive`.

Chaque section indique le dossier à ouvrir, ce qu'il faut avoir lancé avant, et la commande à lancer.

| Projet | Qui le construit | Sert à |
|---|---|---|
| `demo-application-reactive` | le participant, pas à pas | les sept TP qui construisent l'application, dans l'ordre ; la partie WebFlux du TP « Serveur bloquant et non bloquant » ; le TP « Multiplexage HTTP2 » |
| `demo-serveur-bloquant` | le participant, dans le TP « Serveur bloquant et non bloquant » | comparer Spring MVC avec WebFlux dans `demo-application-reactive` |
| les autres projets `demo-…` | fournis prêts à lancer | le TP de même nom |

| Dossier | TP |
|---|---|
| `demo-serveur-bloquant` | Serveur bloquant et non bloquant, partie Spring MVC |
| `demo-list-flux` | List et Flux |
| `demo-mvc` | Mono dans Spring MVC |
| `demo-webclient` | WebClient et service de tarifs |
| `demo-backpressure` | Backpressure |
| `demo-sse` | Server-Sent Events |
| `demo-websocket` | WebSocket |
| `demo-mongodb` | MongoDB réactif |
| `demo-mongo-template` | Requêtes avec ReactiveMongoTemplate |
| `demo-tailable` | Curseur tailable |
| `demo-securite-webflux` | Sécurité WebFlux |
| `demo-utilisateurs-mongodb` | Utilisateurs en base |
| `demo-securite-methodes` | Sécurité des méthodes |
| `demo-stepverifier` | StepVerifier |
| `demo-contextes-test` | Contextes de test |
| `demo-webtestclient` | WebTestClient |

# TP - Projet Spring Boot 4 avec WebFlux

## Créer demo-application-reactive

Ce projet est construit pas à pas par les TP. Le générer avec le formulaire de [Spring Initializr](https://start.spring.io) :

| Champ              | Valeur                                        |
| ------------------ | --------------------------------------------- |
| Project build tool | Maven                                         |
| Language           | Java                                          |
| Spring Boot        | dernière version stable (pré-sélectionnée)    |
| Group              | `com.example` (valeur proposée)               |
| Artifact           | `demo` (valeur proposée)                      |
| Name               | `demo` (valeur proposée)                      |
| Package name       | `com.example.demo` (valeur proposée)          |
| Packaging          | Jar                                           |
| Configuration      | Properties                                    |
| Java               | 17                                            |
| Dependencies       | *Add dependencies…* « Spring Reactive Web »   |

Ne pas ajouter « Spring Web » ! Spring Boot démarre Spring MVC sur Tomcat, pas WebFlux.

Cliquer *Generate*, enregistrer l'archive sous le nom `demo-application-reactive.zip`, puis la décompresser dans un dossier `demo-application-reactive` : l'archive n'a pas de dossier parent.


Sans navigateur, cette commande génère le même projet depuis le dossier de travail :

```bash
curl https://start.spring.io/starter.zip -d type=maven-project -d javaVersion=17 -d groupId=com.example -d artifactId=demo -d name=demo -d packageName=com.example.demo -d dependencies=webflux -o demo-application-reactive.zip
```

```bash
unzip demo-application-reactive.zip -d demo-application-reactive
```

Le projet généré :

```text
demo-application-reactive/
├── mvnw
├── mvnw.cmd
├── .mvn/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/example/demo/DemoApplication.java
    │   └── resources/application.properties
    └── test/java/com/example/demo/DemoApplicationTests.java
```

`pom.xml` déclare le starter WebFlux et son starter de test. Les TP y ajoutent d'autres dépendances, chacun au moment où il en a besoin. `DemoApplication` est la classe qui démarre Spring.

Chaque classe se crée dans `src/main/java/com/example/demo/`, chaque classe de test dans `src/test/java/com/example/demo/`, et chaque fichier commence par `package com.example.demo;` : les blocs de code du Guide pour ce projet omettent cette ligne.

### Démarrer le projet

Se placer dans le dossier `demo-application-reactive` :

```bash
cd demo-application-reactive
```

Puis :

```bash
./mvnw spring-boot:run
```

Attendu, dans le log :

```text
Netty started on port 8080 (http)
Started DemoApplication in 1.321 seconds
```

La durée varie. Si le log annonce Tomcat, « Spring Web » a été ajouté au formulaire : regénérer le projet. Arrêter le serveur par Ctrl-C. Le premier lancement télécharge Maven et les dépendances : le faire avant la séance.

### Un premier contrôleur

Créer `src/main/java/com/example/demo/SalutationController.java` :

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SalutationController {
    @GetMapping("/salutation")
    public String salutation() { return "Bonjour"; }
}
```

Redémarrer le projet, puis, dans un second terminal :

```bash
curl http://localhost:8080/salutation
```

```text
Bonjour
```

Arrêter le serveur.

# TP - Serveur bloquant et non bloquant

## Les trois modèles pendant l’attente

Une requête arrive du client. L’application appelle la base pour répondre, et la base met une seconde. La seconde mesure une attente d’entrée-sortie, pas une seconde de calcul CPU.

Un thread est un fil d’exécution, c’est lui qui déroule les instructions d’une requête. Ce qui distingue les trois modèles, c’est le thread qui reste occupé pendant l’attente.

### MVC avec threads classiques

Spring MVC traite la requête sur un thread classique lié à un thread du système. Le serveur les réutilise depuis un pool. Si le code attend un appel bloquant à la base, le thread reste occupé par cette requête jusqu’à la réponse, puis continue à la ligne suivante.

Dans l’analogie du restaurant, une table représente une requête, un employé de salle un thread, et la cuisine la base. L’employé transmet la commande et attend le plat devant la cuisine. Si les quatre employés attendent, une autre table doit patienter.

### WebFlux non bloquant

L’employé transmet la commande et retourne en salle. Lorsque le plat est prêt, la cuisine le signale ; un employé disponible le porte. Le signal représente une notification, pas un employé supplémentaire.

Dans le serveur, un appel non bloquant rend la main au thread pendant l’attente. Le code décrit la suite à exécuter lorsque le résultat arrive. Une boucle d’événements permet à un petit groupe de threads de traiter les requêtes et événements réseau. La suite peut être exécutée par un autre thread que celui qui a reçu la requête.

Ce modèle demande de composer le traitement avec des API non bloquantes. Un appel bloquant sur un thread de la boucle d’événements l’immobilise et retarde les autres tâches. Le bénéfice dépend donc aussi des bibliothèques utilisées.

### MVC avec threads virtuels

Peut-on conserver le code séquentiel « appeler, attendre, continuer » et libérer le thread système pendant certaines attentes ? Les threads virtuels, disponibles depuis Java 21, le permettent quand Java peut suspendre cette attente.

Un thread virtuel est géré par Java. Pour exécuter ses instructions, Java le monte sur un thread de plateforme appelé porteur, lié à un thread système. Pendant une entrée-sortie compatible, Java peut suspendre le thread virtuel et libérer le porteur. Un autre thread virtuel peut alors s’exécuter sur ce thread système. Quand la réponse arrive, le premier thread virtuel reprend son code, éventuellement sur un autre porteur.

| | Avant l’attente | Pendant une attente compatible | À la reprise |
|---|---|---|---|
| Thread virtuel A | Exécute son code | Suspendu | Continue son code |
| Thread système | Exécute A | Peut exécuter B | Un thread système exécute A |

Le code de A reste séquentiel, son appel reste bloquant. Certaines opérations peuvent retenir le porteur : cette disponibilité n’est pas garantie pour toutes les attentes. Les connexions à la base et sa capacité restent également limitées.

Source : [Java SE 25, threads virtuels](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html).

## Prédire avant de mesurer

Dix requêtes arrivent ensemble. Chacune attend une seconde ; le calcul est négligeable, les attentes sont indépendantes, et aucun service distant ne limite leur concurrence.

- MVC classique dispose de quatre threads de traitement.
- WebFlux dispose de quatre workers HTTP, les threads qui traitent les événements réseau.
- Le TP simule l’attente : aucune base réelle ni pool de connexions.

Prédire le temps nécessaire pour terminer le lot et expliquer quand la cinquième requête commence son attente. Attendu : environ trois secondes en MVC, par vagues de quatre, quatre et deux ; environ une seconde en WebFlux, les attentes se recouvrent. Faire répondre avant de lancer les mesures.

## Préparer les deux mesures

Créer seulement le projet MVC `demo-serveur-bloquant`, à côté de `demo-application-reactive`. Pour WebFlux, réutiliser `demo-application-reactive`, créé au TP « Projet Spring Boot 4 avec WebFlux ». Les deux répondent à `/commande` sur le port 8080 : lancer MVC, mesurer, l’arrêter, puis recommencer avec WebFlux.

Deux terminaux suffisent : le premier se place dans le projet à démarrer, où se trouve `pom.xml` ; le second reste dans le dossier de travail commun aux projets et sert à envoyer les requêtes.

## Créer le projet Spring MVC

Dans [Spring Initializr](https://start.spring.io), saisir :

| Champ | Valeur |
|---|---|
| Project build tool | Maven |
| Language | Java |
| Spring Boot | dernière version stable proposée |
| Group | `com.example` |
| Artifact | `demo-serveur-bloquant` |
| Name | `demo` |
| Package name | `com.example.demo` |
| Packaging | Jar |
| Configuration | Properties |
| Java | 17 |
| Dependencies | « Spring Web » uniquement |

Cliquer *Generate* et enregistrer l’archive sous le nom `demo-serveur-bloquant.zip`.

**Depuis le terminal**, cette commande génère le même projet. La lancer depuis le dossier de travail qui contient `demo-application-reactive` :

```bash
curl --fail --show-error https://start.spring.io/starter.zip -d type=maven-project -d javaVersion=17 -d groupId=com.example -d artifactId=demo-serveur-bloquant -d name=demo -d packageName=com.example.demo -d dependencies=web -o demo-serveur-bloquant.zip
```

Choisir le formulaire ou la commande : une seule génération suffit. Si l’archive existe, passer directement à l’extraction :

```bash
unzip demo-serveur-bloquant.zip -d demo-serveur-bloquant
```

Dans `pom.xml`, le starter serveur est `spring-boot-starter-webmvc`. Créer `src/main/java/com/example/demo/AttenteBloquante.java` :

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AttenteBloquante {
    @GetMapping("/commande")
    public String commande() throws InterruptedException {
        System.out.println("requête reçue par " + Thread.currentThread().getName());
        Thread.sleep(1000);
        return "plat servi";
    }
}
```

Dans ce code, l’annotation associe la méthode à `GET /commande`. La ligne du log montre le nom du thread qui reçoit la requête. `Thread.sleep(1000)` le suspend pendant une seconde : c’est le comportement bloquant simulé ici. La réponse arrive après.

## Créer le script d’envoi des requêtes

Pour envoyer dix requêtes simultanées, créer `envoyer-requetes-paralleles.sh` dans le dossier de travail commun, à côté des deux projets :

```text
dossier-de-travail/
├── envoyer-requetes-paralleles.sh
├── demo-serveur-bloquant/
└── demo-application-reactive/
```

```bash
#!/usr/bin/env bash
set -u
url=${1:?usage : bash envoyer-requetes-paralleles.sh URL NOMBRE}
nombre=${2:?usage : bash envoyer-requetes-paralleles.sh URL NOMBRE}
debut=$SECONDS
pids=()
for ((i=0; i<nombre; i++)); do
    curl --fail --silent --show-error --max-time 15 "$url" > /dev/null &
    pids+=("$!")
done
echec=0
for pid in "${pids[@]}"; do
    wait "$pid" || echec=1
done
printf '%s requêtes terminées en environ %s s\n' "$nombre" "$((SECONDS-debut))"
exit "$echec"
```

Le script envoie les requêtes en parallèle, attend leur fin et signale un échec. Il utilise Bash et `curl`, installés au TP « Projet Spring Boot 4 avec WebFlux ».

## Mesurer Spring MVC

Démarrer MVC avec quatre threads :

```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--server.tomcat.threads.max=4"
```

Attendre le message de démarrage de Tomcat. Dans le second terminal, envoyer dix requêtes :

```bash
time bash envoyer-requetes-paralleles.sh http://localhost:8080/commande 10
```

Ce qu'on doit observer :

| Observation | Attendu |
|---|---|
| Durée du lot | environ 3 s (3,1 s relevées en préparation) |
| Threads de traitement | quatre noms, réutilisés pour quatre, quatre, puis deux requêtes |

Le script affiche les secondes entières ; `time` donne une mesure plus précise. Arrêter MVC avec `Ctrl-C` avant de continuer.

## Préparer WebFlux dans demo-application-reactive

Ouvrir le projet créé au TP « Projet Spring Boot 4 avec WebFlux » avec le starter WebFlux. Conserver son contrôleur de salutation. Créer `src/main/java/com/example/demo/AttenteReactive.java` :

```java
package com.example.demo;

import java.time.Duration;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
public class AttenteReactive {
    @GetMapping("/commande")
    public Mono<String> commande() {
        System.out.println("requête reçue par " + Thread.currentThread().getName());
        return Mono.delay(Duration.ofSeconds(1)).map(fin -> "plat servi");
    }
}
```

Les annotations et l’URL gardent le même rôle que dans MVC. `Mono<String>` représente le résultat qui arrivera plus tard. `Mono.delay` programme un signal après une seconde ; il ne bloque pas le worker HTTP pendant cette attente. `map` décrit le texte renvoyé à l’arrivée du signal. Ces opérateurs seront étudiés au chapitre « Mono et Flux ».

Le minuteur utilise par défaut le scheduler `parallel` de Reactor. Les workers HTTP observés dans le log ne sont donc pas tous les threads de l’application.

## Mesurer WebFlux

La propriété système `reactor.netty.ioWorkerCount=4` règle à quatre les workers HTTP de Reactor Netty. Maven exécute l’application dans une JVM séparée : transmettre la propriété avec `spring-boot.run.jvmArguments`.

```bash
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Dreactor.netty.ioWorkerCount=4"
```

Cette propriété fixe le nombre de workers HTTP, pas tous les threads utilisés par l’application. Source : [Reactor Netty 1.3.7, LoopResources](https://github.com/reactor/reactor-netty/blob/v1.3.7/reactor-netty-core/src/main/java/reactor/netty/resources/LoopResources.java) et [Spring Boot Maven Plugin, arguments JVM](https://docs.spring.io/spring-boot/maven-plugin/run.html).

Attendre le message de démarrage de Netty, puis envoyer le même lot :

```bash
time bash envoyer-requetes-paralleles.sh http://localhost:8080/commande 10
```

Ce qu'on doit observer :

| Observation | Attendu |
|---|---|
| Durée du lot | environ 1 s, plus les coûts de démarrage et d’envoi (1,2 s relevées en préparation) |
| Workers HTTP | quatre noms ; la répartition des dix requêtes varie (deux, trois, trois et deux en préparation) |

Arrêter WebFlux avec `Ctrl-C`.

## Interpréter les mesures

Chronologie simplifiée, seuls les temps d’attente sont représentés :

| Modèle | 0 à 1 s | 1 à 2 s | 2 à 3 s |
|---|---|---|---|
| MVC classique | 4 requêtes | 4 suivantes | 2 dernières |
| WebFlux | 10 requêtes | Lot terminé | Lot terminé |

Chaque requête attend une seconde dans la simulation. Le lot MVC prend environ trois secondes, car les requêtes patientent aussi jusqu’à ce qu’un thread se libère. WebFlux laisse les dix attentes se recouvrir. Le temps de réponse d’une requête et la durée totale du lot sont deux mesures différentes.

Mesures de préparation le 27 octobre 2026 : MVC classique, 3,119 s ; WebFlux, 1,163 s. Les deux mesures utilisent quatre threads de traitement, dix requêtes indépendantes et une attente simulée. Elles montrent le comportement de cette simulation, pas une comparaison de performances universelle.

## Comparer les trois modèles

Même situation : une requête attend la base.

| | MVC classique | WebFlux | MVC avec threads virtuels |
|---|---|---|---|
| Pendant l’attente | Le thread reste bloqué sur la requête. | Le worker HTTP reste disponible. | Le thread virtuel est suspendu ; son thread système peut exécuter autre chose pour une attente compatible. |
| À l’arrivée de la réponse | Le thread poursuit son code. | Un thread exécute la suite programmée. | Le thread virtuel reprend son code. |
| Style de code | Séquentiel, appels bloquants. | Composition avec `Mono` et `Flux`. | Séquentiel, appels bloquants. |
| Nombreuses attentes | Les threads système du pool sont immobilisés. | Quelques workers traitent de nombreuses requêtes en attente. | De nombreux threads virtuels peuvent se partager moins de threads système. |
| Vigilance | Le pool peut être saturé. | Un appel bloquant peut immobiliser un worker. | Les connexions et la capacité de la base restent limitées. |

Ces points de vigilance ne sont pas exclusifs. Les connexions et la capacité des services distants limitent les trois modèles ; certaines opérations peuvent aussi retenir le porteur d’un thread virtuel. Les threads virtuels ne sont pas mesurés dans ce TP.

## Choisir un modèle

| Situation | Première option | Pourquoi ? |
|---|---|---|
| L’application MVC tient la charge et ses performances conviennent. | Conserver MVC classique. | Le modèle actuel répond au besoin. |
| Beaucoup de requêtes attendent des services distants, avec des bibliothèques bloquantes et du code séquentiel. | Évaluer MVC avec threads virtuels. | Conserver ce style de code en réduisant les threads système immobilisés pendant les attentes compatibles. |
| Les flux asynchrones doivent être composés et leur demande régulée ; les dépendances sont non bloquantes. | Évaluer WebFlux. | Reactor permet de composer ces traitements et la backpressure régule la demande entre composants compatibles. |

Exemples : une application de gestion interne dont la charge est absorbée ; une API qui attend de nombreux services ; une agrégation de flux d’événements diffusés progressivement.

Ce sont des orientations, pas des règles automatiques. Examiner les bibliothèques, la charge, les compétences de l’équipe et le facteur qui limite l’application. Le calcul CPU n’est pas accéléré par le seul changement de modèle d’attente.

## Exercice : choisir pour une application

Pour chaque situation, proposer une première option et dire ce qu’il faudrait vérifier avant de la retenir.

- **A. Réservations** : MVC respecte les objectifs, y compris aux pointes de charge.
- **B. Suivi de colis** : de nombreux appels bloquants aux transporteurs saturent le pool de threads ; les services distants ont de la capacité.
- **C. Capteurs** : des sources non bloquantes émettent des événements à combiner et la demande doit être régulée ; l’équipe maîtrise Reactor.

Variation : l’application passe l’essentiel de son temps à calculer une image. Quel bénéfice attendre du seul changement de modèle ?

> [!TIP]
> **Ce qu’il faut retenir**
> Chaque attente dure une seconde ; le nombre de threads disponibles change la durée totale du lot. MVC classique immobilise un thread. WebFlux libère ses workers HTTP. Un thread virtuel suspendu peut libérer son thread système. Le calcul, la base et les connexions restent des limites.

Source : [Spring Framework, concurrence WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux/new-framework.html#webflux-concurrency-model) ; [Java SE 25, threads virtuels](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html).

## Corrigé - Choisir pour une application


| Cas | Première option | Raisonnement |
|---|---|---|
| A. Réservations | Conserver MVC classique. | Les objectifs sont déjà satisfaits ; aucun gain de migration n’est établi. |
| B. Suivi de colis | Évaluer MVC avec threads virtuels. | Le pool est saturé par des appels bloquants et les services distants ont de la capacité. Vérifier que les attentes peuvent suspendre les threads virtuels, puis mesurer sous charge et surveiller les limites des connexions. |
| C. Capteurs | Évaluer WebFlux. | Les sources sont non bloquantes, les flux doivent être combinés et l’équipe connaît Reactor. Vérifier que les producteurs peuvent respecter la demande et mesurer sous charge. |

Ces choix sont des hypothèses à évaluer. En B, augmenter le pool peut aussi être mesuré, en tenant compte des ressources nécessaires. En C, le mot « flux » seul ne suffit pas à imposer WebFlux.

Pour le calcul d’image, aucun des trois modèles n’accélère le calcul par son seul mécanisme d’attente. Il faut examiner le coût CPU.

**Réussite** : nommer la ressource qui attend, relier le choix aux faits de la situation et formuler une vérification ou une limite. Un autre choix est recevable s’il respecte les contraintes et son raisonnement est explicite.

# TP - Multiplexage HTTP2

| Projet | `demo-application-reactive` |
|---|---|
| Fichiers | `application.properties` ; `AttenteReactive.java`, créé au TP « Serveur bloquant et non bloquant » |
| Sur le poste | un terminal Bash et `curl` compilé avec HTTP2 |

## Vérifier que curl parle HTTP2

```bash
curl -V
```

La ligne `Features` doit contenir `HTTP2`. Sans ce mot, ce `curl` ne peut pas demander HTTP2 et le TP ne peut pas se faire avec lui.

## Activer HTTP2

Ajouter cette ligne dans `src/main/resources/application.properties` :

```
server.http2.enabled=true
```

Redémarrer l'application, puis demander la salutation en HTTP2 :

```bash
curl -i --http2-prior-knowledge http://localhost:8080/salutation
```

La première ligne de la réponse donne le protocole :

```text
HTTP/2 200
```

> [!WARNING]
> **Sans l'option, la réponse reste en HTTP 1.1**
> `curl -i http://localhost:8080/salutation` affiche toujours `HTTP/1.1 200 OK` après l'activation. Sur `http://`, sans TLS, `curl` ouvre la connexion en HTTP 1.1 et ne demande pas HTTP2 de lui-même ; le serveur accepte encore HTTP 1.1 et répond dans ce protocole. `--http2-prior-knowledge` demande HTTP2 en clair dès l'ouverture de la connexion.

## Prédire la durée du lot

Trente requêtes vers `/commande` partent ensemble. Chacune attend une seconde côté serveur, comme au TP « Serveur bloquant et non bloquant ».

- En HTTP 1.1, le client ouvre six connexions au plus, comme un navigateur. Une connexion ne porte qu'un échange à la fois.
- En HTTP2, le client ouvre une seule connexion.

Prédire, dans chaque cas, la durée du lot et le nombre de connexions ouvertes.

## Mesurer en HTTP 1.1

L'application tourne avec HTTP2 activé. Dans un second terminal, envoyer le lot en HTTP 1.1 :

```bash
time curl -s -Z --parallel-max 6 --http1.1 "http://localhost:8080/commande?n=[1-30]" -w '%{stderr}%{num_connects}' > /dev/null
```

La commande se lit ainsi :

- `[1-30]` : `curl` génère lui-même trente URL ; le paramètre `n` ne sert qu'à les distinguer, le contrôleur l'ignore.
- `-Z` : les requêtes partent en parallèle.
- `--parallel-max 6` : six transferts au plus en même temps, la limite d'un navigateur en HTTP 1.1.
- `--http1.1` : le protocole est imposé.
- `-w '%{stderr}%{num_connects}'` : après chaque requête, `1` si elle a ouvert une connexion, `0` si elle en a réutilisé une.
- `> /dev/null` : les trente réponses sont masquées.

Ce qu'on doit observer :

| Observation | Attendu |
|---|---|
| Connexions | six `1`, puis vingt-quatre `0` |
| Durée du lot | environ 5 s (5,05 s relevées en préparation) |

## Mesurer en HTTP2

Envoyer le même lot en HTTP2 :

```bash
time curl -s -Z --http2-prior-knowledge "http://localhost:8080/commande?n=[1-30]" -w '%{stderr}%{num_connects}' > /dev/null
```

Ce qu'on doit observer :

| Observation | Attendu |
|---|---|
| Connexions | un seul `1`, pas forcément en tête, et vingt-neuf `0` |
| Durée du lot | environ 1 s (1,04 s relevée en préparation) |

## Relier l'écart aux connexions

| Protocole | Connexions | Vagues de requêtes | Durée du lot |
|---|---|---|---|
| HTTP 1.1, six connexions au plus | 6 | cinq vagues de six | environ 5 s |
| HTTP2 | 1 | une seule | environ 1 s |

Chaque requête attend toujours une seconde côté serveur : HTTP2 ne rend pas le traitement plus rapide. En HTTP 1.1, une connexion porte un échange à la fois, et les vingt-quatre requêtes suivantes attendent qu'une connexion se libère. En HTTP2, les trente échanges circulent ensemble sur la même connexion : c'est le multiplexage.

Pour vérifier que la limite de six connexions explique l'écart, relancer HTTP 1.1 sans elle :

```bash
time curl -s -Z --http1.1 "http://localhost:8080/commande?n=[1-30]" -w '%{stderr}%{num_connects}' > /dev/null
```

Le lot se termine aussi en environ 1 s (1,03 s relevée en préparation), mais avec trente `1` : une connexion par requête. HTTP 1.1 n'obtient le même résultat qu'en multipliant les connexions, ce que les navigateurs s'interdisent au-delà de six par serveur.

Mesures de préparation le 28 octobre 2026, sur macOS avec `curl` 8.7.1. Elles montrent le comportement de cette simulation, pas une comparaison de performances universelle.

> [!TIP]
> **Ce qu'il faut retenir**
> HTTP2 fait passer plusieurs échanges en même temps sur une seule connexion. Le gain apparaît quand de nombreuses requêtes lentes partent ensemble vers le même serveur : elles n'attendent plus qu'une connexion se libère. Chaque requête, elle, dure toujours le même temps.

Sources : [RFC 9113, HTTP/2](https://www.rfc-editor.org/rfc/rfc9113.html) ; [Spring Boot, Configure HTTP/2](https://docs.spring.io/spring-boot/how-to/webserver.html#howto.webserver.configure-http2) ; [Chrome DevTools, Network features reference](https://developer.chrome.com/docs/devtools/network/reference) ; [curl, manuel](https://curl.se/docs/manpage.html).

# TP - List et Flux

> [!warning] IntelliJ ne résout pas les imports
> Si IntelliJ affiche `package reactor.core.publisher does not exist` ou `package org.junit.jupiter.api does not exist`, vérifier deux points :
>
> - Mettre IntelliJ à jour vers une version compatible avec Maven 3.10, utilisé par le wrapper du projet.
> - Dans la vue Project, retirer à `src` le statut de racine de sources, puis marquer `src/main/java` comme **Sources Root** et `src/test/java` comme **Test Sources Root**. Si tout `src` est marqué comme code de production, IntelliJ compile aussi les tests sans leurs dépendances de test.
>
> Recharger ensuite les projets Maven et relancer `DemoFlux` avec son `main`.

| Projet | `demo-list-flux` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, pour Reactor |
| Classe à lancer | `com.example.demo.DemoFlux`, par son `main` |
| Fichiers | `DemoFlux.java` |
| Sur le poste | un IDE Java |

`DemoFlux.java` se lance par son `main` dans l'IDE ; il ne démarre aucun serveur. Les imports et le record sont fournis pour pouvoir rejouer les extraits du Deck.

```java
package com.example.demo;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public class DemoFlux {
    record Article(int numero) {}

    static List<Article> liste() throws InterruptedException {
        List<Article> resultat = new ArrayList<>();
        for (int i = 1; i <= 5; i++) {
            Thread.sleep(200);
            resultat.add(new Article(i));
        }
        return resultat;
    }

    static Flux<Article> articles() {
        return Flux.range(1, 5).map(Article::new)
            .delayElements(Duration.ofMillis(200));
    }

    static Mono<Article> trouver(int numero) {
        return numero >= 1 && numero <= 5
            ? Mono.just(new Article(numero)) : Mono.empty();
    }

    public static void main(String[] args) throws InterruptedException {
        long debutListe = System.currentTimeMillis();
        List<Article> resultat = liste();
        System.out.println((System.currentTimeMillis() - debutListe) + " ms : " + resultat);

        Flux<Article> flux = articles().log();
        System.out.println("flux déclaré");
        Thread.sleep(500); // aucune production pendant cette pause, sans abonnement
        long debutFlux = System.currentTimeMillis();
        flux.subscribe(a -> System.out.println(
            (System.currentTimeMillis() - debutFlux) + " ms : reçu " + a));
        System.out.println("subscribe a rendu la main");
        Thread.sleep(2000); // maintient ce main vivant ; ce n'est pas le traitement du serveur

        Mono<Article> mono = trouver(9).log();
        System.out.println("déclaré");
        mono.subscribe(a -> System.out.println("reçu " + a));
    }
}
```

Ce qu'on doit observer :

| Bloc | Attendu |
|---|---|
| `List` | la liste entière après environ une seconde |
| `Flux` sans abonnement | aucune production |
| `subscribe` | la main rendue avant le premier article (33 ms relevées en préparation) |
| Articles du `Flux` | un toutes les 200 ms environ |
| `Mono` vide | terminé sans ligne `reçu` |

Les millisecondes exactes et les préfixes du log varient.

Pour ne montrer qu'une étape, poser un point d'arrêt avant la suivante dans l'IDE. Avant le dernier bloc, prédire les signaux du `Mono` vide. La pause du `main` sert seulement à maintenir la JVM en vie.

Source : [Reactor, abonnement et exécution](https://github.com/reactor/reactor-core/blob/main/docs/modules/ROOT/pages/reactiveProgramming.adoc).

# TP - Mono et Flux

**Objectif :** écrire un service qui réunit plusieurs résultats asynchrones, sans attendre l'un pour lancer l'autre.

| Dossier à ouvrir | `demo-application-reactive`, après le TP « Serveur bloquant et non bloquant » |
|---|---|
| Avant de commencer | un IDE Java ; aucun serveur à démarrer |
| Commande | le `main` de `FichesProduit`, lancé dans l'IDE |

Une fiche produit réunit le titre, le prix et le stock d'un article. Chacune de ces trois informations vient d'un service différent, qui répond après un délai :

| Service | Ce qu'il renvoie | Délai |
|---|---|---|
| `titre(numero)` | `Mono<String>` | 100 ms |
| `prix(numero)` | `Mono<Double>` | 300 ms |
| `stock(numero)` | `Mono<Integer>` | 200 ms |

> [!IMPORTANT]
> Les trois services prennent le **numéro** de l'article. Aucun n'a besoin du résultat d'un autre pour répondre.

## Le code de départ

Une seule classe, `FichesProduit`, à créer dans `demo-application-reactive`. Reactor y est déjà présent par le starter WebFlux : rien à ajouter au `pom.xml`. Garder la ligne `package` que l'IDE place en tête de la classe, et coller le reste.

Ce que la classe contient :

- le record `Fiche`, qui porte les quatre informations d'une fiche ;
- les trois services, qui simulent leur délai ;
- `fiche(numero)` et `fiches()`, les deux méthodes à écrire ;
- un `main` qui lance l'étape choisie, affiche le temps écoulé devant chaque fiche, et reste en vie le temps que les fiches arrivent.

Ce qu'elle ne contient pas : ni serveur à démarrer, ni base de données, ni réseau. On la lance comme un programme Java ordinaire, par son `main`.

`FichesProduit.java`

```java
import java.time.Duration;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public class FichesProduit {

    record Fiche(int numero, String titre, double prix, int stock) {}

    static final String[] TITRES = {"Clavier", "Souris", "Écran", "Casque", "Webcam"};
    static final double[] PRIX = {49.0, 19.0, 189.0, 79.0, 59.0};
    static final int[] STOCKS = {12, 40, 5, 8, 0};

    // Les trois services : chacun répond après un délai, à partir du seul numéro de l'article.

    static Mono<String> titre(int numero) {
        return Mono.just(TITRES[numero - 1]).delayElement(Duration.ofMillis(100));
    }

    static Mono<Double> prix(int numero) {
        return Mono.just(PRIX[numero - 1]).delayElement(Duration.ofMillis(300));
    }

    static Mono<Integer> stock(int numero) {
        return Mono.just(STOCKS[numero - 1]).delayElement(Duration.ofMillis(200));
    }

    // Étape 1 : la fiche d'un article.
    static Mono<Fiche> fiche(int numero) {
        // À compléter en suivant le Guide.
        return Mono.empty();
    }

    // Étape 2 : les fiches des articles 1 à 5.
    static Flux<Fiche> fiches() {
        // À compléter en suivant le Guide.
        return Flux.empty();
    }

    public static void main(String[] args) throws InterruptedException {
        long debut = System.currentTimeMillis();
        fiche(1).subscribe(f -> afficher(debut, f));      // étape 1
        // fiches().subscribe(f -> afficher(debut, f));   // étape 2
        Thread.sleep(2000);   // garde le programme en vie pendant que les fiches arrivent
    }

    static void afficher(long debut, Fiche fiche) {
        System.out.println((System.currentTimeMillis() - debut) + " ms : " + fiche);
    }
}
```

Lancer le `main` tel quel : rien ne s'affiche. `fiche` renvoie pour l'instant un `Mono` vide, qui se termine sans élément.

> [!NOTE]
> **Question : À quoi sert le `Thread.sleep(2000)` ?**
> Les fiches sont livrées par un autre thread que celui du `main`. Sans cette pause, le `main` se termine juste après `subscribe`, la JVM s'arrête, et rien ne s'affiche, sans aucune erreur.

## Étape 1 : la fiche d'un article

Écrire `fiche(numero)`, qui construit la fiche d'un article à partir des trois services.

Algorithme :

1. Demander le titre, le prix et le stock de l'article, tous les trois en même temps.
2. Attendre d'avoir les trois résultats.
3. Construire la `Fiche` avec le numéro et ces trois résultats.

Critère de réussite : le `main` affiche la fiche de l'article 1 en un peu plus de 300 ms.

```text
3xx ms : Fiche[numero=1, titre=Clavier, prix=49.0, stock=12]
```

Les millisecondes varient d'une machine à l'autre, et le premier lancement de Reactor ajoute quelques dizaines de millisecondes. Ce qui compte : bien moins que 600 ms.

> [!NOTE]
> **Question : Ma fiche met 600 ms. Pourquoi ?**
> 600 ms, c'est 100 + 300 + 200 : les trois appels partent l'un après l'autre. C'est ce qui se passe quand chaque appel est écrit dans la fonction de `flatMap` du précédent. Aucun service n'a besoin du résultat d'un autre : un opérateur qui s'abonne aux trois sources en même temps convient mieux.

## Étape 2 : les fiches de cinq articles

Avant d'écrire la moindre ligne, noter sa prédiction. `fiches()` construit les fiches des articles 1 à 5, avec `flatMap`, et chaque fiche demande environ 300 ms. Au bout de combien de temps la cinquième fiche s'affiche-t-elle : 300 ms ou 1 500 ms ?

Puis écrire `fiches()`.

Algorithme :

1. Produire les numéros 1 à 5.
2. Pour chaque numéro, construire sa fiche avec la méthode `fiche` de l'étape 1.
3. Livrer les fiches au fil de leur arrivée.

Dans le `main`, mettre en commentaire la ligne de l'étape 1 et retirer le commentaire de la ligne de l'étape 2, puis lancer.

Critère de réussite : les cinq fiches s'affichent, et la mesure confirme ou corrige la prédiction.

Sortie attendue, relevée en préparation :

```text
410 ms : Fiche[numero=5, titre=Webcam, prix=59.0, stock=0]
429 ms : Fiche[numero=1, titre=Clavier, prix=49.0, stock=12]
429 ms : Fiche[numero=2, titre=Souris, prix=19.0, stock=40]
429 ms : Fiche[numero=3, titre=Écran, prix=189.0, stock=5]
429 ms : Fiche[numero=4, titre=Casque, prix=79.0, stock=8]
```

L'ordre des fiches et les millisecondes varient d'une exécution à l'autre : `flatMap` livre les fiches dans l'ordre où elles arrivent.

> [!NOTE]
> **Question : Avec `map`, le code ne compile pas. Pourquoi ?**
> La fonction passée à `map` renvoie un `Mono<Fiche>`. `map` le place tel quel dans le flux, qui devient un `Flux<Mono<Fiche>>`, alors que `fiches()` promet un `Flux<Fiche>`. Le compilateur refuse : c'est le signe qu'il faut un opérateur qui s'abonne lui-même à chaque `Mono`.

> [!TIP]
> **Ce qu'il faut retenir**
> - Plusieurs résultats indépendants, assemblés en un seul : `zip`, et la durée du plus lent.
> - Un même traitement asynchrone sur plusieurs éléments : `flatMap`, qui les lance tous ensemble.
> - Des appels écrits l'un dans la fonction de l'autre partent l'un après l'autre, quel que soit l'opérateur.

## Corrigé - Mono et Flux


Les deux méthodes complétées. Le reste de la classe ne change pas.

### FichesProduit.java

```java
    // Étape 1 : la fiche d'un article.
    static Mono<Fiche> fiche(int numero) {
        return Mono.zip(titre(numero), prix(numero), stock(numero))
            .map(t -> new Fiche(numero, t.getT1(), t.getT2(), t.getT3()));
    }

    // Étape 2 : les fiches des articles 1 à 5.
    static Flux<Fiche> fiches() {
        return Flux.range(1, 5).flatMap(n -> fiche(n));
    }
```

**Étape 1.** `Mono.zip` s'abonne aux trois services en même temps, attend un élément de chacun, puis les livre ensemble dans un `Tuple3`. `getT1()` est le titre, `getT2()` le prix, `getT3()` le stock, dans l'ordre des arguments. La fiche arrive au bout du service le plus lent, le prix : un peu plus de 300 ms.

**Étape 2.** `flatMap` reçoit cinq numéros et s'abonne à la fiche de chacun, toutes en même temps. Les cinq fiches arrivent donc presque ensemble, au bout d'un peu plus de 300 ms : la prédiction juste était 300 ms. Exemple de sortie :

```text
410 ms : Fiche[numero=5, titre=Webcam, prix=59.0, stock=0]
429 ms : Fiche[numero=1, titre=Clavier, prix=49.0, stock=12]
429 ms : Fiche[numero=2, titre=Souris, prix=19.0, stock=40]
429 ms : Fiche[numero=3, titre=Écran, prix=189.0, stock=5]
429 ms : Fiche[numero=4, titre=Casque, prix=79.0, stock=8]
```

L'ordre des fiches peut varier d'une exécution à l'autre : `flatMap` les livre dans l'ordre où elles arrivent.

Pour comparer, remplacer `flatMap` par `concatMap` : chaque fiche attend la précédente, et la cinquième arrive au bout d'environ 1 500 ms, dans l'ordre des numéros.

# TP - Threads, erreurs et débogage

**Objectif :** rendre une chaîne réactive robuste : isoler un appel bloquant, rattraper les erreurs d'un service instable, et retrouver l'étape qui échoue.

| Dossier à ouvrir | `demo-application-reactive`, après le TP « Mono et Flux » |
|---|---|
| Avant de commencer | un IDE Java ; aucun serveur à démarrer |
| Commande | le `main` de `FichesRobustes`, lancé dans l'IDE |

Une fiche d'article réunit les avis des clients, le prix et le stock d'un article. Chacune de ces trois informations vient d'un service différent, et aucun ne se comporte bien :

| Service | Ce qu'il renvoie | Comportement |
|---|---|---|
| `lireAvis(numero)` | `String` | bloque le thread qui l'appelle pendant 500 ms : la bibliothèque n'existe qu'en version bloquante |
| `prix(numero)` | `Mono<Double>` | instable : l'article 2 échoue au premier appel, l'article 3 à chaque appel |
| `prixCatalogue(numero)` | `Mono<Double>` | toujours disponible, mais le prix du catalogue papier peut être ancien |
| `stock(numero)` | `Mono<Integer>` | lit le stock dans une table où il manque un article |

> [!IMPORTANT]
> Les services prennent le **numéro** de l'article. Ils affichent une ligne à chaque appel, précédée du nom du thread qui l'exécute : c'est ce qui permet de vérifier chaque étape.

## Le code de départ

Une seule classe, `FichesRobustes`, à créer dans `demo-application-reactive`, à côté de `FichesProduit`. Reactor y est déjà présent par le starter WebFlux : rien à ajouter au `pom.xml`. Garder la ligne `package` que l'IDE place en tête de la classe, et coller le reste.

Ce que la classe contient :

- le record `Fiche`, qui porte les quatre informations d'une fiche ;
- les quatre services, qui simulent leur comportement ;
- `avis(numero)`, `prixFiable(numero)` et `fiche(numero)`, les trois méthodes à écrire, une par étape ;
- un `main` qui lance l'étape choisie et reste en vie le temps que les résultats arrivent ;
- `trace`, qui affiche le nom du thread courant entre crochets, puis un message.

`FichesRobustes.java`

```java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import reactor.core.scheduler.Schedulers;

public class FichesRobustes {

    record Fiche(int numero, String avis, double prix, int stock) {}

    static final String[] AVIS = {"4,5/5", "4,1/5", "3,8/5", "4,7/5"};
    static final double[] PRIX = {49.0, 19.0, 189.0, 79.0};
    static final double[] PRIX_CATALOGUE = {49.0, 17.0, 179.0, 79.0};
    static final Map<Integer, Integer> STOCKS = Map.of(1, 12, 2, 40, 3, 5);   // le stock de l'article 4 manque
    static final Map<Integer, Integer> APPELS = new ConcurrentHashMap<>();

    // Bibliothèque d'avis : bloquante, 500 ms par appel. Elle n'existe qu'en version bloquante.
    static String lireAvis(int numero) {
        trace("lecture des avis", numero);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return AVIS[numero - 1];
    }

    // Service de tarifs instable : l'article 2 échoue au premier appel, l'article 3 à chaque appel.
    static Mono<Double> prix(int numero) {
        return Mono.defer(() -> {
            int appel = APPELS.merge(numero, 1, Integer::sum);
            trace("appel prix", numero);
            if (numero == 3 || (numero == 2 && appel == 1)) {
                return Mono.error(new IllegalStateException("tarif indisponible pour " + numero));
            }
            return Mono.just(PRIX[numero - 1]);
        });
    }

    // Prix du catalogue papier : toujours disponible, parfois ancien.
    static Mono<Double> prixCatalogue(int numero) {
        return Mono.just(PRIX_CATALOGUE[numero - 1]);
    }

    static Mono<Integer> stock(int numero) {
        return Mono.just(numero).map(n -> STOCKS.get(n));
    }

    // Étape 1 : les avis d'un article, sans bloquer le thread qui s'abonne.
    static Mono<String> avis(int numero) {
        // À compléter en suivant le Guide.
        return Mono.empty();
    }

    // Étape 2 : un prix pour chaque article, malgré le service instable.
    static Mono<Double> prixFiable(int numero) {
        // À compléter en suivant le Guide.
        return Mono.empty();
    }

    // Étape 3 : la fiche d'un article, avec une étape nommée par source.
    static Mono<Fiche> fiche(int numero) {
        // À compléter en suivant le Guide.
        return Mono.empty();
    }

    public static void main(String[] args) throws InterruptedException {
        // Étape 1
        avis(1).subscribe(a -> trace("avis reçu", a));
        trace("subscribe a rendu la main");

        // Étape 2
        // Flux.range(1, 3).concatMap(n -> prixFiable(n)).subscribe(p -> trace("prix reçu", p));

        // Étape 3
        // Flux.range(1, 4).concatMap(n -> fiche(n))
        //     .subscribe(f -> trace("fiche", f), Throwable::printStackTrace);

        Thread.sleep(3000);   // garde le programme en vie pendant que les résultats arrivent
    }

    static <T> T trace(String etape, T valeur) {
        System.out.println("[" + Thread.currentThread().getName() + "] " + etape + " " + valeur);
        return valeur;
    }

    static void trace(String message) {
        System.out.println("[" + Thread.currentThread().getName() + "] " + message);
    }
}
```

Lancer le `main` tel quel : seule la ligne `[main] subscribe a rendu la main` s'affiche. `avis` renvoie pour l'instant un `Mono` vide, qui se termine sans élément.

## Étape 1 : les avis sans bloquer

Écrire `avis(numero)`, qui renvoie les avis d'un article sans immobiliser le thread qui s'abonne.

Algorithme :

1. Envelopper l'appel à `lireAvis(numero)` dans un `Mono`, pour qu'il ne s'exécute qu'à l'abonnement.
2. Faire exécuter cet appel sur un Scheduler fait pour les appels bloquants.

Critère de réussite : `subscribe a rendu la main` s'affiche sur `main`, et ni la lecture des avis ni l'avis reçu ne s'affichent sur `main`.

```text
[boundedElastic-1] lecture des avis 1
[main] subscribe a rendu la main
[boundedElastic-1] avis reçu 4,5/5
```

Les deux premières lignes peuvent s'afficher dans l'autre ordre : deux threads écrivent en même temps. Le numéro du thread peut aussi varier ; seul le préfixe compte.

> [!NOTE]
> **Question : Tout s'affiche sur `[main]`, et `subscribe a rendu la main` arrive en dernier. Pourquoi ?**
> L'appel bloquant s'exécute sur le thread qui s'abonne, ici `main`, qui reste immobilisé 500 ms. Envelopper l'appel ne suffit pas : il faut aussi dire sur quel Scheduler l'abonnement doit partir.

## Étape 2 : un prix malgré les pannes

Dans le `main`, mettre en commentaire les deux lignes de l'étape 1 et retirer le commentaire de la ligne de l'étape 2.

Écrire `prixFiable(numero)`, qui renvoie toujours un prix.

Algorithme :

1. Appeler `prix(numero)`.
2. S'il échoue, refaire l'appel, deux fois au plus.
3. S'il échoue encore, prendre le prix du catalogue papier.

Critère de réussite : les trois articles ont un prix, et le nombre d'appels affichés correspond à chaque panne.

```text
[main] appel prix 1
[main] prix reçu 49.0
[main] appel prix 2
[main] appel prix 2
[main] prix reçu 19.0
[main] appel prix 3
[main] appel prix 3
[main] appel prix 3
[main] prix reçu 179.0
```

> [!NOTE]
> **Question : L'article 2 affiche `17.0` après un seul appel. Pourquoi ?**
> `17.0` est le prix du catalogue : le repli a remplacé l'erreur avant qu'elle atteigne l'opérateur qui réessaie. Un opérateur qui réessaie ne réagit qu'à une erreur qui lui parvient. Vérifier l'ordre des deux opérateurs.

## Étape 3 : retrouver l'étape qui échoue

Avant d'écrire la moindre ligne, noter sa prédiction. Dans `fiche`, le prix sera protégé comme à l'étape 2, puis portera un `checkpoint("prix")`. L'article 2 échoue au premier appel de prix. Le nom « prix » apparaîtra-t-il dans une trace d'erreur ?

Puis écrire `fiche(numero)`.

Algorithme :

1. Demander les avis, le prix fiable et le stock de l'article, tous les trois en même temps, avec les méthodes des étapes précédentes pour les deux premiers.
2. Nommer chacune des trois sources par un `checkpoint` : `"avis"`, `"prix"`, `"stock"`.
3. Attendre d'avoir les trois résultats, puis construire la `Fiche`.

Dans le `main`, mettre en commentaire la ligne de l'étape 2 et retirer le commentaire des deux lignes de l'étape 3, puis lancer.

Critère de réussite : les fiches des articles 1 à 3 s'affichent, puis une trace d'erreur pour l'article 4, qui nomme l'étape en cause.

```text
[boundedElastic-1] fiche Fiche[numero=1, avis=4,5/5, prix=49.0, stock=12]
[boundedElastic-2] fiche Fiche[numero=2, avis=4,1/5, prix=19.0, stock=40]
[boundedElastic-1] fiche Fiche[numero=3, avis=3,8/5, prix=179.0, stock=5]
java.lang.NullPointerException: The mapper [FichesRobustes$$Lambda/0x…] returned a null value.
	Suppressed: The stacktrace has been enhanced by Reactor, refer to additional information below: 
Error has been observed at the following site(s):
	*__checkpoint ⇢ stock
Original Stack Trace:
	...
```

Les lignes des services s'intercalent entre les fiches, et les numéros de thread varient d'une exécution à l'autre.

> [!NOTE]
> **Question : Pourquoi la trace ne cite-t-elle aucune ligne de `FichesRobustes` ?**
> La lambda de `stock` s'est terminée sans erreur : elle a renvoyé `null`, et c'est Reactor qui a refusé cette valeur. La pile d'appels ne montre donc que le code de Reactor. Seul le nom du `checkpoint` relie l'erreur à une étape de la fiche.

> [!TIP]
> **Ce qu'il faut retenir**
> - Un appel bloquant inévitable : l'envelopper, puis faire partir l'abonnement sur `boundedElastic`.
> - Un service instable : réessayer d'abord, se replier ensuite.
> - Un `checkpoint` ne nomme que les erreurs qui le traversent sans être rattrapées.

## Corrigé - Threads, erreurs et débogage


Les trois méthodes complétées. Le reste de la classe ne change pas.

### FichesRobustes.java

```java
    // Étape 1 : les avis d'un article, sans bloquer le thread qui s'abonne.
    static Mono<String> avis(int numero) {
        return Mono.fromCallable(() -> lireAvis(numero))
                   .subscribeOn(Schedulers.boundedElastic());
    }

    // Étape 2 : un prix pour chaque article, malgré le service instable.
    static Mono<Double> prixFiable(int numero) {
        return prix(numero)
                   .retry(2)
                   .onErrorResume(e -> prixCatalogue(numero));
    }

    // Étape 3 : la fiche d'un article, avec une étape nommée par source.
    static Mono<Fiche> fiche(int numero) {
        return Mono.zip(avis(numero).checkpoint("avis"),
                        prixFiable(numero).checkpoint("prix"),
                        stock(numero).checkpoint("stock"))
                   .map(t -> new Fiche(numero, t.getT1(), t.getT2(), t.getT3()));
    }
```

**Étape 1.** `Mono.fromCallable` retarde l'appel à `lireAvis` jusqu'à l'abonnement. `subscribeOn(Schedulers.boundedElastic())` fait partir cet abonnement sur un thread de `boundedElastic`, le Scheduler fait pour les appels bloquants : `main` rend la main tout de suite, et la lecture des avis s'exécute ailleurs.

**Étape 2.** `retry(2)` se réabonne à `prix(numero)` après un échec, deux fois au plus : chaque réabonnement est un nouvel appel au service. L'article 2 répond au deuxième appel, `19.0`. L'article 3 échoue trois fois : `onErrorResume` remplace alors l'erreur par le prix du catalogue, `179.0`. Dans l'ordre inverse, `onErrorResume` remplacerait l'erreur dès le premier échec et `retry` n'aurait rien à réessayer : l'article 2 afficherait `17.0` après un seul appel.

**Étape 3.** La prédiction juste était non : l'erreur de l'article 2 est rattrapée par `retry`, aucune trace ne s'affiche, et le nom « prix » n'apparaît nulle part. L'article 4 n'a pas de stock : `STOCKS.get` renvoie `null`, et `map` refuse cette valeur. Cette erreur n'est rattrapée par personne : elle traverse le `checkpoint("stock")`, puis `zip`, et arrive à l'abonné, qui affiche la trace. Seul le nom `stock` y figure, puisque l'erreur n'a traversé aucun des deux autres `checkpoint`.

# TP - Mono dans Spring MVC

| Projet | `demo-mvc` |
|---|---|
| Dépendances | `spring-boot-starter-webmvc`, `spring-boot-starter-webflux` |
| Classe de démarrage | `com.example.demo.DemoMvcApplication` |
| Fichiers | `MvcMono.java` dans le projet ; `envoyer-requetes-paralleles.sh` dans le dossier de travail commun, créé lors de la comparaison des serveurs |
| Sur le poste | Bash et `curl` |

`MvcMono.java`. Le projet déclare les deux starters serveur, sans autre réglage : Spring MVC est retenu. Les deux routes attendent une seconde ; l'une rend le thread pendant l'attente, l'autre le garde.

```java
package com.example.demo;

import java.time.Duration;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
public class MvcMono {
    @GetMapping("/prix/mono")
    public Mono<Double> mono() {
        trace("méthode");
        return Mono.just(49.0).delayElement(Duration.ofSeconds(1))
            .doOnNext(prix -> trace("valeur"));
    }
    @GetMapping("/prix/bloquant")
    public double bloquant() throws InterruptedException {
        Thread.sleep(1000);
        return 49.0;
    }
    static void trace(String message) {
        System.out.println("[" + Thread.currentThread().getName() + "] " + message);
    }
}
```

Arrêter les serveurs précédents, puis lancer :

```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--server.tomcat.threads.max=1"
```

Prédire les durées des deux paires de requêtes, puis lancer `envoyer-requetes-paralleles.sh` depuis le dossier de travail commun, dans un second terminal :

```bash
time bash envoyer-requetes-paralleles.sh http://localhost:8080/prix/mono 2
```

```bash
time bash envoyer-requetes-paralleles.sh http://localhost:8080/prix/bloquant 2
```

Ce qu'on doit observer :

| Observation | Attendu |
|---|---|
| Paire `mono` | environ 1 s (1 036 à 1 041 ms relevées en préparation) |
| Paire bloquante | environ 2 s (2 047 à 2 052 ms relevées en préparation) |
| Log | la méthode sur un thread `http-nio`, la valeur sur `parallel` |

Arrêter le serveur après la comparaison.

Source : [applications web réactives avec Spring Boot](https://docs.spring.io/spring-boot/reference/web/reactive.html).

# TP - Exposer une API WebFlux

**Objectif :** exposer une API réactive par annotations puis par routes fonctionnelles, et la vérifier en ligne de commande.

| Dossier à ouvrir | `demo-application-reactive`, après le TP « Threads, erreurs et débogage » |
|---|---|
| Avant de commencer | `curl` ; aucun autre projet ni service |
| Commande | `./mvnw spring-boot:run` depuis le dossier du projet, puis les appels `curl` de chaque étape |

Une boutique publie les commandes du jour. Le service `Commandes` est fourni : il les garde en mémoire et les livre une par une.

| Méthode du service | Ce qu'elle renvoie |
|---|---|
| `duJour()` | `Flux<Commande>` : les commandes du jour, une toutes les 300 ms |
| `parNumero(numero)` | `Mono<Commande>` : une commande, ou un `Mono` vide si le numéro est inconnu |
| `supprimer(numero)` | `Mono<Void>` : supprime la commande, puis se termine sans valeur |
| `trace(qui, numero)` | affiche dans la console qui répond, et pour quelle commande |

> [!IMPORTANT]
> Les numéros sont des nombres entiers. Les commandes 1 à 4 existent ; la commande 9 n'existe pas.

## Le code de départ

Un seul fichier, `Commandes.java`, à créer dans `demo-application-reactive`, dans le même package que la classe d'application. WebFlux y est déjà présent : rien à ajouter au `pom.xml`. Garder la ligne `package` que l'IDE place en tête du fichier, et coller le reste.

Ce que le fichier contient :

- le record `Commande`, qui porte le numéro, le client et le montant ;
- le service `Commandes`, déclaré comme bean par `@Component`, qui simule les commandes du jour.

Ce qu'il ne contient pas : aucune route. Les classes à écrire, une par étape, sont nommées dans le Guide.

`Commandes.java`

```java
import java.time.Duration;
import java.util.Map;
import java.util.concurrent.ConcurrentSkipListMap;

import org.springframework.stereotype.Component;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

record Commande(long numero, String client, double montant) {}

@Component
class Commandes {

    private final Map<Long, Commande> commandes = new ConcurrentSkipListMap<>(Map.of(
        1L, new Commande(1, "Durand", 120.0),
        2L, new Commande(2, "Martin", 45.5),
        3L, new Commande(3, "Petit", 310.0),
        4L, new Commande(4, "Leroy", 89.9)));

    // Les commandes du jour, une toutes les 300 ms.
    Flux<Commande> duJour() {
        return Flux.fromIterable(commandes.values()).delayElements(Duration.ofMillis(300));
    }

    // Une commande, ou un Mono vide si le numéro est inconnu.
    Mono<Commande> parNumero(long numero) {
        return Mono.justOrEmpty(commandes.get(numero));
    }

    // Supprime une commande, puis se termine sans valeur.
    Mono<Void> supprimer(long numero) {
        return Mono.fromRunnable(() -> commandes.remove(numero));
    }

    // Affiche dans la console qui répond, et pour quelle commande.
    static void trace(String qui, long numero) {
        System.out.println("[" + qui + "] commande " + numero);
    }
}
```

Démarrer l'application, puis lancer :

```bash
curl -i localhost:8080/commandes
```

La réponse est `404 Not Found` : aucune route ne répond encore.

## Étape 1 : le contrôleur annoté

Créer la classe `CommandeController`, un contrôleur annoté qui répond sous `/commandes`.

Algorithme :

1. `GET /commandes` renvoie les commandes du jour, une par ligne, au fur et à mesure qu'elles arrivent.
2. `GET /commandes/{numero}` renvoie une commande. Le motif n'accepte que des chiffres.
3. Au début de la méthode qui renvoie une commande, appeler `Commandes.trace("contrôleur", numero)`.
4. Pour un numéro inconnu, répondre `404 Not Found`.

Critère de réussite : les trois commandes ci-dessous donnent les réponses attendues.

> [!IMPORTANT]
> **Toujours lancer `curl` avec `-N` sur un flux**
> L'option `-N` désactive la mise en mémoire tampon de `curl` : chaque ligne s'affiche dès que le serveur l'envoie. Sans elle, `curl` peut tout afficher d'un bloc à la fin, notamment quand sa sortie passe dans un tube, et le flux semble alors arriver en une seule fois alors que le serveur l'envoie au fil de l'eau.

```bash
curl -N localhost:8080/commandes
```

Une ligne toutes les 300 ms environ, quatre lignes en tout :

```text
{"numero":1,"client":"Durand","montant":120.0}
{"numero":2,"client":"Martin","montant":45.5}
...
```

```bash
curl -i localhost:8080/commandes/9
```

```text
HTTP/1.1 404 Not Found
```

```bash
curl -i localhost:8080/commandes/12.json
```

```text
HTTP/1.1 404 Not Found
```

> [!NOTE]
> **Question : Les commandes s'affichent sur une seule ligne, dans un tableau JSON. Pourquoi ?**
> Sans type de média déclaré, un `Flux` est envoyé en `application/json` : WebFlux écrit chaque commande au fil de l'eau, mais dans un seul tableau, qui n'est un document JSON complet qu'à son `]` final. Il faut déclarer un type de média qui envoie un élément par ligne, chaque ligne étant un document complet.

> [!NOTE]
> **Question : `/commandes/9` répond `200 OK` avec un corps vide. Pourquoi ?**
> Un `Mono` vide n'est pas une erreur : WebFlux répond 200, sans corps. Le code 404 se décide dans la méthode, quand le `Mono` se termine sans élément.

> [!NOTE]
> **Question : `/commandes/12.json` répond `200 OK`. Pourquoi ?**
> Le point fait partie du segment, et une variable `{numero}` sans restriction capture tout le segment. Le motif doit n'accepter que des chiffres.

## Étape 2 : migrer vers le routeur

L'équipe décide de déclarer les routes d'une commande par des routes fonctionnelles. La liste du jour reste dans le contrôleur.

Avant d'écrire la moindre ligne, noter sa prédiction. On ajoute un routeur qui déclare `GET /commandes/{numero}`, sans retirer encore la méthode du contrôleur. Pour `curl localhost:8080/commandes/2`, quel nom s'affichera dans la console : `[contrôleur]` ou `[routeur]` ? Ou l'application refusera-t-elle de démarrer ?

Puis créer deux classes.

`CommandeHandler`, un bean qui contient une méthode par route :

1. `parNumero` lit le numéro dans l'URL, appelle `Commandes.trace("routeur", numero)`, et renvoie la commande, ou `404 Not Found` pour un numéro inconnu.
2. `supprimer` supprime la commande, puis répond `204 No Content`.

`CommandeRoutes`, une classe de configuration qui déclare un bean `RouterFunction`, nommé `routesCommandes` :

1. Sous le préfixe `/commandes`, `GET /{numero}` vers `parNumero`, et `DELETE /{numero}` vers `supprimer`.
2. Le motif n'accepte que des chiffres, comme à l'étape 1.

Démarrer l'application, lancer `curl localhost:8080/commandes/2`, et comparer la console à la prédiction.

Puis retirer du contrôleur la méthode qui renvoie une commande, et redémarrer.

Critère de réussite : les commandes ci-dessous donnent les réponses attendues, et la console n'affiche que `[routeur]`.

| Commande | Réponse attendue |
|---|---|
| `curl -N localhost:8080/commandes` | une ligne toutes les 300 ms, comme à l'étape 1 |
| `curl -i localhost:8080/commandes/2` | `200 OK`, la commande de Martin |
| `curl -i localhost:8080/commandes/9` | `404 Not Found` |
| `curl -i -X DELETE localhost:8080/commandes/2` | `204 No Content` |
| `curl -i localhost:8080/commandes/2`, à nouveau | `404 Not Found` : la commande a été supprimée |

> [!NOTE]
> **Question : Après la migration, j'ai modifié la méthode du contrôleur, et rien ne change. Pourquoi ?**
> Pour une même URL, WebFlux consulte les routes fonctionnelles avant les contrôleurs annotés. Tant que le routeur déclare la route, la méthode du contrôleur n'est jamais appelée, sans erreur ni avertissement. Retirer l'ancienne méthode.

> [!NOTE]
> **Question : Le bean du routeur entre en conflit avec un autre bean nommé `routes`. Pourquoi ?**
> `demo-application-reactive` contient peut-être déjà un routeur des articles, déclaré par une méthode `routes`. Deux beans ne peuvent pas porter le même nom : d'où le nom `routesCommandes`.

> [!TIP]
> **Ce qu'il faut retenir**
> - Un `Flux` livré au fil de l'eau : le type de média `application/x-ndjson`.
> - Un `Mono` vide répond 200 : le 404 se décide avec `defaultIfEmpty` en annoté, `switchIfEmpty` en fonctionnel.
> - Une variable de chemin capture tout le segment, point compris : la restreindre par une expression régulière.
> - Pour une même URL, le routeur passe avant le contrôleur : retirer l'ancienne méthode quand une route migre.

## Corrigé - Exposer une API WebFlux


### CommandeController.java, après l'étape 1

```java
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/commandes")
class CommandeController {

    private final Commandes commandes;

    CommandeController(Commandes commandes) { this.commandes = commandes; }

    @GetMapping(produces = MediaType.APPLICATION_NDJSON_VALUE)
    Flux<Commande> duJour() {
        return commandes.duJour();
    }

    @GetMapping("/{numero:\\d+}")
    Mono<ResponseEntity<Commande>> parNumero(@PathVariable long numero) {
        Commandes.trace("contrôleur", numero);
        return commandes.parNumero(numero)
                        .map(ResponseEntity::ok)
                        .defaultIfEmpty(ResponseEntity.notFound().build());
    }
}
```

**Étape 1.** `produces = MediaType.APPLICATION_NDJSON_VALUE` envoie chaque commande sur sa propre ligne, dès qu'elle est prête : une ligne toutes les 300 ms environ. `{numero:\d+}` n'accepte que des chiffres : `12.json` ne correspond pas au motif, et la requête reçoit un 404. Dans une chaîne Java, la barre oblique inverse se double. Pour un numéro inconnu, `parNumero` renvoie un `Mono` vide : `defaultIfEmpty` le remplace par la réponse 404.

### CommandeHandler.java

```java
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;

import reactor.core.publisher.Mono;

@Component
class CommandeHandler {

    private final Commandes commandes;

    CommandeHandler(Commandes commandes) { this.commandes = commandes; }

    Mono<ServerResponse> parNumero(ServerRequest requete) {
        long numero = Long.parseLong(requete.pathVariable("numero"));
        Commandes.trace("routeur", numero);
        return commandes.parNumero(numero)
                        .flatMap(c -> ServerResponse.ok().bodyValue(c))
                        .switchIfEmpty(ServerResponse.notFound().build());
    }

    Mono<ServerResponse> supprimer(ServerRequest requete) {
        long numero = Long.parseLong(requete.pathVariable("numero"));
        return commandes.supprimer(numero)
                        .then(ServerResponse.noContent().build());
    }
}
```

### CommandeRoutes.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.RouterFunctions;
import org.springframework.web.reactive.function.server.ServerResponse;

@Configuration
class CommandeRoutes {

    @Bean
    RouterFunction<ServerResponse> routesCommandes(CommandeHandler handler) {
        return RouterFunctions.route()
            .path("/commandes", b -> b
                .GET("/{numero:\\d+}", handler::parNumero)
                .DELETE("/{numero:\\d+}", handler::supprimer))
            .build();
    }
}
```

### CommandeController.java, après l'étape 2

La méthode qui renvoie une commande est retirée ; la liste du jour reste servie par le contrôleur.

```java
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;

@RestController
@RequestMapping("/commandes")
class CommandeController {

    private final Commandes commandes;

    CommandeController(Commandes commandes) { this.commandes = commandes; }

    @GetMapping(produces = MediaType.APPLICATION_NDJSON_VALUE)
    Flux<Commande> duJour() {
        return commandes.duJour();
    }
}
```

**Étape 2.** La prédiction juste était `[routeur]` : WebFlux consulte les routes fonctionnelles avant les contrôleurs annotés, et l'application démarre sans erreur ni avertissement. Tant que l'ancienne méthode reste dans le contrôleur, elle n'est jamais appelée pour cette URL. Dans le handler, `switchIfEmpty` joue le rôle de `defaultIfEmpty` : il remplace un `Mono` vide par la réponse 404. `then` attend la fin de la suppression, puis enchaîne la réponse 204. `GET /commandes` n'est déclaré par aucune route du routeur : la requête passe au contrôleur, qui la sert toujours.

# TP - WebClient et service de tarifs

| Projet | `demo-webclient` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-webclient` |
| Classe de démarrage | `com.example.demo.DemoWebclientApplication`, qui démarre le service de tarifs |
| Classe à lancer | `com.example.demo.DemoWebClient`, par son `main`, pendant que le service tourne |
| Fichiers | `ServiceTarifs.java`, `DemoWebClient.java` |
| Sur le poste | un IDE Java et `curl` |

Ce projet sert aussi de service distant au TP « Appeler et diffuser ».

## Démarrer le service distant

`ServiceTarifs.java` est le serveur de test du projet. Les identifiants 1 à 3 existent, 9 répond 404, `muet` accepte l'appel sans terminer sa réponse.

```java
package com.example.demo;

import java.time.Duration;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
public class ServiceTarifs {
    public record Tarif(String id, double montant) {}
    @GetMapping("/tarifs/{id}")
    public Mono<ResponseEntity<Tarif>> tarif(@PathVariable String id) {
        System.out.println("appel tarif " + id);
        if (id.equals("muet")) return Mono.never();
        if (id.equals("9")) return Mono.just(ResponseEntity.notFound().build());
        double montant = switch (id) {
            case "1" -> 49.0;
            case "2" -> 19.0;
            case "3" -> 189.0;
            default -> 0.0;
        };
        return Mono.just(ResponseEntity.ok(new Tarif(id, montant)))
            .delayElement(Duration.ofMillis(300));
    }
}
```

Depuis le dossier `demo-webclient` :

```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8081"
```

## Appeler et mesurer

`DemoWebClient.java` se lance par son `main` dans l'IDE pendant que le service tourne. Le client utilise directement `WebClient.create` dans ce programme autonome ; dans un bean Spring, le Deck montre l'injection du `WebClient.Builder` préconfiguré, fourni par le starter `spring-boot-starter-webclient` du projet.

```java
package com.example.demo;

import java.time.Duration;
import java.util.List;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public class DemoWebClient {
    record Tarif(String id, double montant) {}
    static final WebClient CLIENT = WebClient.create("http://localhost:8081");

    static Mono<Tarif> tarif(String id) {
        return CLIENT.get().uri("/tarifs/{id}", id).retrieve().bodyToMono(Tarif.class);
    }

    static void mesurer(String nom, Flux<Tarif> tarifs) {
        long debut = System.currentTimeMillis();
        tarifs.doOnNext(t -> System.out.println(nom + " " +
            (System.currentTimeMillis() - debut) + " ms : " + t)).blockLast();
    }

    public static void main(String[] args) throws InterruptedException {
        Mono<Tarif> appel = tarif("1");
        System.out.println("Mono déclaré, regarder le serveur : aucun appel nouveau");
        Thread.sleep(1000);
        System.out.println(appel.block());
        mesurer("flatMap", Flux.fromIterable(List.of("1", "2", "3"))
            .flatMap(DemoWebClient::tarif));
        mesurer("concatMap", Flux.fromIterable(List.of("1", "2", "3"))
            .concatMap(DemoWebClient::tarif));
        try {
            tarif("9").block();
        } catch (RuntimeException erreur) {
            System.out.println("404 : " + erreur.getClass().getSimpleName());
        }
        Mono<Tarif> muet = tarif("muet");
        var attente = muet.subscribe(System.out::println, Throwable::printStackTrace);
        Thread.sleep(10000);
        System.out.println("aucun résultat après 10 s ; arrêt volontaire de cet essai");
        attente.dispose();
        long debut = System.currentTimeMillis();
        try {
            muet.timeout(Duration.ofSeconds(2)).block();
        } catch (RuntimeException erreur) {
            System.out.println((System.currentTimeMillis() - debut) + " ms : " + erreur);
        }
    }
}
```

Avant les deux mesures, prédire 300 ms ou 900 ms.

Ce qu'on doit observer :

| Bloc | Attendu |
|---|---|
| Trois tarifs avec `flatMap` | arrivées ensemble sur le serveur, environ 300 ms (316 ms relevées en préparation) |
| Trois tarifs avec `concatMap` | arrivées espacées, environ 900 ms (933 ms relevées en préparation) |
| Service muet, sans `timeout` | aucune réponse au bout de dix secondes |
| Service muet, avec `timeout` | `TimeoutException` au bout d'environ 2 s |
| Tarif `9` | `WebClientResponseException$NotFound` ; `TarifInconnu` avec `tarifMetier` |

Le premier appel peut être plus lent à cause de l'initialisation du client. Pour le service muet, dix secondes sans réponse constituent une observation bornée ; elles ne mesurent pas une attente infinie.

Les appels `block()` et `blockLast()` maintiennent ici le programme de mesure en attente sur son `main`. Ne pas les recopier dans un contrôleur WebFlux.

Pour choisir une erreur métier à la place du 404, ajouter dans `DemoWebClient` :

```java
static class TarifInconnu extends RuntimeException {
    TarifInconnu(String id) { super("Tarif inconnu : " + id); }
}

static Mono<Tarif> tarifMetier(String id) {
    return CLIENT.get().uri("/tarifs/{id}", id).retrieve()
        .onStatus(statut -> statut.value() == 404,
            reponse -> Mono.error(new TarifInconnu(id)))
        .bodyToMono(Tarif.class);
}
```

Dans le `try` qui appelait `tarif("9")`, remplacer cet appel par `tarifMetier("9")`, puis relancer : l'erreur affichée devient `TarifInconnu`. Arrêter ensuite le serveur de tarifs.

Source : [WebClient et retrieve](https://docs.spring.io/spring-framework/reference/web/webflux-webclient/client-retrieve.html).

# TP - Backpressure

| Projet | `demo-backpressure` |
|---|---|
| Dépendances | `spring-boot-starter-webflux` |
| Classe de démarrage | `com.example.demo.DemoBackpressureApplication`, qui démarre le serveur de mesures |
| Classe à lancer | `com.example.demo.DemoDemande`, par son `main` ; aucun serveur ne démarre |
| Fichiers | `DemoDemande.java`, `FluxReseau.java` |
| Sur le poste | un IDE Java et `curl` |

## Observer les demandes locales

Lancer le `main` de `DemoDemande.java` dans l'IDE.

```java
package com.example.demo;

import java.time.Duration;
import java.util.concurrent.atomic.AtomicInteger;
import reactor.core.publisher.Flux;
import reactor.core.scheduler.Schedulers;

public class DemoDemande {
    static void ecrire(long valeur) {
        try { Thread.sleep(10); }
        catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
    public static void main(String[] args) {
        AtomicInteger produits = new AtomicInteger();
        Flux.range(1, 1000).doOnNext(n -> produits.incrementAndGet())
            .subscribe(n -> {
                ecrire(n);
                if (n % 250 == 0) System.out.println("traité " + n + ", produits " + produits);
            });
        Flux.range(1, 12).log("sans-limite").subscribe();
        Flux.range(1, 12).log("lots").limitRate(10).subscribe();
        try {
            Flux.interval(Duration.ofMillis(1)).publishOn(Schedulers.single())
                .doOnNext(DemoDemande::ecrire).blockLast();
        } catch (RuntimeException erreur) {
            System.out.println("source trop rapide : " + erreur);
        }
    }
}
```

Prédire le compteur à 250 éléments traités : la première partie est synchrone, donc elle affiche 250 produits. Lire ensuite la première demande de chaque log : sans limite, puis par lots de dix. Enfin, `interval` poursuit son rythme pendant que l'autre thread écrit lentement : une erreur signale l'épuisement de la demande.

Ce qu'on doit observer :

| Bloc | Attendu |
|---|---|
| `Flux.range` et abonné lent | 250 produits pour 250 traités : la source n'a aucune avance |
| Première demande, sans limite | `request(unbounded)` |
| Première demande, par lots de dix | `request(10)`, puis `request(8)` après chaque tranche de huit |
| `interval` et `publishOn` | `OverflowException` au tick 256, après `request(256)` |

Pour comparer les stratégies, remplacer seulement la dernière chaîne par l'une des variantes suivantes, en gardant le `try/catch`. Relancer entre les variantes :

```java
Flux.interval(Duration.ofMillis(1)).onBackpressureDrop()
    .publishOn(Schedulers.single()).doOnNext(n -> { ecrire(n); System.out.println(n); })
    .take(Duration.ofSeconds(5)).blockLast();
```

```java
Flux.interval(Duration.ofMillis(1)).onBackpressureLatest()
    .publishOn(Schedulers.single()).doOnNext(n -> { ecrire(n); System.out.println(n); })
    .take(Duration.ofSeconds(5)).blockLast();
```

```java
Flux.interval(Duration.ofMillis(1)).onBackpressureBuffer(100)
    .publishOn(Schedulers.single()).doOnNext(DemoDemande::ecrire).blockLast();
```

Avec abandon ou dernière valeur, les valeurs reçues peuvent sauter ; les effets deviennent visibles lorsque la demande de `publishOn` se renouvelle. Les 256 premières valeurs, de 0 à 255, arrivent sans saut : ce sont celles du premier lot demandé par `publishOn`, et l'abonné met environ 3 s à les écrire. Le premier saut suit donc la valeur 255, vers la troisième seconde : 255 puis 2290 avec abandon, 255 puis 2295 avec dernière valeur, relevés en préparation. Le tampon limité finit par signaler une erreur. Les nombres exacts dépendent de la machine.

Ce qu'on doit observer :

| Variante | Attendu |
|---|---|
| `onBackpressureDrop()` | les valeurs reçues sautent : le surplus est abandonné |
| `onBackpressureLatest()` | les valeurs reçues sautent : seule la plus récente est gardée |
| `onBackpressureBuffer(100)` | erreur `The receiver is overrun by more signals than expected` |

## Observer le réseau

`FluxReseau.java` :

```java
package com.example.demo;

import java.time.Duration;
import java.util.concurrent.atomic.AtomicInteger;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;

@RestController
public class FluxReseau {
    public record Mesure(int numero, String contenu) {}
    @GetMapping(value = "/mesures", produces = MediaType.APPLICATION_NDJSON_VALUE)
    public Flux<Mesure> mesures() {
        AtomicInteger produits = new AtomicInteger();
        var compteur = Flux.interval(Duration.ofSeconds(1)).subscribe(t ->
            System.out.println("produits : " + produits.get()));
        return Flux.range(1, 100_000_000)
            .map(n -> new Mesure(n, "x".repeat(1024)))
            .doOnNext(n -> produits.incrementAndGet())
            .doFinally(signal -> { compteur.dispose(); System.out.println("fin : " + signal); });
    }
}
```

Depuis le dossier `demo-backpressure` :

```bash
mvn spring-boot:run
```

Dans un autre terminal :

```bash
curl --limit-rate 2k --max-time 10 http://localhost:8080/mesures -o /dev/null
```

Chaque mesure porte ici un contenu de 1 Ko pour remplir rapidement les tampons réseau. Observer le compteur côté serveur.

Ce qu'on doit observer :

| Observation | Attendu |
|---|---|
| Compteur côté serveur | il augmente au début, le temps que les tampons se remplissent, puis ralentit ou reste stable |

Cette variante rend l'effet visible sans attendre longtemps ; elle ne cherche pas à reproduire le chiffre exact de la slide, qui dépend du contenu, du réseau et de la machine. La fin imposée par `--max-time` produit un code d'erreur `curl` attendu et annule le flux. Arrêter ensuite le serveur.

Source : [modèle réactif et backpressure](https://docs.spring.io/spring-framework/reference/web/webflux/new-framework.html).

# TP - Server-Sent Events

| Projet | `demo-sse` |
|---|---|
| Dépendances | `spring-boot-starter-webflux` |
| Classe de démarrage | `com.example.demo.DemoSseApplication` |
| Fichiers | `CapteurSse.java` |
| Sur le poste | `curl` |

`CapteurSse.java`. Chaque connexion a son flux de mesures ; sa fermeture est visible dans le log.

```java
package com.example.demo;

import java.time.Duration;
import org.springframework.http.MediaType;
import org.springframework.http.codec.ServerSentEvent;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;

@RestController
public class CapteurSse {
    public record Mesure(long numero, double temperature) {}
    Flux<Mesure> mesures() {
        return Flux.interval(Duration.ofSeconds(1))
            .map(n -> new Mesure(n, 20.0 + n % 5))
            .doOnCancel(() -> System.out.println("[capteur] annulation"));
    }
    @GetMapping("/temperatures/json")
    public Flux<Mesure> json() { return mesures(); }
    @GetMapping(value = "/temperatures", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<Mesure> sse() { return mesures(); }
    @GetMapping(value = "/temperatures/nommees", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<Mesure>> nommees(
            @RequestHeader(value = "Last-Event-ID", required = false) String dernier) {
        System.out.println("Last-Event-ID : " + dernier);
        return mesures().map(m -> ServerSentEvent.builder(m)
            .id(Long.toString(m.numero())).event("mesure").build());
    }
}
```

Depuis le dossier `demo-sse` :

```bash
mvn spring-boot:run
```

Observer d'abord le JSON sans fin :

```bash
curl -N --max-time 4 http://localhost:8080/temperatures/json
```

Le tableau commence mais ne se ferme pas. Passer ensuite au SSE :

```bash
curl -N --max-time 4 http://localhost:8080/temperatures
```

Observer une ligne `data:` par mesure et une ligne vide entre événements. Prédire ce qui arrive au capteur quand le client ferme ; à la fin des quatre secondes, lire `[capteur] annulation` côté serveur. Le code d'erreur de délai de `curl` est voulu ici.

Enfin, observer les champs nommés et la réception du dernier identifiant :

```bash
curl -N --max-time 4 -H 'Last-Event-ID: 2' http://localhost:8080/temperatures/nommees
```

Les événements portent `id:`, `event:mesure` et `data:`. Le serveur affiche l'en-tête `2`, mais le compteur repart à zéro : cet exemple reçoit l'identifiant sans implémenter la reprise. Arrêter le serveur.

Récapitulatif de ce qu'on doit observer :

| Requête | Attendu |
|---|---|
| Flux sans type déclaré | un tableau JSON qui commence et ne se ferme jamais |
| SSE | une ligne `data:` par mesure, une ligne vide entre deux événements |
| Fermeture de `curl` | `[capteur] annulation` dans la console du serveur |
| Événements nommés | `id:`, `event:mesure` et `data:` ; l'en-tête `2` affiché par le serveur |

Source : [Server-sent events, standard HTML](https://html.spec.whatwg.org/multipage/server-sent-events.html).

# TP - WebSocket

| Projet | `demo-websocket` |
|---|---|
| Dépendances | `spring-boot-starter-webflux` |
| Classe de démarrage | `com.example.demo.DemoWebsocketApplication` |
| Fichiers | `EchoWebSocket.java` |
| Sur le poste | un navigateur et sa console de développement |

`EchoWebSocket.java`. La configuration associe `/echo` au handler ; celui-ci renvoie chaque texte avec le préfixe `écho :`.

```java
package com.example.demo;

import java.util.Map;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.HandlerMapping;
import org.springframework.web.reactive.handler.SimpleUrlHandlerMapping;
import org.springframework.web.reactive.socket.WebSocketHandler;
import org.springframework.web.reactive.socket.WebSocketMessage;
import org.springframework.web.reactive.socket.server.support.WebSocketHandlerAdapter;

@Configuration
public class EchoWebSocket {
    @Bean
    WebSocketHandler echoHandler() {
        return session -> session.send(session.receive()
            .map(WebSocketMessage::getPayloadAsText)
            .map(texte -> session.textMessage("écho : " + texte)));
    }
    @Bean
    HandlerMapping echoMapping(WebSocketHandler echoHandler) {
        return new SimpleUrlHandlerMapping(Map.of("/echo", echoHandler), -1);
    }
    @Bean
    WebSocketHandlerAdapter echoAdapter() { return new WebSocketHandlerAdapter(); }
}
```

Depuis le dossier `demo-websocket` :

```bash
mvn spring-boot:run
```

Ouvrir `http://localhost:8080/` dans le navigateur, puis sa console de développement, même si la page répond 404. Cela place le client sur le même serveur HTTP local. Coller :

```javascript
const ws = new WebSocket("ws://localhost:8080/echo");
ws.onmessage = evenement => console.log(evenement.data);
ws.onerror = evenement => console.error("connexion WebSocket en échec", evenement);
ws.onopen = () => ws.send("bonjour");
```

Attendu : `écho : bonjour`. Envoyer ensuite un autre message sur la même connexion :

```javascript
ws.send("encore");
```

Fermer la connexion avant de relancer la manipulation :

```javascript
ws.close();
```

Source : [WebSockets avec WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux-websocket.html).

## Filtrer et passer en majuscules

Modifier le handler pour filtrer les textes vides et renvoyer le texte en majuscules. Redémarrer le serveur, rouvrir la connexion du navigateur, puis envoyer `bonjour`, puis une chaîne vide.

Sortie attendue : une seule réponse, `BONJOUR`. La chaîne vide n'en produit aucune. Le corrigé est à la fin de ce chapitre.

## Corrigé - WebSocket


Dans `EchoWebSocket.java`, du projet `demo-websocket`, remplacer dans le bean `echoHandler` la construction des réponses par :

```java
return session -> session.send(session.receive()
    .map(WebSocketMessage::getPayloadAsText)
    .filter(texte -> !texte.isBlank())
    .map(texte -> session.textMessage(texte.toUpperCase(java.util.Locale.ROOT))));
```

Redémarrer le serveur, rouvrir la connexion du navigateur et envoyer `bonjour`, puis `""`. Une seule réponse revient : `BONJOUR`.

# TP - Appeler et diffuser

**Objectif :** appeler un service distant sans bloquer, et diffuser au client un flux continu d'événements.

| Dossier à ouvrir | `demo-application-reactive`, après le TP « Exposer une API WebFlux » |
|---|---|
| Avant de commencer | le projet autonome `demo-webclient` démarré sur le port 8081, par la commande donnée dans « Le code de départ » |
| Commande | `./mvnw spring-boot:run` depuis `demo-application-reactive`, puis `curl -N localhost:8080/tarifs/1/direct` |

Une boutique affiche le tarif d'un article en direct. Le tarif vient d'un service distant, qui répond en 300 ms. L'application de `demo-application-reactive` l'interroge chaque seconde avec `WebClient`, et pousse chaque tarif reçu au client en Server-Sent Events. Le service distant peut s'arrêter : le client doit alors continuer à recevoir un événement par seconde, qui signale que le tarif est indisponible.

Deux applications tournent en même temps :

| Application | Port | Rôle |
|---|---|---|
| `demo-webclient`, fourni prêt à lancer | 8081 | le service distant : `GET /tarifs/{id}` répond en 300 ms |
| `demo-application-reactive` | 8080 | l'application à écrire : elle appelle le service distant et diffuse les tarifs |

Le service distant est `ServiceTarifs`, celui du TP « WebClient et service de tarifs » :

| Requête | Réponse |
|---|---|
| `GET /tarifs/1` | `{"id":"1","montant":49.0}`, après 300 ms |
| `GET /tarifs/2` | `{"id":"2","montant":19.0}`, après 300 ms |
| `GET /tarifs/3` | `{"id":"3","montant":189.0}`, après 300 ms |
| `GET /tarifs/9` | `404 Not Found` |

> [!IMPORTANT]
> Le TP n'utilise que les tarifs 1 à 3. Le service distant est le projet `demo-webclient` : on l'arrête et on le relance pendant le TP, sans toucher à son code.

## Le code de départ

Prérequis :

- le projet `demo-webclient`, fourni prêt à lancer, qui contient `ServiceTarifs.java` ;
- `demo-application-reactive`, dans son état des TP précédents. WebFlux y est déjà présent.

Dans `demo-application-reactive`, ajouter au `pom.xml`, dans `<dependencies>`, le starter qui fournit le `WebClient.Builder` préconfiguré :

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webclient</artifactId>
</dependency>
```

Puis créer un seul fichier, `Tarif.java`, dans le même package que la classe d'application. Garder la ligne `package` que l'IDE place en tête du fichier, et coller le reste.

`Tarif.java`

```java
// Un tarif reçu du service distant. Le montant vaut null quand le tarif est indisponible.
record Tarif(String id, Double montant) {}
```

Ce que le projet contient pour ce TP : le record `Tarif`, qui reprend les deux champs du JSON du service distant. Le montant est un `Double`, et non un `double`, pour pouvoir valoir `null` à l'étape 2.

Ce qu'il ne contient pas : aucun appel au service distant, aucune route `/tarifs`. Les deux classes à écrire sont nommées dans les étapes. Aucune page web n'est fournie : tout se vérifie avec `curl`.

Démarrer le service distant, depuis le dossier `demo-webclient`, dans un premier terminal :

```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8081"
```

Vérifier dans un deuxième terminal :

```bash
curl localhost:8081/tarifs/1
```

```text
{"id":"1","montant":49.0}
```

Démarrer ensuite `demo-application-reactive`, depuis son dossier, puis lancer :

```bash
curl -i localhost:8080/tarifs/1/direct
```

La réponse est `404 Not Found` : aucune route ne répond encore.

## Étape 1 : relayer les tarifs en SSE

Créer deux classes.

`TarifClient`, un bean `@Service` qui appelle le service distant :

1. Le constructeur reçoit le `WebClient.Builder` de Spring Boot et l'adresse du service distant, lue dans la propriété `tarifs.url`, `http://localhost:8081` par défaut. Il en construit un `WebClient` dont c'est l'adresse de base.
2. La méthode `Mono<Tarif> tarif(String id)` envoie `GET /tarifs/{id}` et convertit le corps de la réponse en `Tarif`.

`TarifController`, un contrôleur annoté :

1. `GET /tarifs/{id}/direct` renvoie un `Flux<Tarif>` en Server-Sent Events.
2. Chaque seconde, il appelle `tarif(id)` et pousse le tarif reçu au client.
3. Le flux ne s'arrête pas tant que le client reste connecté.

Critère de réussite : avec le service distant démarré, la commande ci-dessous affiche une ligne `data:` par seconde environ, séparée de la suivante par une ligne vide.

```bash
curl -N localhost:8080/tarifs/1/direct
```

```text
data:{"id":"1","montant":49.0}

data:{"id":"1","montant":49.0}

data:{"id":"1","montant":49.0}
```

Le terminal du service distant affiche une ligne `appel tarif 1` par seconde. Arrêter `curl` par Ctrl-C : les lignes `appel tarif 1` cessent aussitôt.

> [!NOTE]
> **Question : L'application ne démarre pas : « required a bean of type 'org.springframework.web.reactive.function.client.WebClient$Builder' that could not be found ». Pourquoi ?**
> Depuis Spring Boot 4, le `WebClient.Builder` préconfiguré vient du starter `spring-boot-starter-webclient`, distinct du starter WebFlux. Vérifier qu'il a été ajouté au `pom.xml`, puis recharger le projet Maven dans l'IDE.

> [!NOTE]
> **Question : Le compilateur refuse la méthode du contrôleur : « incompatible types: inference variable V has incompatible bounds ». Pourquoi ?**
> Chaque seconde produit un appel, et chaque appel est un `Mono<Tarif>`. Avec `map`, on obtient un flux de `Mono`, pas un flux de tarifs. `flatMap` s'abonne à chaque `Mono` et transmet le tarif qu'il produit.

> [!NOTE]
> **Question : `curl` affiche une erreur 500 au bout d'une seconde, puis s'arrête. Pourquoi ?**
> Le service distant ne tourne pas : le premier appel échoue, et l'erreur arrive avant que la réponse ait commencé. Démarrer `demo-webclient` sur le port 8081.

> [!NOTE]
> **Question : Les tarifs s'affichent sans `data:`, dans un tableau JSON. Pourquoi ?**
> Sans type de média déclaré, un `Flux` est envoyé en `application/json`, dans un tableau qui ne se ferme jamais. Déclarer le type `text/event-stream` sur la route.

## Étape 2 : un repli plutôt qu'une coupure

Avant d'écrire la moindre ligne, noter sa prédiction. `curl -N localhost:8080/tarifs/1/direct` tourne, et le service distant s'arrête. Que se passe-t-il pour `curl` ?

- Il continue d'afficher le dernier tarif reçu.
- Il reste ouvert, sans rien afficher.
- Il s'arrête.

Constater, avec le code de l'étape 1 :

1. Lancer `curl -N localhost:8080/tarifs/1/direct`, et laisser passer deux ou trois lignes.
2. Arrêter le service distant par Ctrl-C dans son terminal.
3. Observer `curl`, puis la console de `demo-application-reactive`, et comparer à la prédiction.

Puis modifier `TarifController` : quand un appel au service distant échoue, pousser `new Tarif(id, null)` à la place du tarif. Le flux doit continuer, une ligne par seconde, que le service distant réponde ou non.

Critère de réussite, avec `curl -N localhost:8080/tarifs/1/direct` lancé et laissé ouvert pendant toute la manipulation :

| Moment | Ce que `curl` affiche |
|---|---|
| Service distant démarré | `data:{"id":"1","montant":49.0}`, une fois par seconde |
| Après Ctrl-C dans le terminal du service distant | `data:{"id":"1","montant":null}`, une fois par seconde, sans interruption |
| Après la relance du service distant, par la même commande | de nouveau `data:{"id":"1","montant":49.0}`, sans relancer `curl` |

> [!NOTE]
> **Question : Après l'arrêt du service distant, `curl` affiche une seule ligne `null`, puis s'arrête. Pourquoi ?**
> Le repli est placé après le `flatMap`, sur le flux entier. L'erreur du premier appel en échec arrive jusqu'au flux principal : `onErrorReturn` la remplace par une dernière valeur, puis termine le flux, comme pour toute erreur Reactor. Le repli doit être placé sur chaque appel, à l'intérieur du `flatMap` : l'erreur d'un appel ne remonte alors plus jusqu'au flux.

> [!NOTE]
> **Question : Après la relance du service distant, quelques lignes `null` continuent d'arriver. Est-ce une erreur ?**
> Non. Le service distant met quelques secondes à démarrer ; pendant ce temps, les appels échouent encore. Le premier tarif revient à la seconde qui suit son démarrage.

> [!TIP]
> **Ce qu'il faut retenir**
> - `WebClient` appelle un service distant sans bloquer : l'appel renvoie un `Mono`, et part à l'abonnement.
> - Pour transformer chaque seconde en appel, `flatMap`, et non `map`.
> - Un `Flux` déclaré en `text/event-stream` pousse chaque élément au client, sur une connexion qui reste ouverte.
> - Une erreur qui atteint le flux principal le termine : le repli se place sur chaque appel, à l'intérieur du `flatMap`.

## Corrigé - Appeler et diffuser


### TarifClient.java

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;

import reactor.core.publisher.Mono;

@Service
class TarifClient {

    private final WebClient client;

    TarifClient(WebClient.Builder builder,
                @Value("${tarifs.url:http://localhost:8081}") String url) {
        this.client = builder.baseUrl(url).build();
    }

    Mono<Tarif> tarif(String id) {
        return client.get()
                     .uri("/tarifs/{id}", id)
                     .retrieve()
                     .bodyToMono(Tarif.class);
    }
}
```

### TarifController.java, après l'étape 1

```java
import java.time.Duration;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;

@RestController
class TarifController {

    private final TarifClient tarifs;

    TarifController(TarifClient tarifs) {
        this.tarifs = tarifs;
    }

    @GetMapping(value = "/tarifs/{id}/direct", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    Flux<Tarif> direct(@PathVariable String id) {
        return Flux.interval(Duration.ofSeconds(1))
                   .flatMap(n -> tarifs.tarif(id));
    }
}
```

**Étape 1.** `TarifClient` reçoit le `WebClient.Builder` de Spring Boot et y fixe l'adresse du service distant. `@Value("${tarifs.url:http://localhost:8081}")` lit cette adresse dans la propriété `tarifs.url`, et prend `http://localhost:8081` quand la propriété n'est pas définie : un test peut ainsi pointer le client vers un autre serveur. `tarif` décrit la requête ; rien ne part tant que personne ne s'abonne. `Flux.interval` émet un nombre chaque seconde, et `flatMap` transforme chacun en appel, puis transmet le tarif reçu : avec `map`, le flux contiendrait des `Mono`, et le compilateur refuse le type de retour. `produces = MediaType.TEXT_EVENT_STREAM_VALUE` fait de chaque tarif un événement SSE, sur une connexion qui reste ouverte. C'est WebFlux qui s'abonne au flux, à l'arrivée de la requête ; quand `curl` se ferme, il annule le flux, et les appels cessent.

### TarifController.java, après l'étape 2

Seule la méthode `direct` change : le repli est placé sur chaque appel.

```java
import java.time.Duration;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;

@RestController
class TarifController {

    private final TarifClient tarifs;

    TarifController(TarifClient tarifs) {
        this.tarifs = tarifs;
    }

    @GetMapping(value = "/tarifs/{id}/direct", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    Flux<Tarif> direct(@PathVariable String id) {
        return Flux.interval(Duration.ofSeconds(1))
                   .flatMap(n -> tarifs.tarif(id)
                                       .onErrorReturn(new Tarif(id, null)));
    }
}
```

**Étape 2.** La prédiction juste était : `curl` s'arrête. Le premier appel en échec produit une `WebClientRequestException`, « Connection refused ». L'erreur remonte jusqu'au flux renvoyé par le contrôleur, et la réponse SSE a déjà commencé : WebFlux ne peut plus envoyer de code d'erreur, et ferme la connexion. La console du projet l'écrit : `ServerHttpResponse already committed (200 OK)`, puis `Closing connection`. Avec `onErrorReturn` placé sur l'appel, l'erreur s'arrête au `Mono` de cet appel : il produit `Tarif(id, null)` à la place du tarif, et le flux principal continue. À la seconde suivante, un nouvel appel part ; dès que le service distant répond de nouveau, les vrais tarifs reviennent, sur la même connexion.

# TP - MongoDB réactif

Ce TP enregistre et relit des articles dans MongoDB, sans qu'aucun thread n'attende la base. Elle regarde ensuite ce que Spring a réellement écrit en base, puis ce que le serveur lit quand le client lit lentement.

| Projet | `demo-mongodb` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-data-mongodb-reactive` |
| Classe de démarrage | `com.example.demo.DemoMongodbApplication` |
| Fichiers | paquet `com.example.demo` : `Article.java`, `ArticleRepository.java`, `ArticleController.java`, `CommandesMongo.java` |
| Sur le poste | Docker et `curl` ; collection `articles` de la base `test` |

Aucune propriété n'est à configurer : sans configuration, Spring Boot se connecte à `mongodb://localhost/test`, c'est-à-dire au port 27017 et à la base `test`.

## Enregistrer et relire

Lancer MongoDB dans un conteneur nommé `mongo` :

```bash
docker run -d --name mongo -p 27017:27017 mongo:8
```

> [!WARNING]
> **Le conteneur existe déjà**
> Un second `docker run` échoue sur un conflit de nom. Le conteneur créé une première fois se relance avec `docker start mongo`. C'est le même conteneur qui servira au TP du chapitre.

Le paquet `com.example.demo` contient quatre fichiers, dans `src/main/java/com/example/demo/` : le document `Article`, son dépôt `ArticleRepository`, le contrôleur `ArticleController`, et `CommandesMongo`, qui affiche dans la console chaque lecture que le pilote envoie à MongoDB.

`Article.java`

```java
package com.example.demo;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("articles")
public record Article(@Id String id, String titre, double prix, String categorie) {}
```

`@Document` associe le record à la collection `articles`. `@Id` désigne l'identifiant : laissé à `null`, il est généré à l'enregistrement.

`ArticleRepository.java`

```java
package com.example.demo;

import org.springframework.data.mongodb.repository.ReactiveMongoRepository;
import reactor.core.publisher.Flux;

public interface ArticleRepository extends ReactiveMongoRepository<Article, String> {
    Flux<Article> findByCategorie(String categorie);
}
```

`ReactiveMongoRepository` fournit `save`, `findById`, `findAll`, `deleteById`… qui renvoient des `Mono` et des `Flux`. `findByCategorie` est une requête dérivée : Spring la déduit du nom de la méthode, comme avec Spring Data JPA.

`ArticleController.java`

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/articles")
public class ArticleController {

    private final ArticleRepository depot;

    public ArticleController(ArticleRepository depot) { this.depot = depot; }

    @GetMapping
    public Flux<Article> tous() { return depot.findAll(); }

    @GetMapping(params = "categorie")
    public Flux<Article> parCategorie(@RequestParam String categorie) {
        return depot.findByCategorie(categorie);
    }

    @PostMapping
    public Mono<Article> creer(@RequestBody Article article) { return depot.save(article); }
}
```

Le contrôleur renvoie directement ce que renvoie le dépôt : WebFlux s'abonne au `Mono` ou au `Flux`.

`CommandesMongo.java`

```java
package com.example.demo;

import com.mongodb.event.CommandListener;
import com.mongodb.event.CommandStartedEvent;
import org.springframework.boot.mongodb.autoconfigure.MongoClientSettingsBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CommandesMongo {

    private long debut;
    private int total;

    @Bean
    MongoClientSettingsBuilderCustomizer afficherLesCommandes() {
        return reglages -> reglages.addCommandListener(new CommandListener() {
            @Override
            public void commandStarted(CommandStartedEvent e) {
                String commande = e.getCommandName();
                if (commande.equals("find")) { debut = System.currentTimeMillis(); total = 0; }
                else if (!commande.equals("getMore") && !commande.equals("killCursors")) return;
                var lot = e.getCommand().get("batchSize");
                if (lot != null) total += lot.asInt32().getValue();
                System.out.printf("[mongo] t=%d ms %s%s%n", System.currentTimeMillis() - debut, commande,
                        lot == null ? "" : " batchSize=" + lot.asInt32().getValue() + " total=" + total);
            }
        });
    }
}
```

`CommandesMongo` ne sert qu'à observer : il ajoute au client MongoDB un écouteur de commandes. Une lecture commence par une commande `find`, qui demande un premier lot de documents ; chaque lot suivant est demandé par un `getMore` ; `killCursors` abandonne une lecture inachevée. `batchSize` est le nombre de documents demandés dans le lot, `total` leur somme depuis le `find`, et `t` le temps écoulé depuis le `find`.

Démarrer le projet, depuis le dossier `demo-mongodb` :

```bash
mvn spring-boot:run
```

Dans un second terminal, créer un article. On n'envoie pas d'identifiant :

```bash
curl -s -X POST localhost:8080/articles -H 'Content-Type: application/json' -d '{"titre":"Clavier","prix":49.0,"categorie":"bureau"}'
```

```text
{"id":"6ad98581286dbfe67c0d03c7","titre":"Clavier","prix":49.0,"categorie":"bureau"}
```

L'article revient avec l'identifiant donné par MongoDB, différent à chaque création. Relire tous les articles :

```bash
curl -s localhost:8080/articles
```

```text
[{"id":"6ad98581286dbfe67c0d03c7","titre":"Clavier","prix":49.0,"categorie":"bureau"}]
```

Puis ceux d'une catégorie, par la requête dérivée :

```bash
curl -s "localhost:8080/articles?categorie=bureau"
```

> [!NOTE]
> **Question : où se règle l'adresse de la base ?**
> Dans la propriété `spring.mongodb.uri`, par exemple `spring.mongodb.uri=mongodb://serveur:27017/catalogue`. Avant Spring Boot 4, elle s'appelait `spring.data.mongodb.uri`. L'ancien nom n'est plus lu, et sans aucun message : l'application démarre et se connecte à `localhost:27017`, comme si rien n'était écrit.

## Le document en base

L'API renvoie un champ `id`. Avant d'exécuter la commande suivante, écrire le document que MongoDB a enregistré pour l'article créé : quels champs, sous quels noms ?

`mongosh` est le shell de MongoDB ; il est présent dans le conteneur :

```bash
docker exec mongo mongosh test --quiet --eval 'db.articles.findOne()'
```

```text
{
  _id: ObjectId('6ad98581286dbfe67c0d03c7'),
  titre: 'Clavier',
  prix: 49,
  categorie: 'bureau',
  _class: 'com.example.demo.Article'
}
```

Deux différences avec le JSON de l'API :

- le composant `id` du record est stocké dans `_id`, le nom que MongoDB impose à l'identifiant, et converti en `ObjectId`. À la lecture, Spring le rend en chaîne de caractères ;
- un cinquième champ, `_class`, porte le nom complet de la classe Java. Spring s'en sert pour savoir quel type recréer à la lecture.

`prix` s'affiche `49` : `mongosh` n'écrit pas la décimale d'un nombre à virgule entier.

Source : [Spring Data MongoDB, Type Mapping](https://docs.spring.io/spring-data/mongodb/reference/mongodb/converters-type-mapping.html).

## Un client lent

Remplacer le contenu de la collection par 300 000 articles, environ 27 Mo de JSON. La commande prend quelques secondes et affiche le nombre de documents :

```bash
docker exec mongo mongosh test --quiet --eval 'db.articles.drop(); for (let k = 0; k < 30; k++) { const lot = []; for (let i = 1; i <= 10000; i++) lot.push({ titre: "Article " + (k * 10000 + i), prix: i % 100, categorie: "bureau" }); db.articles.insertMany(lot); } db.articles.countDocuments()'
```

```text
300000
```

> [!WARNING]
> **Ne plus relire la collection sans limite**
> Un `curl localhost:8080/articles` sans `-o` déverse désormais 27 Mo dans le terminal, et la requête par catégorie aussi, puisque les 300 000 articles sont de la catégorie `bureau`.

Le client de la commande suivante lit 20 Ko par seconde (`--limit-rate 20k`), écrit ce qu'il reçoit dans `lent.json`, et s'arrête au bout de 10 s (`-m 10`). Avant de la lancer, prédire combien d'articles le serveur lit en base pendant ces 10 s : les 300 000 ? à peu près ce que le client reçoit ? autre chose ?

```bash
curl -s --limit-rate 20k -m 10 -o lent.json localhost:8080/articles
```

`curl` se termine au bout de 10 s avec le code 28, celui d'un délai dépassé : il est voulu ici. Compter les articles reçus :

```bash
grep -o '"id"' lent.json | wc -l
```

```text
2487
```

Lire ensuite les lignes `[mongo]` de la console du projet. Exécuté sur Spring Boot 4.1.1, pilote MongoDB 5.8.1 et MongoDB 8.3.13 :

```text
[mongo] t=0 ms find batchSize=256 total=256
[mongo] t=29 ms getMore batchSize=173 total=429
[mongo] t=43 ms getMore batchSize=15 total=444
…
[mongo] t=640 ms getMore batchSize=254 total=9216
[mongo] t=10011 ms killCursors
```

Les tailles de lots varient d'une exécution à l'autre, et le total avec elles : entre 8 896 et 9 216 articles sur onze exécutions, alors que `curl` en a reçu 2 487 à chaque fois. Ici, le serveur a lu 9 216 articles en 0,6 s, puis n'a plus rien lu pendant 9,4 s :

- le pilote lit par lots, et la taille de chaque lot est la demande de l'aval. Il ne demande le lot suivant que lorsque l'aval a de nouveau demandé ;
- les 6 400 à 6 700 articles lus mais pas encore reçus attendent dans les tampons du serveur et du réseau. Une fois cette réserve pleine, la demande s'arrête, et la lecture en base avec elle ;
- à l'arrêt de `curl`, WebFlux annule le flux, et le pilote abandonne la lecture : c'est le `killCursors`.

La backpressure va donc jusqu'à la base, au-delà d'une réserve de quelques milliers d'articles. Une petite collection, elle, est lue en entier : avec 10 000 articles, 900 Ko, et le même client, toute la collection est lue en 0,6 s.

Arrêter le projet avec Ctrl-C, puis vider la collection, qui n'a plus d'usage :

```bash
docker exec mongo mongosh test --quiet --eval 'db.articles.drop()'
```

Le conteneur `mongo` peut rester démarré : le TP du chapitre s'en sert.

> [!TIP]
> **Ce qu'il faut retenir**
> - Un dépôt réactif étend `ReactiveMongoRepository` et renvoie des `Mono` et des `Flux` ; les requêtes dérivées du nom de méthode fonctionnent comme avec Spring Data JPA.
> - Le document en base n'est pas le JSON de l'API : `id` devient `_id`, et Spring ajoute `_class`.
> - Le pilote lit par lots, à la taille de la demande : un client lent arrête la lecture en base une fois les tampons pleins, et sa déconnexion ferme le curseur.

Source : [Spring Boot, Working with NoSQL Technologies](https://docs.spring.io/spring-boot/reference/data/nosql.html).

# TP - Requêtes avec ReactiveMongoTemplate

Ce TP cherche des articles selon des critères facultatifs, puis baisse de 10 % le prix d'une catégorie entière de deux façons : en relisant chaque article avec le dépôt, et en une seule commande avec `ReactiveMongoTemplate`.

| Projet | `demo-mongo-template` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-data-mongodb-reactive` |
| Classe de démarrage | `com.example.demo.DemoMongoTemplateApplication` |
| Fichiers | paquet `com.example.demo` : `Article.java` et `ArticleRepository.java`, identiques à ceux de `demo-mongodb` et montrés dans le TP « MongoDB réactif », puis `CatalogueArticles.java` et `RechercheController.java` |
| Sur le poste | Docker et `curl` ; collection `articles` de la base `test` |

## Chercher avec trois critères

Démarrer MongoDB. Au premier usage, le conteneur se crée par `docker run -d --name mongo -p 27017:27017 mongo:8` ; ensuite, il se relance :

```bash
docker start mongo
```

Le paquet `com.example.demo` contient, à côté du document `Article` et de son dépôt `ArticleRepository`, deux fichiers : `CatalogueArticles`, qui construit les requêtes, et `RechercheController`, qui les expose.

`CatalogueArticles.java`

```java
package com.example.demo;

import com.mongodb.client.result.UpdateResult;
import org.springframework.data.domain.Sort;
import org.springframework.data.mongodb.core.ReactiveMongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Service
public class CatalogueArticles {

    private final ReactiveMongoTemplate template;
    private final ArticleRepository depot;

    public CatalogueArticles(ReactiveMongoTemplate template, ArticleRepository depot) {
        this.template = template; this.depot = depot;
    }

    public Flux<Article> chercher(String categorie, Double max, String mot) {
        Query requete = new Query();
        if (categorie != null) requete.addCriteria(Criteria.where("categorie").is(categorie));
        if (max != null)       requete.addCriteria(Criteria.where("prix").lt(max));
        if (mot != null)       requete.addCriteria(Criteria.where("titre").regex(mot, "i"));
        return template.find(requete.with(Sort.by("prix")), Article.class);
    }

    public Mono<Long> solder(String categorie) {
        Query requete = Query.query(Criteria.where("categorie").is(categorie));
        Update baisse = new Update().multiply("prix", 0.9);
        return template.updateMulti(requete, baisse, Article.class)
                       .map(UpdateResult::getModifiedCount);
    }

    public Mono<Long> solderAvecLeDepot(String categorie) {
        return depot.findByCategorie(categorie)
                .map(a -> new Article(a.id(), a.titre(), a.prix() * 0.9, a.categorie()))
                .flatMap(depot::save)
                .count();
    }
}
```

`ReactiveMongoTemplate` est fourni par Spring Boot avec le starter réactif : il suffit de le demander dans le constructeur.

- `chercher` part d'une requête vide et ajoute un critère par paramètre reçu : `is` pour une égalité, `lt` pour « strictement inférieur », `regex(mot, "i")` pour un mot contenu dans le titre, sans tenir compte des majuscules. Sans aucun paramètre, la requête renvoie tous les articles. `Sort.by("prix")` les trie par prix.
- `solder` décrit la modification, multiplier `prix` par 0,9, et `updateMulti` l'applique à tous les articles de la catégorie. Le `Mono` livre le nombre de documents modifiés.
- `solderAvecLeDepot` fait le même travail avec le dépôt : il relit chaque article, calcule son nouveau prix, puis l'enregistre avec `save`.

`RechercheController.java`

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/articles")
public class RechercheController {

    private final CatalogueArticles catalogue;

    public RechercheController(CatalogueArticles catalogue) { this.catalogue = catalogue; }

    @GetMapping("/recherche")
    public Flux<Article> recherche(@RequestParam(required = false) String categorie,
                                   @RequestParam(required = false) Double max,
                                   @RequestParam(required = false) String mot) {
        return catalogue.chercher(categorie, max, mot);
    }

    @PostMapping("/solde")
    public Mono<Long> solde(@RequestParam String categorie) { return catalogue.solder(categorie); }

    @PostMapping("/solde-depot")
    public Mono<Long> soldeDepot(@RequestParam String categorie) { return catalogue.solderAvecLeDepot(categorie); }
}
```

Les trois paramètres de la recherche sont facultatifs : `required = false`, et un type objet, `Double` et non `double`, pour qu'un paramètre absent puisse valoir `null`.

Démarrer le projet :

```bash
mvn spring-boot:run
```

Dans un second terminal, enregistrer six articles :

```bash
docker exec mongo mongosh test --quiet --eval 'db.articles.drop(); db.articles.insertMany([{titre:"Clavier",prix:49.0,categorie:"bureau"},{titre:"Souris",prix:25.0,categorie:"bureau"},{titre:"Écran",prix:189.0,categorie:"bureau"},{titre:"Clavier mécanique",prix:119.0,categorie:"bureau"},{titre:"Lampe",prix:35.0,categorie:"maison"},{titre:"Coussin",prix:19.0,categorie:"maison"}]); db.articles.countDocuments()'
```

```text
6
```

Puis chercher, en combinant les critères :

```bash
curl -s "localhost:8080/articles/recherche?max=50"; echo
curl -s "localhost:8080/articles/recherche?categorie=bureau&mot=clav"; echo
curl -s "localhost:8080/articles/recherche?categorie=bureau&max=50&mot=clav"; echo
```

```text
[{"id":"6ad9dce81f7211b893f5851b","titre":"Coussin","prix":19.0,"categorie":"maison"},{"id":"6ad9dce81f7211b893f58517","titre":"Souris","prix":25.0,"categorie":"bureau"},{"id":"6ad9dce81f7211b893f5851a","titre":"Lampe","prix":35.0,"categorie":"maison"},{"id":"6ad9dce81f7211b893f58516","titre":"Clavier","prix":49.0,"categorie":"bureau"}]
[{"id":"6ad9dce81f7211b893f58516","titre":"Clavier","prix":49.0,"categorie":"bureau"},{"id":"6ad9dce81f7211b893f58519","titre":"Clavier mécanique","prix":119.0,"categorie":"bureau"}]
[{"id":"6ad9dce81f7211b893f58516","titre":"Clavier","prix":49.0,"categorie":"bureau"}]
```

Les identifiants changent à chaque insertion. La même méthode répond aux trois recherches : seuls les critères reçus filtrent, et les articles arrivent triés par prix.

> [!NOTE]
> **Question : un index sur `titre` accélérerait-il la recherche par mot ?**
> Non. Une expression régulière insensible aux majuscules, comme `regex(mot, "i")`, ne profite pas d'un index : MongoDB parcourt tout l'index au lieu de toute la collection. Exécuté sur MongoDB 8.3.13 avec 10 004 articles : sans index, 10 004 documents examinés ; avec un index sur `titre`, 10 004 entrées d'index examinées. Un index texte ne convient pas non plus : il cherche des mots entiers, et ne trouve rien pour `clav`. Source : [MongoDB, $regex, Index Use](https://www.mongodb.com/docs/manual/reference/operator/query/regex/).

## Solder une catégorie

Remplacer le contenu de la collection par 10 000 articles de la catégorie `bureau` :

```bash
docker exec mongo mongosh test --quiet --eval 'db.articles.drop(); const lot = []; for (let i = 1; i <= 10000; i++) lot.push({ titre: "Article " + i, prix: 10 + i % 100, categorie: "bureau" }); db.articles.insertMany(lot); db.articles.countDocuments()'
```

```text
10000
```

Avant de lancer les deux soldes, prédire combien de commandes `update` chacun envoie à MongoDB : `solderAvecLeDepot`, qui relit et enregistre chaque article, et `solder`, qui appelle `updateMulti`.

MongoDB compte les commandes qu'il reçoit. Ce compteur se lit avec `mongosh` :

```bash
docker exec mongo mongosh test --quiet --eval 'Number(db.serverStatus().opcounters.update)'
```

Il s'additionne depuis le démarrage du conteneur : seul compte l'écart entre deux lectures. Lire le compteur, solder avec le dépôt, puis relire le compteur :

```bash
docker exec mongo mongosh test --quiet --eval 'Number(db.serverStatus().opcounters.update)'
curl -s -w ' en %{time_total} s\n' -X POST "localhost:8080/articles/solde-depot?categorie=bureau"
docker exec mongo mongosh test --quiet --eval 'Number(db.serverStatus().opcounters.update)'
```

```text
0
10000 en 4.365788 s
10000
```

Recréer les 10 000 articles avec la commande précédente, puis faire de même avec `updateMulti` :

```bash
docker exec mongo mongosh test --quiet --eval 'Number(db.serverStatus().opcounters.update)'
curl -s -w ' en %{time_total} s\n' -X POST "localhost:8080/articles/solde?categorie=bureau"
docker exec mongo mongosh test --quiet --eval 'Number(db.serverStatus().opcounters.update)'
```

```text
10000
10000 en 0.124726 s
10001
```

Exécuté sur Spring Boot 4.1.1, Spring Data MongoDB 5.1.1 et MongoDB 8.3.13. Les deux soldes modifient 10 000 articles, mais pas de la même façon :

- le dépôt envoie une commande `update` par article : il relit l'article, calcule le prix en Java, puis renvoie le document entier. Sur neuf exécutions, de 1 à 4,4 s ;
- `updateMulti` envoie une seule commande, qui demande à MongoDB de multiplier le prix de tous les articles de la catégorie. Entre 0,07 et 0,16 s.

Les durées varient d'une exécution à l'autre, la première étant la plus lente ; l'écart du compteur, lui, ne varie pas.

> [!NOTE]
> **Question : un dépôt ne sait-il donc pas modifier plusieurs documents d'un coup ?**
> Si : une méthode de dépôt annotée `@Update` le fait, par exemple `@Update("{ '$mul' : { 'prix' : ?1 } }") Mono<Long> findAndMultiplyPrixByCategorie(String categorie, double facteur);`. Exécuté : la même commande `update` unique que `updateMulti`. Ce qui coûte 10 000 commandes, c'est relire puis enregistrer avec `save`. Le template reste nécessaire quand la requête elle-même dépend des paramètres reçus. Source : [Spring Data MongoDB, Update Methods](https://docs.spring.io/spring-data/mongodb/reference/mongodb/repositories/modifying-methods.html).

Arrêter le projet avec Ctrl-C, puis vider la collection :

```bash
docker exec mongo mongosh test --quiet --eval 'db.articles.drop()'
```

> [!TIP]
> **Ce qu'il faut retenir**
> - `ReactiveMongoTemplate` construit la requête en Java, un critère par paramètre reçu, et renvoie un `Flux`.
> - `updateMulti` modifie tous les documents concernés en une seule commande, sans les relire ; relire puis enregistrer avec `save` coûte une commande par document.
> - Une expression régulière insensible aux majuscules parcourt toute la collection, ou tout l'index.

Source : [Spring Data MongoDB, Template API](https://docs.spring.io/spring-data/mongodb/reference/mongodb/template-crud-operations.html).

# TP - Curseur tailable

Ce TP affiche en direct les commandes enregistrées en base, sans relire la collection. Elle observe ensuite ce que le serveur envoie à MongoDB quand personne ne commande.

| Projet | `demo-tailable` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-data-mongodb-reactive` |
| Classe de démarrage | `com.example.demo.DemoTailableApplication` |
| Fichiers | paquet `com.example.demo` : `CommandesMongo.java`, identique à celui de `demo-mongodb` et montré dans le TP « MongoDB réactif », puis `Commande.java`, `JournalCommandes.java`, `CommandeRepository.java` et `CommandeController.java` |
| Sur le poste | Docker et `curl` ; collection plafonnée `commandes` de la base `test` |

`CommandesMongo` affiche dans la console chaque lecture que le pilote envoie à MongoDB : une ligne `[mongo]` par commande `find`, `getMore` ou `killCursors`.

## Les commandes en direct

Démarrer MongoDB. Au premier usage, le conteneur se crée par `docker run -d --name mongo -p 27017:27017 mongo:8` ; ensuite, il se relance :

```bash
docker start mongo
```

Le paquet `com.example.demo` contient, à côté de `CommandesMongo`, quatre fichiers.

`Commande.java`

```java
package com.example.demo;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("commandes")
public record Commande(@Id String id, String client, double montant) {}
```

`JournalCommandes.java`

```java
package com.example.demo;

import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.core.CollectionOptions;
import org.springframework.data.mongodb.core.ReactiveMongoTemplate;
import reactor.core.publisher.Mono;

@Configuration
public class JournalCommandes {
    @Bean
    ApplicationRunner creerCollection(ReactiveMongoTemplate template) {
        return args -> template.collectionExists(Commande.class)
            .flatMap(existe -> existe ? Mono.empty()
                : template.createCollection(Commande.class,
                      CollectionOptions.empty().capped().size(1_000_000)))
            .block();
    }
}
```

Au démarrage, `JournalCommandes` crée la collection `commandes` si elle n'existe pas, plafonnée à 1 000 000 octets : pleine, elle remplace ses documents les plus anciens. `block()` est admis ici : le code s'exécute une fois, au démarrage, hors de toute requête.

`CommandeRepository.java`

```java
package com.example.demo;

import org.springframework.data.mongodb.repository.ReactiveMongoRepository;
import org.springframework.data.mongodb.repository.Tailable;
import reactor.core.publisher.Flux;

public interface CommandeRepository extends ReactiveMongoRepository<Commande, String> {
    @Tailable
    Flux<Commande> findWithTailableCursorBy();
}
```

`@Tailable` demande un curseur qui reste ouvert après avoir livré les documents présents : le `Flux` livre ensuite chaque nouvelle commande, et ne se termine pas de lui-même. Le nom `findWithTailableCursorBy` est une requête dérivée sans critère : toutes les commandes.

`CommandeController.java`

```java
package com.example.demo;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/commandes")
public class CommandeController {

    private final CommandeRepository depot;

    public CommandeController(CommandeRepository depot) { this.depot = depot; }

    @PostMapping
    public Mono<Commande> creer(@RequestBody Commande commande) { return depot.save(commande); }

    @GetMapping(value = "/direct", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<Commande> direct() { return depot.findWithTailableCursorBy(); }
}
```

`GET /commandes/direct` renvoie ce `Flux` en Server-Sent Events.

Démarrer le projet :

```bash
mvn spring-boot:run
```

Dans un second terminal, suivre les commandes alors que la collection est encore vide :

```bash
curl -N localhost:8080/commandes/direct
```

`curl` se termine aussitôt, sans rien afficher. Sur une collection vide, la requête ne trouve rien, et MongoDB ferme le curseur. Enregistrer trois commandes :

```bash
curl -s -X POST localhost:8080/commandes -H 'Content-Type: application/json' -d '{"client":"Ana","montant":42.0}'; echo
curl -s -X POST localhost:8080/commandes -H 'Content-Type: application/json' -d '{"client":"Bruno","montant":15.0}'; echo
curl -s -X POST localhost:8080/commandes -H 'Content-Type: application/json' -d '{"client":"Chloe","montant":80.0}'; echo
```

Relancer le suivi dans le second terminal :

```bash
curl -N localhost:8080/commandes/direct
```

```text
data:{"id":"6ad9dd0eedf7979df808dddf","client":"Ana","montant":42.0}

data:{"id":"6ad9dd0eedf7979df808dde0","client":"Bruno","montant":15.0}

data:{"id":"6ad9dd0eedf7979df808dde1","client":"Chloe","montant":80.0}
```

Dans un troisième terminal, enregistrer une commande :

```bash
curl -s -X POST localhost:8080/commandes -H 'Content-Type: application/json' -d '{"client":"Dina","montant":12.0}'; echo
```

```text
data:{"id":"6ad9dd10edf7979df808dde2","client":"Dina","montant":12.0}
```

La commande apparaît aussitôt dans le second terminal, et `curl` reste ouvert. L'arrêter avec Ctrl-C.

## Dix secondes sans commande

Le client suivant reçoit les quatre commandes présentes, puis s'arrête au bout de 10 s (`-m 10`). Personne ne commande pendant ce temps. Avant de le lancer, prédire combien de requêtes le serveur envoie à MongoDB pendant ces 10 s.

```bash
curl -s -N -m 10 localhost:8080/commandes/direct
```

Lire ensuite les lignes `[mongo]` de la console du projet. Exécuté sur Spring Boot 4.1.1, pilote MongoDB 5.8.1 et MongoDB 8.3.13 :

```text
[mongo] t=0 ms find batchSize=256 total=256
[mongo] t=3 ms getMore batchSize=253 total=509
[mongo] t=1005 ms getMore batchSize=253 total=762
[mongo] t=2013 ms getMore batchSize=253 total=1015
[mongo] t=3019 ms getMore batchSize=253 total=1268
[mongo] t=4024 ms getMore batchSize=253 total=1521
[mongo] t=5031 ms getMore batchSize=253 total=1774
[mongo] t=6037 ms getMore batchSize=253 total=2027
[mongo] t=7040 ms getMore batchSize=253 total=2280
[mongo] t=8045 ms getMore batchSize=253 total=2533
[mongo] t=9049 ms getMore batchSize=253 total=2786
[mongo] t=10056 ms killCursors
```

Dix `getMore` en 10 s, sur trois exécutions. Ici, `total` additionne les documents demandés, pas les documents reçus : aucune commande n'est arrivée.

- Le curseur ne reçoit rien sans le demander : le pilote envoie un `getMore` dès que le lot précédent est livré.
- MongoDB retient ce `getMore` tant qu'aucun document n'arrive, jusqu'à 1 s par défaut, puis répond par un lot vide ; le pilote en envoie aussitôt un autre.
- Si une commande arrive pendant l'attente, MongoDB répond immédiatement : c'est pourquoi la commande de Dina est apparue aussitôt. Mesuré sur cinq commandes : reçues entre 24 et 31 ms après leur envoi.
- À l'arrêt de `curl`, WebFlux annule le flux, et le pilote ferme le curseur : c'est le `killCursors`.

Relire la collection toutes les secondes ferait autant de requêtes, mais chacune relirait les commandes déjà vues, et une nouvelle commande attendrait le tour suivant.

> [!NOTE]
> **Question : peut-on supprimer une commande de la collection plafonnée ?**
> Oui : exécuté sur MongoDB 8.3.13, `db.commandes.deleteOne({client:"Ana"})` la supprime. Ce qui la réserve à un journal, c'est qu'une fois pleine, elle écrase ses documents les plus anciens, que MongoDB y déconseille les mises à jour, et qu'elle traite les écritures une par une. La documentation recommande, en général, un index TTL plutôt qu'une collection plafonnée pour expirer des données. Source : [MongoDB, Capped Collections](https://www.mongodb.com/docs/manual/core/capped-collections/).

Arrêter le projet avec Ctrl-C. Le conteneur `mongo` peut rester démarré : le TP du chapitre s'en sert.

> [!TIP]
> **Ce qu'il faut retenir**
> - Sur une collection plafonnée, `@Tailable` renvoie un `Flux` qui livre les documents présents, puis chaque nouveau document, sans se terminer.
> - Le pilote envoie un `getMore` par seconde même sans nouveauté ; MongoDB le retient et répond dès qu'un document arrive.
> - Sur une collection vide, le curseur se ferme aussitôt.
> - En production, les *change streams* suivent aussi les modifications et les suppressions, et exigent un *replica set*.

Source : [Spring Data MongoDB, Tailable Cursors](https://docs.spring.io/spring-data/mongodb/reference/mongodb/tailable-cursors.html).

# TP - Persister avec MongoDB réactif

**Objectif :** lire, écrire et suivre en continu des documents MongoDB depuis une API réactive.

| Dossier à ouvrir | `demo-application-reactive`, après le TP « Appeler et diffuser » |
|---|---|
| Avant de commencer | Docker, et le conteneur `mongo` lancé par la commande donnée dans « Le code de départ » |
| Commande | `./mvnw spring-boot:run` depuis le dossier du projet, puis les appels `curl` de chaque étape |

Une bibliothèque met son catalogue en ligne. Les livres sont enregistrés dans MongoDB, et l'API les crée, les relit et les cherche sans qu'aucun thread n'attende la base. Chaque création alimente aussi un journal des nouveautés, que les lecteurs suivent en direct.

Un livre, tel que l'API le reçoit et le renvoie :

```text
{"id":"6ad8e71e87f1385be6e841cc","titre":"La Peste","auteur":"Camus","annee":1947,"genre":"roman"}
```

L'identifiant est donné par MongoDB à l'enregistrement : à la création, on ne l'envoie pas.

| Étape | Ce qu'on construit |
|---|---|
| 1 | Le catalogue en base : créer, relire tous les livres, relire les livres d'un auteur |
| 2 | La recherche : genre, année minimale et mot du titre, chacun facultatif |
| 3 | Les nouveautés en direct : un journal dans une collection plafonnée, suivi en SSE |

## Le code de départ

Prérequis :

- Docker, pour lancer MongoDB sur le poste ;
- `demo-application-reactive`, dans son état des TP précédents. WebFlux y est déjà présent.

Lancer MongoDB, une seule fois. Le premier lancement télécharge l'image `mongo:8`, un peu moins de 300 Mo :

```bash
docker run -d --name mongo -p 27017:27017 mongo:8
```

Vérifier qu'il répond :

```bash
docker exec mongo mongosh --quiet --eval 'db.version()'
```

La commande affiche la version du serveur, qui commence par `8`.

Dans `demo-application-reactive`, ajouter au `pom.xml`, dans `<dependencies>`, le starter qui apporte Spring Data MongoDB et le pilote réactif :

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
</dependency>
```

Aucune configuration à ajouter : sans propriété, Spring Boot se connecte à `mongodb://localhost/test`, c'est-à-dire au MongoDB lancé ci-dessus, base `test`.

Ce que le TP fournit : la dépendance ci-dessus, et les commandes `curl` qui remplissent le catalogue à l'étape 1. Ce qu'il ne fournit pas : aucune classe Java. Toutes les classes à écrire sont nommées dans les étapes, et se créent dans le même package que la classe d'application.

Redémarrer `demo-application-reactive`, puis lancer :

```bash
curl -i localhost:8080/livres
```

La réponse est `404 Not Found` : aucune route ne répond encore.

## Étape 1 : le catalogue en base

Créer trois types.

`Livre`, un record enregistré dans la collection `livres` :

1. Ses champs : `id`, `titre`, `auteur`, `annee`, `genre`.
2. `id` est l'identifiant du document, une chaîne générée par MongoDB quand il vaut `null`.

`LivreRepository`, un dépôt réactif de `Livre`, dont l'identifiant est une chaîne :

1. Il hérite de `save`, `findAll` et des autres méthodes de base.
2. Il déclare une requête dérivée qui renvoie les livres d'un auteur.

`LivreController`, un contrôleur annoté qui répond sous `/livres` :

1. `GET /livres` renvoie tous les livres.
2. `GET /livres?auteur=Camus` renvoie les livres de l'auteur demandé.
3. `POST /livres` enregistre le livre reçu en JSON et renvoie le livre enregistré, identifiant compris.

Redémarrer l'application, puis remplir le catalogue en exécutant ce bloc d'un coup :

```bash
curl -s -X POST localhost:8080/livres -H 'Content-Type: application/json' -d '{"titre":"Le Petit Prince","auteur":"Saint-Exupéry","annee":1943,"genre":"conte"}'; echo
curl -s -X POST localhost:8080/livres -H 'Content-Type: application/json' -d '{"titre":"Vol de nuit","auteur":"Saint-Exupéry","annee":1931,"genre":"roman"}'; echo
curl -s -X POST localhost:8080/livres -H 'Content-Type: application/json' -d '{"titre":"La Peste","auteur":"Camus","annee":1947,"genre":"roman"}'; echo
curl -s -X POST localhost:8080/livres -H 'Content-Type: application/json' -d '{"titre":"La Chute","auteur":"Camus","annee":1956,"genre":"roman"}'; echo
curl -s -X POST localhost:8080/livres -H 'Content-Type: application/json' -d '{"titre":"Fondation","auteur":"Asimov","annee":1951,"genre":"science-fiction"}'; echo
```

Chaque ligne affiche le livre enregistré, avec son identifiant :

```text
{"id":"6ad8e71e87f1385be6e841ca","titre":"Le Petit Prince","auteur":"Saint-Exupéry","annee":1943,"genre":"conte"}
...
```

Critère de réussite : les commandes ci-dessous donnent les réponses attendues. Les identifiants changent d'un poste à l'autre.

| Commande | Réponse attendue |
|---|---|
| `curl -s localhost:8080/livres` | les cinq livres |
| `curl -s "localhost:8080/livres?auteur=Camus"` | La Peste et La Chute |
| `curl -s localhost:8080/livres`, après un redémarrage de l'application | toujours les cinq livres : ils sont en base, pas en mémoire |

Pour voir le document tel que MongoDB le stocke :

```bash
docker exec mongo mongosh --quiet test --eval 'printjson(db.livres.findOne({titre:"La Peste"}))'
```

```text
{
  _id: ObjectId('6ad8e71e87f1385be6e841cc'),
  titre: 'La Peste',
  auteur: 'Camus',
  annee: 1947,
  genre: 'roman',
  _class: 'com.example.demo.Livre'
}
```

> [!NOTE]
> **Question : Le document contient un champ `_class`, que le record ne déclare pas. D'où vient-il ?**
> Spring Data MongoDB enregistre par défaut le nom complet de la classe Java dans `_class`, pour savoir quel type reconstruire à la lecture. Le nom affiché dépend du package du projet.

> [!NOTE]
> **Question : `curl` attend une trentaine de secondes, puis reçoit une erreur 500. Pourquoi ?**
> MongoDB ne tourne pas : le pilote attend un serveur pendant 30 secondes, puis abandonne. La console de l'application affiche `Timed out while waiting for a server`. Démarrer le conteneur avec `docker start mongo`.

> [!NOTE]
> **Question : `docker run` répond « Conflict. The container name "/mongo" is already in use ». Pourquoi ?**
> Le conteneur existe déjà, créé par un premier `docker run`. Il suffit de le démarrer : `docker start mongo`.

## Étape 2 : chercher avec des critères facultatifs

La bibliothèque veut chercher ses livres selon trois critères, chacun facultatif : le genre, une année de parution minimale, un mot contenu dans le titre, sans tenir compte des majuscules. Avec des requêtes dérivées, il faudrait une méthode par combinaison de critères.

Dans `LivreController`, ajouter `GET /livres/recherche` :

1. Les paramètres `genre`, `anneeMin` et `mot` sont tous facultatifs.
2. La requête se construit avec `ReactiveMongoTemplate`, un critère par paramètre présent ; sans aucun paramètre, elle renvoie tous les livres.
3. Les livres sont triés par année de parution.

Critère de réussite :

| Commande | Livres attendus, dans l'ordre |
|---|---|
| `curl -s "localhost:8080/livres/recherche?genre=roman"` | Vol de nuit, La Peste, La Chute |
| `curl -s "localhost:8080/livres/recherche?anneeMin=1945"` | La Peste, Fondation, La Chute |
| `curl -s "localhost:8080/livres/recherche?mot=la"` | La Peste, La Chute |
| `curl -s "localhost:8080/livres/recherche?genre=roman&anneeMin=1945&mot=pe"` | La Peste |
| `curl -s "localhost:8080/livres/recherche"` | les cinq livres, de Vol de nuit (1931) à La Chute (1956) |
| `curl -s "localhost:8080/livres/recherche?genre=poesie"` | `[]` |

> [!NOTE]
> **Question : Sans `anneeMin`, la recherche répond 500, et la console parle d'un « primitive type ». Pourquoi ?**
> Le paramètre est déclaré `int`. Un paramètre absent vaut `null`, qu'un type primitif ne peut pas recevoir : la console affiche `Optional int parameter 'anneeMin' is present but cannot be translated into a null value due to being declared as a primitive type`. Le déclarer `Integer`.

## Étape 3 : les nouveautés en direct

Chaque livre créé doit apparaître aussitôt chez les lecteurs qui suivent les nouveautés, sans qu'ils interrogent la base toutes les secondes. Les nouveautés sont un journal : on y ajoute, on n'y supprime rien. Elles vivent donc dans leur propre collection, plafonnée, et non dans `livres`.

Avant d'écrire la moindre ligne, noter sa prédiction. Le journal contient déjà deux nouveautés. Un lecteur lance `curl -N localhost:8080/livres/nouveautes`, puis un livre est créé.

1. Combien de lignes `data:` le lecteur reçoit-il ?
2. `curl` se termine-t-il de lui-même ?

Puis créer trois types et modifier le contrôleur.

`Nouveaute`, un record enregistré dans la collection `nouveautes` : `id`, généré par MongoDB, `titre` et `auteur`.

`NouveauteRepository`, un dépôt réactif de `Nouveaute`, qui déclare une méthode `@Tailable` renvoyant toutes les nouveautés, puis les suivantes au fil de leur insertion.

`JournalNouveautes`, une classe de configuration qui déclare un bean `ApplicationRunner`. Au démarrage :

1. il vérifie si la collection de `Nouveaute` existe ;
2. si elle n'existe pas, il la crée plafonnée, avec une taille maximale de 1 000 000 octets ;
3. il attend la fin de cette opération avant de laisser l'application continuer.

Dans `LivreController` :

1. `POST /livres` enregistre le livre, puis enregistre une nouveauté avec son titre et son auteur, et renvoie toujours le livre enregistré.
2. `GET /livres/nouveautes` renvoie le flux du curseur tailable en Server-Sent Events.

> [!WARNING]
> Après cette étape, l'application ne démarre plus sans MongoDB : la création du journal attend la base, puis l'application s'arrête au bout de 30 secondes avec `Application run failed`. Pour rejouer un autre TP dans le même projet, laisser le conteneur `mongo` démarré.

Redémarrer l'application, puis créer deux livres :

```bash
curl -s -X POST localhost:8080/livres -H 'Content-Type: application/json' -d '{"titre":"Les Misérables","auteur":"Hugo","annee":1862,"genre":"roman"}'; echo
curl -s -X POST localhost:8080/livres -H 'Content-Type: application/json' -d '{"titre":"Notre-Dame de Paris","auteur":"Hugo","annee":1831,"genre":"roman"}'; echo
```

Dans un premier terminal, suivre les nouveautés :

```bash
curl -N localhost:8080/livres/nouveautes
```

Dans un deuxième terminal, créer un livre, et comparer le premier terminal à la prédiction :

```bash
curl -s -X POST localhost:8080/livres -H 'Content-Type: application/json' -d '{"titre":"Dune","auteur":"Herbert","annee":1965,"genre":"science-fiction"}'; echo
```

Critère de réussite :

| Moment | Ce qu'affiche le premier terminal |
|---|---|
| À l'abonnement | deux lignes `data:`, Les Misérables puis Notre-Dame de Paris |
| À la création de Dune | une troisième ligne `data:`, aussitôt |
| Ensuite | rien, mais `curl` reste ouvert jusqu'au Ctrl-C |

```text
data:{"id":"6ad8e7c3fffa5984b3f86de6","titre":"Les Misérables","auteur":"Hugo"}

data:{"id":"6ad8e7c3fffa5984b3f86de8","titre":"Notre-Dame de Paris","auteur":"Hugo"}

data:{"id":"6ad8e7c6fffa5984b3f86dea","titre":"Dune","auteur":"Herbert"}
```

Les cinq livres de l'étape 1 n'y figurent pas : ils ont été créés avant le journal.

> [!NOTE]
> **Question : `curl -N localhost:8080/livres/nouveautes` se termine aussitôt, sans rien afficher. Pourquoi ?**
> Le journal est vide. Sur une collection plafonnée vide, MongoDB ferme le curseur tailable dès l'abonnement : le flux se termine, et une création faite ensuite n'arrive pas. Créer d'abord un livre, puis relancer `curl`.

> [!NOTE]
> **Question : Le suivi des nouveautés répond 500, et la console affiche « tailable cursor requested on non capped collection ». Pourquoi ?**
> La collection `nouveautes` existait déjà, ordinaire, avant le démarrage : par exemple, un livre a été créé avec le nouveau `POST` avant que `JournalNouveautes` existe. Le runner ne crée la collection que si elle est absente, et ne la transforme pas. Supprimer la collection, puis redémarrer l'application.

> [!CAUTION]
> Cette commande supprime le journal des nouveautés et tout son contenu. Les livres ne sont pas touchés.
>
> ```bash
> docker exec mongo mongosh --quiet test --eval 'db.nouveautes.drop()'
> ```

> [!TIP]
> **Ce qu'il faut retenir**
> - Un dépôt réactif étend `ReactiveMongoRepository` : `findAll` renvoie un `Flux`, `save` un `Mono`, et une requête dérivée se déclare par son nom, comme avec JPA.
> - Une requête dont les critères dépendent des paramètres reçus se construit avec `ReactiveMongoTemplate`, un critère par paramètre présent.
> - Un paramètre facultatif se déclare avec un type objet, `Integer` et non `int`.
> - Une collection plafonnée accepte un curseur tailable, qui livre l'existant puis chaque nouvelle insertion ; elle convient à un journal, pas à un catalogue : pleine, elle écrase ses documents les plus anciens.

## Corrigé - Persister avec MongoDB réactif


### Livre.java

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("livres")
record Livre(@Id String id, String titre, String auteur, int annee, String genre) {}
```

### LivreRepository.java

```java
import org.springframework.data.mongodb.repository.ReactiveMongoRepository;

import reactor.core.publisher.Flux;

interface LivreRepository extends ReactiveMongoRepository<Livre, String> {

    Flux<Livre> findByAuteur(String auteur);
}
```

### LivreController.java, après l'étape 1

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/livres")
class LivreController {

    private final LivreRepository depot;

    LivreController(LivreRepository depot) {
        this.depot = depot;
    }

    @GetMapping
    Flux<Livre> tous() {
        return depot.findAll();
    }

    @GetMapping(params = "auteur")
    Flux<Livre> parAuteur(@RequestParam String auteur) {
        return depot.findByAuteur(auteur);
    }

    @PostMapping
    Mono<Livre> creer(@RequestBody Livre livre) {
        return depot.save(livre);
    }
}
```

**Étape 1.** `@Document("livres")` associe le record à la collection `livres`, et `@Id` désigne l'identifiant : un livre reçu sans `id` arrive avec `null`, et MongoDB en génère un à l'enregistrement. `ReactiveMongoRepository<Livre, String>` fournit `save` et `findAll` ; Spring écrit leur implémentation. `findByAuteur` est une requête dérivée : Spring la déduit du nom de la méthode. `params = "auteur"` distingue les deux `GET /livres` : la méthode `parAuteur` ne répond que si le paramètre est présent. Le contrôleur renvoie directement le `Flux` et le `Mono` du dépôt ; WebFlux s'y abonne, et aucun thread n'attend la base.

### LivreController.java, après l'étape 2

```java
import org.springframework.data.domain.Sort;
import org.springframework.data.mongodb.core.ReactiveMongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/livres")
class LivreController {

    private final LivreRepository depot;
    private final ReactiveMongoTemplate template;

    LivreController(LivreRepository depot, ReactiveMongoTemplate template) {
        this.depot = depot;
        this.template = template;
    }

    @GetMapping
    Flux<Livre> tous() {
        return depot.findAll();
    }

    @GetMapping(params = "auteur")
    Flux<Livre> parAuteur(@RequestParam String auteur) {
        return depot.findByAuteur(auteur);
    }

    @GetMapping("/recherche")
    Flux<Livre> recherche(@RequestParam(required = false) String genre,
                          @RequestParam(required = false) Integer anneeMin,
                          @RequestParam(required = false) String mot) {
        Query requete = new Query();
        if (genre != null)    requete.addCriteria(Criteria.where("genre").is(genre));
        if (anneeMin != null) requete.addCriteria(Criteria.where("annee").gte(anneeMin));
        if (mot != null)      requete.addCriteria(Criteria.where("titre").regex(mot, "i"));
        return template.find(requete.with(Sort.by("annee")), Livre.class);
    }

    @PostMapping
    Mono<Livre> creer(@RequestBody Livre livre) {
        return depot.save(livre);
    }
}
```

**Étape 2.** `ReactiveMongoTemplate` est fourni par Spring Boot avec le starter réactif. La requête part vide, et reçoit un critère par paramètre présent : `is` pour l'égalité, `gte` pour « supérieur ou égal », `regex` avec l'option `"i"` pour un mot contenu dans le titre, sans tenir compte des majuscules. Sans aucun critère, elle renvoie tous les livres. `Sort.by("annee")` trie par année. `find` renvoie un `Flux` : rien n'est envoyé à MongoDB avant l'abonnement. Les trois paramètres sont des types objets, pour valoir `null` quand ils sont absents.

### Nouveaute.java

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("nouveautes")
record Nouveaute(@Id String id, String titre, String auteur) {}
```

### NouveauteRepository.java

```java
import org.springframework.data.mongodb.repository.ReactiveMongoRepository;
import org.springframework.data.mongodb.repository.Tailable;

import reactor.core.publisher.Flux;

interface NouveauteRepository extends ReactiveMongoRepository<Nouveaute, String> {

    @Tailable
    Flux<Nouveaute> findWithTailableCursorBy();
}
```

### JournalNouveautes.java

```java
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.core.CollectionOptions;
import org.springframework.data.mongodb.core.ReactiveMongoTemplate;

import reactor.core.publisher.Mono;

@Configuration
class JournalNouveautes {

    @Bean
    ApplicationRunner creerJournal(ReactiveMongoTemplate template) {
        return args -> template.collectionExists(Nouveaute.class)
            .flatMap(existe -> existe ? Mono.empty()
                : template.createCollection(Nouveaute.class,
                      CollectionOptions.empty().capped().size(1_000_000)))
            .block();
    }
}
```

### LivreController.java, après l'étape 3

```java
import org.springframework.data.domain.Sort;
import org.springframework.data.mongodb.core.ReactiveMongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/livres")
class LivreController {

    private final LivreRepository depot;
    private final ReactiveMongoTemplate template;
    private final NouveauteRepository nouveautes;

    LivreController(LivreRepository depot, ReactiveMongoTemplate template,
                    NouveauteRepository nouveautes) {
        this.depot = depot;
        this.template = template;
        this.nouveautes = nouveautes;
    }

    @GetMapping
    Flux<Livre> tous() {
        return depot.findAll();
    }

    @GetMapping(params = "auteur")
    Flux<Livre> parAuteur(@RequestParam String auteur) {
        return depot.findByAuteur(auteur);
    }

    @GetMapping("/recherche")
    Flux<Livre> recherche(@RequestParam(required = false) String genre,
                          @RequestParam(required = false) Integer anneeMin,
                          @RequestParam(required = false) String mot) {
        Query requete = new Query();
        if (genre != null)    requete.addCriteria(Criteria.where("genre").is(genre));
        if (anneeMin != null) requete.addCriteria(Criteria.where("annee").gte(anneeMin));
        if (mot != null)      requete.addCriteria(Criteria.where("titre").regex(mot, "i"));
        return template.find(requete.with(Sort.by("annee")), Livre.class);
    }

    @PostMapping
    Mono<Livre> creer(@RequestBody Livre livre) {
        return depot.save(livre)
                    .flatMap(cree -> nouveautes.save(new Nouveaute(null, cree.titre(), cree.auteur()))
                                               .map(n -> cree));
    }

    @GetMapping(value = "/nouveautes", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    Flux<Nouveaute> nouveautes() {
        return nouveautes.findWithTailableCursorBy();
    }
}
```

**Étape 3.** La prédiction juste était : trois lignes, et `curl` ne se termine pas. Le curseur tailable livre d'abord les deux nouveautés présentes, puis reste ouvert et livre Dune dès son insertion ; le `Flux` ne se termine que si le client ferme la connexion. Le runner crée la collection plafonnée avant toute insertion ; `block()` y est admis, puisqu'il s'exécute une fois, au démarrage, hors de toute requête. Dans `creer`, `flatMap` enchaîne l'enregistrement de la nouveauté après celui du livre, puis `map` renvoie le livre enregistré à la place de la nouveauté. La route SSE renvoie le `Flux` du curseur en `text/event-stream`, comme au chapitre « Appeler et diffuser ».

# TP - Sécurité WebFlux

Ce TP protège une petite API d'articles : tout le monde peut lire les articles, un utilisateur connu peut en créer, seul un administrateur peut en supprimer. Elle vérifie chaque règle avec `curl`, puis observe ce que change l'ordre des règles.

| Projet | `demo-securite-webflux` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-security` |
| Classe de démarrage | `com.example.demo.DemoSecuriteWebfluxApplication` |
| Fichiers | paquet `com.example.demo` : `ArticlesController.java`, `UtilisateursConfig.java`, `SecuriteConfig.java` |
| Sur le poste | `curl` |

Les articles et les utilisateurs restent en mémoire : aucune base de données n'est nécessaire.

## Trois règles vérifiées avec curl

Le paquet `com.example.demo` contient trois fichiers, dans `src/main/java/com/example/demo/` : le contrôleur `ArticlesController`, les utilisateurs `UtilisateursConfig` et les règles `SecuriteConfig`.

`ArticlesController.java`

```java
package com.example.demo;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/articles")
class ArticlesController {

    record Article(String id, String titre) {}

    @GetMapping
    Flux<Article> tous() {
        return Flux.just(new Article("1", "Clavier"), new Article("2", "Souris"));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    Mono<Article> creer(@RequestBody Article article) {
        return Mono.just(article);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    Mono<Void> supprimer(@PathVariable String id) {
        System.out.println("[secu] article " + id + " supprimé");
        return Mono.empty();
    }
}
```

Le contrôleur ne contient aucune règle de sécurité. La suppression affiche une ligne dans la console : elle montre si la requête a atteint le contrôleur.

`UtilisateursConfig.java`

```java
package com.example.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.userdetails.MapReactiveUserDetailsService;
import org.springframework.security.core.userdetails.ReactiveUserDetailsService;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
class UtilisateursConfig {

    @Bean
    PasswordEncoder encodeur() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    ReactiveUserDetailsService utilisateurs(PasswordEncoder encodeur) {
        UserDetails ana = User.withUsername("ana")
            .password(encodeur.encode("secret")).roles("USER").build();
        UserDetails admin = User.withUsername("admin")
            .password(encodeur.encode("secret")).roles("ADMIN").build();
        return new MapReactiveUserDetailsService(ana, admin);
    }
}
```

Deux utilisateurs, `ana` de rôle `USER` et `admin` de rôle `ADMIN`, avec le mot de passe `secret`. Le chargement des utilisateurs et le `PasswordEncoder` sont détaillés plus loin dans le chapitre ; ce fichier sert ici à disposer de deux comptes.

`SecuriteConfig.java`

```java
package com.example.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@Configuration
class SecuriteConfig {

    @Bean
    @Profile("!large")
    SecurityWebFilterChain securite(ServerHttpSecurity http) {
        return http
            .authorizeExchange(regles -> regles
                .pathMatchers(HttpMethod.GET, "/articles/**").permitAll()
                .pathMatchers(HttpMethod.DELETE, "/articles/**").hasRole("ADMIN")
                .anyExchange().authenticated())
            .httpBasic(Customizer.withDefaults())
            .csrf(ServerHttpSecurity.CsrfSpec::disable)
            .build();
    }

    @Bean
    @Profile("large")
    SecurityWebFilterChain regleLargeEnPremier(ServerHttpSecurity http) {
        return http
            .authorizeExchange(regles -> regles
                .pathMatchers("/articles/**").permitAll()
                .pathMatchers(HttpMethod.DELETE, "/articles/**").hasRole("ADMIN")
                .anyExchange().authenticated())
            .httpBasic(Customizer.withDefaults())
            .csrf(ServerHttpSecurity.CsrfSpec::disable)
            .build();
    }
}
```

Lecture du premier bean, `securite` :

- les règles de `authorizeExchange` s'évaluent **dans l'ordre**, et la première qui correspond à la requête l'emporte ;
- `permitAll` laisse passer sans authentification, `hasRole("ADMIN")` exige le rôle `ADMIN`, `authenticated` exige un utilisateur connu ;
- `anyExchange` attrape tout ce qui reste : il vient en dernier ;
- `httpBasic` accepte un identifiant et un mot de passe dans l'en-tête `Authorization`, ce que `curl -u` envoie ;
- la protection CSRF est désactivée parce que l'API n'est appelée que par des clients hors navigateur, comme `curl`. Une application avec formulaires la garde.

Le second bean, `regleLargeEnPremier`, ne sert qu'à la section suivante : il n'est actif qu'avec le profil Spring `large`, et le premier bean est alors écarté par `@Profile("!large")`.

Aucune classe ne porte `@EnableWebFluxSecurity` : Spring Boot active la sécurité WebFlux dès que le starter est présent. Sans Spring Boot, l'annotation est nécessaire.

Démarrer le projet :

```bash
mvn spring-boot:run
```

Dans un second terminal, lancer les six appels :

```bash
curl -s -o /dev/null -w "GET    anonyme  %{http_code}\n" localhost:8080/articles
curl -s -o /dev/null -w "POST   anonyme  %{http_code}\n" -X POST -H "Content-Type: application/json" -d '{"id":"3","titre":"Écran"}' localhost:8080/articles
curl -s -o /dev/null -w "POST   ana      %{http_code}\n" -u ana:secret -X POST -H "Content-Type: application/json" -d '{"id":"3","titre":"Écran"}' localhost:8080/articles
curl -s -o /dev/null -w "DELETE anonyme  %{http_code}\n" -X DELETE localhost:8080/articles/1
curl -s -o /dev/null -w "DELETE ana      %{http_code}\n" -u ana:secret -X DELETE localhost:8080/articles/1
curl -s -o /dev/null -w "DELETE admin    %{http_code}\n" -u admin:secret -X DELETE localhost:8080/articles/1
```

Exécuté sur Spring Boot 4.1.1 et Spring Security 7.1.1, avec le même résultat sur trois exécutions :

```text
GET    anonyme  200
POST   anonyme  401
POST   ana      201
DELETE anonyme  401
DELETE ana      403
DELETE admin    204
```

- 401 : on ne sait pas qui appelle. La réponse porte l'en-tête `WWW-Authenticate: Basic`, qui demande un identifiant.
- 403 : on sait qui appelle, et il n'a pas le droit. `ana` est connue, mais n'a pas le rôle `ADMIN`.
- La console du projet n'affiche qu'une ligne `[secu] article 1 supprimé`, celle de `admin` : les refus n'atteignent pas le contrôleur.

> [!NOTE]
> **Question : un mauvais mot de passe sur une URL publique ?**
> `curl -u ana:faux localhost:8080/articles` répond 401, alors que `GET /articles` est ouvert à tous. Dès qu'un identifiant est envoyé, Spring Security le vérifie, et un identifiant faux est refusé avant que les règles d'URL soient lues.

## La règle large en premier

Le second bean ne change qu'une ligne : la première règle, `pathMatchers("/articles/**").permitAll()`, ne précise plus la méthode HTTP. Avant de lancer les appels, prédire ce que répondent un `DELETE` et un `POST` sans identifiant.

Arrêter le projet avec Ctrl-C, puis le relancer avec le profil Spring `large` :

```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--spring.profiles.active=large"
```

Relancer les six mêmes appels. Exécuté sur Spring Boot 4.1.1 et Spring Security 7.1.1, avec le même résultat sur trois exécutions :

```text
GET    anonyme  200
POST   anonyme  201
POST   ana      201
DELETE anonyme  204
DELETE ana      204
DELETE admin    204
```

- `/articles/**` sans méthode correspond aussi aux `POST` et aux `DELETE` : la première règle laisse tout passer, et la règle `DELETE` n'est jamais lue.
- La console affiche trois lignes `[secu] article 1 supprimé` : un anonyme a supprimé un article.
- Rien ne signale l'erreur, ni au démarrage, ni dans les réponses.

> [!WARNING]
> **`anyExchange` avant une autre règle**
> Placé en premier, `anyExchange().authenticated()` rendrait toutes les règles suivantes inutiles. Ici, Spring Security refuse de démarrer : `Cannot register … which would be unreachable because anyExchange() has already been registered`. Ce contrôle ne porte que sur `anyExchange` : une règle large écrite avec `pathMatchers` passe sans message, comme on vient de le voir.

Arrêter le projet avec Ctrl-C.

> [!TIP]
> **Ce qu'il faut retenir**
> - Une application WebFlux se sécurise par un bean `SecurityWebFilterChain`, construit à partir de `ServerHttpSecurity`.
> - Les règles s'évaluent dans l'ordre, la première qui correspond l'emporte : la plus précise vient en premier, `anyExchange` en dernier.
> - 401 : utilisateur inconnu ou identifiant faux ; 403 : utilisateur connu, sans le droit.
> - Une requête refusée n'atteint pas le contrôleur.

Sources : [Spring Security, WebFlux Security](https://docs.spring.io/spring-security/reference/reactive/configuration/webflux.html), [Spring Security, Cross Site Request Forgery](https://docs.spring.io/spring-security/reference/features/exploits/csrf.html), [Spring Boot, WebFlux Security](https://docs.spring.io/spring-boot/reference/web/spring-security.html).

# TP - Utilisateurs en base

Ce TP remplace les deux comptes en mémoire par des comptes lus dans MongoDB. Elle compte ensuite combien de fois Spring Security relit un utilisateur, puis observe ce que devient un `UserDetailsService` de Spring MVC dans une application WebFlux.

| Projet | `demo-utilisateurs-mongodb` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-security`, `spring-boot-starter-data-mongodb-reactive` |
| Classe de démarrage | `com.example.demo.DemoUtilisateursMongodbApplication` |
| Fichiers | paquet `com.example.demo` : `ArticlesController.java`, `SecuriteConfig.java`, `Utilisateur.java`, `UtilisateurRepository.java`, `UtilisateursMongo.java`, `ComptesConfig.java`, `ServiceBloquantConfig.java` |
| Sur le poste | Docker et `curl` ; collection `utilisateurs` de la base `test` |

Le projet reprend le contrôleur et les règles du TP « Sécurité WebFlux », sans `UtilisateursConfig` : les utilisateurs viennent désormais de la base.

- `ArticlesController.java` est celui de `demo-securite-webflux`, montré dans le TP « Sécurité WebFlux », avec la ligne `package com.example.demo;`.
- `SecuriteConfig.java` garde ses règles d'URL, sans la variante `large` :

```java
package com.example.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@Configuration
class SecuriteConfig {

    @Bean
    SecurityWebFilterChain securite(ServerHttpSecurity http) {
        return http
            .authorizeExchange(regles -> regles
                .pathMatchers(HttpMethod.GET, "/articles/**").permitAll()
                .pathMatchers(HttpMethod.DELETE, "/articles/**").hasRole("ADMIN")
                .anyExchange().authenticated())
            .httpBasic(Customizer.withDefaults())
            .csrf(ServerHttpSecurity.CsrfSpec::disable)
            .build();
    }
}
```

## Les comptes en base

Démarrer MongoDB. Au premier usage, le conteneur se crée par `docker run -d --name mongo -p 27017:27017 mongo:8` ; ensuite, il se relance :

```bash
docker start mongo
```

À côté du contrôleur et des règles, le paquet `com.example.demo` contient cinq fichiers, dans `src/main/java/com/example/demo/` : le document `Utilisateur`, son dépôt `UtilisateurRepository`, le service `UtilisateursMongo`, les comptes `ComptesConfig`, et `ServiceBloquantConfig`, qui ne sert qu'à la dernière section.

`Utilisateur.java`

```java
package com.example.demo;

import java.util.List;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("utilisateurs")
record Utilisateur(@Id String nom, String motDePasseHache, List<String> roles) {}
```
`UtilisateurRepository.java`

```java
package com.example.demo;

import org.springframework.data.mongodb.repository.ReactiveMongoRepository;

interface UtilisateurRepository extends ReactiveMongoRepository<Utilisateur, String> {}
```

Un utilisateur est un document de la collection `utilisateurs` : son nom sert d'identifiant, à côté de son mot de passe haché et de la liste de ses rôles.

`UtilisateursMongo.java`

```java
package com.example.demo;

import org.springframework.context.annotation.Profile;
import org.springframework.security.core.userdetails.ReactiveUserDetailsService;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Mono;

@Component
@Profile("!bloquant")
class UtilisateursMongo implements ReactiveUserDetailsService {

    private final UtilisateurRepository depot;

    UtilisateursMongo(UtilisateurRepository depot) { this.depot = depot; }

    @Override
    public Mono<UserDetails> findByUsername(String nom) {
        System.out.println("[secu] findByUsername(" + nom + ")");
        return depot.findById(nom)
            .map(u -> User.withUsername(u.nom())
                          .password(u.motDePasseHache())
                          .roles(u.roles().toArray(String[]::new))
                          .build());
    }
}
```

`findByUsername` est la seule méthode de `ReactiveUserDetailsService`. Elle cherche le document par son nom et le convertit en `UserDetails`, le type que Spring Security sait lire. Pour un nom inconnu, `findById` renvoie un `Mono` vide : Spring Security répond alors 401, sans qu'aucune exception soit levée ici. La ligne affichée dans la console montre chaque appel.

`@Profile("!bloquant")` écarte ce service avec le profil Spring `bloquant`, qui sert à la dernière section à comparer avec un `UserDetailsService` de Spring MVC.

`ComptesConfig.java`

```java
package com.example.demo;

import java.util.List;
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
class ComptesConfig {

    @Bean
    PasswordEncoder encodeur() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    ApplicationRunner comptes(UtilisateurRepository depot, PasswordEncoder encodeur) {
        return args -> depot.deleteAll()
            .thenMany(depot.saveAll(List.of(
                new Utilisateur("ana", encodeur.encode("secret"), List.of("USER")),
                new Utilisateur("admin", encodeur.encode("secret"), List.of("ADMIN")))))
            .then()
            .doOnSuccess(v -> System.out.println("[secu] comptes ana et admin enregistrés"))
            .block();
    }
}
```

- `createDelegatingPasswordEncoder` hache avec BCrypt. La valeur enregistrée commence par `{bcrypt}`, qui dit à la vérification quel algorithme employer.
- `comptes` s'exécute au démarrage : il vide la collection, puis enregistre `ana`, de rôle `USER`, et `admin`, de rôle `ADMIN`, avec le mot de passe `secret` haché. La ligne `[secu] comptes ana et admin enregistrés` signale que les comptes sont prêts.

Démarrer le projet :

```bash
mvn spring-boot:run
```

Attendre la ligne `[secu] comptes ana et admin enregistrés`, puis lancer les quatre appels dans un second terminal :

```bash
curl -s -o /dev/null -w "POST   ana      %{http_code}\n" -u ana:secret -X POST -H "Content-Type: application/json" -d '{"id":"3","titre":"Écran"}' localhost:8080/articles
curl -s -o /dev/null -w "DELETE ana      %{http_code}\n" -u ana:secret -X DELETE localhost:8080/articles/1
curl -s -o /dev/null -w "DELETE admin    %{http_code}\n" -u admin:secret -X DELETE localhost:8080/articles/1
curl -s -o /dev/null -w "POST   zoe      %{http_code}\n" -u zoe:secret -X POST -H "Content-Type: application/json" -d '{"id":"3","titre":"Écran"}' localhost:8080/articles
```

Exécuté sur Spring Boot 4.1.1, Spring Security 7.1.1 et MongoDB 8.3.13, avec le même résultat sur trois exécutions :

```text
POST   ana      201
DELETE ana      403
DELETE admin    204
POST   zoe      401
```

Les règles n'ont pas changé, seule l'origine des comptes a changé : les codes sont ceux du TP précédent. `zoe` n'existe pas en base et reçoit 401.

> [!WARNING]
> **Un mot de passe enregistré en clair**
> Un document dont le mot de passe vaut `secret`, sans préfixe, ne donne pas 401 mais 500 : `IllegalArgumentException: Given that there is no default password encoder configured, each password must have a password encoding prefix…`. Un rôle enregistré sous la forme `ROLE_ADMIN` donne lui aussi 500 : `ROLE_ADMIN cannot start with ROLE_ (it is automatically added)`. Hacher le mot de passe avec le même `PasswordEncoder` à la création, et enregistrer les rôles sans préfixe.

> [!NOTE]
> **Question : garder aussi le service en mémoire ?**
> L'application ne démarre pas : Spring Security attend un seul `ReactiveUserDetailsService`, et le message indique « required a single bean, but 2 were found ». C'est pourquoi `demo-utilisateurs-mongodb` ne contient pas `UtilisateursConfig`.

## Dix requêtes de ana

Laisser le projet tourner. Avant de lancer les appels, prédire combien de fois `findByUsername` sera appelé pour dix requêtes de `ana`.

Dix requêtes avec `ana`, puis dix requêtes anonymes, chacune affichant sa durée en secondes :

```bash
for i in 1 2 3 4 5 6 7 8 9 10; do curl -s -o /dev/null -w "%{time_total}\n" -u ana:secret localhost:8080/articles; done
for i in 1 2 3 4 5 6 7 8 9 10; do curl -s -o /dev/null -w "%{time_total}\n" localhost:8080/articles; done
```

Exécuté sur Spring Boot 4.1.1, Spring Security 7.1.1 et MongoDB 8.3.13, avec le même résultat sur trois exécutions :

- la console affiche dix lignes `[secu] findByUsername(ana)`, une par requête, et aucune pour les requêtes anonymes ;
- chaque requête avec `ana` dure environ 0,155 s, chaque requête anonyme environ 0,003 s.

`httpBasic` n'enregistre l'utilisateur nulle part entre deux requêtes : chaque requête repart de l'en-tête `Authorization`, relit l'utilisateur en base et vérifie son mot de passe. La plus grande part du temps est la vérification BCrypt, volontairement lente pour freiner qui essaie des mots de passe ; Spring Security la fait tourner sur un thread `boundedElastic`, pour ne pas immobiliser le thread qui reçoit les requêtes.

La recherche revient à chaque requête authentifiée : c'est pour cela qu'elle ne doit pas bloquer, et que `findByUsername` renvoie un `Mono`.

Arrêter le projet avec Ctrl-C.

## Garder son UserDetailsService

Une application Spring MVC charge ses utilisateurs avec un `UserDetailsService`. Le fichier `ServiceBloquantConfig` en déclare un, réduit à un seul compte, `ana`, et actif seulement avec le profil Spring `bloquant` ; `UtilisateursMongo` est alors écarté.

`ServiceBloquantConfig.java`

```java
package com.example.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@Profile("bloquant")
class ServiceBloquantConfig {

    @Bean
    UserDetailsService utilisateursBloquants(PasswordEncoder encodeur) {
        String hache = encodeur.encode("secret");
        return nom -> {
            System.out.println("[secu] loadUserByUsername(" + nom + ")");
            if (!nom.equals("ana")) throw new UsernameNotFoundException(nom);
            return User.withUsername("ana").password(hache).roles("USER").build();
        };
    }
}
```

Avant de lancer l'appel, prédire ce que répond `curl -u ana:secret` à un `POST`.

Relancer le projet avec le profil Spring `bloquant` :

```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--spring.profiles.active=bloquant"
```

Puis, dans le second terminal :

```bash
curl -s -o /dev/null -w "POST   ana      %{http_code}\n" -u ana:secret -X POST -H "Content-Type: application/json" -d '{"id":"3","titre":"Écran"}' localhost:8080/articles
```

Exécuté sur Spring Boot 4.1.1 et Spring Security 7.1.1, avec le même résultat sur trois exécutions :

```text
POST   ana      401
```

- La console n'affiche aucune ligne `[secu] loadUserByUsername(ana)` : le service n'est jamais appelé.
- Au démarrage, elle affiche `Using generated security password: …`, sans autre avertissement.

Spring Security WebFlux n'appelle qu'un `ReactiveUserDetailsService`. N'en trouvant aucun, Spring Boot crée le sien, avec l'utilisateur `user` et un mot de passe généré : c'est ce mot de passe qu'affiche la console. Le `UserDetailsService` reste dans l'application, ignoré. La correction est d'implémenter `ReactiveUserDetailsService`, comme `UtilisateursMongo`.

Arrêter le projet avec Ctrl-C.

> [!TIP]
> **Ce qu'il faut retenir**
> - Une application WebFlux charge ses utilisateurs par un `ReactiveUserDetailsService`, dont la seule méthode, `findByUsername`, renvoie un `Mono<UserDetails>`.
> - Un utilisateur inconnu est un `Mono` vide, que Spring Security transforme en 401.
> - Avec `httpBasic`, l'utilisateur est relu et son mot de passe vérifié à chaque requête : la recherche ne doit pas bloquer.
> - Un `UserDetailsService` est ignoré par Spring Security WebFlux, sans message.
> - Le mot de passe s'enregistre haché par le même `PasswordEncoder`, avec son préfixe `{bcrypt}`.

Sources : [Spring Security, Reactive Authentication](https://docs.spring.io/spring-security/reference/reactive/authentication/index.html), [Spring Security, Password Storage](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html), [Spring Security 7.1.1, ServerHttpSecurity](https://github.com/spring-projects/spring-security/blob/7.1.1/config/src/main/java/org/springframework/security/config/web/server/ServerHttpSecurity.java#L1113), [Spring Boot 4.1.1, ReactiveUserDetailsServiceAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/v4.1.1/module/spring-boot-security/src/main/java/org/springframework/boot/security/autoconfigure/ReactiveUserDetailsServiceAutoConfiguration.java).

# TP - Sécurité des méthodes

Ce TP pose une règle qui dépend de la donnée : un utilisateur n'annule que ses propres commandes, un administrateur les annule toutes. Les règles d'URL de `SecuriteConfig` ne savent pas l'exprimer : elles voient l'URL et la méthode HTTP, jamais à qui appartient la commande. La règle se pose donc sur la méthode du service qui annule, et elle vaut pour tout contrôleur qui l'appelle.

| Projet | `demo-securite-methodes` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-security`, `spring-boot-starter-data-mongodb-reactive` |
| Classe de démarrage | `com.example.demo.DemoSecuriteMethodesApplication` |
| Fichiers | paquet `com.example.demo` : les comptes et les règles, `SecuriteConfig.java`, `Utilisateur.java`, `UtilisateurRepository.java`, `UtilisateursMongo.java`, `ComptesConfig.java` ; les commandes, `Commande.java`, `CommandeRepository.java`, `MethodesConfig.java`, `CommandeService.java`, `CommandesController.java`, `CommandesConfig.java` |
| Sur le poste | Docker et `curl` ; collections `utilisateurs` et `commandes_clients` de la base `test` |

Les comptes et les règles sont ceux de `demo-utilisateurs-mongodb`, montrés dans le TP « Utilisateurs en base » : les comptes `ana`, de rôle `USER`, et `admin`, de rôle `ADMIN`, sont lus dans MongoDB. Une seule différence : ce projet n'a pas de variante `bloquant`, et `UtilisateursMongo` n'y porte pas `@Profile("!bloquant")`. Les règles de `SecuriteConfig` couvrent aussi les routes `/commandes/**` : `anyExchange().authenticated()` les réserve aux utilisateurs connus.

## Annuler une commande

Démarrer MongoDB. Au premier usage, le conteneur se crée par `docker run -d --name mongo -p 27017:27017 mongo:8` ; ensuite, il se relance :

```bash
docker start mongo
```

Le paquet `com.example.demo` contient six fichiers propres à ce TP, dans `src/main/java/com/example/demo/` : le document `Commande` et son dépôt `CommandeRepository`, l'activation `MethodesConfig`, le service `CommandeService`, le contrôleur `CommandesController` et les données de départ `CommandesConfig`.

`Commande.java`

```java
package com.example.demo;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("commandes_clients")
record Commande(@Id String numero, String client) {}
```

`CommandeRepository.java`

```java
package com.example.demo;

import org.springframework.data.mongodb.repository.ReactiveMongoRepository;
import reactor.core.publisher.Mono;

interface CommandeRepository extends ReactiveMongoRepository<Commande, String> {
    Mono<Void> deleteByClientAndNumero(String client, String numero);
}
```

Une commande est un document de la collection `commandes_clients` : son numéro sert d'identifiant, à côté du nom du client. La collection ne s'appelle pas `commandes`, nom de la collection plafonnée du TP « Curseur tailable », dans la même base `test`. Spring Data déduit la requête du nom de `deleteByClientAndNumero` : elle supprime la commande seulement si le client et le numéro correspondent tous les deux.

`MethodesConfig.java`

```java
package com.example.demo;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.security.config.annotation.method.configuration.EnableReactiveMethodSecurity;

@Configuration
@Profile("!sansmethodes")
@EnableReactiveMethodSecurity
class MethodesConfig {}
```

`@EnableReactiveMethodSecurity` active la vérification des annotations de sécurité posées sur les méthodes. Spring Boot ne l'active pas de lui-même. `@Profile("!sansmethodes")` écarte cette classe avec le profil Spring `sansmethodes`, employé dans un encadré plus bas.

`CommandeService.java`

```java
package com.example.demo;

import java.util.List;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.context.ReactiveSecurityContextHolder;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Service
class CommandeService {

    private final CommandeRepository depot;

    CommandeService(CommandeRepository depot) { this.depot = depot; }

    @PreAuthorize("hasRole('ADMIN') or #client == authentication.name")
    Mono<Void> annuler(String client, String numero) {
        System.out.println("[secu] annuler(" + client + ", " + numero + ") sur "
            + Thread.currentThread().getName());
        return depot.deleteByClientAndNumero(client, numero);
    }

    Mono<Void> annulerPlusieurs(String client, List<String> numeros) {
        return Flux.fromIterable(numeros).concatMap(n -> annuler(client, n)).then();
    }

    @PreAuthorize("hasRole('ADMIN') or #client == authentication.name")
    String annulerBloquant(String client, String numero) {
        return "annulée";
    }

    Mono<String> nomCourant() {
        return ReactiveSecurityContextHolder.getContext()
            .map(ctx -> ctx.getAuthentication().getName());
    }
}
```

Lecture de `annuler` :

- `@PreAuthorize` porte une expression que Spring Security vérifie avant d'exécuter la méthode ; si elle est fausse, la méthode n'est pas appelée ;
- `hasRole('ADMIN')` teste le rôle de l'utilisateur connecté, `authentication.name` est son nom ;
- `#client` désigne le paramètre `client` de la méthode, le client dont on annule la commande. L'expression se lit : administrateur, ou client qui annule sa propre commande ;
- la méthode renvoie un `Mono`, et c'est obligatoire : Spring Security lit l'utilisateur dans le contexte Reactor de la requête, qu'il atteint par ce `Mono` ;
- la ligne `[secu] annuler(…)` s'affiche dans la console à chaque exécution du corps de la méthode : elle montre si la méthode a réellement été appelée.

Les trois autres méthodes servent plus loin : `annulerPlusieurs` à la section suivante, `annulerBloquant` et `nomCourant` dans des encadrés.

`CommandesController.java`

```java
package com.example.demo;

import java.util.List;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/commandes")
class CommandesController {

    private final CommandeService service;
    private final CommandeRepository depot;

    CommandesController(CommandeService service, CommandeRepository depot) {
        this.service = service;
        this.depot = depot;
    }

    @GetMapping
    Flux<Commande> toutes() { return depot.findAll(); }

    @GetMapping("/moi")
    Mono<String> moi() { return service.nomCourant(); }

    @DeleteMapping("/{client}/{numero}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    Mono<Void> annuler(@PathVariable String client, @PathVariable String numero) {
        return service.annuler(client, numero);
    }

    @DeleteMapping("/{client}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    Mono<Void> annulerPlusieurs(@PathVariable String client, @RequestParam List<String> numeros) {
        return service.annulerPlusieurs(client, numeros);
    }

    @DeleteMapping("/bloquant/{client}/{numero}")
    String annulerBloquant(@PathVariable String client, @PathVariable String numero) {
        return service.annulerBloquant(client, numero);
    }
}
```

Le contrôleur ne porte aucune règle : il transmet au service le client et le numéro lus dans l'URL. `DELETE /commandes/{client}/{numero}` annule une commande, `GET /commandes` liste celles qui restent.

`CommandesConfig.java`

```java
package com.example.demo;

import java.util.List;
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
class CommandesConfig {

    @Bean
    ApplicationRunner commandes(CommandeRepository depot) {
        return args -> depot.deleteAll()
            .thenMany(depot.saveAll(List.of(new Commande("42", "ana"), new Commande("17", "bob"))))
            .then()
            .doOnSuccess(v -> System.out.println("[secu] commandes 42 de ana et 17 de bob enregistrées"))
            .block();
    }
}
```

Au démarrage, `commandes` vide la collection, puis enregistre deux commandes : la 42, de `ana`, et la 17, de `bob`. Chaque relance du projet les remet donc en place. `bob` n'a pas de compte : il n'est que le client de la commande 17.

Démarrer le projet :

```bash
mvn spring-boot:run
```

Attendre les deux lignes `[secu] comptes ana et admin enregistrés` et `[secu] commandes 42 de ana et 17 de bob enregistrées`. Elles s'affichent après la ligne `Started` : un appel lancé avant elles reçoit 401, les comptes n'étant pas encore en base.

Avant de lancer les appels, prédire le code HTTP de chacun : `ana` annule sa commande 42, `ana` annule la commande 17 de `bob`, `admin` annule la commande 17 de `bob`.

Dans un second terminal, lancer les trois appels :

```bash
curl -s -o /dev/null -w "DELETE ana/42 avec ana    %{http_code}\n" -u ana:secret -X DELETE localhost:8080/commandes/ana/42
curl -s -o /dev/null -w "DELETE bob/17 avec ana    %{http_code}\n" -u ana:secret -X DELETE localhost:8080/commandes/bob/17
curl -s -o /dev/null -w "DELETE bob/17 avec admin  %{http_code}\n" -u admin:secret -X DELETE localhost:8080/commandes/bob/17
```

Puis lister les commandes qui restent :

```bash
curl -s -u admin:secret localhost:8080/commandes
```

Exécuté sur Spring Boot 4.1.1, Spring Security 7.1.1 et MongoDB 8.3.13, avec le même résultat sur trois exécutions :

```text
DELETE ana/42 avec ana    204
DELETE bob/17 avec ana    403
DELETE bob/17 avec admin  204
```

```text
[]
```

- 403 et non 401 : `ana` est connue de Spring Security, elle n'a pas le droit d'annuler la commande de `bob`.
- Les deux commandes ont disparu : `ana` a annulé la sienne, `admin` celle de `bob`.
- La console n'affiche que deux lignes `[secu] annuler(…)` : `annuler(ana, 42)` pour `ana`, puis `annuler(bob, 17)` pour `admin`. Aucune pour le refus.

Spring Security décide d'abord, et n'appelle la méthode qu'ensuite : en cas de refus, le corps de `annuler` ne s'exécute pas, et ni le dépôt ni la base ne sont sollicités. Dans le code de Spring Security, l'appel de la méthode est enveloppé dans un `Mono.defer`, qui ne s'exécute qu'une fois l'autorisation accordée.

`#client` se retrouve par le nom du paramètre, que le compilateur ne conserve qu'avec l'option `-parameters`. Le parent Spring Boot du `pom.xml` l'active. Sans elle, `#client` vaut `null` : `ana` reçoit 403 pour sa propre commande, sans aucun message.

> [!NOTE]
> **Question : un appelant anonyme refusé par @PreAuthorize ?**
> Il reçoit 401, avec l'en-tête `WWW-Authenticate: Basic`, et non 403 : Spring Security ne sait pas qui appelle. Le 403 ne vaut que pour un utilisateur connu. Dans ce projet, le cas ne se présente pas : `anyExchange().authenticated()` arrête l'anonyme avant même le service.

> [!NOTE]
> **Question : oublier @EnableReactiveMethodSecurity ?**
> Arrêter le projet avec Ctrl-C, puis le relancer avec le profil Spring `sansmethodes`, qui écarte `MethodesConfig` :
>
> ```bash
> mvn spring-boot:run -Dspring-boot.run.arguments="--spring.profiles.active=sansmethodes"
> ```
>
> Les trois mêmes appels répondent 204, 204, 204, sur trois exécutions : `ana` annule la commande de `bob`. Rien ne le signale, ni au démarrage ni dans la console : sans l'annotation, `@PreAuthorize` est ignorée.

> [!NOTE]
> **Question : une méthode qui renvoie un String ?**
> `annulerBloquant` porte la même règle que `annuler`, mais renvoie un `String`. L'application démarre normalement, c'est l'appel qui échoue :
>
> ```bash
> curl -s -o /dev/null -w "%{http_code}\n" -u ana:secret -X DELETE localhost:8080/commandes/bloquant/ana/42
> ```
>
> Il répond 500, sur trois exécutions, et la console affiche `IllegalStateException: The returnType class java.lang.String on … must return an instance of org.reactivestreams.Publisher (for example, a Mono or Flux) or the function must be a Kotlin coroutine in order to support Reactor Context`. Sans `Mono` ni `Flux`, Spring Security n'a pas accès au contexte Reactor qui porte l'utilisateur.

Arrêter le projet avec Ctrl-C.

## Annuler plusieurs commandes

`CommandeService` contient une seconde méthode d'annulation, `annulerPlusieurs`, appelée par `DELETE /commandes/{client}?numeros=…`. Elle ne porte aucune annotation : elle appelle `annuler`, protégée, pour chaque numéro, l'un après l'autre.

Avant de lancer l'appel, prédire ce que répond `ana` quand elle annule ainsi la commande 17 de `bob`.

Relancer le projet, ce qui remet en place les commandes 42 et 17 :

```bash
mvn spring-boot:run
```

Attendre les deux lignes `[secu]`, puis, dans le second terminal :

```bash
curl -s -o /dev/null -w "DELETE bob?numeros=17 avec ana  %{http_code}\n" -u ana:secret -X DELETE "localhost:8080/commandes/bob?numeros=17"
```

Puis lister les commandes qui restent :

```bash
curl -s -u admin:secret localhost:8080/commandes
```

Exécuté sur Spring Boot 4.1.1, Spring Security 7.1.1 et MongoDB 8.3.13, avec le même résultat sur trois exécutions :

```text
DELETE bob?numeros=17 avec ana  204
```

```text
[{"numero":"42","client":"ana"}]
```

- `ana` a annulé la commande de `bob` : il ne reste que la commande 42.
- La console affiche `[secu] annuler(bob, 17)` : `annuler` s'est exécutée sans que sa règle soit vérifiée.

Le contrôleur ne reçoit pas le service lui-même, mais un proxy que Spring place devant lui. C'est ce proxy qui vérifie la règle de `@PreAuthorize`, puis délègue au service. `annulerPlusieurs` appelle `annuler` sur `this`, à l'intérieur du service : l'appel ne traverse pas le proxy, et la règle n'est pas vérifiée. Le même piège vaut pour toute annotation appliquée par un proxy, `@Transactional` par exemple.

La correction consiste à protéger aussi la méthode appelée de l'extérieur. Arrêter le projet avec Ctrl-C, puis poser sur `annulerPlusieurs`, dans `CommandeService.java`, la même annotation que sur `annuler` :

```java
    @PreAuthorize("hasRole('ADMIN') or #client == authentication.name")
    Mono<Void> annulerPlusieurs(String client, List<String> numeros) {
        return Flux.fromIterable(numeros).concatMap(n -> annuler(client, n)).then();
    }
```

Relancer le projet, attendre les deux lignes `[secu]`, puis lancer l'appel de `ana` et le même appel avec `admin` :

```bash
curl -s -o /dev/null -w "DELETE bob?numeros=17 avec ana    %{http_code}\n" -u ana:secret -X DELETE "localhost:8080/commandes/bob?numeros=17"
curl -s -o /dev/null -w "DELETE bob?numeros=17 avec admin  %{http_code}\n" -u admin:secret -X DELETE "localhost:8080/commandes/bob?numeros=17"
```

Exécuté sur Spring Boot 4.1.1, Spring Security 7.1.1 et MongoDB 8.3.13, avec le même résultat sur trois exécutions :

```text
DELETE bob?numeros=17 avec ana    403
DELETE bob?numeros=17 avec admin  204
```

La règle est désormais vérifiée à l'entrée de `annulerPlusieurs`, par le proxy : `ana` est refusée, `admin` passe.

> [!NOTE]
> **Question : lire l'utilisateur dans le code ?**
> Quand la règle dépasse une expression, par exemple parce qu'elle dépend d'une donnée lue en base, le service lit l'utilisateur lui-même. C'est ce que fait `nomCourant`, avec `ReactiveSecurityContextHolder`. Celui-ci remplace le `SecurityContextHolder` de Spring MVC : il ne lit pas une variable du thread, mais le contexte Reactor de la requête, puisque celle-ci change de thread en cours de traitement.
>
> ```bash
> curl -s -u ana:secret localhost:8080/commandes/moi
> ```
>
> L'appel renvoie `ana`, avec le code 200 ; sans identifiant, il reçoit 401. Mesuré sur trois exécutions.

Arrêter le projet avec Ctrl-C.

> [!TIP]
> **Ce qu'il faut retenir**
> - Une règle qui dépend de la donnée se pose sur la méthode du service, avec `@PreAuthorize` : elle vaut pour tout appelant.
> - `@EnableReactiveMethodSecurity` est indispensable : sans elle, `@PreAuthorize` est ignorée, sans message.
> - La méthode protégée renvoie un `Mono` ou un `Flux`.
> - La règle est vérifiée avant la méthode : un refus n'exécute pas son corps et ne touche pas la base.
> - La vérification vit dans un proxy : un appel interne à la classe ne le traverse pas.

Sources : [Spring Security, Reactive Method Security](https://docs.spring.io/spring-security/reference/reactive/authorization/method.html), [Spring Security 7.1.1, AuthorizationManagerBeforeReactiveMethodInterceptor, ligne 138](https://github.com/spring-projects/spring-security/blob/7.1.1/core/src/main/java/org/springframework/security/authorization/method/AuthorizationManagerBeforeReactiveMethodInterceptor.java#L138), [Spring Framework, Understanding AOP Proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html).

# TP - Sécuriser une application réactive

**Objectif :** protéger une API réactive par des règles d'URL et par des règles sur ses méthodes, avec des utilisateurs lus en base.

| Dossier à ouvrir | `demo-application-reactive`, après le TP « Persister avec MongoDB réactif » |
|---|---|
| Avant de commencer | le conteneur `mongo` relancé par `docker start mongo` : le projet ne démarre plus sans MongoDB |
| Commande | `./mvnw spring-boot:run` depuis le dossier du projet, puis les appels `curl -u` de chaque étape |

La bibliothèque ouvre son catalogue à tous, mais seul un compte connu ajoute un livre, et seul un administrateur en retire un. Les commandes du jour suivent la même logique : un client n'annule que ses propres commandes. Le TP protège `demo-application-reactive`, sans nouveau domaine : le catalogue de livres du TP « Persister avec MongoDB réactif », et les commandes du TP « Exposer une API WebFlux ».

Les trois comptes du TP :

| Compte | Rôle | Mot de passe |
|---|---|---|
| `Durand` | `USER` | `secret` |
| `Martin` | `USER` | `secret` |
| `admin` | `ADMIN` | `secret` |

`Durand` et `Martin` sont aussi les clients des commandes 1 et 2.

| Étape | Ce qu'on construit |
|---|---|
| 1 | Les règles d'URL, avec les trois comptes en mémoire |
| 2 | Les mêmes comptes, lus dans MongoDB |
| 3 | Chacun ses commandes : une règle posée sur une méthode |

## Le code de départ

Prérequis :

- Docker, et le conteneur `mongo` du TP « Persister avec MongoDB réactif » ;
- `demo-application-reactive`, dans son état de fin du TP « Persister avec MongoDB réactif » : depuis son étape 3, il ne démarre plus sans MongoDB.

Démarrer MongoDB :

```bash
docker start mongo
```

Dans `demo-application-reactive`, ajouter au `pom.xml`, dans `<dependencies>`, le starter de Spring Security :

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Ce que le TP fournit : cette dépendance, et les appels `curl` de chaque étape. Ce qu'il ne fournit pas : aucune classe Java. Toutes les classes à écrire sont nommées dans les étapes, et se créent dans le même package que la classe d'application.

Avant de redémarrer, noter sa prédiction. Avec le seul starter, sans aucune classe de sécurité :

1. Que répond `curl -i localhost:8080/livres` ?
2. Que répond un `POST /livres` sans identifiant ?

Redémarrer le projet, puis lancer :

```bash
curl -i localhost:8080/livres
```

Puis :

```bash
curl -s -w "  %{http_code}\n" -X POST -H 'Content-Type: application/json' -d '{"titre":"La Peste","auteur":"Camus","annee":1947,"genre":"roman"}' localhost:8080/livres
```

Exécuté sur Spring Boot 4.1.1 et Spring Security 7.1.1, avec le même résultat sur trois exécutions :

- `GET /livres` répond `401 Unauthorized`, avec l'en-tête `WWW-Authenticate: Basic realm="Realm"` ;
- le `POST` répond 403, avec le corps `Access Denied` ;
- la console du projet affiche `Using generated security password: …`.

Spring Boot, ne trouvant aucun service de comptes, crée un utilisateur `user`, dont le mot de passe est celui affiché dans la console, et ferme toutes les routes : sans identifiant, la réponse est 401. Le `POST` reçoit 403, et non 401 : la protection CSRF, active par défaut, refuse une écriture qui ne porte pas de jeton CSRF, avant même de chercher qui appelle. Avec `user` et le mot de passe généré, `GET /livres` répond 200, mais le `POST` répond toujours 403.

Les routes des TP précédents sont fermées elles aussi : `GET /commandes/1` sans identifiant répond 401.

## Étape 1 : les règles d'URL

Créer deux classes de configuration, et compléter le contrôleur des livres.

`SecuriteConfig`, une classe de configuration qui déclare un bean `SecurityWebFilterChain` :

1. `GET /livres/**` est ouvert à tous ;
2. `DELETE /livres/**` est réservé au rôle `ADMIN` ;
3. toute autre requête demande un utilisateur connu ;
4. les règles s'écrivent dans cet ordre, la plus précise d'abord ;
5. l'authentification se fait par `httpBasic`, et la protection CSRF est désactivée : l'API n'est appelée que par `curl`.

`ComptesMemoire`, une classe de configuration qui déclare deux beans :

1. un `PasswordEncoder` délégant, créé par `PasswordEncoderFactories` ;
2. un `ReactiveUserDetailsService` en mémoire, avec les trois comptes du tableau, dont les mots de passe sont hachés par cet encodeur.

Dans `LivreController`, ajouter `DELETE /livres/{id}`, qui supprime le livre et répond 204.

Redémarrer l'application, puis lancer ces trois appels. Le troisième crée un livre avec `Durand` :

```bash
curl -s -o /dev/null -w "GET    /livres anonyme   %{http_code}\n" localhost:8080/livres
curl -s -o /dev/null -w "POST   /livres anonyme   %{http_code}\n" -X POST -H 'Content-Type: application/json' -d '{"titre":"La Peste","auteur":"Camus","annee":1947,"genre":"roman"}' localhost:8080/livres
curl -s -o /dev/null -w "POST   /livres Durand    %{http_code}\n" -u Durand:secret -X POST -H 'Content-Type: application/json' -d '{"titre":"La Peste","auteur":"Camus","annee":1947,"genre":"roman"}' localhost:8080/livres
```

Relire le catalogue, et copier l'identifiant du livre créé :

```bash
curl -s localhost:8080/livres; echo
```

Dans les deux premières lignes ci-dessous, remplacer `IDENTIFIANT` par l'identifiant copié, puis lancer les trois appels :

```bash
curl -s -o /dev/null -w "DELETE /livres/{id} Durand  %{http_code}\n" -u Durand:secret -X DELETE localhost:8080/livres/IDENTIFIANT
curl -s -o /dev/null -w "DELETE /livres/{id} admin   %{http_code}\n" -u admin:secret -X DELETE localhost:8080/livres/IDENTIFIANT
curl -s -o /dev/null -w "DELETE /commandes/1 anonyme %{http_code}\n" -X DELETE localhost:8080/commandes/1
```

Critère de réussite :

| Appel | Code attendu |
|---|---|
| `GET /livres` anonyme | 200 |
| `POST /livres` anonyme | 401 |
| `POST /livres` avec `Durand` | 200 |
| `DELETE /livres/{id}` avec `Durand` | 403 |
| `DELETE /livres/{id}` avec `admin` | 204 |
| `DELETE /commandes/1` anonyme | 401 |

La console n'affiche plus `Using generated security password`.

> [!NOTE]
> **Question : `POST /livres` répond 403, même avec `Durand`. Pourquoi ?**
> La protection CSRF n'est pas désactivée. Exécuté sans la ligne qui la désactive : `POST /livres` anonyme 403, avec `Durand` 403 et le corps `Access Denied`, et même `DELETE /commandes/1` avec `admin` 403. Elle refuse toute écriture sans jeton CSRF, avant que les règles d'URL soient lues.

## Étape 2 : les comptes en base

Les mêmes comptes passent dans MongoDB. Seule leur origine change : les règles, et donc les codes de l'étape 1, restent les mêmes.

`Compte`, un record enregistré dans la collection `comptes` : le nom, qui sert d'identifiant, le mot de passe haché et la liste des rôles.

`CompteRepository`, un dépôt réactif de `Compte`, dont l'identifiant est le nom.

`ComptesMongo`, un composant qui implémente `ReactiveUserDetailsService` :

1. `findByUsername` cherche le compte par son nom ;
2. il le convertit en `UserDetails`, avec son mot de passe haché et ses rôles ;
3. pour un nom inconnu, il renvoie un `Mono` vide.

`ComptesConfig`, une classe de configuration qui déclare deux beans :

1. un `PasswordEncoder` délégant, comme à l'étape 1 ;
2. un `ApplicationRunner` qui enregistre les trois comptes, mots de passe hachés, attend la fin de l'enregistrement, puis affiche `Comptes Durand, Martin et admin enregistrés`.

Supprimer `ComptesMemoire`.

Redémarrer l'application, et attendre la ligne `Comptes Durand, Martin et admin enregistrés`. Relancer les appels de l'étape 1, en recopiant l'identifiant d'un nouveau livre, puis un appel avec un compte qui n'existe pas :

```bash
curl -s -o /dev/null -w "POST   /livres zoe       %{http_code}\n" -u zoe:secret -X POST -H 'Content-Type: application/json' -d '{"titre":"La Peste","auteur":"Camus","annee":1947,"genre":"roman"}' localhost:8080/livres
```

Critère de réussite :

| Appel | Code attendu |
|---|---|
| Les six appels de l'étape 1 | les mêmes codes qu'à l'étape 1 |
| `POST /livres` avec `zoe` | 401 |

Pour voir un compte tel que MongoDB le stocke :

```bash
docker exec mongo mongosh --quiet test --eval 'printjson(db.comptes.findOne({_id:"Durand"}))'
```

```text
{
  _id: 'Durand',
  motDePasseHache: '{bcrypt}$2a$10$…',
  roles: [
    'USER'
  ],
  _class: 'com.example.demo.Compte'
}
```

Le mot de passe est enregistré haché, et commence par `{bcrypt}` ; la suite diffère d'un poste à l'autre.

> [!NOTE]
> **Question : `Durand` reçoit 401 juste après le démarrage. Pourquoi ?**
> Au premier démarrage, la collection `comptes` est vide : un appel lancé avant la ligne `Comptes Durand, Martin et admin enregistrés` reçoit 401. Les comptes restent ensuite en base : aux démarrages suivants, l'appel passe dès le début.

> [!NOTE]
> **Question : L'application ne démarre plus après l'étape 2. Pourquoi ?**
> `ComptesMemoire` n'a pas été supprimé. `ComptesMemoire` et `ComptesConfig` déclarent chacun un bean `encodeur`, et Spring refuse ce doublon : `The bean 'encodeur' … could not be registered. A bean with that name has already been defined … and overriding is disabled`. Supprimer `ComptesMemoire`.

## Étape 3 : chacun ses commandes

Avant d'écrire la moindre ligne, noter sa prédiction. `Martin`, un compte connu, annule la commande 1, qui est celle de `Durand`. Que répond l'application, et que devient la commande ?

Les commandes vivent en mémoire : chaque redémarrage remet les quatre commandes en place. Redémarrer l'application, attendre la ligne `Comptes Durand, Martin et admin enregistrés`, puis lancer :

```bash
curl -s -o /dev/null -w "DELETE /commandes/1 Martin  %{http_code}\n" -u Martin:secret -X DELETE localhost:8080/commandes/1
curl -s -o /dev/null -w "GET    /commandes/1 Durand  %{http_code}\n" -u Durand:secret localhost:8080/commandes/1
```

Exécuté sur Spring Boot 4.1.1, Spring Security 7.1.1 et MongoDB 8.3.13, avec le même résultat sur trois exécutions :

```text
DELETE /commandes/1 Martin  204
GET    /commandes/1 Durand  404
```

`Martin` a annulé la commande de `Durand`, qui a disparu. `Martin` est connu, et `DELETE /commandes/{numero}` n'exige qu'un utilisateur connu : l'URL ne dit pas à qui appartient la commande 1. La règle dépend de la donnée : elle se pose sur la méthode qui annule, une fois la commande lue.

Dans `Commandes`, ajouter une méthode `annuler(Commande commande)` :

1. elle supprime la commande reçue, en réutilisant `supprimer` ;
2. elle est protégée par `@PreAuthorize` : seul un administrateur, ou le client de la commande, peut l'appeler ;
3. le client se lit dans la commande reçue en paramètre, comme `#client` désignait le paramètre `client` dans le TP « Sécurité des méthodes ».

Dans `CommandeHandler`, la méthode `supprimer` :

1. lit d'abord la commande par son numéro ;
2. appelle `annuler` avec la commande lue ;
3. répond 204, comme avant.

Sur `SecuriteConfig`, ajouter `@EnableReactiveMethodSecurity`.

Redémarrer l'application, attendre la ligne `Comptes Durand, Martin et admin enregistrés`, puis lancer les appels dans cet ordre :

```bash
curl -s -o /dev/null -w "DELETE /commandes/1 Martin  %{http_code}\n" -u Martin:secret -X DELETE localhost:8080/commandes/1
curl -s -o /dev/null -w "GET    /commandes/1 Durand  %{http_code}\n" -u Durand:secret localhost:8080/commandes/1
curl -s -o /dev/null -w "DELETE /commandes/1 Durand  %{http_code}\n" -u Durand:secret -X DELETE localhost:8080/commandes/1
curl -s -o /dev/null -w "DELETE /commandes/2 admin   %{http_code}\n" -u admin:secret -X DELETE localhost:8080/commandes/2
curl -s -o /dev/null -w "GET    /commandes/1 Durand  %{http_code}\n" -u Durand:secret localhost:8080/commandes/1
curl -s -o /dev/null -w "DELETE /commandes/9 Martin  %{http_code}\n" -u Martin:secret -X DELETE localhost:8080/commandes/9
curl -s -o /dev/null -w "DELETE /commandes/3 anonyme %{http_code}\n" -X DELETE localhost:8080/commandes/3
```

Critère de réussite :

| Appel, dans l'ordre | Code attendu |
|---|---|
| `DELETE /commandes/1`, la commande de `Durand`, avec `Martin` | 403 |
| `GET /commandes/1` avec `Durand` | 200 : la commande est toujours là |
| `DELETE /commandes/1` avec `Durand` | 204 |
| `DELETE /commandes/2`, la commande de `Martin`, avec `admin` | 204 |
| `GET /commandes/1` avec `Durand` | 404 : la commande a été annulée |
| `DELETE /commandes/9`, qui n'existe pas, avec `Martin` | 204 |
| `DELETE /commandes/3` anonyme | 401 |

> [!NOTE]
> **Question : `Martin` annule toujours la commande de `Durand`. Pourquoi ?**
> `@EnableReactiveMethodSecurity` manque sur `SecuriteConfig`. Exécuté sans elle : 204, puis 404 pour la commande 1, sans aucun message, ni au démarrage ni dans la console. Sans cette annotation, `@PreAuthorize` est ignorée.

> [!NOTE]
> **Question : `DELETE /commandes/9` répond 204, alors que la commande n'existe pas. Pourquoi ?**
> La commande 9 est inconnue : `parNumero` renvoie un `Mono` vide, `flatMap` ne reçoit rien et n'appelle pas `annuler`. Il n'y a donc rien à vérifier, et la réponse reste 204, comme avant le TP.

> [!TIP]
> **Ce qu'il faut retenir**
> - Le starter seul ferme toutes les routes, et la protection CSRF refuse les écritures avant même de chercher qui appelle.
> - Les règles d'URL s'évaluent dans l'ordre : la plus précise d'abord, `anyExchange` en dernier.
> - Entre l'étape 1 et l'étape 2, seule l'origine des comptes change : les codes restent les mêmes.
> - Une règle qui dépend d'une donnée se pose sur la méthode, après avoir lu la donnée.
> - Sans `@EnableReactiveMethodSecurity`, `@PreAuthorize` est ignorée, sans message.

Sources : [Spring Security, WebFlux Security](https://docs.spring.io/spring-security/reference/reactive/configuration/webflux.html), [Spring Security, Cross Site Request Forgery](https://docs.spring.io/spring-security/reference/features/exploits/csrf.html), [Spring Security, Reactive Authentication](https://docs.spring.io/spring-security/reference/reactive/authentication/index.html), [Spring Security, Reactive Method Security](https://docs.spring.io/spring-security/reference/reactive/authorization/method.html), [Spring Boot, Spring Security](https://docs.spring.io/spring-boot/reference/web/spring-security.html).

## Corrigé - Sécuriser une application réactive


### SecuriteConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@Configuration
class SecuriteConfig {

    @Bean
    SecurityWebFilterChain securite(ServerHttpSecurity http) {
        return http
            .authorizeExchange(regles -> regles
                .pathMatchers(HttpMethod.GET, "/livres/**").permitAll()
                .pathMatchers(HttpMethod.DELETE, "/livres/**").hasRole("ADMIN")
                .anyExchange().authenticated())
            .httpBasic(Customizer.withDefaults())
            .csrf(ServerHttpSecurity.CsrfSpec::disable)
            .build();
    }
}
```

### ComptesMemoire.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.userdetails.MapReactiveUserDetailsService;
import org.springframework.security.core.userdetails.ReactiveUserDetailsService;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
class ComptesMemoire {

    @Bean
    PasswordEncoder encodeur() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    ReactiveUserDetailsService comptes(PasswordEncoder encodeur) {
        return new MapReactiveUserDetailsService(
            User.withUsername("Durand").password(encodeur.encode("secret")).roles("USER").build(),
            User.withUsername("Martin").password(encodeur.encode("secret")).roles("USER").build(),
            User.withUsername("admin").password(encodeur.encode("secret")).roles("ADMIN").build());
    }
}
```

### LivreController.java, avec la suppression

```java
import org.springframework.data.domain.Sort;
import org.springframework.data.mongodb.core.ReactiveMongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/livres")
class LivreController {

    private final LivreRepository depot;
    private final ReactiveMongoTemplate template;
    private final NouveauteRepository nouveautes;

    LivreController(LivreRepository depot, ReactiveMongoTemplate template,
                    NouveauteRepository nouveautes) {
        this.depot = depot;
        this.template = template;
        this.nouveautes = nouveautes;
    }

    @GetMapping
    Flux<Livre> tous() {
        return depot.findAll();
    }

    @GetMapping(params = "auteur")
    Flux<Livre> parAuteur(@RequestParam String auteur) {
        return depot.findByAuteur(auteur);
    }

    @GetMapping("/recherche")
    Flux<Livre> recherche(@RequestParam(required = false) String genre,
                          @RequestParam(required = false) Integer anneeMin,
                          @RequestParam(required = false) String mot) {
        Query requete = new Query();
        if (genre != null)    requete.addCriteria(Criteria.where("genre").is(genre));
        if (anneeMin != null) requete.addCriteria(Criteria.where("annee").gte(anneeMin));
        if (mot != null)      requete.addCriteria(Criteria.where("titre").regex(mot, "i"));
        return template.find(requete.with(Sort.by("annee")), Livre.class);
    }

    @PostMapping
    Mono<Livre> creer(@RequestBody Livre livre) {
        return depot.save(livre)
                    .flatMap(cree -> nouveautes.save(new Nouveaute(null, cree.titre(), cree.auteur()))
                                               .map(n -> cree));
    }

    @GetMapping(value = "/nouveautes", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    Flux<Nouveaute> nouveautes() {
        return nouveautes.findWithTailableCursorBy();
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    Mono<Void> retirer(@PathVariable String id) {
        return depot.deleteById(id);
    }
}
```

**Étape 1.** La prédiction juste était : 401 pour le `GET`, 403 pour le `POST`. Dès qu'un bean `SecurityWebFilterChain` existe, Spring Boot n'applique plus sa configuration par défaut, qui fermait tout. Les règles s'évaluent dans l'ordre, et la première qui correspond l'emporte : la lecture des livres est ouverte, leur suppression réservée à `ADMIN`, et `anyExchange` attrape tout le reste, en dernier. `Durand` reçoit 403 sur la suppression : il est connu, sans le rôle. `createDelegatingPasswordEncoder` hache avec BCrypt, et `MapReactiveUserDetailsService` garde les comptes en mémoire. Dès qu'un `ReactiveUserDetailsService` est déclaré, Spring Boot ne crée plus l'utilisateur `user` : la ligne du mot de passe généré disparaît. `retirer` renvoie le `Mono<Void>` de `deleteById`, et `@ResponseStatus(HttpStatus.NO_CONTENT)` donne 204. `POST /livres` répond 200 et non 201 : `creer` ne porte pas de `@ResponseStatus`.

### Compte.java

```java
import java.util.List;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document("comptes")
record Compte(@Id String nom, String motDePasseHache, List<String> roles) {}
```

### CompteRepository.java

```java
import org.springframework.data.mongodb.repository.ReactiveMongoRepository;

interface CompteRepository extends ReactiveMongoRepository<Compte, String> {}
```

### ComptesMongo.java

```java
import org.springframework.security.core.userdetails.ReactiveUserDetailsService;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import reactor.core.publisher.Mono;

@Component
class ComptesMongo implements ReactiveUserDetailsService {

    private final CompteRepository depot;

    ComptesMongo(CompteRepository depot) {
        this.depot = depot;
    }

    @Override
    public Mono<UserDetails> findByUsername(String nom) {
        return depot.findById(nom)
                    .map(c -> User.withUsername(c.nom())
                                  .password(c.motDePasseHache())
                                  .roles(c.roles().toArray(String[]::new))
                                  .build());
    }
}
```

### ComptesConfig.java

```java
import java.util.List;

import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
class ComptesConfig {

    @Bean
    PasswordEncoder encodeur() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    ApplicationRunner enregistrerComptes(CompteRepository depot, PasswordEncoder encodeur) {
        return args -> depot.saveAll(List.of(
                new Compte("Durand", encodeur.encode("secret"), List.of("USER")),
                new Compte("Martin", encodeur.encode("secret"), List.of("USER")),
                new Compte("admin", encodeur.encode("secret"), List.of("ADMIN"))))
            .then()
            .doOnSuccess(v -> System.out.println("Comptes Durand, Martin et admin enregistrés"))
            .block();
    }
}
```

**Étape 2.** `ComptesMongo` est désormais le seul `ReactiveUserDetailsService` : Spring Security l'appelle à chaque requête authentifiée, puisque `httpBasic` ne garde rien entre deux requêtes. Un nom inconnu donne un `Mono` vide, que Spring Security transforme en 401 : c'est la réponse reçue par `zoe`. Le mot de passe s'enregistre haché par le même encodeur, et son préfixe `{bcrypt}` dit à la vérification quel algorithme employer. Le runner bloque une seule fois, au démarrage, hors de toute requête, comme `JournalNouveautes`. `saveAll` remplace les comptes de même nom : relancer l'application ne crée pas de doublon. Les codes de l'étape 1 n'ont pas changé : seule l'origine des comptes a changé.

### Commandes.java, après l'étape 3

```java
import java.time.Duration;
import java.util.Map;
import java.util.concurrent.ConcurrentSkipListMap;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Component;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

record Commande(long numero, String client, double montant) {}

@Component
class Commandes {

    private final Map<Long, Commande> commandes = new ConcurrentSkipListMap<>(Map.of(
        1L, new Commande(1, "Durand", 120.0),
        2L, new Commande(2, "Martin", 45.5),
        3L, new Commande(3, "Petit", 310.0),
        4L, new Commande(4, "Leroy", 89.9)));

    // Les commandes du jour, une toutes les 300 ms.
    Flux<Commande> duJour() {
        return Flux.fromIterable(commandes.values()).delayElements(Duration.ofMillis(300));
    }

    // Une commande, ou un Mono vide si le numéro est inconnu.
    Mono<Commande> parNumero(long numero) {
        return Mono.justOrEmpty(commandes.get(numero));
    }

    // Supprime une commande, puis se termine sans valeur.
    Mono<Void> supprimer(long numero) {
        return Mono.fromRunnable(() -> commandes.remove(numero));
    }

    // Annule une commande : réservé à son client ou à un administrateur.
    @PreAuthorize("hasRole('ADMIN') or #commande.client() == authentication.name")
    Mono<Void> annuler(Commande commande) {
        return supprimer(commande.numero());
    }

    // Affiche dans la console qui répond, et pour quelle commande.
    static void trace(String qui, long numero) {
        System.out.println("[" + qui + "] commande " + numero);
    }
}
```

### CommandeHandler.java, après l'étape 3

```java
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;

import reactor.core.publisher.Mono;

@Component
class CommandeHandler {

    private final Commandes commandes;

    CommandeHandler(Commandes commandes) { this.commandes = commandes; }

    Mono<ServerResponse> parNumero(ServerRequest requete) {
        long numero = Long.parseLong(requete.pathVariable("numero"));
        Commandes.trace("routeur", numero);
        return commandes.parNumero(numero)
                        .flatMap(c -> ServerResponse.ok().bodyValue(c))
                        .switchIfEmpty(ServerResponse.notFound().build());
    }

    Mono<ServerResponse> supprimer(ServerRequest requete) {
        long numero = Long.parseLong(requete.pathVariable("numero"));
        return commandes.parNumero(numero)
                        .flatMap(commandes::annuler)
                        .then(ServerResponse.noContent().build());
    }
}
```

### SecuriteConfig.java, après l'étape 3

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.method.configuration.EnableReactiveMethodSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@Configuration
@EnableReactiveMethodSecurity
class SecuriteConfig {

    @Bean
    SecurityWebFilterChain securite(ServerHttpSecurity http) {
        return http
            .authorizeExchange(regles -> regles
                .pathMatchers(HttpMethod.GET, "/livres/**").permitAll()
                .pathMatchers(HttpMethod.DELETE, "/livres/**").hasRole("ADMIN")
                .anyExchange().authenticated())
            .httpBasic(Customizer.withDefaults())
            .csrf(ServerHttpSecurity.CsrfSpec::disable)
            .build();
    }
}
```

**Étape 3.** La prédiction juste était : 204, et la commande disparaît. Le handler lit d'abord la commande, puis appelle `annuler` : Spring Security vérifie l'expression avec la commande reçue en paramètre, avant d'exécuter la méthode. `#commande` désigne ce paramètre, `client()` son client, `authentication.name` l'utilisateur connecté. `Martin` reçoit 403 : il est connu, sans droit sur la commande de `Durand`. Un refus arrête la chaîne : `then` n'est jamais atteint, et la commande reste en mémoire. `annuler` renvoie un `Mono`, comme l'exige la sécurité réactive des méthodes. `@EnableReactiveMethodSecurity` active la vérification ; sans elle, l'annotation est ignorée, sans message.

# TP - StepVerifier

> [!WARNING]
> **Ne jamais lancer `mvn test` sans `-Dtest`**
> Certains tests des TP StepVerifier, Contextes de test et WebTestClient, et un test du TP « Tester une application réactive », échouent volontairement. Chaque section donne la commande ciblée et annonce l'échec attendu. Dans `demo-application-reactive`, les tests générés démarrent toute l'application et nécessitent MongoDB à partir du TP « Persister avec MongoDB réactif ».

Ce TP vérifie par des tests automatiques ce qu'un `Flux` émet : ses éléments, dans l'ordre, puis sa fin ou son erreur. Elle reprend l'exemple des slides du chapitre « Threads, erreurs et débogage » : un catalogue de trois articles, Clavier, Souris et Écran, et un service de prix qui échoue pour la Souris. Elle part du réflexe, attendre le résultat avec `block()`, puis passe à `StepVerifier`, qui vérifie chaque signal.

| Projet | `demo-stepverifier` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, et en portée test `spring-boot-starter-webflux-test` |
| Classe de démarrage | `com.example.demo.DemoStepverifierApplication`, que les tests ne démarrent pas |
| Fichiers | paquet `com.example.demo` : `Catalogue.java` ; le test `CatalogueTest.java` se crée pendant le TP |
| Sur le poste | un IDE Java |

Les tests ne démarrent pas Spring : aucun serveur, aucune base de données.

Le `pom.xml` du projet déclare le starter de test de WebFlux :

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux-test</artifactId>
  <scope>test</scope>
</dependency>
```

Elle apporte JUnit et `reactor-test`, qui contient `StepVerifier`. En Spring Boot 4.1.1, `spring-boot-starter-test` seul n'apporte pas `reactor-test`.

Les tests se lancent un par un, en nommant la méthode, par exemple :

```bash
mvn test "-Dtest=CatalogueTest#troisTitres"
```

> [!WARNING]
> **Ne pas lancer `mvn test` sans `-Dtest`**
> Quatre tests du TP échouent volontairement : `prixAvecBlock`, `ordreInverse`, `titreDeTrop` et `fluxHorsDuFournisseur`. Lancés ensemble, ils donnent `BUILD FAILURE`, et c'est le résultat attendu.

Maven affiche la durée de chaque test. Elle dépend du poste : le Guide en donne un ordre de grandeur, qui dit quel type de résultat attendre.

## block() et la Souris

Le projet contient `Catalogue`, dans `src/main/java/com/example/demo/`. Créer `CatalogueTest`, dans `src/test/java/com/example/demo/`.

`Catalogue.java`

```java
package com.example.demo;

import java.time.Duration;
import java.util.Map;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public class Catalogue {

    public record Article(int numero, String titre) {}

    public static class TarifIndisponible extends RuntimeException {
        public TarifIndisponible(String message) {
            super(message);
        }
    }

    private static final Map<Integer, Double> PRIX = Map.of(1, 49.0, 3, 189.0);

    // Clavier, Souris, Écran, sans délai.
    public static Flux<Article> catalogue() {
        return Flux.just(new Article(1, "Clavier"), new Article(2, "Souris"), new Article(3, "Écran"));
    }

    // Répond en 100 ms ; le service de tarifs échoue pour la Souris.
    public static Mono<Double> prix(Article article) {
        return Mono.delay(Duration.ofMillis(100))
                   .flatMap(t -> article.numero() == 2
                           ? Mono.error(new TarifIndisponible("tarif indisponible pour " + article.titre()))
                           : Mono.just(PRIX.get(article.numero())));
    }

    // L'article de ce numéro, ou un Mono vide s'il n'existe pas.
    public static Mono<Article> trouver(int numero) {
        return catalogue().filter(a -> a.numero() == numero).next();
    }
}
```

- `catalogue()` émet les trois articles, sans délai.
- `prix(article)` répond au bout de 100 ms : 49.0 pour le Clavier, 189.0 pour l'Écran, et une erreur `TarifIndisponible` pour la Souris.
- `trouver(numero)` renvoie l'article de ce numéro, ou un `Mono` vide s'il n'existe pas.

`CatalogueTest.java`

```java
package com.example.demo;

import static com.example.demo.Catalogue.catalogue;
import static com.example.demo.Catalogue.prix;
import static com.example.demo.Catalogue.trouver;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.time.Duration;
import java.util.List;

import org.junit.jupiter.api.Test;

import com.example.demo.Catalogue.Article;
import com.example.demo.Catalogue.TarifIndisponible;
import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

class CatalogueTest {

    // block() : les trois titres, dans l'ordre.
    @Test
    void troisTitres() {
        List<String> titres = catalogue().map(Article::titre).collectList().block();
        assertEquals(List.of("Clavier", "Souris", "Écran"), titres);
    }

    // block() sur les prix : le Clavier, puis l'erreur de la Souris.
    @Test
    void prixAvecBlock() {
        List<Double> prix = catalogue().concatMap(a -> prix(a)).collectList().block();
        assertEquals(List.of(49.0), prix);
    }

    // StepVerifier : chaque signal, dans l'ordre.
    @Test
    void troisTitresPuisFin() {
        StepVerifier.create(catalogue().map(Article::titre))
            .expectNext("Clavier", "Souris", "Écran")
            .verifyComplete();
    }

    // Les titres dans le désordre.
    @Test
    void ordreInverse() {
        StepVerifier.create(catalogue().map(Article::titre))
            .expectNext("Souris", "Clavier", "Écran")
            .verifyComplete();
    }

    // Un titre de moins que ce que le Flux émet.
    @Test
    void titreDeTrop() {
        StepVerifier.create(catalogue().map(Article::titre))
            .expectNext("Clavier", "Souris")
            .verifyComplete();
    }

    // Le prix du Clavier, puis l'erreur de la Souris.
    @Test
    void prixPuisErreur() {
        StepVerifier.create(catalogue().concatMap(a -> prix(a)))
            .expectNext(49.0)
            .expectError(TarifIndisponible.class)
            .verify();
    }

    // Un Mono vide : aucun élément, puis la fin.
    @Test
    void articleInconnu() {
        StepVerifier.create(trouver(9))
            .verifyComplete();
    }

    // Des attentes fausses, sans verify.
    @Test
    void sansVerify() {
        StepVerifier.create(catalogue().map(Article::titre))
            .expectNext("faux")
            .expectComplete();
    }

    // Deux heures de flux, en temps virtuel.
    @Test
    void deuxMesuresEnDeuxHeures() {
        StepVerifier.withVirtualTime(() -> Flux.interval(Duration.ofHours(1)).take(2))
            .expectSubscription()
            .thenAwait(Duration.ofHours(2))
            .expectNext(0L, 1L)
            .verifyComplete();
    }

    // Le Flux construit hors du fournisseur : il garde l'horloge réelle.
    @Test
    void fluxHorsDuFournisseur() {
        Flux<Long> mesures = Flux.interval(Duration.ofHours(1)).take(2);
        StepVerifier.withVirtualTime(() -> mesures)
            .expectSubscription()
            .thenAwait(Duration.ofHours(2))
            .expectNext(0L, 1L)
            .expectComplete()
            .verify(Duration.ofSeconds(3));
    }
}
```

La classe contient tous les tests du TP, chacun précédé d'un commentaire qui dit ce qu'il vérifie. Les sections suivantes les lancent un par un.

Le premier test, `troisTitres`, est celui qu'on écrit spontanément : `collectList().block()` attend la liste des titres, puis `assertEquals` la compare à la liste attendue. Dans un test, c'est le thread du test qui attend : le bloquer ne gêne personne.

```bash
mvn test "-Dtest=CatalogueTest#troisTitres"
```

Exécuté sur Java 25, Spring Boot 4.1.1 et Reactor 3.8.7, avec le même résultat sur trois exécutions : le test passe en moins d'une seconde.

```text
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```

Comparer la liste entière vérifie aussi l'ordre des titres.

`prixAvecBlock` fait le même geste sur les prix : `concatMap` demande le prix de chaque article, l'un après l'autre. Le Clavier donne 49.0, puis la Souris échoue, et l'Écran n'est jamais demandé. Le test attend la liste `[49.0]`. Avant de le lancer, prédire : passe-t-il ?

```bash
mvn test "-Dtest=CatalogueTest#prixAvecBlock"
```

Exécuté sur Java 25, Spring Boot 4.1.1 et Reactor 3.8.7, avec le même résultat sur trois exécutions : le test échoue en quelques dixièmes de seconde, au moins 0,2 s, le temps des deux appels de prix.

```text
[ERROR] Errors: 
[ERROR]   CatalogueTest.prixAvecBlock » TarifIndisponible tarif indisponible pour Souris
```

`block()` relance l'erreur reçue : `assertEquals` n'est jamais atteint. L'erreur ne passe donc pas inaperçue, mais `block()` rend la liste ou l'exception, jamais les deux. Le 49.0 reçu avant l'erreur est perdu, et « le prix du Clavier, puis l'erreur de la Souris » ne peut pas s'écrire avec `block()`.

> [!NOTE]
> **Question : et un `Flux` infini ?**
> Un `Flux` infini ne se termine jamais : `block()` l'attend sans fin. Avec un délai maximal de deux secondes :
>
> ```java
> Flux.interval(Duration.ofMillis(100)).collectList().block(Duration.ofSeconds(2));
> ```
>
> Le test échoue après un peu plus de 2 s, sur `IllegalStateException: Timeout on blocking read for 2000000000 NANOSECONDS`. `StepVerifier`, présenté à la section suivante, sait vérifier les premiers éléments puis abandonner le flux avec `thenCancel` :
>
> ```java
> StepVerifier.create(Flux.interval(Duration.ofMillis(100)))
>     .expectNext(0L, 1L, 2L)
>     .thenCancel()
>     .verify();
> ```
>
> Ce test passe en moins d'une seconde, au moins 0,3 s, le temps des trois éléments émis à 100 ms d'intervalle. Mesuré trois fois chacun, dans une classe de test séparée.

## Vérifier signal par signal

`troisTitresPuisFin` vérifie les mêmes titres avec `StepVerifier` :

- `StepVerifier.create` reçoit le `Flux` à vérifier ;
- `expectNext("Clavier", "Souris", "Écran")` attend ces trois éléments, dans cet ordre ;
- `verifyComplete()` attend la fin du `Flux`, puis lance la vérification : c'est lui qui s'abonne.

```bash
mvn test "-Dtest=CatalogueTest#troisTitresPuisFin"
```

Exécuté sur Java 25, Spring Boot 4.1.1 et Reactor 3.8.7, avec le même résultat sur trois exécutions : le test passe en moins d'une seconde.

Deux variantes fausses montrent ce que `StepVerifier` signale. `ordreInverse` attend les titres dans le désordre, `titreDeTrop` n'en attend que deux :

```bash
mvn test "-Dtest=CatalogueTest#ordreInverse"
```

```text
[ERROR] Failures: 
[ERROR]   CatalogueTest.ordreInverse:47 expectation "expectNext(Souris)" failed (expected value: Souris; actual value: Clavier)
```

```bash
mvn test "-Dtest=CatalogueTest#titreDeTrop"
```

```text
[ERROR] Failures: 
[ERROR]   CatalogueTest.titreDeTrop:55 expectation "expectComplete" failed (expected: onComplete(); actual: onNext(Écran))
```

Exécuté sur Java 25, Spring Boot 4.1.1 et Reactor 3.8.7, avec le même résultat sur trois exécutions. Chaque message nomme l'attente qui échoue, puis ce qui était attendu et ce qui est arrivé : la Souris attendue, le Clavier reçu ; la fin attendue, l'Écran reçu.

`prixPuisErreur` écrit enfin le cas que `block()` ne savait pas écrire : le prix du Clavier, puis l'erreur de la Souris. `expectError(TarifIndisponible.class)` attend la fin par une erreur de ce type, et `verify()` lance la vérification.

```bash
mvn test "-Dtest=CatalogueTest#prixPuisErreur"
```

Exécuté sur Java 25, Spring Boot 4.1.1 et Reactor 3.8.7, avec le même résultat sur trois exécutions : le test passe en quelques dixièmes de seconde, au moins 0,2 s, le temps des deux appels de prix.

> [!NOTE]
> **Question : et un `Mono` vide ?**
> `verifyComplete()` sans aucun `expectNext` : aucun élément, puis la fin. C'est le test `articleInconnu`, sur `trouver(9)`, un article qui n'existe pas. Lancé avec `mvn test "-Dtest=CatalogueTest#articleInconnu"`, il passe, sur trois exécutions.

## Oublier verify

`sansVerify` déclare des attentes fausses : le `Flux` émet trois titres, pas `"faux"`. Il se termine par `expectComplete()`, et non par `verifyComplete()`. Avant de le lancer, prédire : le test échoue-t-il ?

```bash
mvn test "-Dtest=CatalogueTest#sansVerify"
```

Exécuté sur Java 25, Spring Boot 4.1.1 et Reactor 3.8.7, avec le même résultat sur trois exécutions : le test passe.

`create` et les méthodes `expect…` ne font que décrire les attentes. C'est une méthode `verify…` qui s'abonne au `Flux` et compare chaque signal : sans elle, personne ne s'abonne, et rien n'est vérifié. `verifyComplete()` fait les deux d'un seul appel, `expectComplete()` puis `verify()`. Le piège ne se voit pas, puisque le test est vert.

## Deux heures en quelques millisecondes

`deuxMesuresEnDeuxHeures` teste un `Flux` qui émet une mesure par heure, deux fois : `Flux.interval(Duration.ofHours(1)).take(2)`. Avec l'horloge réelle, le test durerait deux heures.

- `StepVerifier.withVirtualTime` remplace l'horloge de Reactor par une horloge virtuelle, que le test avance lui-même.
- Le `Flux` est construit dans le fournisseur passé à `withVirtualTime`, la lambda `() -> …` : il est créé après le remplacement de l'horloge.
- `thenAwait(Duration.ofHours(2))` avance l'horloge virtuelle de deux heures : les deux mesures, 0 et 1, sont émises aussitôt.

```bash
mvn test "-Dtest=CatalogueTest#deuxMesuresEnDeuxHeures"
```

Exécuté sur Java 25, Spring Boot 4.1.1 et Reactor 3.8.7, avec le même résultat sur trois exécutions : le test passe en moins d'une seconde.

`fluxHorsDuFournisseur` déclare les mêmes attentes, mais construit le `Flux` avant d'appeler `withVirtualTime`, puis le renvoie depuis le fournisseur. Il termine par `verify(Duration.ofSeconds(3))`, qui abandonne au bout de trois secondes.

```bash
mvn test "-Dtest=CatalogueTest#fluxHorsDuFournisseur"
```

Exécuté sur Java 25, Spring Boot 4.1.1 et Reactor 3.8.7, avec le même résultat sur trois exécutions : le test échoue après un peu plus de 3 s.

```text
[ERROR] Failures: 
[ERROR]   CatalogueTest.fluxHorsDuFournisseur:101 VerifySubscriber timed out on reactor.core.publisher.FluxLimitRequest$FluxLimitRequestSubscriber@…
```

Ce qui suit `@` est un identifiant d'objet, qui peut différer d'un poste à l'autre.

`Flux.interval(Duration)` choisit son horloge au moment où il est construit. Construit hors du fournisseur, il garde l'horloge réelle : `thenAwait` avance une horloge virtuelle qu'il n'utilise pas, et la première mesure n'arrivera qu'au bout d'une heure réelle. `verify()` n'a aucun délai par défaut : sans `verify(Duration.ofSeconds(3))`, le test attendrait les deux heures réelles. Ce dernier point se lit dans le code de Reactor ; il n'a pas été mesuré.

> [!TIP]
> **Ce qu'il faut retenir**
> - `block()` rend la liste ou l'exception : il ne peut pas vérifier ce qui précède une erreur.
> - `StepVerifier` vérifie chaque signal dans l'ordre : les éléments, puis la fin ou l'erreur.
> - Rien n'est vérifié sans une méthode `verify…` : c'est elle qui s'abonne.
> - `withVirtualTime` teste un flux lent sans l'attendre, à condition de construire le `Flux` dans le fournisseur.

Sources : [Reactor, Testing](https://projectreactor.io/docs/core/release/reference/testing.html), [Reactor 3.8.7, StepVerifier, lignes 41 à 45](https://github.com/reactor/reactor-core/blob/v3.8.7/reactor-test/src/main/java/reactor/test/StepVerifier.java#L41-L45), [ligne 96](https://github.com/reactor/reactor-core/blob/v3.8.7/reactor-test/src/main/java/reactor/test/StepVerifier.java#L96) et [lignes 168 à 191](https://github.com/reactor/reactor-core/blob/v3.8.7/reactor-test/src/main/java/reactor/test/StepVerifier.java#L168-L191), [Reactor 3.8.7, BlockingSingleSubscriber, ligne 102](https://github.com/reactor/reactor-core/blob/v3.8.7/reactor-core/src/main/java/reactor/core/publisher/BlockingSingleSubscriber.java#L102), [Reactor 3.8.7, Flux.interval, ligne 1245](https://github.com/reactor/reactor-core/blob/v3.8.7/reactor-core/src/main/java/reactor/core/publisher/Flux.java#L1245).

# TP - Contextes de test

Ce TP teste un service Spring avec l'injection de ses dépendances, sans démarrer toute l'application. Le service testé est `Commandes`, du TP « Exposer une API WebFlux » : quatre commandes du jour, livrées une toutes les 300 ms, et une suppression. Le TP compare d'abord un contexte réduit à l'application entière, puis compte les contextes que Spring crée, montre comment un contexte partagé transporte l'état d'une classe de test à l'autre, et lance enfin deux classes en parallèle.

| Projet | `demo-contextes-test` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-webclient`, et en portée test `spring-boot-starter-webflux-test` |
| Classe de démarrage | `com.example.demo.DemoContextesTestApplication`, que les tests ne démarrent pas |
| Fichiers | paquet `com.example.demo`, comme dans `demo-application-reactive` : `Commandes.java`, le fichier de départ du TP « Exposer une API WebFlux » ; `CommandeHandler.java`, son corrigé `CommandeHandler.java` ; `Tarif.java`, le record du TP « Appeler et diffuser » ; `TarifClient.java`, son corrigé `TarifClient.java`. Les tests et leurs ressources se créent pendant le TP. |
| Sur le poste | un IDE Java |

Ces classes sont celles des TP, recopiées telles quelles : le projet ne dépend pas de `demo-application-reactive`. Les tests ne démarrent ni serveur ni service de tarifs, et le projet n'a pas de base de données.

Le `pom.xml` du projet déclare le starter de test de WebFlux :

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux-test</artifactId>
    <scope>test</scope>
</dependency>
```

Il apporte JUnit, `spring-test`, qui contient les annotations de test de Spring, et `reactor-test`, qui contient `StepVerifier`.

Les classes de test se créent dans `src/test/java/com/example/demo/`. Elles se lancent une par une, ou par groupe, en les nommant avec `-Dtest`.

> [!WARNING]
> **Ne pas lancer `mvn test` sans `-Dtest`**
> Trois tests du TP échouent volontairement : `TarifClientTest`, `WebConfigTest`, et `CommandesTest` quand il passe après `SuppressionTest`. Lancés ensemble, ils donnent `BUILD FAILURE`, et c'est le résultat attendu.

## Un contexte réduit

`CommandesTest.java`

```java
package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

import reactor.test.StepVerifier;

@SpringJUnitConfig(Commandes.class)
class CommandesTest {

    @Autowired
    Commandes commandes;

    @Test
    void quatreCommandesDuJour() {
        StepVerifier.create(commandes.duJour())
            .expectNextCount(4)
            .verifyComplete();
    }
}
```

- `@SpringJUnitConfig` réunit deux annotations : `@ExtendWith(SpringExtension.class)`, qui branche Spring sur JUnit, et `@ContextConfiguration`, qui nomme les classes à charger.
- Ici, une seule classe est nommée, `Commandes`. Spring en fait un bean, et l'injecte dans le champ `@Autowired`. Rien d'autre n'est chargé.
- `StepVerifier` compte les quatre commandes du jour, puis attend la fin du `Flux`.

```bash
mvn test "-Dtest=CommandesTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : le test passe en quelques secondes. Plus d'une seconde vient des quatre commandes, émises à 300 ms d'intervalle ; le reste est le démarrage du premier contexte.

```text
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```

Le contexte réduit coûte peu. Mesuré avec une classe de test chronométrée : le contexte de `@SpringJUnitConfig(Commandes.class)` compte 7 beans, et se crée en quelques millisecondes quand un autre contexte a déjà été chargé dans la même exécution, en moins d'une seconde s'il est le premier. Avec `@SpringBootTest`, qui démarre l'application entière, `demo-application-reactive` en fin de TP « Sécuriser une application réactive » compte 236 beans : son contexte demande plusieurs secondes, et MongoDB doit tourner, puisque `ComptesConfig` enregistre les comptes au démarrage.

Le prix du contexte réduit : Spring ne crée que ce qu'on lui nomme. L'auto-configuration de Spring Boot, qui fournit d'ordinaire les beans d'infrastructure, ne s'applique pas. `TarifClientTest` nomme `TarifClient`, du TP « Appeler et diffuser », dont le constructeur reçoit un `WebClient.Builder` :

`TarifClientTest.java`

```java
package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

@SpringJUnitConfig(TarifClient.class)
class TarifClientTest {

    @Autowired
    TarifClient client;

    @Test
    void clientInjecte() {
    }
}
```

```bash
mvn test "-Dtest=TarifClientTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : le test échoue, sans même s'exécuter. Maven résume l'erreur par `Failed to load ApplicationContext` ; sa cause se lit plus haut dans la console, sur la dernière ligne `Caused by` :

```text
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'org.springframework.web.reactive.function.client.WebClient$Builder' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}
```

Dans l'application, c'est Spring Boot qui fournit le `WebClient.Builder`. Dans un contexte réduit, ce que Spring Boot fournirait, le test doit le déclarer lui-même.

`@SpringJUnitWebConfig` réunit `@SpringJUnitConfig` et `@WebAppConfiguration` : le contexte devient un contexte web Servlet, avec un `ServletContext` simulé. `WebConfigTest` l'emploie sur le même service :

`WebConfigTest.java`

```java
package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit.jupiter.web.SpringJUnitWebConfig;

@SpringJUnitWebConfig(Commandes.class)
class WebConfigTest {

    @Autowired
    Commandes commandes;

    @Test
    void contexteWeb() {
    }
}
```

```bash
mvn test "-Dtest=WebConfigTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : le test ne démarre pas.

```text
[ERROR] Errors: 
[ERROR]   WebConfigTest » BeanInstantiation Failed to instantiate [org.springframework.test.context.web.WebDelegatingSmartContextLoader]: Constructor threw exception
```

Les deux lignes `Caused by` de la console donnent la raison :

```text
Caused by: java.lang.NoClassDefFoundError: jakarta/servlet/ServletContext
Caused by: java.lang.ClassNotFoundException: jakarta.servlet.ServletContext
```

Un projet WebFlux n'a pas l'API Servlet. L'annotation est valide, mais elle appartient au monde Spring MVC : elle sert à tester une configuration Spring MVC sans Spring Boot.

## Combien de contextes

Spring garde en mémoire chaque contexte de test qu'il crée, et le réutilise pour toute classe qui déclare la même configuration, dans la même exécution. La configuration qui fait la clé : les classes nommées, mais aussi les profils actifs, les propriétés de test et les doublures comme `@MockitoBean`. Une classe qui change l'une d'elles obtient un autre contexte.

Pour voir ce que fait le cache, créer `src/test/resources/logback-test.xml` :

```bash
mkdir -p src/test/resources && touch src/test/resources/logback-test.xml
```

```
<configuration>
  <appender name="C" class="ch.qos.logback.core.ConsoleAppender">
    <encoder><pattern>%d{HH:mm:ss.SSS} [%thread] %logger{20} %msg%n</pattern></encoder>
  </appender>
  <logger name="org.springframework.test.context.cache" level="DEBUG"/>
  <root level="WARN"><appender-ref ref="C"/></root>
</configuration>
```

Il affiche en `DEBUG` le journal du cache des contextes, et le reste en `WARN` seulement. Puis créer deux classes de test.

`SuppressionTest.java`

```java
package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

import reactor.test.StepVerifier;

@SpringJUnitConfig(Commandes.class)
class SuppressionTest {

    @Autowired
    Commandes commandes;

    @Test
    void supprimerLaDeux() {
        StepVerifier.create(commandes.supprimer(2)).verifyComplete();
        StepVerifier.create(commandes.parNumero(2)).verifyComplete();
    }
}
```

`SuppressionTest` supprime la commande 2, puis vérifie que `parNumero(2)` ne renvoie plus rien. Il nomme la même classe que `CommandesTest`.

`HandlerTest.java`

```java
package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.mock.web.reactive.function.server.MockServerRequest;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

import reactor.test.StepVerifier;

@SpringJUnitConfig({Commandes.class, CommandeHandler.class})
class HandlerTest {

    @Autowired
    CommandeHandler handler;

    @Test
    void laCommandeUn() {
        StepVerifier.create(handler.parNumero(MockServerRequest.builder().pathVariable("numero", "1").build()))
            .expectNextMatches(r -> r.statusCode() == HttpStatus.OK)
            .verifyComplete();
    }
}
```

`HandlerTest` nomme deux classes, `Commandes` et `CommandeHandler`, et appelle le handler avec une requête simulée par `MockServerRequest` : la commande 1 doit répondre 200.

Avant de lancer les trois classes, prédire : combien de contextes Spring crée-t-il pour `SuppressionTest`, `CommandesTest` et `HandlerTest` ?

```bash
mvn test -Dsurefire.runOrder=reversealphabetical "-Dtest=SuppressionTest,CommandesTest,HandlerTest"
```

Sans option, Maven Surefire lance les classes dans l'ordre où le système de fichiers les donne, qui n'est pas garanti. `-Dsurefire.runOrder=reversealphabetical` impose l'ordre alphabétique inverse : `SuppressionTest`, `HandlerTest`, puis `CommandesTest`.

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions. La console affiche plusieurs lignes de statistiques du cache ; la dernière se termine ainsi :

```text
Spring test ApplicationContext cache statistics: [DefaultContextCache@… size = 2, maxSize = 32, contextUsageCount = 1, parentContextCount = 0, hitCount = 29, missCount = 2, failureCount = 0]
```

- `missCount = 2` : deux fois, Spring n'a pas trouvé de contexte en cache et en a chargé un.
- `size = 2` : deux contextes sont en cache. `SuppressionTest` et `CommandesTest` partagent celui de `Commandes`. `HandlerTest` nomme deux classes : c'est une autre configuration, donc un second contexte, même si elle contient la première.

Le build se termine pourtant par `BUILD FAILURE`, sur `CommandesTest` : c'est l'objet de la sous-section suivante.

> [!NOTE]
> **Question : comment compter les contextes ?**
> En passant le logger `org.springframework.test.context.cache` au niveau `DEBUG`, comme le fait `logback-test.xml`. Spring y affiche ses statistiques : `missCount` compte les contextes chargés, `size` ceux qui sont en cache.

## Une suppression qui déborde

Dans l'exécution précédente, `SuppressionTest` a supprimé la commande 2, puis `CommandesTest` a attendu quatre commandes du jour. Aucun test ne s'exécutait en parallèle. Avant de lire le résultat, prédire : `CommandesTest` passe-t-il ?

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : `SuppressionTest` et `HandlerTest` passent, `CommandesTest` échoue.

```text
[ERROR] Failures: 
[ERROR]   CommandesTest.quatreCommandesDuJour expectation "expectNextCount(4)" failed (expected: count = 4; actual: counted = 3; signal: onComplete())
```

Lancé seul, à la première sous-section, `CommandesTest` passait. Les deux classes reçoivent le même contexte, donc le même bean `Commandes`, et sa map garde la suppression d'une classe à l'autre. Le résultat dépend de l'ordre des classes : un test qui passe seul et échoue dans la suite est le symptôme d'un état partagé.

Deux corrections sont possibles : écrire un test qui ne laisse pas d'état derrière lui, ou demander à Spring de fermer le contexte après la classe qui le modifie. C'est le rôle de `@DirtiesContext`. L'ajouter sur `SuppressionTest`, avec son import :

```java
package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

import reactor.test.StepVerifier;

@SpringJUnitConfig(Commandes.class)
@DirtiesContext
class SuppressionTest {

    @Autowired
    Commandes commandes;

    @Test
    void supprimerLaDeux() {
        StepVerifier.create(commandes.supprimer(2)).verifyComplete();
        StepVerifier.create(commandes.parNumero(2)).verifyComplete();
    }
}
```

```bash
mvn test -Dsurefire.runOrder=reversealphabetical "-Dtest=SuppressionTest,CommandesTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : les deux classes passent, et la dernière ligne de statistiques porte `size = 1` et `missCount = 2`. Le contexte de `SuppressionTest` a été fermé après la classe, et `CommandesTest` en a chargé un nouveau, avec ses quatre commandes. La correction se paie d'un rechargement.

Retirer ensuite `@DirtiesContext` et son import : la sous-section suivante part du fichier d'origine.

> [!WARNING]
> **Pourquoi deux `StepVerifier` dans `SuppressionTest` ?**
> Écrit en une seule chaîne, `commandes.supprimer(2).then(commandes.parNumero(2))` émet encore la commande 2 : `parNumero` lit la map au moment où la chaîne est construite, donc avant la suppression. Exécuté sur une classe de test séparée, trois fois : `expectation "expectComplete" failed (expected: onComplete(); actual: onNext(Commande[numero=2, client=Martin, montant=45.5]))`. Deux `StepVerifier` successifs suppriment d'abord, puis relisent.

## Deux classes en parallèle

`CommandesBisTest` est une copie de `CommandesTest` sous un autre nom : deux classes qui lisent les commandes, sans rien modifier.

`CommandesBisTest.java`

```java
package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

import reactor.test.StepVerifier;

@SpringJUnitConfig(Commandes.class)
class CommandesBisTest {

    @Autowired
    Commandes commandes;

    @Test
    void quatreCommandesDuJour() {
        StepVerifier.create(commandes.duJour())
            .expectNextCount(4)
            .verifyComplete();
    }
}
```

D'abord en séquence :

```bash
mvn test "-Dtest=CommandesTest,CommandesBisTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : les deux classes passent, l'une après l'autre. La première dure un peu plus de deux secondes, démarrage du contexte compris ; la seconde un peu plus d'une seconde, le temps de ses quatre commandes.

Créer ensuite `src/test/resources/junit-platform.properties` :

```
junit.jupiter.execution.parallel.enabled=true
junit.jupiter.execution.parallel.mode.default=concurrent
```

La première propriété active l'exécution parallèle de JUnit ; la seconde rend les tests concurrents par défaut. Relancer la même commande.

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur cinq exécutions : les deux classes passent, et durent ensemble à peu près le temps de la première, un peu plus de deux secondes. Le journal du cache montre deux threads de JUnit, `ForkJoinPool-1-worker-1` et `ForkJoinPool-1-worker-2` : les deux classes tournent en même temps, et reçoivent le même contexte en cache.

La condition est celle que la sous-section précédente a montrée : un bean partagé ne doit pas garder d'état modifié par un test. Exécuté cinq fois en parallèle avec `SuppressionTest` à la place de `CommandesBisTest` : `CommandesTest` échoue chaque fois, sur `counted = 3`. La documentation de Spring demande de ne pas exécuter en parallèle les tests qui emploient `@DirtiesContext`, `@MockitoBean` ou `@MockitoSpyBean`, ou qui modifient un état partagé, comme une base de données ou des fichiers.

Supprimer ensuite `junit-platform.properties`, pour que les classes du projet s'exécutent de nouveau en séquence.

> [!TIP]
> **Ce qu'il faut retenir**
> - `@SpringJUnitConfig` ne charge que les classes nommées : un contexte petit et rapide, sans l'auto-configuration de Spring Boot.
> - `@SpringJUnitWebConfig` demande l'API Servlet : il ne démarre pas dans un projet WebFlux.
> - Spring réutilise un contexte pour toute classe de même configuration : un bean qui garde un état le transporte d'une classe à l'autre.
> - `@DirtiesContext` ferme le contexte après la classe, au prix d'un rechargement.
> - Les tests en parallèle partagent les mêmes contextes : ils ne doivent modifier aucun état partagé.

Sources : [Spring Framework 7.0.9, SpringJUnitConfig, lignes 51 et 52](https://github.com/spring-projects/spring-framework/blob/v7.0.9/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringJUnitConfig.java#L51-L52), [Spring Framework 7.0.9, SpringJUnitWebConfig, lignes 55 à 57](https://github.com/spring-projects/spring-framework/blob/v7.0.9/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/web/SpringJUnitWebConfig.java#L55-L57), [Spring Framework, Context Caching](https://docs.spring.io/spring-framework/reference/testing/testcontext-framework/ctx-management/caching.html), [Spring Framework, Parallel Test Execution](https://docs.spring.io/spring-framework/reference/testing/testcontext-framework/parallel-test-execution.html), [Maven Surefire, runOrder](https://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html#runOrder).

# TP - WebTestClient

Les TP ont vérifié chaque route à la main, avec `curl`. À chaque modification du code, il faudrait tout refaire. `WebTestClient` envoie une requête à l'application et vérifie son statut, ses en-têtes et son corps, dans un test automatique, sans démarrer de serveur. Ce TP teste ainsi les routes des commandes, puis les règles de sécurité du TP « Sécuriser une application réactive », puis le client `TarifClient` contre un faux serveur.

| Projet | `demo-webtestclient` |
|---|---|
| Dépendances | `spring-boot-starter-webflux`, `spring-boot-starter-webclient`, `spring-boot-starter-security`, `spring-boot-starter-data-mongodb-reactive`, et en portée test `spring-boot-starter-webflux-test` ; deux dépendances de test s'ajoutent pendant le TP |
| Classe de démarrage | `com.example.demo.DemoWebtestclientApplication`, que les tests ne démarrent pas |
| Fichiers | paquet `com.example.demo`, comme dans `demo-application-reactive` : les classes de `demo-application-reactive` en fin de TP « Sécuriser une application réactive » qui servent aux tests, recopiées depuis les corrigés (liste ci-dessous). Les tests se créent pendant le TP. |
| Sur le poste | un IDE Java |

| Fichier | Corrigé recopié |
|---|---|
| `Commandes.java` | `Commandes.java, après l'étape 3`, TP « Sécuriser une application réactive » |
| `CommandeController.java` | `CommandeController.java, après l'étape 2`, TP « Exposer une API WebFlux » |
| `CommandeHandler.java` | `CommandeHandler.java, après l'étape 3`, TP « Sécuriser une application réactive » |
| `CommandeRoutes.java` | `CommandeRoutes.java`, TP « Exposer une API WebFlux » |
| `SecuriteConfig.java` | `SecuriteConfig.java, après l'étape 3`, TP « Sécuriser une application réactive » |
| `Livre.java`, `LivreRepository.java`, `Nouveaute.java`, `NouveauteRepository.java` | TP « Persister avec MongoDB réactif » |
| `LivreController.java` | `LivreController.java, avec la suppression`, TP « Sécuriser une application réactive » |
| `Tarif.java`, `TarifClient.java` | TP « Appeler et diffuser » |

Le projet ne dépend pas de `demo-application-reactive`. Il ne contient ni comptes, ni journal des nouveautés, ni autre contrôleur : seulement ce que les tests appellent. Les tests ne démarrent ni serveur, ni MongoDB, ni service de tarifs.

> [!WARNING]
> **Ne pas lancer `mvn test` sans `-Dtest`**
> Plusieurs tests de ce TP échouent volontairement. Lancés ensemble, ils donnent `BUILD FAILURE`, et c'est le résultat attendu.

## Un contrôleur seul

`CommandeControllerTest.java`

```java
package com.example.demo;

import static org.mockito.Mockito.when;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webflux.test.autoconfigure.WebFluxTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.reactive.server.WebTestClient;

import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

@WebFluxTest(CommandeController.class)
class CommandeControllerTest {

    @Autowired
    WebTestClient client;

    @MockitoBean
    Commandes commandes;

    @Test
    void commandesDuJour() {
        when(commandes.duJour()).thenReturn(Flux.just(
            new Commande(1, "Durand", 120.0),
            new Commande(2, "Martin", 45.5)));

        Flux<Commande> corps = client.get().uri("/commandes")
            .accept(MediaType.APPLICATION_NDJSON)
            .exchange()
            .expectStatus().isOk()
            .returnResult(Commande.class)
            .getResponseBody();

        StepVerifier.create(corps)
            .expectNext(new Commande(1, "Durand", 120.0))
            .expectNext(new Commande(2, "Martin", 45.5))
            .verifyComplete();
    }
}
```

- `@WebFluxTest(CommandeController.class)` démarre la couche WebFlux et le seul contrôleur nommé, sans services ni base de données. Il fournit le `WebTestClient`, injecté par `@Autowired`.
- `CommandeController` a besoin d'un `Commandes`. `@MockitoBean` met à sa place une doublure, dont `when(...).thenReturn(...)` programme les réponses : ici, deux commandes du jour.
- `accept(MediaType.APPLICATION_NDJSON)` demande le flux, une commande par ligne, comme le `curl` du TP « Exposer une API WebFlux ». `expectStatus().isOk()` vérifie le statut.
- `returnResult(Commande.class).getResponseBody()` rend le corps sous forme de `Flux<Commande>`, que `StepVerifier` vérifie commande par commande. `Commande` est un record : deux commandes de mêmes valeurs sont égales.

`@WebFluxTest` a changé de paquetage en Spring Boot 4 : `org.springframework.boot.webflux.test.autoconfigure`. Spring Boot 3 le plaçait dans `org.springframework.boot.test.autoconfigure.web.reactive` : un import copié d'un projet Spring Boot 3 ne compile plus.

```bash
mvn test "-Dtest=CommandeControllerTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : le test passe en quelques secondes, démarrage du contexte compris.

> [!NOTE]
> **Question : et `@WebFluxTest` sans nommer de contrôleur ?**
> Il charge alors tous les contrôleurs du projet, dont `LivreController`, qui demande un `LivreRepository` qu'aucune doublure ne fournit. Exécuté trois fois : le contexte ne se charge pas, sur `No qualifying bean of type '…LivreRepository'`. Nommer le contrôleur testé limite le contexte à ce dont il a besoin.

## La commande 1

`CommandeControllerTest` reçoit un second test, `commandeConnue`, et l'import de `Mono` qu'il emploie. La doublure renvoie la commande 1 pour `parNumero(1)`, puis le test demande `GET /commandes/1` et attend 200 :

```java
package com.example.demo;

import static org.mockito.Mockito.when;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webflux.test.autoconfigure.WebFluxTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.reactive.server.WebTestClient;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

@WebFluxTest(CommandeController.class)
class CommandeControllerTest {

    @Autowired
    WebTestClient client;

    @MockitoBean
    Commandes commandes;

    @Test
    void commandesDuJour() {
        when(commandes.duJour()).thenReturn(Flux.just(
            new Commande(1, "Durand", 120.0),
            new Commande(2, "Martin", 45.5)));

        Flux<Commande> corps = client.get().uri("/commandes")
            .accept(MediaType.APPLICATION_NDJSON)
            .exchange()
            .expectStatus().isOk()
            .returnResult(Commande.class)
            .getResponseBody();

        StepVerifier.create(corps)
            .expectNext(new Commande(1, "Durand", 120.0))
            .expectNext(new Commande(2, "Martin", 45.5))
            .verifyComplete();
    }

    @Test
    void commandeConnue() {
        when(commandes.parNumero(1)).thenReturn(Mono.just(new Commande(1, "Durand", 120.0)));

        client.get().uri("/commandes/1").exchange().expectStatus().isOk();
    }
}
```

Avant de lancer le test, prédire le code reçu.

```bash
mvn test "-Dtest=CommandeControllerTest#commandeConnue"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions :

```text
[ERROR] Failures: 
[ERROR]   CommandeControllerTest.commandeConnue:48 Status expected:<200 OK> but was:<404 NOT_FOUND>
```

Depuis l'étape 2 du TP « Exposer une API WebFlux », `GET /commandes/{numero}` est déclaré dans `CommandeRoutes`, une route fonctionnelle servie par `CommandeHandler`, et non plus par une méthode de `CommandeController`. `@WebFluxTest` ne détecte que les contrôleurs annotés : aucune route ne répond à `/commandes/1`, d'où le 404, quelle que soit la doublure. Un test « commande 9 → 404 », écrit seul, aurait été vert sans jamais exécuter le handler.

La correction : importer la route et son handler, en gardant le contrôleur nommé. `CommandeRoutesTest.java` :

```java
package com.example.demo;

import static org.mockito.Mockito.when;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webflux.test.autoconfigure.WebFluxTest;
import org.springframework.context.annotation.Import;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.reactive.server.WebTestClient;

import reactor.core.publisher.Mono;

@WebFluxTest(CommandeController.class)
@Import({CommandeRoutes.class, CommandeHandler.class})
class CommandeRoutesTest {

    @Autowired
    WebTestClient client;

    @MockitoBean
    Commandes commandes;

    @Test
    void commandeConnue() {
        when(commandes.parNumero(1)).thenReturn(Mono.just(new Commande(1, "Durand", 120.0)));

        client.get().uri("/commandes/1")
              .exchange()
              .expectStatus().isOk()
              .expectBody(Commande.class).isEqualTo(new Commande(1, "Durand", 120.0));
    }

    @Test
    void commandeInconnue() {
        when(commandes.parNumero(9)).thenReturn(Mono.empty());

        client.get().uri("/commandes/9").exchange().expectStatus().isNotFound();
    }
}
```

```bash
mvn test "-Dtest=CommandeRoutesTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : les deux tests passent, 200 pour la commande 1, 404 pour la commande 9.

> [!NOTE]
> **Question : pourquoi ne pas utiliser `@SpringBootTest` ?**
> Il charge toute l'application : dans `demo-application-reactive` en fin de TP « Sécuriser une application réactive », plus de deux cents beans, plusieurs secondes de démarrage, et MongoDB doit tourner, comme l'a montré le TP « Contextes de test ». Les routes et la sécurité y seraient, mais aussi la base. `@WebFluxTest` vérifie la couche web seule, en une fraction de seconde une fois le premier contexte chargé.

## Le module de test de sécurité

Le projet porte `spring-boot-starter-security` et les règles de `SecuriteConfig`, comme `demo-application-reactive` depuis le TP « Sécuriser une application réactive », où `curl` sans compte reçoit 401 sur `/commandes`. Pourtant, les tests ci-dessus passent sans aucun compte.

Ajouter au `pom.xml`, dans `<dependencies>`, le starter de test de sécurité, sans modifier les tests :

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

Avant de relancer les tests, prédire : restent-ils verts ?

```bash
mvn test "-Dtest=CommandeControllerTest,CommandeRoutesTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9, Spring Security 7.1.1 et JUnit 6.0.3, avec le même résultat sur trois exécutions : les quatre tests échouent.

```text
[ERROR] Failures: 
[ERROR]   CommandeControllerTest.commandeConnue:48 Status expected:<200 OK> but was:<401 UNAUTHORIZED>
[ERROR]   CommandeControllerTest.commandesDuJour:34 Status expected:<200 OK> but was:<401 UNAUTHORIZED>
[ERROR]   CommandeRoutesTest.commandeConnue:30 Status expected:<200 OK> but was:<401 UNAUTHORIZED>
[ERROR]   CommandeRoutesTest.commandeInconnue:38 Status expected:<404 NOT_FOUND> but was:<401 UNAUTHORIZED>
```

En Spring Boot 4.1.1, la sécurité n'entre dans un test `@WebFluxTest` que par le module `spring-boot-security-test`, que ce starter apporte. Sans lui, la sécurité était absente des tests : ils étaient verts sur une API qui, lancée pour de vrai, répond 401. Avec lui, mais sans les règles du projet, c'est la sécurité par défaut de Spring Boot qui s'applique : tout exige un compte. `SecuriteConfig` est une classe `@Configuration`, que `@WebFluxTest` ne détecte pas.

> [!NOTE]
> **Question : ajouter seulement `spring-security-test`, comme en Spring Boot 3 ?**
> Cela ne suffit plus. Exécuté trois fois avec `spring-security-test` seul et `@Import(SecuriteConfig.class)` : le contexte ne se charge pas, sur `No qualifying bean of type 'org.springframework.security.config.web.server.ServerHttpSecurity'`.

Les tests doivent donc importer les règles du projet, et appeler l'API avec un utilisateur. Supprimer `CommandeControllerTest`, et créer `src/test/java/com/example/demo/CommandeSecuriteTest.java` :

```java
package com.example.demo;

import static org.mockito.Mockito.when;
import static org.springframework.security.test.web.reactive.server.SecurityMockServerConfigurers.mockUser;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webflux.test.autoconfigure.WebFluxTest;
import org.springframework.context.annotation.Import;
import org.springframework.http.MediaType;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.reactive.server.WebTestClient;

import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

@WebFluxTest(CommandeController.class)
@Import(SecuriteConfig.class)
class CommandeSecuriteTest {

    @Autowired
    WebTestClient client;

    @MockitoBean
    Commandes commandes;

    @Test
    void sansCompte() {
        client.get().uri("/commandes").exchange().expectStatus().isUnauthorized();
    }

    @Test
    void commandesDuJour() {
        when(commandes.duJour()).thenReturn(Flux.just(
            new Commande(1, "Durand", 120.0),
            new Commande(2, "Martin", 45.5)));

        Flux<Commande> corps = client.mutateWith(mockUser("Durand"))
            .get().uri("/commandes")
            .accept(MediaType.APPLICATION_NDJSON)
            .exchange()
            .expectStatus().isOk()
            .returnResult(Commande.class)
            .getResponseBody();

        StepVerifier.create(corps)
            .expectNext(new Commande(1, "Durand", 120.0))
            .expectNext(new Commande(2, "Martin", 45.5))
            .verifyComplete();
    }
}
```

Remplacer ensuite le contenu de `CommandeRoutesTest.java` :

```java
package com.example.demo;

import static org.mockito.Mockito.when;
import static org.springframework.security.test.web.reactive.server.SecurityMockServerConfigurers.mockUser;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webflux.test.autoconfigure.WebFluxTest;
import org.springframework.context.annotation.Import;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.reactive.server.WebTestClient;

import reactor.core.publisher.Mono;

@WebFluxTest(CommandeController.class)
@Import({SecuriteConfig.class, CommandeRoutes.class, CommandeHandler.class})
class CommandeRoutesTest {

    @Autowired
    WebTestClient client;

    @MockitoBean
    Commandes commandes;

    @Test
    void commandeConnue() {
        when(commandes.parNumero(1)).thenReturn(Mono.just(new Commande(1, "Durand", 120.0)));

        client.mutateWith(mockUser("Durand"))
              .get().uri("/commandes/1")
              .exchange()
              .expectStatus().isOk()
              .expectBody(Commande.class).isEqualTo(new Commande(1, "Durand", 120.0));
    }

    @Test
    void commandeInconnue() {
        when(commandes.parNumero(9)).thenReturn(Mono.empty());

        client.mutateWith(mockUser("Durand"))
              .get().uri("/commandes/9")
              .exchange()
              .expectStatus().isNotFound();
    }
}
```

- `@Import(SecuriteConfig.class)` charge le `SecurityWebFilterChain` du TP : les règles d'URL du projet s'appliquent dans le test.
- `mockUser("Durand")`, de `spring-security-test`, place un utilisateur dans la requête, sans mot de passe, sans authentification et sans compte en base.
- `sansCompte` vérifie qu'une requête sans utilisateur reçoit 401, comme `curl` sans `-u`.

```bash
mvn test "-Dtest=CommandeSecuriteTest,CommandeRoutesTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9, Spring Security 7.1.1 et JUnit 6.0.3, avec le même résultat sur trois exécutions : les quatre tests passent.

## Les règles de sécurité

Les règles d'URL du TP portent surtout sur les livres : `GET /livres/**` ouvert à tous, `DELETE /livres/**` réservé au rôle `ADMIN`. `LivreSecuriteTest.java` les vérifie :

```java
package com.example.demo;

import static org.mockito.Mockito.when;
import static org.springframework.security.test.web.reactive.server.SecurityMockServerConfigurers.mockUser;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webflux.test.autoconfigure.WebFluxTest;
import org.springframework.context.annotation.Import;
import org.springframework.data.mongodb.core.ReactiveMongoTemplate;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.reactive.server.WebTestClient;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@WebFluxTest(LivreController.class)
@Import(SecuriteConfig.class)
class LivreSecuriteTest {

    @Autowired
    WebTestClient client;

    @MockitoBean
    LivreRepository depot;

    @MockitoBean
    ReactiveMongoTemplate template;

    @MockitoBean
    NouveauteRepository nouveautes;

    @Test
    void lectureAnonyme() {
        when(depot.findAll()).thenReturn(Flux.empty());

        client.get().uri("/livres").exchange().expectStatus().isOk();
    }

    @Test
    void suppressionAnonyme() {
        client.delete().uri("/livres/1").exchange().expectStatus().isUnauthorized();
    }

    @Test
    void suppressionParUnUtilisateur() {
        client.mutateWith(mockUser("Martin").roles("USER"))
              .delete().uri("/livres/1")
              .exchange()
              .expectStatus().isForbidden();
    }

    @Test
    void suppressionParUnAdministrateur() {
        when(depot.deleteById("1")).thenReturn(Mono.empty());

        client.mutateWith(mockUser("admin").roles("ADMIN"))
              .delete().uri("/livres/1")
              .exchange()
              .expectStatus().isNoContent();
    }
}
```

- `LivreController` reçoit trois dépendances dans son constructeur : il faut trois doublures. Avec la seule doublure de `LivreRepository`, exécuté trois fois, le contexte ne se charge pas, sur `No qualifying bean of type 'org.springframework.data.mongodb.core.ReactiveMongoTemplate'`.
- `mockUser("Martin").roles("USER")` et `mockUser("admin").roles("ADMIN")` simulent les deux rôles du TP.
- Les quatre tests reprennent les vérifications faites avec `curl` au TP : lecture anonyme 200, suppression anonyme 401, suppression par `Martin` 403, suppression par `admin` 204.

```bash
mvn test "-Dtest=LivreSecuriteTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9, Spring Security 7.1.1 et JUnit 6.0.3, avec le même résultat sur trois exécutions : les quatre tests passent.

> [!NOTE]
> **Question : et sans `@Import(SecuriteConfig.class)` ?**
> C'est la sécurité par défaut de Spring Boot qui s'applique. Exécuté trois fois sans cette ligne :
>
> ```text
> [ERROR] Failures: 
> [ERROR]   LivreSecuriteTest.lectureAnonyme:35 Status expected:<200 OK> but was:<401 UNAUTHORIZED>
> [ERROR]   LivreSecuriteTest.suppressionAnonyme:40 Status expected:<401 UNAUTHORIZED> but was:<403 FORBIDDEN>
> [ERROR]   LivreSecuriteTest.suppressionParUnAdministrateur:58 Status expected:<204 NO_CONTENT> but was:<403 FORBIDDEN>
> ```
>
> `suppressionParUnUtilisateur` passe quand même : `Martin` reçoit 403, mais pour une mauvaise raison. La protection CSRF de la sécurité par défaut refuse toute suppression sans jeton CSRF, quel que soit le rôle. Un test qui ne vérifierait que « Martin → 403 » serait vert sans tester la règle : c'est le test de l'administrateur qui révèle l'oubli.

> [!NOTE]
> **Question : et sans `when(depot.deleteById("1"))` ?**
> Une doublure non programmée renvoie `null`. Exécuté trois fois sans cette ligne : le test de l'administrateur passe quand même, en 204. `retirer` porte `@ResponseStatus(HttpStatus.NO_CONTENT)` : WebFlux pose le statut 204 avant de regarder la valeur renvoyée, puis, pour une méthode qui renvoie un `Mono<Void>`, remplace `null` par un `Mono` vide. Ce rattrapage ne vaut que dans ce cas précis. La ligne `when` dit explicitement ce que le dépôt renvoie, sans compter sur lui.

Chaque combinaison de doublures et d'imports forme une configuration différente, donc un contexte de plus dans le cache. La consigne du TP « Contextes de test » s'applique : ne pas exécuter en parallèle les tests qui emploient `@MockitoBean`.

## Un faux serveur

`TarifClient`, du TP « Appeler et diffuser », appelle le service de tarifs. Il lit son adresse dans la propriété `tarifs.url`, `http://localhost:8081` par défaut. Un test peut donc le construire avec une autre adresse : celle d'un faux serveur.

MockWebServer démarre un vrai serveur HTTP sur le poste, sur un port libre, et renvoie les réponses qu'on lui a préparées. Le `WebClient` de `TarifClient` passe par le réseau, comme en production, sans que le vrai service tourne.

Ajouter au `pom.xml`, dans `<dependencies>` :

```
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver3</artifactId>
    <version>5.3.0</version>
    <scope>test</scope>
</dependency>
```

La version s'écrit dans le `pom.xml` : Spring Boot 4.1.1 ne la gère pas. 5.3.0 est celle qu'emploie Spring Framework 7.0.9. Le premier lancement télécharge environ 3,3 Mo, avec OkHttp, Okio et la bibliothèque Kotlin.

`TarifClientServeurTest.java`

```java
package com.example.demo;

import java.io.IOException;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.reactive.function.client.WebClientResponseException;

import mockwebserver3.MockResponse;
import mockwebserver3.MockWebServer;
import mockwebserver3.RecordedRequest;
import reactor.test.StepVerifier;

class TarifClientServeurTest {

    MockWebServer serveur = new MockWebServer();
    TarifClient tarifs;

    @BeforeEach
    void demarrer() throws IOException {
        serveur.start();
        tarifs = new TarifClient(WebClient.builder(), serveur.url("/").toString());
    }

    @AfterEach
    void arreter() {
        serveur.close();
    }

    @Test
    void tarifRecu() throws InterruptedException {
        serveur.enqueue(new MockResponse.Builder()
            .addHeader("Content-Type", "application/json")
            .body("{\"id\":\"1\",\"montant\":49.0}")
            .build());

        StepVerifier.create(tarifs.tarif("1"))
            .expectNext(new Tarif("1", 49.0))
            .verifyComplete();

        RecordedRequest requete = serveur.takeRequest();
        System.out.println("[serveur] " + requete.getMethod() + " " + requete.getTarget());
    }

    @Test
    void tarifInconnu() {
        serveur.enqueue(new MockResponse.Builder().code(404).build());

        StepVerifier.create(tarifs.tarif("9"))
            .expectError(WebClientResponseException.NotFound.class)
            .verify();
    }
}
```

- `serveur.start()` démarre le faux serveur ; `serveur.url("/")` donne son adresse, transmise à `TarifClient` à la place de `tarifs.url`.
- `enqueue` met en file la réponse que le serveur renverra au prochain appel : un tarif en JSON, ou un 404.
- `takeRequest()` rend la requête que le serveur a reçue : le test peut vérifier ce que le client a réellement envoyé.
- Un 404 du serveur donne une erreur `WebClientResponseException.NotFound`, que `expectError` vérifie.

```bash
mvn test "-Dtest=TarifClientServeurTest"
```

Exécuté sur Java 25, Spring Boot 4.1.1, Spring Framework 7.0.9 et JUnit 6.0.3, avec le même résultat sur trois exécutions : les deux tests passent, et la console affiche la requête reçue par le faux serveur.

```text
[serveur] GET /tarifs/1
```

> [!WARNING]
> **Les exemples écrits pour OkHttp 4 ne compilent pas**
> Beaucoup d'exemples en ligne construisent la réponse avec `new MockResponse().setHeader(...).setBody(...)`. Avec `mockwebserver3`, la compilation échoue : `no suitable constructor found for MockResponse(no arguments)`. La réponse se construit avec `MockResponse.Builder`, comme ci-dessus.

> [!TIP]
> **Ce qu'il faut retenir**
> - `@WebFluxTest` teste la couche web seule : nommer le contrôleur, remplacer ses dépendances par des doublures `@MockitoBean`, importer ce qu'il ne détecte pas, routes fonctionnelles et règles de sécurité.
> - Sans `spring-boot-starter-security-test`, la sécurité est absente du test : des tests verts peuvent porter sur une API qui répond 401.
> - `mockUser` simule un utilisateur et ses rôles, sans mot de passe ni compte en base.
> - Un test de 403 seul peut passer pour une mauvaise raison : vérifier aussi le cas autorisé.
> - Un client `WebClient` se teste contre un faux serveur, MockWebServer, à qui l'on donne l'adresse du serveur.

Sources : [Spring Boot, Auto-configured Spring WebFlux Tests](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html#testing.spring-boot-applications.spring-webflux-tests), [Spring Framework, WebTestClient](https://docs.spring.io/spring-framework/reference/testing/webtestclient.html), [Spring Boot, Test Modules](https://docs.spring.io/spring-boot/reference/testing/test-modules.html), [Spring Boot 4.1.1, spring-boot-security-test, auto-configurations ajoutées à WebTestClient](https://github.com/spring-projects/spring-boot/blob/v4.1.1/module/spring-boot-security-test/src/main/resources/META-INF/spring/org.springframework.boot.webtestclient.autoconfigure.AutoConfigureWebTestClient.imports), [Spring Security, Testing Authentication](https://docs.spring.io/spring-security/reference/reactive/test/web/authentication.html), [Spring Framework 7.0.9, InvocableHandlerMethod, lignes 221 à 232](https://github.com/spring-projects/spring-framework/blob/v7.0.9/spring-webflux/src/main/java/org/springframework/web/reactive/result/method/InvocableHandlerMethod.java#L221-L232), [Spring Framework 7.0.9, ReactiveAdapter, lignes 109 et 110](https://github.com/spring-projects/spring-framework/blob/v7.0.9/spring-core/src/main/java/org/springframework/core/ReactiveAdapter.java#L109-L110), [Spring Framework, WebClient, Testing](https://docs.spring.io/spring-framework/reference/web/webflux-webclient/client-testing.html), [Spring Framework 7.0.9, version de mockwebserver3, ligne 40](https://github.com/spring-projects/spring-framework/blob/v7.0.9/framework-platform/framework-platform.gradle#L40).

# TP - Tester une application réactive

**Objectif :** écrire les tests automatiques d'une application réactive, du flux isolé jusqu'à l'API sécurisée.

| Dossier à ouvrir | `demo-application-reactive`, après le TP « Sécuriser une application réactive » |
|---|---|
| Avant de commencer | rien à démarrer : ni MongoDB, ni `demo-webclient` |
| Commande | `./mvnw test -Dtest=` suivi du nom de la classe de test, depuis le dossier du projet, ou le test lancé dans l'IDE ; jamais `./mvnw test` seul |

Deux comportements de `demo-application-reactive` n'ont été vérifiés qu'à la main. Au TP « Appeler et diffuser », le repli des tarifs en direct se constatait en arrêtant le service distant, `curl -N` ouvert. Au TP « Sécuriser une application réactive », la règle d'annulation des commandes se constatait avec `curl -u`, compte par compte. Ce TP en fait des tests qui tournent en quelques secondes, sans service distant ni `curl`.

| Étape | Ce qu'on construit |
|---|---|
| 1 | Le test du repli : un tarif par seconde, puis un tarif vide pendant une panne, en temps virtuel |
| 2 | Le test de la règle d'annulation : qui peut annuler quelle commande |
| 3 | Aucune ligne de test : casser une règle à la fois, et compter les tests qui échouent |

## Le code de départ

`demo-application-reactive`, dans son état de fin du TP « Sécuriser une application réactive ». Les classes testées y sont déjà :

- `TarifController`, dont la méthode `direct` appelle `TarifClient` chaque seconde et remplace un appel en échec par `new Tarif(id, null)` ;
- `Commandes`, qui garde en mémoire quatre commandes, et dont la méthode `annuler` porte la règle `@PreAuthorize("hasRole('ADMIN') or #commande.client() == authentication.name")` ;
- `CommandeRoutes` et `CommandeHandler`, qui routent `DELETE /commandes/{numero}` vers `annuler` ;
- `SecuriteConfig`, avec ses règles d'URL et `@EnableReactiveMethodSecurity`.

Les quatre commandes de `Commandes` :

| Numéro | Client |
|---|---|
| 1 | Durand |
| 2 | Martin |
| 3 | Petit |
| 4 | Leroy |

Dans le `pom.xml`, vérifier la présence de ces deux dépendances dans `<dependencies>`, et les ajouter si elles manquent. La première apporte JUnit, Mockito, `StepVerifier` et `@WebFluxTest` ; la seconde, `mockUser` et la sécurité dans `@WebFluxTest`.

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

Les classes de test se créent dans `src/test/java`, dans le même package que la classe d'application. Aucun test ne démarre l'application entière : ni MongoDB ni le service distant des tarifs n'ont besoin de tourner. Pas de dépendance à ajouter pour un faux serveur : aucun appel HTTP ne sort des tests.

Un test se lance depuis l'IDE, ou en ligne de commande, à la racine du projet :

```bash
mvn test -Dtest=TarifControllerTest
```

## Étape 1 : le repli en temps virtuel

`TarifController.direct` émet un tarif par seconde, sans fin. Le tester sur l'horloge réelle prendrait des secondes par test, et vérifier le repli demanderait d'arrêter le service distant. Le temps virtuel règle le premier point, une doublure de `TarifClient` le second.

Créer `src/test/java/com/example/demo/TarifControllerTest.java` :

1. Le contexte ne contient que `TarifController` : `@SpringJUnitConfig(TarifController.class)`.
2. `TarifClient` est remplacé par une doublure, `@MockitoBean`, et le contrôleur est injecté par `@Autowired`.
3. Premier test, un tarif par seconde : la doublure renvoie `new Tarif("1", 12.5)` pour l'identifiant `"1"`. Avec `StepVerifier.withVirtualTime`, vérifier que rien n'arrive pendant la première seconde, puis le tarif, puis encore rien pendant une seconde, puis le tarif. Le flux ne se termine jamais : finir par `thenCancel()`, puis `verify()`.
4. Second test, le repli : la doublure renvoie `Mono.error(new IllegalStateException("service arrêté"))`. Avancer l'horloge de deux secondes, puis vérifier deux fois `new Tarif("1", null)`, et finir de la même façon.

Critère de réussite :

| Vérification | Résultat attendu |
|---|---|
| `mvn test -Dtest=TarifControllerTest` | les deux tests passent, la classe en une fraction de seconde |
| Le même test, après avoir retiré `.onErrorReturn(new Tarif(id, null))` de `TarifController` | le test du repli échoue ; remettre la ligne ensuite |

Le message d'échec, sans le repli :

```text
expectation "expectNext(Tarif[id=1, montant=null])" failed (expected: onNext(Tarif[id=1, montant=null]); actual: onError(java.lang.IllegalStateException: service arrêté))
```

C'est la coupure constatée au TP « Appeler et diffuser » : l'erreur d'un appel termine le flux entier.

> [!NOTE]
> **Question : Le test passe, mais en trois secondes au lieu d'une. Pourquoi ?**
> Le flux est construit hors du fournisseur passé à `withVirtualTime`, par exemple `var flux = controleur.direct("1");` puis `withVirtualTime(() -> flux)`. `Flux.interval` choisit son horloge au moment où il est construit : il a pris l'horloge réelle, et le test attend deux vraies secondes. Appeler `controleur.direct("1")` dans le fournisseur : `withVirtualTime(() -> controleur.direct("1"))`.

## Étape 2 : qui peut annuler une commande

La règle d'annulation est portée par `Commandes.annuler`. Pour la tester avec `WebTestClient`, le contexte doit contenir le vrai `Commandes`, pas une doublure.

Créer `src/test/java/com/example/demo/AnnulationTest.java` en collant ce fichier :

```java
import static org.springframework.security.test.web.reactive.server.SecurityMockServerConfigurers.mockUser;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webflux.test.autoconfigure.WebFluxTest;
import org.springframework.context.annotation.Import;
import org.springframework.test.web.reactive.server.WebTestClient;

@WebFluxTest(CommandeController.class)
@Import({SecuriteConfig.class, CommandeRoutes.class, CommandeHandler.class, Commandes.class})
class AnnulationTest {

    @Autowired
    WebTestClient client;

    @Test
    void martinNAnnulePasLaCommandeDeDurand() {
        client.mutateWith(mockUser("Martin"))
              .delete().uri("/commandes/1")
              .exchange()
              .expectStatus().isForbidden();
    }

    @Test
    void durandAnnuleSaCommande() {
        client.mutateWith(mockUser("Durand"))
              .delete().uri("/commandes/1")
              .exchange()
              .expectStatus().isNoContent();
    }

    @Test
    void adminAnnuleLaCommandeDeMartin() {
        client.mutateWith(mockUser("admin").roles("ADMIN"))
              .delete().uri("/commandes/2")
              .exchange()
              .expectStatus().isNoContent();
    }

    @Test
    void anonymeRefuse() {
        client.delete().uri("/commandes/3")
              .exchange()
              .expectStatus().isUnauthorized();
    }
}
```

`@WebFluxTest` ne charge que `CommandeController` ; `@Import` ajoute la configuration de sécurité, les routes fonctionnelles, leur handler et le service `Commandes`. Aucun `@MockitoBean` : les quatre commandes sont celles du service.

**Prédiction, à noter avant de lancer la classe :** quel statut reçoit Martin quand il tente d'annuler la commande 1, celle de Durand ?

Lancer la classe :

```bash
mvn test -Dtest=AnnulationTest
```

Puis lancer seul le test de Martin :

```bash
mvn test -Dtest='AnnulationTest#martinNAnnulePasLaCommandeDeDurand'
```

Comparer les deux résultats à la prédiction, puis corriger la classe :

1. Chaque test vise sa propre commande, qu'aucun autre test ne touche.
2. Durand annule la commande 1, la sienne.
3. `admin` annule la commande 2, celle de Martin.
4. Martin tente d'annuler la commande 3, celle de Petit.
5. L'anonyme tente d'annuler la commande 4.

Critère de réussite, `mvn test -Dtest=AnnulationTest` :

| Test | Statut attendu |
|---|---|
| Martin, commande 3 | 403 |
| Durand, commande 1 | 204 |
| `admin`, commande 2 | 204 |
| anonyme, commande 4 | 401 |

> [!NOTE]
> **Question : Pourquoi ne pas doubler `Commandes` avec `@MockitoBean`, comme pour un contrôleur ?**
> La doublure n'a pas la règle. Avec `@MockitoBean Commandes`, Martin reçoit 204 si `annuler` est programmée par `when(...).thenReturn(Mono.empty())`, et 500 si elle ne l'est pas, puisqu'une doublure non programmée renvoie `null`. Jamais 403. `@PreAuthorize` s'applique par un proxy que Spring pose autour du bean qu'il crée ; la doublure est enregistrée telle quelle, sans ce proxy. Pour tester une règle posée sur une méthode, garder le vrai bean qui la porte.

## Étape 3 : casser une règle

Une suite verte dit que le code respecte les tests, pas que chaque règle est testée. Pour le savoir, casser une règle, relancer les deux classes du TP, et compter les tests qui échouent.

**Prédiction, à noter avant de casser quoi que ce soit :** pour chacune des trois règles ci-dessous, combien des six tests du TP échouent ?

| Règle cassée | Modification |
|---|---|
| a | Retirer `@EnableReactiveMethodSecurity` de `SecuriteConfig` |
| b | Dans l'expression de `Commandes.annuler`, retirer `hasRole('ADMIN') or ` |
| c | Dans `SecuriteConfig`, remplacer `.anyExchange().authenticated()` par `.anyExchange().permitAll()` |

Casser une seule règle à la fois, lancer les deux classes, noter le nombre d'échecs, puis remettre la règle avant de passer à la suivante :

```bash
mvn test -Dtest='TarifControllerTest,AnnulationTest'
```

Critère de réussite : les trois nombres sont relevés, chaque règle est remise, et les six tests passent de nouveau.

Sortie attendue :

| Règle cassée | Tests en échec |
|---|---|
| a, sans `@EnableReactiveMethodSecurity` | 1 : Martin, `expected:<403 FORBIDDEN> but was:<204 NO_CONTENT>` ; sans elle, `@PreAuthorize` est ignorée |
| b, sans `hasRole('ADMIN') or` | 1 : `admin`, `expected:<204 NO_CONTENT> but was:<403 FORBIDDEN>` ; la commande 2 n'est pas la sienne |
| c, `anyExchange().permitAll()` | 0 : l'anonyme reçoit toujours 401, car `@PreAuthorize` le refuse à son tour |

> [!TIP]
> **Ce qu'il faut retenir**
> - Un flux temporisé se teste en temps virtuel, à condition d'être construit dans le fournisseur passé à `withVirtualTime`.
> - Une doublure remplace ce dont dépend la classe testée : le repli se vérifie sans arrêter de service distant.
> - Une règle `@PreAuthorize` se teste avec le vrai bean qui la porte ; une doublure n'a pas de proxy de sécurité.
> - Un test ne compte pas sur un autre : l'ordre des méthodes d'une classe n'est pas forcément celui du fichier, et un état partagé fait échouer un test pour une raison qui n'est pas la sienne.
> - Casser une règle et compter les échecs montre ce que la suite vérifie vraiment.

## Corrigé - Tester une application réactive


### TarifControllerTest.java

```java
import static org.mockito.Mockito.when;

import java.time.Duration;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

import reactor.core.publisher.Mono;
import reactor.test.StepVerifier;

@SpringJUnitConfig(TarifController.class)
class TarifControllerTest {

    @Autowired
    TarifController controleur;

    @MockitoBean
    TarifClient tarifs;

    @Test
    void unTarifParSeconde() {
        when(tarifs.tarif("1")).thenReturn(Mono.just(new Tarif("1", 12.5)));

        StepVerifier.withVirtualTime(() -> controleur.direct("1"))
            .expectSubscription()
            .expectNoEvent(Duration.ofSeconds(1))
            .expectNext(new Tarif("1", 12.5))
            .expectNoEvent(Duration.ofSeconds(1))
            .expectNext(new Tarif("1", 12.5))
            .thenCancel()
            .verify();
    }

    @Test
    void repliPendantLaPanne() {
        when(tarifs.tarif("1")).thenReturn(Mono.error(new IllegalStateException("service arrêté")));

        StepVerifier.withVirtualTime(() -> controleur.direct("1"))
            .thenAwait(Duration.ofSeconds(2))
            .expectNext(new Tarif("1", null), new Tarif("1", null))
            .thenCancel()
            .verify();
    }
}
```

**Étape 1.** `@SpringJUnitConfig(TarifController.class)` ne crée que le contrôleur ; `@MockitoBean` y place une doublure de `TarifClient`, que `when(...).thenReturn(...)` programme. `withVirtualTime` remplace l'horloge de Reactor par une horloge que le test avance lui-même : `expectNoEvent(Duration.ofSeconds(1))` l'avance d'une seconde en vérifiant que rien n'arrive, `thenAwait` l'avance sans rien vérifier. Le flux de `direct` ne se termine jamais : `thenCancel()` annule l'abonnement une fois les attentes vérifiées. Dans le second test, chaque appel à la doublure échoue ; `onErrorReturn`, placé sur l'appel, remplace l'erreur par `Tarif("1", null)`, et le flux continue à la seconde suivante. Sans lui, l'erreur atteint le flux principal et le termine : c'est le message d'échec obtenu en retirant la ligne.

### AnnulationTest.java

```java
import static org.springframework.security.test.web.reactive.server.SecurityMockServerConfigurers.mockUser;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webflux.test.autoconfigure.WebFluxTest;
import org.springframework.context.annotation.Import;
import org.springframework.test.web.reactive.server.WebTestClient;

@WebFluxTest(CommandeController.class)
@Import({SecuriteConfig.class, CommandeRoutes.class, CommandeHandler.class, Commandes.class})
class AnnulationTest {

    @Autowired
    WebTestClient client;

    @Test
    void martinNAnnulePasLaCommandeDePetit() {
        client.mutateWith(mockUser("Martin"))
              .delete().uri("/commandes/3")
              .exchange()
              .expectStatus().isForbidden();
    }

    @Test
    void durandAnnuleSaCommande() {
        client.mutateWith(mockUser("Durand"))
              .delete().uri("/commandes/1")
              .exchange()
              .expectStatus().isNoContent();
    }

    @Test
    void adminAnnuleLaCommandeDeMartin() {
        client.mutateWith(mockUser("admin").roles("ADMIN"))
              .delete().uri("/commandes/2")
              .exchange()
              .expectStatus().isNoContent();
    }

    @Test
    void anonymeRefuse() {
        client.delete().uri("/commandes/4")
              .exchange()
              .expectStatus().isUnauthorized();
    }
}
```

**Étape 2.** La prédiction juste était : 204. Lancée en entier, la classe de départ fait échouer le seul test de Martin :

```text
Status expected:<403 FORBIDDEN> but was:<204 NO_CONTENT>
```

Lancé seul, le même test passe : la règle fonctionne. JUnit n'exécute pas forcément les méthodes dans l'ordre du fichier ; il suit un ordre fixe d'une exécution à l'autre, mais volontairement non évident. Ici, le test de Durand passe en premier et supprime la commande 1 du vrai `Commandes`, que tous les tests de la classe partagent puisqu'ils partagent le même contexte. Martin vise ensuite une commande qui n'existe plus : `flatMap` ne reçoit rien, `annuler` n'est pas appelée, et la réponse est 204, comme pour une commande inconnue. Dans la version corrigée, chaque test vise sa propre commande, et aucun ne dépend de l'ordre des autres.

**Étape 3.** Les réponses justes étaient : 1, 1 et 0.

| Règle cassée | Tests en échec |
|---|---|
| a, sans `@EnableReactiveMethodSecurity` | 1 : Martin, `expected:<403 FORBIDDEN> but was:<204 NO_CONTENT>` ; sans elle, `@PreAuthorize` est ignorée |
| b, sans `hasRole('ADMIN') or` | 1 : `admin`, `expected:<204 NO_CONTENT> but was:<403 FORBIDDEN>` ; la commande 2 n'est pas la sienne |
| c, `anyExchange().permitAll()` | 0 : l'anonyme reçoit toujours 401, car `@PreAuthorize` le refuse à son tour |

Le cas c montre ce que la suite ne vérifie pas : la règle d'URL qui exige un compte pour `/commandes`. Pour la couvrir, il faudrait un test d'une route que seule cette règle protège, comme `GET /commandes` sans compte, qui doit répondre 401.

Sources : [Reactor, Testing](https://projectreactor.io/docs/core/release/reference/testing.html), [Spring Security, Reactive Method Security](https://docs.spring.io/spring-security/reference/reactive/authorization/method.html), [Spring Security, Testing Authentication](https://docs.spring.io/spring-security/reference/reactive/test/web/authentication.html), [JUnit, TestMethodOrder](https://github.com/junit-team/junit-framework/blob/r6.0.3/junit-jupiter-api/src/main/java/org/junit/jupiter/api/TestMethodOrder.java#L37-L38).
