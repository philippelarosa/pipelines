**Project description**
* Type  : Veille techno, Proof of concept :  essayer d'appréhender les fonctionnalités liés au « Running Services »du moteurs de containers Singularity,
* Bioinformaticians contact: Philippe La Rosa (philippe.larosa@curie.fr)

**Singularity : les services**

Il existe différentes manières d'exécuter des conteneurs Singularity. On peut utiliser des commandes telles que run, exec et shell pour interagir avec les processus du conteneur, dans ces cas les conteneurs Singularity sont exécuté au premier plan; il s’agit des modes les plus couramment utilisés. Singularity, permet en outre d'exécuter des conteneurs en mode « détaché » ou « démon » qui peut exécuter différents services en arrière-plan. Un « service » est essentiellement un processus exécuté en arrière-plan que plusieurs clients différents peuvent utiliser. Par exemple, un serveur Web ou une base de données. Pour exécuter des services dans un conteneur Singularity, il faut utiliser des instances. Une instance de conteneur est une version persistante et isolée de l'image de conteneur qui s'exécute en arrière-plan.

**Aperçu**

Singularity v2.4 a introduit le concept d'instances permettant aux utilisateurs d'exécuter des services dans Singularity. Cette page vous aidera à comprendre les instances à l'aide d'un exemple élémentaire suivi d'un exemple plus utile exécutant un serveur Web NGINX à l'aide d'instances. En fin de compte, vous trouverez un exemple plus détaillé d'exécution d'une instance d'API qui convertit les URL en PDF.

Pour commencer, supposons que vous souhaitiez exécuter un serveur Web NGINX en dehors d'un conteneur. Sur Ubuntu, vous pouvez simplement installer NGINX et démarrer le service en :

```
$ sudo apt-get update && sudo apt-get install -y nginx
$ sudo service nginx start
```

Si vous deviez faire quelque chose comme ça à partir d'un conteneur, vous verriez également le démarrage du service et le serveur Web en cours d'exécution. Mais si vous deviez quitter le conteneur, le processus continuerait à s'exécuter dans un espace de noms de montage inaccessible. Le processus serait toujours en cours d'exécution, mais vous ne pourriez pas facilement le tuer ou l'interfacer. C'est ce qu'on appelle un processus orphelin. Les instances de singularité vous permettent de gérer correctement les services.

**Instances de conteneur dans singularity**

Pour la démonstration, utilisons un exemple simple (bien que quelque peu inutile) d'image alpine_latest.sif de la bibliothèque de conteneurs :

`$ singularity pull library://alpine`

La commande ci-dessus enregistrera l'image alpine de la bibliothèque de conteneurs sous le nom alpine_latest.sif.

Pour démarrer une instance, vous devez suivre cette procédure :

```
[command]                      [image]              [name of instance]
$ singularity instance start   alpine_latest.sif     instance1
```

Cette commande amène Singularity à créer un environnement isolé dans lequel les services de conteneur vivent. On peut confirmer qu'une instance est en cours d'exécution en utilisant la commande instance list comme suit :

```
$ singularity instance list

INSTANCE NAME    PID      IMAGE

instance1        12715    /home/ysub/alpine_latest.sif
```

**A noter**

Les instances sont liées à votre user. Assurez-vous donc d'exécuter toutes les commandes d'instance avec ou sans le privilège sudo. Si vous démarrez une instance avec sudo et que vous devez ensuite la répertorier avec sudo, sinon vous ne pourrez pas localiser l'instance.

Si vous souhaitez exécuter plusieurs instances à partir de la même image, c'est aussi simple que d'exécuter la commande plusieurs fois avec des noms d'instance différents. Le nom de l'instance identifie de manière unique les instances, de sorte qu'elles ne peuvent pas être répétées.

```
$ singularity instance start alpine_latest.sif instance2
$ singularity instance start alpine_latest.sif instance3
```

Et encore une fois pour confirmer que les instances fonctionnent comme prévu :

```
$ singularity instance list

INSTANCE NAME    PID      IMAGE
instance1        12715    /home/ysub/alpine_latest.sif
instance2        12795    /home/ysub/alpine_latest.sif
instance3        12837    /home/ysub/alpine_latest.sif
```


Vous pouvez utiliser les commandes run/exec de singularité sur les instances :

```
$ singularity run instance://instance1
$ singularity exec instance://instance2 cat /etc/os-release
```

Lorsque vous utilisez run avec un URI d'instance, le runscript sera exécuté à l'intérieur de l'instance. De même avec exec, il exécutera la commande donnée dans l'instance.

Si vous voulez fouiller à l'intérieur de votre instance, vous pouvez exécuter une commande shell de singularité normale, mais donnez-lui l'URI de l'instance :

```
$ singularity shell instance://instance3
Singularity>
```

Lorsque vous avez terminé avec votre instance, vous pouvez la nettoyer avec la commande `instance stop` comme suit :

`$ singularity instance stop instance1`

Si plusieurs instances sont en cours d'exécution et que vous souhaitez toutes les arrêter, vous pouvez le faire avec un caractère générique ou l'indicateur –all. Les trois commandes suivantes sont toutes identiques :

```
$ singularity instance stop \*

$ singularity instance stop --all

$ singularity instance stop --a
```

**A noter**
échapper le caractère générique avec une barre oblique inverse comme celle-ci \* pour le transmettre correctement.

**Nginx “Hello-world” in Singularity**

L'exemple ci-dessus, bien que pas très utile, devrait servir d'introduction juste au concept d'instances de Singularity et d'exécution de services en arrière-plan. Ce qui suit illustre un exemple plus utile de configuration d'un exemple de serveur Web NGINX à l'aide d'instances. Nous allons d'abord créer un fichier de définition de base (appelons-le nginx.def) :

```
Bootstrap: docker
From: nginx
Includecmd: no

%startscript
   nginx
```

Cela télécharge le conteneur officiel NGINX Docker, le convertit en une image Singularity et lui dit d'exécuter NGINX lorsque vous démarrez l'instance. Puisque nous exécutons un serveur Web, nous allons exécuter les commandes suivantes en tant que root.

```
$ sudo singularity build nginx.sif nginx.def
$ sudo singularity instance start --writable-tmpfs nginx.sif web
```

**A noter**
La commande de démarrage ci-dessus nécessite sudo car nous exécutons un serveur Web. De plus, pour permettre à l'instance d'écrire des fichiers temporaires pendant l'exécution, vous devez utiliser --writable-tmpfs lors du démarrage de l'instance.

Juste comme ça, nous avons téléchargé, construit et exécuté une image NGINX Singularity. Et pour confirmer qu'il fonctionne correctement :

```
$ curl localhost

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
 body {
     width: 35em;
     margin: 0 auto;
     font-family: Tahoma, Verdana, Arial, sans-serif;
 }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Visitez localhost sur votre navigateur, vous devriez voir un message de bienvenue !

**Pour aller plus loin**

Dans cette section, nous allons montrer un exemple d'empaquetage d'un service dans un conteneur et de son exécution. Le service que nous allons emballer est un serveur API qui convertit une page Web en PDF, et peut être trouvé ici. Vous pouvez créer l'image en suivant les étapes décrites ci-dessous ou vous pouvez simplement télécharger l'image finale directement à partir de la bibliothèque de conteneurs, exécutez simplement :

`$ singularity pull url-to-pdf.sif library://sylabs/doc-examples/url-to-pdf:latest`

**Construire l'image**

Cette section décrira les conditions requises pour créer le fichier de définition (url-to-pdf.def) qui sera utilisé pour créer l'image du conteneur. url-to-pdf-api est basé sur un serveur Node 8 qui utilise une version sans tête de Chromium appelée Puppeteer. Commençons par choisir une base à partir de laquelle construire notre conteneur, dans ce cas le docker image node:8 qui est pré-installé avec Node 8 a été utilisé :

```
Bootstrap: docker
From: node:8
Includecmd: no
```


Puppeteer nécessite également l'installation manuelle d'un grand nombre de dépendances en plus du nœud 8, nous pouvons donc les ajouter dans la section %post ainsi que le script d'installation pour l'url-to-pdf :

```
%post

    apt-get update && apt-get install -yq gconf-service libasound2 \
        libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 \
        libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 \
        libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 \
        libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 \
        libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 \
        libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates \
        fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils \
        wget curl && rm -r /var/lib/apt/lists/*
    git clone https://github.com/alvarcarto/url-to-pdf-api.git pdf_server
    cd pdf_server
    npm install
    chmod -R 0755 .
```

Et maintenant, nous devons définir ce qui se passe lorsque nous démarrons une instance du conteneur. Dans cette situation, nous ajoutons les commandes qui démarrent le service url-to-pdf :

```
%startscript
    cd /pdf_server
    # Use nohup and /dev/null to completely detach server process from terminal
    nohup npm start > /dev/null 2>&1 < /dev/null &
```


De plus, le service url-to-pdf nécessite la définition de certaines variables d'environnement, ce que nous pouvons faire dans la section environnement :

```
%environment
    NODE_ENV=development
    PORT=9000
    ALLOW_HTTP=true
    URL=localhost
    export NODE_ENV PORT ALLOW_HTTP URL
```

**Le fichier de définition complet  :**

```
Bootstrap: docker
From: node:8
Includecmd: no

%post

    apt-get update && apt-get install -yq gconf-service libasound2 \
        libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 \
        libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 \
        libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 \
        libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 \
        libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 \
        libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates \
        fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils \
        wget curl && rm -r /var/lib/apt/lists/*
    git clone https://github.com/alvarcarto/url-to-pdf-api.git pdf_server
    cd pdf_server
    npm install
    chmod -R 0755 .

%startscript
    cd /pdf_server
    # Use nohup and /dev/null to completely detach server process from terminal
    nohup npm start > /dev/null 2>&1 < /dev/null &

%environment
    NODE_ENV=development
    PORT=9000
    ALLOW_HTTP=true
    URL=localhost
    export NODE_ENV PORT ALLOW_HTTP URL
```

**Le conteneur peut être construit comme ceci :**

`$ sudo singularity build url-to-pdf.sif url-to-pdf.def`

- [ ] 

**Démarrage d’une instance et exécution du service :**

`$ sudo singularity instance start url-to-pdf.sif pdf`

A noter
S'il se produit une erreur liée au refus de la connexion au port lors du démarrage de l'instance ou lors de son utilisation ultérieure, vous pouvez essayer de spécifier différents numéros de port dans la section %environment du fichier de définition ci-dessus.

Pour confirmer que cela fonctionne en envoyant au serveur une requête http en utilisant curl :

```
$ curl -o sylabs.pdf localhost:9000/api/render?url=http://sylabs.io/docs

% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                         Dload  Upload   Total   Spent    Left  Speed

100 73750  100 73750    0     0  14583      0  0:00:05  0:00:05 --:--:-- 19130
```


Un fichier PDF devrait être généré comme illustré ci-dessous :

....


Pour vérifier si l'instance fonctionne bien, il est possible de voir les processus en cours :

```
$ sudo singularity shell instance://pdf

Singularity: Invoking an interactive shell within container...

Singularity final.sif:/home/ysub> ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       461  0.0  0.0  18204  3188 pts/1    S    17:58   0:00 /bin/bash --norc
root       468  0.0  0.0  36640  2880 pts/1    R+   17:59   0:00  \_ ps auxf
root         1  0.0  0.1 565392 12144 ?        Sl   15:10   0:00 sinit
root        16  0.0  0.4 1113904 39492 ?       Sl   15:10   0:00 npm
root        26  0.0  0.0   4296   752 ?        S    15:10   0:00  \_ sh -c nodemon --watch ./src -e js src/index.js
root        27  0.0  0.5 1179476 40312 ?       Sl   15:10   0:00      \_ node /pdf_server/node_modules/.bin/nodemon --watch ./src -e js src/index.js
root        39  0.0  0.7 936444 61220 ?        Sl   15:10   0:02          \_ /usr/local/bin/node src/index.js

Singularity final.sif:/home/ysub> exit
```

**Enfin, pour aller encore plus loin :**

Maintenant que nous avons la confirmation que le serveur fonctionne, rendons-le un peu plus propre. Il est difficile de se souvenir de la commande curl exacte et de la syntaxe de l'URL chaque fois que vous souhaitez demander un PDF, alors automatisons-le. Pour ce faire, nous pouvons utiliser des applications SCIF (Standard Container Integration Format), qui sont intégrées directement dans singularity. Si vous ne l'avez pas déjà fait, consultez la documentation du système de fichiers scientifique pour vous familiariser avec.

Tout d'abord, nous allons déplacer l'installation de l'url-to-pdf dans une application, afin qu'il y ait un endroit désigné pour placer les fichiers de sortie. Pour ce faire, nous souhaitons ajouter une section à notre fichier de définition pour construire le serveur :

```
%appinstall pdf_server
    git clone https://github.com/alvarcarto/url-to-pdf-api.git pdf_server
    cd pdf_server
    npm install
    chmod -R 0755 .
```

Et mettre à jour notre script de démarrage pour pointer vers l'emplacement de l'application :

```
%startscript
    cd /scif/apps/pdf_server/scif/pdf_server
    # Use nohup and /dev/null to completely detach server process from terminal
    nohup npm start > /dev/null 2>&1 < /dev/null &
```

Définir l'application pdf_client, que nous exécuterons pour envoyer les requêtes au serveur :

```
%apprun pdf_client
    if [ -z "${1:-}" ]; then
        echo "Usage: singularity run --app pdf <instance://name> <URL> [output file]"
        exit 1
    fi
    curl -o "${SINGULARITY_APPDATA}/output/${2:-output.pdf}" "${URL}:${PORT}/api/render?url=${1}"
```

On peut maintenant voir, l'application pdf_client vérifier que l'utilisateur fournit au moins un argument.

Le fichier def complet ressemblera à ceci :

```
Bootstrap: docker
From: node:8
Includecmd: no

%post

    apt-get update && apt-get install -yq gconf-service libasound2 \
        libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 \
        libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 \
        libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 \
        libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 \
        libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 \
        libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates \
        fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils \
        wget curl && rm -r /var/lib/apt/lists/*

%appinstall pdf_server
    git clone https://github.com/alvarcarto/url-to-pdf-api.git pdf_server
    cd pdf_server
    npm install
    chmod -R 0755 .

%startscript
    cd /scif/apps/pdf_server/scif/
    # Use nohup and /dev/null to completely detach server process from terminal
    nohup npm start > /dev/null 2>&1 < /dev/null &

%environment
    NODE_ENV=development
    PORT=9000
    ALLOW_HTTP=true
    URL=localhost
    export NODE_ENV PORT ALLOW_HTTP URL

%apprun pdf_client
    if [ -z "${1:-}" ]; then
        echo "Usage: singularity run --app pdf <instance://name> <URL> [output file]"
        exit 1
    fi
    curl -o "${SINGULARITY_APPDATA}/output/${2:-output.pdf}" "${URL}:${PORT}/api/render?url=${1}"
```

Création du conteneur comme précédemment. L'option --force écrasera l'ancien conteneur :

`$ sudo singularity build --force url-to-pdf.sif url-to-pdf.def`

Maintenant que nous avons un répertoire de sortie dans le conteneur, nous devons l'exposer à l'hôte à l'aide d'un montage de liaison. Une fois que nous avons reconstruit le conteneur, création 'un nouveau répertoire appelé /tmp/out pour les fichiers PDF générés.

`$ mkdir /tmp/out`

Après avoir construit l'image à partir du fichier de définition édité, démarrage simple de l'instance :

`$ singularity instance start --bind /tmp/out/:/output url-to-pdf.sif pdf`

Pour demander un pdf il suffit de faire :

`$ singularity run --app pdf_client instance://pdf http://sylabs.io/docs sylabs.pdf`

Pour confirmer que tout a bien fonctionné et donc que le fichier pdf est bien au bon endroit :

```
$ ls /tmp/out/
sylabs.pdf
```

Lorsque vous avez terminé, utilisez la commande instance stop pour fermer toutes les instances en cours d'exécution.

`$ singularity instance stop --all`

**A noter **
Si le service que vous souhaitez exécuter dans votre instance nécessite un montage de liaison, vous devez alors passer l'option --bind lors de l'appel du démarrage de l'instance. 

Exemple, si l'on souhaite récupérer la sortie de l'instance du conteneur Web qui est placée à /output/ à l'intérieur du conteneur, vous pouvez faire :

`$ singularity instance start --bind output/dir/outside/:/output/ nginx.sif  web`


Voila c'est fini (pour le moment ...)

Philippe La Rosa
