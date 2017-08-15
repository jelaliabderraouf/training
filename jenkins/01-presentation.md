# Présentation de Jenkins


En 2008 Jenkins devient LA solution pour l'intégration continue en remplacement de [CruiseControl](https://fr.wikipedia.org/wiki/CruiseControl). 

Le puis fort de Jenkins en plus de ça grande communauté est qu'il existe un nombre impressionnant de __plugins__ permettant d'étendre les fonctionnalités simple de l'outil.

Il est possible de lié Jenkins  avec __Subversion__ , __Git__ , et d'autre contrôleur de révision, il est possible de l'utiliser comme planificateur de tâche comme **cron** afin de réalisé des opérations périodique. Vous pouvez aussi lancer des exécutions à la demande ou utiliser des __webhook__ pour déclencher une opération par programmation. Nous allons couvrir quelque possibilité cependant telle que mentionné avec le nombre de plugins disponible les champs de possibilités sont nombreuses.

[Site web officiel de Jenkins](https://jenkins.io/). 

Que pouvons nous exécuter avec Jenkins ? 

* Des projets basés sur Apache Ant 
* Apache Maven 
* Des scripts arbitraires en shell Unix ou batch Windows. ( En d'autre mot N'IMPORTE QUOI :D ) 

L'application est développé en Java et sous [Licence MIT](https://fr.wikipedia.org/wiki/Licence_MIT)

## Architecture de Jenkins 

Avant de débuter avec l'installation voyons l'idée général du rôle de Jenkins et ça conception . 

Chaque tâche est une job, lors de l'exécution de la job nous exécutons un **build** , même si le résultat n'est QUE l'envoie d'un courriel ou le traitement de donnée, mais qu'aucune compilation n'est réalisé :D.

Jenkins fonctionne en mode master / slave :

* **Master** : reçoit les évènements que nous parlions d'évènement généré par le serveur Subversion, git, exécution d'un build manuellement par un utilisateur. Selon les critères saisie le système choisi un **slave** pour réaliser les opérations, nous verrons que nous utilisons les labels pour assigner des jobs au slave. Le serveur **master** contient l'ensemble des définitions **centralisé** contenant la description des opérations à faire ! Si tous les slaves sont déjà en utilisation le serveur va mettre en place un file d'attente.
* **Slave** : Le serveur slave est donc un agent du master, il réalisera l'opération demandé par le serveur Jenkins , bien entendu si le serveur Jenkins essaye d'exécuter une commande sur la machine elle doit être installé. Il est important donc d'avoir un __template__ , modèle pour que l'ensemble des serveurs slave est la même configuration. Comme Jenkins est développé en Java il est possible d'avoir des agents sur plusieurs type de système d'exploitation 

Voici une représentation graphique du fonctionnalité simple de Jenkins :

![](./imgs/jenkins-architecture.png)

Dans la pratique le serveur Jenkins peut être aussi être utilisé comme slave, cependant ceci est limitatif autant en terme de puissance que des possibilités d'avoir plusieurs architecture et / ou des lieux de traitement à plusieurs endroits.

Telle que mentionné plus tôt le nombre de plugins sont impressionnant : [https://plugins.jenkins.io/](https://plugins.jenkins.io/)

## Installation de Jenkins 

Bon nous orientons surtout la formation sur la pratique donc débutons avec l'installation , honnêtement je suis super paresseux et depuis l'arrivée **docker** ça n'a pas arrangé les choses. En d'autre mot je suis un bon DevOps / sysadmin :D , la paresse est un art mais demande beaucoup de temps pour la mettre en place :D. Nous réaliserons donc l'installation avec un conteneur !!

### Installation avec Docker

Bon voilà le problème en ce moment avec **docker** et les images officiel parfois elle ne sont plus disponible :-/ , originalement il y avait [https://hub.docker.com/_/jenkins/](https://hub.docker.com/_/jenkins/) , mais si vous regardez la page il y a le message :

> DEPRECATED
> 
> This image has been deprecated in favor of the jenkins/jenkins:lts image provided and maintained by Jenkins Community as part of project's release process
> The images found here will receive no further updates after LTS 2.60.x. Please adjust your usage accordingly.

Donc nous devons nous tourner vers l'image : [https://hub.docker.com/r/jenkins/jenkins/](https://hub.docker.com/r/jenkins/jenkins/) , ceci est l'image suggérer par la [documentation officiel de jenkins](https://jenkins.io/doc/book/getting-started/installing/#docker) . Il est évident que ceci n'aide pas l'utilisateur de docker mais que voulez vous ... :-/

Pour les besoins de la démonstration je vais principalement utiliser le service de Jenkins en __standalone__ sans slave , je couvrirai plus la partie de l'ajout d'agent par la suite . 

Bien entendu nous utiliserons un fichier de **docker-compose.yml** pour partir du bon pied :D.

Donc définition du [docker-compose-v1.yml](./dockers/docker-compose-v1.yml):

```
version: '2'
services:
    jenkins:
        image: jenkins/jenkins
        container_name : 'x3-jenkins-f'
        hostname: jenkins.train.x3rus.com
        environment:
            - TZ=America/Montreal
        volumes:
            - "/srv/docker/x3-jenkins-f/jenkins-data:/var/jenkins_home"
        # ports:
            #- 8080:8080   # Web interface
            #- 50000:50000 # Build Executors
```

Bon rapidement même si maintenant on gère , mais pour les nouveaux on veut que tous le monde embarque :

* L'image : Le conteneur de référence original , ici celui fournit par la documentation de Jenkins , donc récupération depuis hub.docker.com
* container_name : Le nom lors de l'exécution sur le docker host
* hostname : Car on aime avoir un beau hostname ... :P
* environnement : Je définie que le conteneur à le fuseaux horaire de Montréal 
* Volumes : Comme nous désirons conserver les données de traitement du Jenkins en cas de crash du conteneur ou simplement une mise à jour il est important de le sortir du conteneur !!
* Ports : Ici je ne met aucune redirection de port pour l'adresse IP du docker host , car je n'utiliserai que l'IP du conteneur dans son réseau interne au système . Bien entendu si vous activé le conteneur Jenkins pour être interrogé depuis une autre machine vous devrez mettre en place cette redirection . 

C'est partie pour le démarrage du conteneur .... Avec un problème de permission : D.

```bash
$ docker-compose up
Starting x3-jenkins-f ... 
Starting x3-jenkins-f ... done
Attaching to x3-jenkins-f
x3-jenkins-f | touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
x3-jenkins-f | Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
x3-jenkins-f exited with code 1

$ ls -ld /srv/docker/x3-jenkins-f/jenkins-data/
drwxr-xr-x 2 root root 4096 Aug 10 17:28 /srv/docker/x3-jenkins-f/jenkins-data/
```

Effectivement par défaut la création du répertoire est réalisé par l'utilisateur **root** alors que l'usager qui démarre le service à le UID 1000 ainsi que son groupe . Correction et on redémarre .

```bash
$ sudo chown  1000:1000 /srv/docker/x3-jenkins-f/jenkins-data/
$ docker-compose up                                                 
Starting x3-jenkins-f ...                                                      
Starting x3-jenkins-f ... done                                                 
Attaching to x3-jenkins-f                                                      
x3-jenkins-f | Running from: /usr/share/jenkins/jenkins.war                
x3-jenkins-f | webroot: EnvVars.masterEnvVars.get("JENKINS_HOME")              
x3-jenkins-f | Aug 10, 2017 5:35:30 PM Main deleteWinstoneTempContents                                                                                        
x3-jenkins-f | WARNING: Failed to delete the temporary Winstone file /tmp/winstone/jenkins.war 
x3-jenkins-f | Aug 10, 2017 5:35:30 PM org.eclipse.jetty.util.log.Log initialized 
x3-jenkins-f | INFO: Logging initialized @624ms to org.eclipse.jetty.util.log.JavaUtilLog
x3-jenkins-f | Aug 10, 2017 5:35:30 PM winstone.Logger logInternal          
[ .... ]
x3-jenkins-f | Jenkins initial setup is required. An admin user has been created and a password generated.
x3-jenkins-f | Please use the following password to proceed to installation:   
x3-jenkins-f |                         
x3-jenkins-f | 041688506d8b4f1484ff13ff9c0367ac                                
x3-jenkins-f |                         
x3-jenkins-f | This may also be found at: /var/jenkins_home/secrets/initialAdminPassword 
[ ... ]
x3-jenkins-f | Aug 10, 2017 5:35:42 PM hudson.model.AsyncPeriodicWork$1 run    
x3-jenkins-f | INFO: Finished Download metadata. 6,278 ms                      
x3-jenkins-f | --> setting agent port for jnlp                                 
x3-jenkins-f | --> setting agent port for jnlp... done 
```

Visualisation du conteneur :

```bash
$ docker ps
CONTAINER ID    IMAGE               COMMAND                  CREATED             STATUS          PORTS                 NAMES
459a6bc1985f    jenkins/jenkins     "/bin/tini -- /usr..."   9 minutes ago       Up 2 minutes    8080/tcp, 50000/tcp   x3-jenkins-f

$ docker inspect x3-jenkins-f | grep IPAddr
        "SecondaryIPAddresses": null,
            "IPAddress": "",
            "IPAddress": "172.31.0.2",

```

Quand nous allons sur le service : http://172.31.0.2:8080 , nous aurons ceci .

![](./imgs/01-initialisation-jenkins.png) 

Nous devrons dons saisir la clé qui fut affiché lors du démarrage , dans notre cas : 041688506d8b4f1484ff13ff9c0367ac 

Nous installerons les plugins suggérer quand on débutte ces toujours une bonne idée :P 

![](./imgs/02-initialisation-jenkins-select-plugins.png)

* Création du compte admin 

![](./imgs/03-initialisation-jenkins-creation-compte-admin.png)

* Prêt à jouer :D 

![](./imgs/04-initialisation-jenkins-pret-a-usage.png)

Cette installation est simpliste mais fonctionnel , avant de voir comment nous allons pouvoir améliorer notre déploiement pour que ce soit plus complet lors du démarrage prenons le temps de comprendre ce que nous allons vouloir d'inclus dans le déploiement. Nous couvrirons donc cette partie dans la section [Amélioration de l'installation](#Amélioration de l'installation).


## Tour d'horizon de Jenkins 


Si nous reprenons la page d'accueil du logiciel :

![](./imgs/05-home-jenkins.png)

* **New Item** : Parfois on appel ça job , parfois item , j'ai voulu le mettre sous le projecteur
* **Manage Jenkins** : Même raison , quand on vient de déployer une application et que l'on veut voir les options de configuration disponible , le voir tous de suite c'est cool :D.
* **Agent / slave** :  Telle que mentionné pour le moment nous utiliserons le serveur Jenkins comme **orchestrateur** et serveur d'exécution , mais lorsque nous allons avoir d'autre slave nous aurons une plus grande liste !

Nous allons faire une petite job très simple pour voir le comportement et les options , avant de voir les paramètres disponible , je crois qu'une présentation de ce que peux faire Jenkins est idéal . Plusieurs d'entre vous connaisse déjà le produit ce sera une révision et pour les autres ça permettra de mettre tous le monde à peu près sur le même pied.

### Création d'un Tâche simple

* Cliquez sur **new item** .

![](./imgs/06-creation-job-selection-type.png)

Vous devez choisir le type de projet que vous désirez , si vous avez déjà un serveur Jenkins dans votre organisation et que la liste et plus ou moins longue sachez que la mise en place de plugins change le comportement de Jenkins donc ne vous étonnez pas. Le premier exemple qui me vient à l'esprit est que vous aurez peut-être l'option de faire la création d'un projet **Maven** qui n'est pas lister ici. 

* Vous devez attribuer un nom à la job , ici : **demo-tache-simple**

Dans notre cas nous allons faire quelques chose des très simples , nous allons sélectionner **Freestyle project**, ceci nous laissera les mains libre pour faire **n'importe quoi** quelque chose de bien ou de mal :P. 

Une fois cliquez sur **OK** vous aurez l'ensemble des possibilités de configuration , si vous regardez la configuration est structuré par section :

![](./imgs/07-1-setup-job-tabs.png)

**RAPPEL** : il est probable que vous ayez plus ou moins de champs du à la mise en place de plugins ! (C'est le dernier avertissement :P)

Nous avons donc plusieurs section :

* **General** : Regroupant les informations global de la tâches , description et autre
* **Source code Management** : Il est possible , mais non obligatoire que votre job récupère depuis un dépôt Subversion, git , etc des scripts ou des données.
* **Build Triggers** : Ceci permet d'indiquer qu'est-ce qui déclenchera l'exécution de la tâche , nous y reviendrons 
* **Build Environments** : Spécification de l'environnement de build , nous y retrouverons des variables et autre permettant de définir le contexte du build.* **Build** : La tâche proprement dite , donc les commandes et instruction de la job
* **Post-build Action** : Définition de la tâche quand la job est terminé , nous verrons que nous allons pouvoir par exemple indiquer au système que si le build à bien fonctionné de poussé le résultat dans une voûte ou un docker registry ... 

Ce sont les grosses étapes d'une tâche Jenkins , regardons les en détail en réalisant un job. Je vous préviens ne je vais pas couvrir chaque petit option mais celle que je juge importante . De plus je pense que ce sera plus pertinent de réaliser une job et de la modifier en cours de route plutôt que de descendre la page et expliquer le fonctionnement. 

#### Au plus simple 

Nous allons laisser l'ensemble des options par défaut et modifier les paramètres et option au fur et à mesure de la découverte. 

Donc allons directement à l'étape de build , vous pouvez cliquer sur l'onglet en haut **build**.
Nous allons ajouter une étape , je vais sélectionner **Execute shell**, pour garder ça simple mais honnêtement c'est principalement ce que j'utilise.

![](./imgs/07-2-setup-job-add-build.png)

Et je vais mettre une petite job simple qui affiche l'heure et le type de CPU de la machine de build, si vous vous dites c'est null oui je sais mais je manque d'imagination . :P

Donc voici la commande :

```bash
$ cat /proc/cpuinfo  | grep 'model name'  | uniq
```

Et le résultat dans Jenkins :

![](./imgs/07-3-setup-job-build-cmd.png)

On peut pas dire que c'est bien différent d'un script bash classique , nous avons l'interpréteur de commande au début et la / les commandes. Nous sauvegardons puis on se réjouit du résultat !

Il suffit de cliquer sur **Build NOW** .

![](./imgs/07-4-setup-job-run-build.png)

Nous cliquez sur le bouton et maintenant il y a dans la boite **build history** , le build __#1__, il faut sélectionner le build et cliquer sur console pour voir le résultat :

![](./imgs/07-5-setup-job-view-build-result.png)

Et voilà le résultat 

![](./imgs/07-5-setup-job-view-build-result-consol.png) 

Nous voyons donc le résultat , il pourrait être plus beau mais bon ... c'est exactement comme dans la console :D. Pas de panique la complexité arrive cependant le point important de cette partie est que n'importe quelle script devrais fonctionner si l'ensemble des commandes sont présente sur la machine où le build sera réalisé. 

En d'autre mot , si vous utilisez python3 et que ce dernier n'est pas présent sur la machine de build ça ne passera pas ... Je sais c'est étrange :P.


#### Du simple un peu dynamique

Bon c'est cool, mais bon vos scripts ne sont pas si statique , il y a toujours des paramètres à fournir puis personne ne veut écrire les valeurs en dure dans le script qui est fournit . Introduisons le concept de variables ... Nous retourne dans la section configuration ...

Dans la section **General** vous avez l'option **This project is parameterized** , vous allons donc pouvoir fournir des paramètres.

Nous allons pouvoir ajouter plusieurs type de paramètres :

![](./imgs/07-6-setup-job-type-of-parameter.png)

Dans mon cas je ne vais définir 2 type :

* **string** :  Nous allons écrire le résultat de l'opération dans un fichier qui sera passé en paramètre
* **boolean** : Juste pour en mettre un deuxième :P , qui indiquera si oui ou non on crée un fichier ou juste l'afficher à l'écran :D

Voici le résultat de la définition :

![](./imgs/07-7-setup-job-define-parameter.png)

Et nous allons modifier le code pour les utiliser 


Voici le code :

```bash
 #!/bin/bash
 #
 # Description : un exemple de script
 #########################################

echo " debut script : "

echo "Valeur de B_MUST_WRITE : $B_MUST_WRITE "
echo "Valeur de FILE_NAME_2_WRITE : $FILE_NAME_2_WRITE "

if [ "$B_MUST_WRITE" == "false" ] ; then

    cat /proc/cpuinfo  | grep 'model name'  | uniq
else
    cat /proc/cpuinfo  | grep 'model name'  | uniq > $FILE_NAME_2_WRITE 
fi
``` 

![](./imgs/07-8-setup-job-code-with-parameter.png)

Le résultat lors de l'exécution :

![](./imgs/07-9-setup-job-build-with-parameter.png)

Comme vous pouvez le voir l'ensemble des arguments / paramètres fournit sont des variables d'environnement au script ce qui permet de faire la manipulation simplement. Jenkins offre plusieurs variables d'environnement , si vous regardez en bas de la boite pour saisir votre code vous avez le lien : __See the list of available environment variables__ en cliquant dessus vous aurez la liste . Nous y reviendrons avec un cas concret ...

#### Gestion du paramètre et visualisation de l'erreur

Pour terminer ce cas simple nous allons ajouter une validation pour le nom du fichier , je ne veux pas que les utilisateurs puisse définir un full __PATH__ avec des __/__ , sinon il risquerait de pouvoir définir un chemin d'un fichier système. Bon honnêtement j'exagère sur le niveau de risque, car le service Jenkins n'est pas exécuté comme **root**, mais bon pas toujours facile de trouver des exemples :P , a vous d'extrapoler pour votre usage.

Voici le résultat du nouveau code :

```bash
 #!/bin/bash
 #
 # Description : un exemple de script
 #########################################

echo " debut script : "

echo "Valeur de B_MUST_WRITE : $B_MUST_WRITE "
echo "Valeur de FILE_NAME_2_WRITE : $FILE_NAME_2_WRITE "

if [ "$B_MUST_WRITE" == "false" ] ; then

	cat /proc/cpuinfo  | grep 'model name'  | uniq
else
	
   	if echo $FILE_NAME_2_WRITE | grep -q '/' ; then
    	echo "Le nom du fichier ne peut pas conteneur de / "
        echo "c'est pour ne pas que vous ecriviez partout :P !!! "
    	exit 1
    else 
		cat /proc/cpuinfo  | grep 'model name'  | uniq > $FILE_NAME_2_WRITE 
    fi
fi
```

Rien de très compliqué si vous réalisez déjà un peu de __scripting__ , Si nous utilisons la job pour afficher à l'écran donc sans coché l'option d'écrire dans un fichier :

* Cas 1 : affichage à l'écran , pas de paramètre :
    ![](./imgs/08-1-exemple-cas-parameter-B.png)
    ![](./imgs/08-1-exemple-cas-parameter-A.png)
* Cas 2 : Écriture dans un fichier , pas d'erreur :
    ![](./imgs/08-2-exemple-cas-parameter-B.png)
    ![](./imgs/08-2-exemple-cas-parameter-A.png)
* Cas 3 : Écriture dans un fichier , mais mauvais non de fichier :
    ![](./imgs/08-3-exemple-cas-parameter-B.png)
    ![](./imgs/08-3-exemple-cas-parameter-A.png)

De plus lors de l'affichage général de la tâche vous aurez le statu de l'ensemble des jobs.

![](./imgs/08-4-job-status.png) 

Vous savez comme moi que peu importe la qualité de votre script , même s'il est le plus merveilleux du monde et qu'il fait tous SUPER bien , quand vous dites à certaine personne d'ouvrir une console et de démarrer le script sur la ligne de commande ça bloque !!! 
L'inquiétude , la peur , le stress de pas saisir la bonne chose , l'angoisse de ne pas savoir si ça a bien fonctionner , etc je présume que la liste peu être longue. 

Avec Jenkins ceci vous offre la possibilité de réaliser une "interface" simple pour l'utilisateur pour démarrer l'exécution de la tâche !

Comme vous pouvez le voir Jenkins utilise le code de retour du script pour savoir si l'exécution c'est passé adéquatement :

* 0 == Pas de problème 
* n == n'importe quelle autre valeur problème !!

Pour la visualisation du problème vous pouvez en plus configurer la tâche pour qu'elle transmettre un courriel au personne afin de les aviser du problème.
Ceci est SUPER pour les tâches automatisé que l'on regarde jamais :P .

Si nous retournons à la job vous pouvez définir un **Post-build Actions** :

1. **Add post-build action**
2. Sélectionnez **E-mail Notification**

![](./imgs/08-5-job-email-notification.png)

#### L'espace de travail ou workspace 

Dans un des exemples j'ai réalisé l'écriture dans un fichier , dans la configuration du paramètre j'ai bloquer la possibilité de définir un chemin contenant un **/** donc le répertoire sera dans le répertoire courent. Bien entendu la question est il est où ? 

L'ensemble des tâches sous Jenkins sont exécuté dans un espace de travail , en anglais __workspace__ . Vous pouvez le consulter et le télécharger depuis l'interface de Jenkins.

![](./imgs/09-workspace.png)

Comme vous pouvez le voir vous pouvez aussi supprimer le contenu de l'espace du travail en un clique. Ceci est parfois requis , car par défaut il n'y a pas de suppression de l'espace de travail , ce dernier est conserver et les données s'ajoute, lors d'un processus de compilation ceci peut faire gagner du temps mais aussi causer des problèmes. 

Comme vous pouvez le voir sur la copie d'écran j'avais fait une erreur avec un nom de fichier contenant de **-** , le fichier __-NAME-2-WRITE__. 

Mais il est où ? :P 

Ce répertoire de travail peut être configurable que ce soit sur le serveur __master__ ou __slave__ , regardons par défaut sur le conteneur .

```bash
$ # répertoire contenant l'ensemble de la configuration de Jenkins
$ docker exec  x3-jenkins-f ls /var/jenkins_home
config.xml
copy_reference_file.log
hudson.model.UpdateCenter.xml
hudson.plugins.emailext.ExtendedEmailPublisher.xml
hudson.plugins.git.GitTool.xml
identity.key.enc
init.groovy.d
jenkins.CLI.xml
jenkins.install.InstallUtil.lastExecVersion
jenkins.install.UpgradeWizard.state
jobs
logs
nodeMonitors.xml
nodes
plugins
queue.xml.bak
secret.key
secret.key.not-so-secret
secrets
updates
userContent
users
war
workflow-libs
workspace

$ # répertoire de l'espace de travail
$ docker exec  x3-jenkins-f ls -l /var/jenkins_home/workspace         
total 4                                
drwxr-xr-x 2 jenkins jenkins 4096 Aug 11 17:40 demo-tache-simple 

$ # et voilà 
$ docker exec  x3-jenkins-f ls -l /var/jenkins_home/workspace/demo-tache-simple                                                                      
total 8                                
-rw-r--r-- 1 jenkins jenkins 54 Aug 11 17:39 -NAME-2-WRITE                     
-rw-r--r-- 1 jenkins jenkins 54 Aug 14 08:21 toto   
```

Comme nous avons exporté le répertoire __/var/jenkins\_home__ du conteneur normalement ce répertoire est aussi disponible directement sur votre file système sans passer par le conteneur ! Et j'espère que vous l'avez fait sinon à la prochaine mise à jour vous perdrez vos configurations :D.

### Quelque paramètre intéressant 

Maintenant que nous avons une petite compréhension simpliste de ce que fait une job, prenons le temps de regarder les autres options disponible de la tache.

* **Section général**
    * **Discard old builds** :
        Comme vous avez pu le voir le système conserve les informations sur les builds antérieur , le problème est que ça prendra de la place puis surtout il est possible que vous n'en avez rien à faire de se qui fut réalisé il y a + de 7 jours. De même après 10 build peut importent le contenu des vieux , il est donc possible de définir une rotation / suppression. **ATTENTION** ceci est traité lors de l'exécution d'une build , en d'autre mot si vous avez 20 build en historie et que vous activé cette configuration il ne va pas faire le traitement de nettoyage il faudra attendre la prochaine exécution.
        ![](./imgs/10-1-parametre-discard-old.png)

* **Source Code Management**
    * C'est le prochain sujet, donc pas ça arrive .

* **Build Triggers**
    * **Trigger builds remotely (e.g., from scripts)** : 
        Nous prendrons un peu de temps pour voir cette possibilité plus tard avec un exemple concret , cependant il est possible d'appeler la page Jenkins avec un TOKEN secret pour faire l'appel de la tâche très intéressant lors de la mis en place d'intégration tous en conservant une visibilité
    * **Build periodically** : 
        Permet de définir sous la forme de la syntaxe de **cron** une période d'exécution, une chose que je trouve intéressant avec cette méthode est que nous avons la possibilité d'avoir la tâche planifier et nous permettons à l'utilisateur de l'exécuter manuel quand il veut. De plus comme il voit si c'est en cours il ne pourra pas l'exécuter de manière simultané :D.

* **Build Environment**
    * **Delete workspace before build starts**:
        Il est possible d'indiquer au processus de build de supprimer l'espace de travail avant de débutter le processus, il est même possible en cliquant sur advance de définir un pattern . Donc vous ne supprimer pas tous, mais uniquement un type de fichier, il est aussi possible d'utiliser une variables d'environnement pour l'avoir configurable.
        ![](./imgs/10-2-delete-workspace-advance.png)
    * **Abort the build if it's stuck**:
        Définition de ce qui doit ce produire si la tâche ne progresse plus et oui parfois notre code contient des erreurs que faire alors ? Attendre indéfiniment ou l'arrêter ?!?! Tous comme pour l'avantage de la configuration du cron dans Jenkins ceci à l'avantage de ne pas être requis dans votre script . Vous laissez cette tâche ingrate à Jenkins :D.
        ![](./imgs/10-3-build-stuck-so.png)
    * **Add timestamps to the Console Output**:
        Très bien ça surtout pour les tâches longue , car elle permet de voir l'évolution du script dans le temps directement dans le log de la console.

## Tour d'horizon de la configuration de Jenkins 

On peut pas dire que l'on maitrise Jenkins a ce stade cependant, nous avons une légère petite meilleur compréhension de ce que l'on peut faire , j'étais dans un problème d'œuf ou la poule. Montrer la configuration sans savoir les conséquences ou mettre en place une tâche sans la configuration, comme vous pouvez le constater j'ai fait un choix :D. 

Revenons donc sur les configurations possible, encore une fois, j'ai une configuration minimal, très peu de plugins . Il est donc possible d'augmenter le nombre de fonctionnalité, nous en verrons quelques une par la suite.

Pour accéder à la configuration dans le menu de gauche choisir **Manage Jenkins**, je ne couvrirai pas l'ensemble des options nous les explorerons au fur et à mesure.

### Configure System : Configuration général de Jenkins

Nous avons dans cette section les configurations général de Jenkins ( PATH, env globale , ... )

* **Répertoire de Jenkins** : 
    Vous pouvez re définir les répertoires pour Jenkins , le lieu où seront stocké les **workspace** , j'en fait mention surtout si vous avez acheter des disque spéciaux pour avoir une plus grande performance d'accès disque. Il est possible que vous vouliez les utilisés pour les lieux de travail.

    ![](./imgs/11-1-global-config-path-jenkins.png)

 

# TODO truc

* plugin :
    jobConfigHistory:latest                
    nested-view:latest                     
    global-build-stats:latest              
    gitlab-plugin:latest  