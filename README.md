# Cluster NiFi

Création cluster NiFi dans docker : Apache Nifi 1.27.0 en date du 19/09/2024

Voir ici [Running a cluster with Apache Nifi and Docker](https://www.nifi.rocks/apache-nifi-docker-compose-cluster/) 

Utilise l'image docker suivante : [Apache NiFi Image](https://hub.docker.com/r/apache/nifi).


# Sommaire : 

* [Installation](#Installation)
* [Lancement](#Lancement)
* [Création ou chargement de Flows](#Template)
* [Kafka](#Kafka)
* [Process de données](#Process)
* [Stop](#Stop)
* [Exécution de versions spécifiques](#Images)
* [Registry](#Registry)
* [Pour aller plus loin](#Subdocs)

# <a name="installation"></a>Installation

#### Remarque : si on utilise un environnement sur VM, on va d'abord cloner l'environnement :
```
git clone https://github.com/crystalloide/nifi-cluster

cd nifi-cluster

```
Avant de commencer, on va créer un nouveau dépôt git pour stocker les flows.

```
git init ../flow_storage
sudo chown -R 1000.1000 ../flow_storage
```

## <a name="Lancement"></a>Lancement

Lancement du cluster et de son écosystème :
   
1. Récupération des images : 
   ``docker compose up -d``

2. Pour suivre le bon démarrage :
  ``docker ps -a ``

3. Création de quelques topics dans Kafka :
   ``bin/launch-script.sh``

4. Accès au proxy Nginx proxy ici [http://localhost:80](http://localhost:80/)
    
5. On peut aussi accéder directement  à l'UI de NiFi - après quelques minutes - sur l'URL via un navigateur Web :

   [http://localhost:8080/nifi/](http://localhost:8080/nifi)
   
6. Définir quelques flows et traiter des données.
   
7. Si NiFi Registry est bien lancé également [registry](#registry) : 
   
   alors on fait le lien entre le cluster NiFi et NiFi Registry :
   
    ``bin/add-registry.sh``

Il faut patienter un peu, le temps que le cluster NiFi soit effectivement opérationnel, 

c'est-à-dire le temps que les noeuds NiFi soient correctement configurés et connectés en cluster

9°) Quelques templates sont disponibles ici : [template](#Template) 

  ``/home/user/nifi-cluster/flow_templates``

ou dans le NiFi Registry [NiFi registry](#Registry) 

10°) On peut regarder la production et consommation de messages dans Kafka [Kafka](#Kafka)


11°) Gestion de Kafka : 

## <a name="kafka"></a>Kafka

### Exécution de commandes

Vous pouvez exécuter des commandes Kafka ponctuelles en utilisant la commande :

``docker compose run <service> <...>`` 

qui lance un conteneur distinct en utilisant la même image mais en exécutant la commande. 

Cela peut cependant être assez lent car vous devez lancer le conteneur pour chaque commande.

Après un certain temps, ces conteneurs s'accumulent, ce que vous pouvez voir avec la commande :
``docker compose ps -a``

Si cela devient un problème, nettoyez-les avec la commande : 
``docker container prune``

12°) Exécution de commandes spécifique dans un container : 

docker exec -it <nom_du_container> commande

``docker compose run kafka bash ``

```
[+] Running 1/0
 ⠿ Container zookeeper Running 0.0s
kafka 09:24:44.07
kafka 09:24:44.07 Welcome to the Bitnami kafka container
kafka 09:24:44.08 Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-kafka
kafka 09:24:44.08 Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-kafka/issues
kafka 09:24:44.08

I have no name!@d2f135d230e4:/$ echo $PATH
/opt/bitnami/kafka/bin:/opt/bitnami/java/bin:/opt/bitnami/java/bin:/opt/bitnami/common/bin:/opt/bitnami/kafka/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

I have no name!@d2f135d230e4:/$ ls /opt/bitnami/kafka/bin
connect-distributed.sh        kafka-console-consumer.sh    ...

I have no name!@d2f135d230e4:/$ exit
exit
$
```

#### Exécution d'une commande dans le container :

Vous pouvez exécuter des scripts ou des commandes spécifiques qui se trouvent déjà dans le conteneur.

On peut utiliser l'alias de service « kafka » comme raccourci pour joindre le premier broker kafka disponible.

**Création d'un Topic**
```
docker compose run kafka kafka-topics.sh \
  --bootstrap-server kafka:9092 \
  --create --topic my.source.topic \
  --replication-factor 3 --config retention.ms=36000000
```

[+] Running 1/0
⠿ Container zookeeper Running 0.0s
kafka 09:43:21.17
kafka 09:43:21.18 Welcome to the Bitnami kafka container
kafka 09:43:21.18 Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-kafka
kafka 09:43:21.18 Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-kafka/issues
kafka 09:43:21.19

WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic my.source.topic.


**Description d'un Topic**
```
docker compose run kafka kafka-topics.sh \
  --bootstrap-server kafka:9092 \
  --describe --topic my.source.topic
```

[+] Running 1/0
⠿ Container zookeeper Running 0.0s
kafka 09:45:45.41
kafka 09:45:45.41 Welcome to the Bitnami kafka container
kafka 09:45:45.42 Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-kafka
kafka 09:45:45.42 Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-kafka/issues
kafka 09:45:45.42

Topic: my.source.topic  TopicId: ou824ZiQRo-gELS07nh3mg PartitionCount: 1       ReplicationFactor: 3    Configs: segment.bytes=1073741824,retention.ms=36000000
        Topic: my.source.topic  Partition: 0    Leader: 1001    Replicas: 1001,1003,1002        Isr: 1001,1003,1002


### Exécution de Scripts

On peut également utiliser la commande run pour monter un répertoire local, 
puis exécuter tous les scripts qui pourraient s'y trouver. 
Les chemins doivent être donnés en "absolus" (arborescence complète dpeuis /)

```
docker compose run --volume <path-to-mount>:<mount-point> kafka <mount-point>/<script-name>
```

Note : on peut jeter un oeil aux scripts *bin/launch-script.sh* et *bin/create-topics.sh* pour voir la manière dont cela est fait.

## <a name="process de données"></a>Process

À présent, vous devriez avoir chargé le flow à partir du template dans NiFi et configuré les topics dans Kafka. 

On peut maintenant travailler avec les données.

1. Démarrer tous les processus du flux en appuyant sur le bouton de démarrage de la boîte de dialogue *Operate*.
1. Envoyer un ou plusieurs messages dans le topic source. (*ctrl-D* pour terminer)
1. Observer les messages en cours de traitement dans le flux.
1. Récupérer le message du topic final. (*ctrl-C* pour terminer)

Pour plus de visibilité, on peut exécuter les deux commandes Kafka dans des consoles séparées afin de voir chaque message être traité.

### Production de quelques messages dans le topic source :

```
docker compose run kafka kafka-console-producer.sh \
 --bootstrap-server kafka:9092 --topic my.source.topic
```
>hello world
>now is the time
>one is the number
> ^D


Remarque : cela peut être fait aussi avec le script : 
``bin/launch-script.sh producer``.

### Affichage des messages produits : 

```
docker compose run kafka kafka-console-consumer.sh \
 --bootstrap-server kafka:9092 --topic my.sink.topic --offset earliest --partition 0
```
hello world
now is the time
one is the number
^C

Remarque : cela peut être fait aussi avec le script : 
``bin/launch-script.sh consumer``

## <a name="stop"></a>Stop

Il suffit de faire la commande : 
``docker compose down``
pour arrêter le cluster et détruire les conteneurs. 
Si vous souhaitez conserver les conteneurs, utilisez plutôt : 
``docker compose stop``

# <a name="images"></a>Running Specific Versions Of NiFi

The cluster uses a locally built image of NiFi based on the official NiFi image. This gives scope to add extra tools at the build stage instead of waiting until run time. At present this only involves installing the package *redis-tools* which is used in one of the [experiments](docs/experiment-redis_direct.md) where an ExecuteStreamCommand processor runs the tools in a shell to run ad hoc Redis commands.

This image uses the build script in *build-nifi/Dockerfile* to perform this task, and is referred from *docker-compose.yml* so that the build is performed automatically for you.

You can however rebuild this image manually any time you wish with ``docker compose build``.

You can also override the "latest" tag within the build file to run a specific version of NiFi. For example:

```
docker compose build --build-arg NIFI_VERSION=1.16.0
```

The image is still tagged as latest so will be used the next time "up" is called.

# <a name="registry"></a>Utilisation d'un Nifi Registry

Un service de NiFi Registry a été ajouté pour faciliter la persistance des flux plutôt que d'avoir à utiliser les Templates 

(A noter que les Templates seront déppréciés d'ailleurs en NiFi 2.0.)

On se connectez-vous à l'interface graphique de NiFi Registry ici :

http://localhost:18080/nifi-registry


## 1ère fois : 

La première fois qu'on utilise un Nifi Registry, on doit configurer un bucket et éventuellement y placer un flux. 

Il s'agit d'un processus manuel.

1. Dans le registre, cliquer sur la clé puis créez un nouveau bucket.
   
2. Dans NiFi, utiliser le menu "Controller Settings" -> "Registry Clients"
 
3. Ajouter un nouveau client avec l'URL "`http://registry:18080/`"
   
4. Sur le bureau, créer un processor group
 
5. À l'intérieur du groupe, faire glisser le template de flux test.
   
6. En arrière-plan, faire un clic droit et sélectionnez "Version" ->  "Start version control"

7. Dans la boîte de dialogue, donner un nom au flux et cliquer sur Enregistrer.

Vous verrez que le bucket "test" et le snapshot du flux ont été créés dans le dépôt git.

The first time you use the registry you need to set up the bucket, and optionally put a flow into it. This is a manual process.


## Ensuite :

Une fois le registre configuré, tous les flux créés seront stockés dans le dépôt git local, ce qui vous assure la persistance. 

Si on redémarre le cluster, on verra dans le registre que les définitions de flux ont été conservées.

Sur NiFi, on doit toujours créer le lien vers le Registry comme décrit ci-dessus vers "`http://registry:18080/`"

On peux importer ensuite le flux sur le canevas : 

1. Faire glisser un Process Group de la barre de menu vers le canevas
1. Cliquer sur "Import from Registry"
1. Sélectionner le bucket, le flow et la version souhaitée
1. Cliquer sur « Import »


## Ajout automatique du NiFi Registry dans le cluster NiFi : 

A lancer manuellement : 
``bin/add-registry.sh`` 


# <a name="Subdocs"></a>Pour aller plus loin :

* [Custom Processors](docs/custom_processors.md)
* [Issues](docs/issues.md)
* [Working With Elasticsearch](docs/elasticsearch.md)
* [Experiments](docs/experiments.md)
  * [Standard Processors](docs/experiment-standard_processors.md)
  * [Using Tab Separation](docs/experiment-tab_separation.md)
  * [Convert To ECS](docs/experiment-convert_to_ecs.md)
  * [Enrich From Redis](docs/experiment-enrich_from_redis.md)
  * [Write To Redis](docs/experiment-write_to_redis.md)
  * [Some Specific Transform Cases](docs/experiment-some_specific_transform_cases.md)
  * [Unpacking Lookups](docs/experiment-unpacking_lookups.md)
  * [Fork / Join Enrichment](docs/experiment-fork_join_enrichment.md)
  * [Grok Filtering](docs/experiment-grok_filtering.md)
  * [Running Redis Commands Within NiFi](docs/experiment-redis_direct.md)
