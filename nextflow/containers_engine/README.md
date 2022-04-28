---

title: "nextflow nouveaux containers engine"
author: "Philippe La Rosa"
team: "HPC benchmark/conception"
date: "15 Juin 2021"
output: html_document

---

Project description
-------------------

- Type : "Proof of concept" essayer d'appréhender les nouveaux moteurs de containers utilisables via nextflow :
podman, shifter, charliecloud.
- Bioinformaticians contact: Philippe La Rosa (philippe.larosa@curie.fr)


Dependencies
------------

- nextflow 
- podman containers (This feature requires Nextflow version 20.01.0 or later.)
_**This is an incubating feature. The use in production environment is not recommended.**_
- shifter containers (This feature requires Shifter version 18.03 (or later) and Nextflow 19.10.0 (or later).)
- charliecloud containers (This feature requires Nextflow version 21.03.0-edge or later and Charliecloud ``v0.22` or later.)
_**This is an incubating feature. The use in production environment is not recommended.**_



Podman containers
-------------------

- Podman est un outsider qui est prometteur, mais il a probablement besoin d'un ou deux ans supplémentaires pour la plupart des cas d'utilisation de HPC.

Podman est un environnement d'exécution de conteneur développé par Red Hat. Son objectif principal est de remplacer Docker. Bien qu'il ne soit pas explicitement conçu pour les cas d'utilisation HPC, il se veut un « wrapper » léger pour exécuter des conteneurs sans la surcharge du démon Docker complet. De plus, l'équipe de développement de Podman étudie récemment une meilleure prise en charge des cas d'utilisation HPC.

Podman cas d'utilisation HPC qui ne sont pas pris en charge actuellemnt pour certaines des raisons suivantes :

Prise en charge manquante des systèmes de fichiers parallèles (par exemple, IBM Spectrum Scale)
Rootless Podman a été conçu pour utiliser des espaces de noms d'utilisateurs du noyau qui ne sont pas compatibles avec la plupart des systèmes de fichiers parallèles (peut changer dans un an ou deux)
Pas encore possible de définir les valeurs par défaut de la stratégie de site du système
L'extraction d'images Docker/OCI nécessite plusieurs subuids/subgids (peut changer dans un an ou deux)
Là où Podman brille, c'est en fournissant un moyen d'exécuter et de créer des conteneurs sans accès root ni setuid.

Les mêmes défis et problèmes nécessaires à Podman pour exécuter des conteneurs OCI dans un environnement HPC sont les mêmes problèmes rencontrés par Singularity pour créer des images SIF sans racine dans l'environnement HPC : mapper les UID aux sous-utilisateurs/sous-gids sur les nœuds de calcul. Plus intéressant encore, Buildah offre un moyen prometteur de permettre aux utilisateurs de créer des images de conteneur en tant qu'images Docker/OCI, le tout sans racine. Il est plausible d'utiliser Buildah comme mécanisme de livraison d'image de conteneur et d'échanger l'implémentation d'exécution du conteneur (Podman vs Singularity) en fonction des besoins et des exigences spécifiques.

- Prerequis

Podman doit être installé sur l'environnement d'exécution, par ex. la BIWS ou un cluster distribué, selon l'endroit où doit vous s'exécuter le pipeline. L'exécution en mode sans racine nécessite une configuration appropriée du système d'exploitation. En raison des limites actuelles de Podman, l'utilisation de cpuset pour les processeurs et la mémoire n'est possible qu'en utilisant sudo.  Espaces de noms d'utilisateurs dans le noyau Linux.

Pour le moment podman est installé sur ma BIWS : `u900-bdd-1-112n-6990.curie.fr`

- Comment ça fonctionne via nextflow

Il n'est pas necessaire de modifier le script Nextflow pour l'exécuter avec Podman. Spécifiez simplement l'image Podman à partir de laquelle les conteneurs sont démarrés à l'aide de l'option de ligne de commande `-with-podman`. Par example:

`nextflow run <script> -with-podman [image du conteneur OCI]`

Chaque fois que le script lance une exécution de processus, Nextflow l'exécute dans un conteneur Podman créé à l'aide de l'image spécifiée. En pratique, Nextflow encapsulera automatiquement les processus et les exécutera en exécutant la commande **podman run** avec l'image fournie.

Il est possible de ne pas utiliser une seule image Podman en tant que paramètre de la ligne de commande, il faut dans ce cas les définir dans le fichier de configuration Nextflow. 

Par exemple, ajouter les lignes suivantes dans le fichier nextflow.config :

```
process.container = 'nextflow/examples:latest'
podman.enabled = true
```

- Comment ça fonctionne via la ligne de commande

```
podman run nextflow/exemples:latest
```

Cas d'erreur prévisible car podman veut utiliser des espaces de noms d'utilisateurs du noyau qui ne sont pas compatibles avec la plupart des systèmes de fichiers parallèles
```
podman run nextflow/exemples:latest
cannot clone: Invalid argument
user namespaces are not enabled in /proc/sys/user/max_user_namespaces
Error: could not get runtime: cannot re-exec process
```

- Pour plus de détails consulter : 

[Nextflow Podman containers](https://www.nextflow.io/docs/latest/podman.html)

[scope Podman](https://www.nextflow.io/docs/latest/config.html#config-podman)

[Podman](https://podman.io)



Shifter containers
-------------------

- Shifter est une alternative au moteur de conteneur à Docker. Shifter fonctionne en convertissant les images Docker dans un format commun qui peut ensuite être distribué et lancé sur des systèmes HPC. L'interface utilisateur de Shifter permet à un utilisateur de sélectionner une image dans le registre dockerhub, puis de soumettre des tâches qui s'exécutent entièrement dans le conteneur.

Nextflow fournit une prise en charge intégrée de Shifter. Cela permet de contrôler l'environnement d'exécution des processus d'un pipeline en les exécutant dans des conteneurs isolés avec leurs dépendances.

De plus, le support fourni par Nextflow pour différentes technologies de conteneurs permet au même pipeline d'être exécuté de manière transparente avec les conteneurs Docker, Singularity ou Shifter, en fonction du moteur disponible dans les plates-formes d'exécution cibles.

- Prerequis

Vous devez installer Shifter et Shifter image gateway dans votre environnement d'exécution, c'est-à-dire : ordinateur personnel (BIWS) ou le nœud d'entrée d'un cluster distribué. Dans le cas du cluster distribué, Shifter doit être installé sur tous les nœuds de calcul et la commande shifterimg doit également être disponible et Shifter correctement configuré pour accéder à la passerelle Image, pour plus d'informations, consultez la documentation officielle.

Pour le moment podman est installé sur ma BIWS : u900-bdd-1-112n-6990.curie.fr

- Images

Shifter convertit une image docker en couches squashfs qui sont distribuées et lancées dans le cluster. Pour plus d'informations sur la création d'images Shifter, consultez la documentation officielle.

- Comment ça fonctionne via nextflow

L'intégration pour Shifter, à l'heure actuelle, nécessite de configurer les paramètres suivants dans le fichier de configuration :

```
process.container = "dockerhub_user/image_name:image_tag"
shifter.enabled = true
```
et shifter essaiera toujours de rechercher les images dans le registre Docker Hub.

A Noter que si la balise d'image n'est pas spécifiée, c'est la balise ‘latest’ qui est récupérée par défaut.

Il est possible de spécifier une image Shifter différente pour chaque définition de processus dans le script de pipeline.

- Comment ça fonctionne via la ligne de commande

Récupérer une image docker et la convertir en image shifter :

```
shifterimg pull ubuntu:14.04
```

ou

```
shifterimg pull scanon/shanetest:latest
```

Cas d'erreur prévisible car shifter veut utiliser le système image gateway qui doit être disponible dans l'environnement d'exécution.

```
shifterimg pull ubuntu:14.04 
ERROR: failed to contact the image gateway.
```

- Pour plus de détails consulter : 

[Nextflow Shifter containers](https://www.nextflow.io/docs/latest/shifter.html)

[scope Process](https://www.nextflow.io/docs/latest/config.html#config-process)

[Shifter](https://github.com/NERSC/shifter/tree/master/doc)


Charliecloud containers
-------------------

- Charliecloud est un moteur de conteneur alternatif à Docker, mieux adapté à une utilisation dans les environnements HPC. Son principal avantage est qu'il peut être utilisé avec des autorisations non privilégiées, en utilisant les espaces de noms d'utilisateurs dans le noyau Linux. Charliecloud est capable d'extraire des registres Docker.

En utilisant cette fonctionnalité, tout processus d'un script Nextflow peut être exécuté de manière transparente dans un conteneur Charliecloud. Cela peut être extrêmement utile pour empaqueter les dépendances binaires d'un script dans un format standard et portable qui peut être exécuté sur n'importe quelle plate-forme prenant en charge le moteur Charliecloud.

- Prerequis

Vous devez installer Charliecloud version 0.22 ou ultérieure dans votre environnement d'exécution, c'est-à-dire : ordinateur personnel (BIWS) ou le nœud d'entrée d'un cluster distribué. Pour plus d'informations, consultez la documentation officielle.  Espaces de noms d'utilisateurs dans le noyau Linux.

- Comment ça fonctionne via nextflow

Il est inutile de modifier le script Nextflow pour exécuter avec Charliecloud. Il faut Spécifier simplement l'image docker à partir de laquelle les conteneurs sont démarrés à l'aide de l'option de ligne de commande `-with-charliecloud`. Par example: 

`nextflow run <your script> -with-charliecloud [container]`

Ajouter la varible d'env : NXF_CHARLIECLOUD_CACHEDIR au lancement du pipeline.
exemple :

`export NXF_CHARLIECLOUD_CACHEDIR=/data/tmp/plarosa/charlieImg`

nextflow va exécuter <votre script> -with-charliecloud [conteneur] à chaque fois que votre script lance une exécution de processus, Nextflow l'exécute dans un conteneur charliecloud créé à l'aide de l'image de conteneur spécifiée. En pratique, Nextflow encapsulera automatiquement vos processus et les exécutera en exécutant la commande **ch-run** avec le conteneur que vous avez fourni.

Il est possible de ne pas utiliser une seule image charliecloud en tant que paramètre de la ligne de commande, il faut dans ce cas les définir dans le fichier de configuration Nextflow. 

Par exemple, ajouter les lignes suivantes dans le fichier nextflow.config :

```
process {
    withName:foo {
        container = 'image_name_1'
    }
    withName:bar {
        container = 'image_name_2'
    }
}
charliecloud {
    enabled = true
}
```

Pour que cela fonctionne le conteneur doit être au format de répertoire plat Charliecloud. 
Voir la section ci-dessous pour la compatibilité avec les registres Docker.
Lisez la page Configuration pour en savoir plus sur le fichier nextflow.config et comment l'utiliser pour configurer l'exécution du pipeline : [scope charliecloud]( https://www.nextflow.io/docs/latest/config.html#scope-charliecloud)

Attention :

Nextflow gère automatiquement les montages du système de fichiers à chaque lancement d'un conteneur en fonction des fichiers d'entrée du processus. Notez, cependant, que lorsqu'une entrée de processus est un lien symbolique, le fichier lié doit être stocké dans le même dossier où se trouve le lien symbolique, ou dans l'un de ses sous-dossiers. Sinon, l'exécution du processus échouera car le conteneur lancé ne pourra pas accéder au fichier lié. 

- Charliecloud et Docker Hub

Nextflow est capable d'extraire de manière transparente les images de conteneurs distants stockées dans n'importe quel registre compatible Docker et les convertit au format compatible Charliecloud.

Par défaut, lorsqu'un nom de conteneur est spécifié, Nextflow vérifie si un conteneur portant ce nom existe dans le système de fichiers local. S'il existe, il est utilisé pour exécuter le conteneur. Si un fichier correspondant n'existe pas, Nextflow essaie automatiquement d'extraire une image avec le nom spécifié à partir du Docker Hub.

Pour extraire des images d'un registre Docker tiers, fournissez simplement l'URL de l'image. Si aucune URL n'est fournie, Docker Hub est supposé. Par exemple, cela peut être utilisé pour extraire une image de quay.io et la convertir automatiquement au format de conteneur Charliecloud :

```
process.container = 'https://quay.io/biocontainers/multiqc:1.3--py35_2'
charliecloud.enabled = true
```

Alors que sous Docker Hub par exemple pour une image rnaseq-nf

```
process.container = 'nextflow/rnaseq-nf:latest'
charliecloud.enabled = true
```
Si le cacheDir de charliecloud n'est pas défini : Nextflow va creer par défaut l'image dans le work : .`./work/charliecloud` 

Lorsque l'image existe sur le file systeme on peut faire :

```
process.container = '/data/tmp/plarosa/charlieImg/rnaseq-nf-latest'
charliecloud.enabled = true
```

ou idéalement : compatible si l'image existe sur le file systeme ou non lors de premier lancement du pipeline :

```
charliecloud.cacheDir = "/data/tmp/plarosa/charlieImg"
process.container = 'nextflow/rnaseq-nf:latest'
charliecloud.enabled = true
```

Ce qui donne pour la runtime concrètement sous nextflow : 

si l'image existe dans charliecloud.cacheDir

> nxf_launch() {
>     ch-run --no-home -w /data/tmp/plarosa/charlieImg/nextflow-rnaseq-nf-latest -- bash -c "mkdir -p /data/tmp/plarosa/nextflowdsl2/rnaseq-nf "$PWD"";ch-run --no-home --unset-env="*" -w --set-env=/data/tmp/plarosa/charlieImg/nextflow-rnaseq-nf-latest/ch/environment --set-env=<( echo "NXF_DEBUG=${NXF_DEBUG:=0}" ) -b /data/tmp/plarosa/nextflowdsl2/rnaseq-nf:/data/tmp/plarosa/nextflowdsl2/rnaseq-nf -b "$PWD":"$PWD" -c "$PWD" /data/tmp/plarosa/charlieImg/nextflow-rnaseq-nf-latest -- /bin/bash -c "/bin/bash /data/tmp/plarosa/nextflowdsl2/rnaseq-nf/DSL1/work/04/fc2fa56faa35052c8d6460b274d77c/.command.run nxf_trace"
> }


ou si l'image n'existe pas et que charliecloud.cacheDir n'est pas définie, nextflow commence par pull l'image dans le work et l'utilise ensuite 

> nxf_launch() {
>     ch-run --no-home -w /data/tmp/plarosa/nextflowdsl2/rnaseq-nf/DSL1/work/charliecloud/nextflow-rnaseq-nf-latest -- bash -c "mkdir -p /data/tmp/plarosa/nextflowdsl2/rnaseq-nf "$PWD"";ch-run --no-home --unset-env="*" -w --set-env=/data/tmp/plarosa/nextflowdsl2/rnaseq-nf/DSL1/work/charliecloud/nextflow-rnaseq-nf-latest/ch/environment --set-env=<( echo "NXF_DEBUG=${NXF_DEBUG:=0}" ) -b /data/tmp/plarosa/nextflowdsl2/rnaseq-nf:/data/tmp/plarosa/nextflowdsl2/rnaseq-nf -b "$PWD":"$PWD" -c "$PWD" /data/tmp/plarosa/nextflowdsl2/rnaseq-nf/DSL1/work/charliecloud/nextflow-rnaseq-nf-latest -- /bin/bash -c "/bin/bash /data/tmp/plarosa/nextflowdsl2/rnaseq-nf/DSL1/work/bb/38d3f1e7613160ca403872d6170c8d/.command.run nxf_trace"
> }


Cas d'erreur prévisible quand charliecloud veut utiliser des espaces de noms d'utilisateurs du noyau qui ne sont pas compatibles avec la plupart des systèmes de fichiers parallèles

```
Command error:
  ch-run[22456]: can't init user+mount namespaces: Invalid argument (ch_core.c:358 22)

```


- Comment ça fonctionne via la ligne de commande

récupérer une image docker et la convertir en image charliecloud :

`ch-image pull ewels/multiqc:dev /data/tmp/plarosa/charlieImg/ewels-multiqc-dev`

ou récupérer une image via un repo et la convertir en image charliecloud :

`ch-image pull https://quay.io/biocontainers/multiqc:1.3--py35_2 /data/tmp/plarosa/charlieImg/quay.io-biocontainers-multiqc-1.3--py35_2`

runtime

```
ch-run /data/foo -- echo hello
  hello
```



Conclusion
------------

Nous avons choisi depuis deja plusieurs années d'utiliser le moteur de conteneurs Singularity pour nos pipelines (recherche/prod), ce qui semble représenter aujourd'hui la meilleure alternative à Docker. Les principaux avantages de Singularity sont qu'il peut être utilisé avec des autorisations non privilégiées et ne nécessite pas de processus démon séparé. Ainsi que d'autres fonctionnalités, comme par exemple la prise en charge des montages autofs, Singularity est également capable d'utiliser des images Docker existantes et de les extraire des registres Docker. Ces avantages font de Singularity un moteur de conteneur mieux adapté aux exigences des charges de travail HPC. 
De plus, le support fourni par Nextflow pour différentes technologies de conteneurs, permet au même pipeline d'être exécuté de manière transparente avec les conteneurs Docker ou Singularity, en fonction du moteur disponible dans les plates-formes d'exécution cibles.

Les tests réalisés sur les autres alternatives à Docker ont permis de renforcer ce premier choix.
En effet, les trois autres moteurs de conteneurs ne semblent pas à l'heure actuelle être de bons candidats.
Ils ne sont pas encore suffisement matures et/ou ne conviennent pas à notre infrastructure :
- podman containers - doit utiliser pour fonctionner les espaces de noms d'utilisateurs du noyau qui ne sont pas compatibles avec la plupart des systèmes de fichiers parallèles, de plus la documentation de nexflow précise, qu'il n'est pas recommandé de l'utiliser en environnement de production.

- shifter containers - shifter doit utiliser le système image gateway qui n'est pas nécessairement disponible dans l'environnement d'exécution.

- charliecloud containers - comme podman, charliecloud doit utiliser pour fonctionner les espaces de noms d'utilisateurs du noyau qui ne sont pas compatibles avec la plupart des systèmes de fichiers parallèles, de plus la documentation de nexflow précise, qu'il n'est pas recommandé de l'utiliser en environnement de production.



