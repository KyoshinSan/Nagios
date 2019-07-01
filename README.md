# Nagios Centos 7
Nagios est l’outil de contrôle open source le plus largement utilisé. Il nous aide à surveiller les services et les applications exécutées sous Windows, Linux, les routeurs et d’autres périphériques réseau.

Avec l’aide de Nagios, on peut surveiller les services et attributs de base. Nous pouvons accéder à Nagios à l’aide de l’interface Web fournie avec le kit et la configuration doit être effectuée au niveau du fichier.

## Type de configuration

1. [Configuration simple](https://github.com/KyoshinSan/Nagios/blob/master/README.md#1-configuration-simple)
2. [Configuration check_by_ssh](https://github.com/KyoshinSan/Nagios/blob/master/README.md#2-configuration-check_by_ssh)
3. [Configuration NRPE (Nagios Remote Plugin Executor)](https://github.com/KyoshinSan/Nagios/blob/master/README.md#3-configuration-nrpe)

## Liste de services
Ce tutoriel explique comment contrôler les **services** et les **attributs** des serveurs Linux / UNIX, tels que:

### Attributs
- Charge CPU
- Utilisation de la RAM
- Utilisation du disque dur
- Le nombre de d'utilisateur connecté
- Les processus en cours
- etc...

### Services
- HTTP
- FTP
- SSH
- SMTP
- etc...

## Les prérequis 
Avant d'installer **Nagios**, le système doit répondre à la configuration requise pour installer Nagios. Installez donc le serveur Web (httpd), PHP, les compilateurs et les bibliothèques de développement.

Installez tous les packages en une seule commande.

```
yum -y install httpd php gcc glibc glibc-common wget perl gd gd-devel unzip zip
```

Créez un utilisateur **`nagios`** et un groupe **nagcmd** pour permettre l'exécution des commandes externes via l'interface Web. Ajoutez l'utilisateur **`nagios`** et **`apache`** à faire partie du groupe **nagcmd**.

```
useradd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios
usermod -a -G nagcmd apache
```
## Installation du Serveur Nagios

Téléchargez la dernière version de Nagios Core à l'aide du terminal.

```
cd /tmp/
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.3.tar.gz
tar -zxvf nagios-4.4.3.tar.gz
cd /tmp/nagios-4.4.3
```

Compiler et installer Nagios.

```
./configure --with-nagios-group=nagios --with-command-group=nagcmd
make all
make install
make install-init
make install-config
make install-commandmode
```
> Ajouter `--prefix=/etc/nagios` pour une installation propre (à tester)

## Installation de l'interface Web de Nagios

Installez la configuration Web de Nagios à l’aide de la commande suivante.

```
make install-webconf
```

Exécutez la commande suivante pour installer le thème **exfoliation** de Nagios.

> Thèmes qui changent l'apparence de Nagios Core.

```
make install-exfoliation
```

Créez un compte d'utilisateur (**nagiosadmin**) pour vous connecter à l'interface Web de Nagios. Rappelez-vous le mot de passe que vous avez attribué à cet utilisateur - vous en aurez besoin plus tard.

```
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

Redémarrez le serveur Web Apache pour que les nouveaux paramètres prennent effet.

```
systemctl restart httpd
systemctl enable httpd
```

# 1. Configuration simple
Des exemples de fichiers de configuration ont maintenant été installés dans le répertoire `/usr/local/nagios/etc`. Ces exemples de fichiers devraient fonctionner correctement pour commencer à utiliser Nagios. Vous devrez juste faire un changement avant de continuer.

Modifiez le fichier de configuration `/usr/local/nagios/etc/objects/contacts.cfg` avec votre éditeur favori et remplacez l’adresse e-mail associée à la définition de contact **nagiosadmin** par celle que vous souhaitez utiliser pour recevoir des alertes.
> Ne pas oublier d'installer un serveur mail
```
vi /usr/local/nagios/etc/objects/contacts.cfg
```

Modifiez le champ **`Email`** pour recevoir la notification.

```
define contact{
        contact_name                    nagiosadmin             ; Short name of user
        use                             generic-contact         ; Inherit default values from generic-contact template (defined above)
        alias                           Nagios Admin            ; Full name of user

        email                           toto@exemple.com       ; <<***** L'adresse mail à changer ******
        }
```

## Installation des Plugins Nagios

Téléchargez les plugins Nagios dans le répertoire **/tmp**.

```
cd /tmp
wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
tar -zxvf nagios-plugins-2.2.1.tar.gz
cd /tmp/nagios-plugins-2.2.1/
```

Compilez et installez les plugins Nagios.

```
./configure --with-nagios-user=nagios --with-nagios-group=nagios
make
make install
```

## Démarrer le serveur Nagios

Vérifiez le fichiers de configuration de Nagios.

```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

**Output:**

```
Nagios Core 4.4.3
Copyright (c) 2009-present Nagios Core Development Team and Community Contributors
Copyright (c) 1999-2009 Ethan Galstad
Last Modified: 2019-01-15
License: GPL

Website: https://www.nagios.org
Reading configuration data...
   Read main config file okay...
   Read object config files okay...

Running pre-flight check on configuration data...

Checking objects...
        Checked 8 services.
        Checked 1 hosts.
        Checked 1 host groups.
        Checked 0 service groups.
        Checked 1 contacts.
        Checked 1 contact groups.
        Checked 24 commands.
        Checked 5 time periods.
        Checked 0 host escalations.
        Checked 0 service escalations.
Checking for circular paths...
        Checked 1 hosts
        Checked 0 service dependencies
        Checked 0 host dependencies
        Checked 5 timeperiods
Checking global event handlers...
Checking obsessive compulsive processor commands...
Checking misc settings...

Total Warnings: 0
Total Errors:   0

Things look okay - No serious problems were detected during the pre-flight check
```

S'il n'y a pas d'erreur, lancez le service Nagios.

```
service nagios start
```

Démarrez Nagios au démarrage du système.

```
chkconfig nagios on
```

## SELinux

Voir si **SELinux** est en mode **Enforcing**.

```
getenforce
```

Mettez **SELinux** en mode **Permissive** ou **désactivez-le**.

```
setenforce 0
```

Pour rendre ce changement permanent, vous devrez modifier **/etc/selinux/config** et redémarrer le système.

## Firewall

Assurez-vous d'autoriser l'accès au serveur Web via le pare-feu.

```bash
### FirewallD ###

firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

## Accéder à l'interface Web de Nagios

Accédez maintenant à l'interface Web Nagios à l'aide de l'URL suivante. Vous serez invité à entrer le nom d'utilisateur (**nagiosadmin**) et le mot de passe que vous avez spécifiés précédemment.

[http://ip-du-serveur/nagios/](#)

![Screenshot_1](https://raw.githubusercontent.com/KyoshinSan/Nagios/master/Doc%20nagios/Screenshot_1.png)

La console Nagios ressemblera à celle ci-dessous.

![Screenshot_2](https://raw.githubusercontent.com/KyoshinSan/Nagios/master/Doc%20nagios/Screenshot_2.png)

Cliquez sur **Host** dans le volet de gauche pour obtenir une liste des systèmes surveillés par Nagios. Nous n’avons ajouté aucun hôte à Nagios, il surveille donc simplement l’hôte local lui-même.

Screenshot

Cliquez sur **Services** dans le volet de gauche pour obtenir le statut des services surveillés avec Nagios.

Screenshot

# 2. Configuration check_by_ssh

Dans cette configuration, nous allons apprendre à configurer la surveillance de nagios à l’aide de ssh. SSH étant généralement installé sur presque toutes les distributions Linux, aucun paquet supplémentaire ne doit être installé. Nous n'avons besoin que du plugin check_by_ssh, ce plugin permet à Nagios d'exécuter des plugins de surveillance et des scripts sur la machine distante de manière sécurisée, sans avoir à fournir d'informations d'authentification.

Commençons donc la configuration pour configurer la surveillance Nagios à l’aide de SSH.

## Configurer la connexion entre le serveur et le client Nagios

Maintenant, nous devons configurer un mot de passe sans connexion entre le serveur et le client Nagios en utilisant la configuration de la clé publique/privée ssh. Pour créer les clés publique/privée, connectez-vous au **serveur nagios** et changez l’utilisateur en nagios.

```
su nagios
```

Ensuite créer les clés en utilisant la commande suivante :

```
ssh-keygen
```

Appuyez sur **Entrée** pour sélectionner le nom de fichier et le mot de passe par défaut. Une fois les clés générées, vous pouvez les localiser dans le dossier ‘/home/nagios/.ssh ’. Aller à ce dossier :

```
cd /home/nagios/.ssh
```

Copiez la clé publique (appelée **id_rsa.pub**) sur la machine cliente à l’aide de la commande suivante :

```
ssh-copy-id -i ~/.ssh/id_rsa.pub nagios@client_IP
```

Ensuite, connectez-vous à la machine cliente et au dossier ‘/home/nagios/.ssh’ et assurez-vous que les autorisations pour le fichier ‘authorised_keys’ est de 700, de sorte que seul l’utilisateur 'nagios ’puisse lire, écrire et exécuter ce dossier.

```
chmod 700 /home/nagios/.ssh/authorized_keys
```

## Tester la connexion

Pour que check_by_ssh fonctionne, nous devrions pouvoir nous connecter à la machine cliente à partir du serveur nagios sans aucune authentification. Nous avons déjà apporté les modifications nécessaires sur la machine cliente et le serveur Nagios. Tout ce que nous avons à faire est de tester la connexion entre les deux.

Pour tester la connexion, ssh sur la machine cliente à partir du serveur Nagios en utilisant la commande suivante :

```
ssh nagios@client_IP
```

Nous devrions pouvoir directement nous connecter à la machine cliente, sans nom d'utilisateur ni mot de passe.

Une fois que nous avons testé avec succès la connexion entre le serveur et le client nagios, nous vérifierons ensuite la connectivité du plug-in check_by_ssh d'un serveur à l'autre. Alors déconnectez-vous du client et reconnectez-vous au serveur nagios et exécutez la commande suivante :

```
/usr/local/nagios/libexec/check_by_ssh -H client_ip -C uptime
```

Nous devrions obtenir la sortie suivante :

```
12:58:54 up 16 days, 21:29,  1 user,  load average: 0.00, 0.01, 0.05
```

Cela montre que le plugin check_by_ssh fonctionne également comme prévu.

## Configuration de check_by_ssh sur le serveur

Ce sera la dernière étape pour configurer la surveillance Nagios à l'aide de SSH. Nous devons maintenant définir la définition des commandes pour utiliser check_by_ssh pour la connexion aux clients. Pour ce faire, nous devrons éditer **commandes.cfg**, situé dans le dossier **/usr/local/nagios/etc/objects/**. Ouvrir le fichier :

```
vim /usr/local/nagios/etc/objects/commands.cfg
```

Ajouter les commandes suivantes au fichier : 

```
define command {

	command_name check_remote_disk
	command_line $USER1$/check_by_ssh -H $HOSTADDRESS$ -C ‘/usr/lib/nagios/plugins/check_disk -w $ARG1$ -c $ARG2$’
}

define command{

	command_name check_remote_users
	command_line $USER1$/check_by_ssh -H $HOSTADDRESS$ -C ‘/usr/lib/nagios/plugins/check_users -w $ARG2$ -c $ARG3$’
}

define command{

	command_name check_remote_load
	command_line $USER1$/check_by_ssh -H $HOSTADDRESS$ -C ‘/usr/lib/nagios/plugins/check_load -w $ARG1$ -c $ARG2$’

}

define command{

	command_name check_remote_procs
	command_line $USER1$/check_by_ssh -H $HOSTADDRESS$ -C ‘/usr/lib/nagios/plugins/check_procs -w $ARG1$ -c $ARG2$ -s $ARG3$’

}

define command{

	command_name check_remote_swap
	command_line $USER1$/check_by_ssh -H $HOSTADDRESS$ -C ‘/usr/lib/nagios/plugins/check_swap -w $ARG1$ -c $ARG2$’
}
```

Selon la manière dont vous avez configuré nagios et les plugins, vous devrez peut-être changer la même chose dans les commandes mentionnées ci-dessus. Maintenant, sauvegardez le fichier, quittez et redémarrez le serveur nagios pour appliquer les modifications.

```
service nagios restart
```

Visitez maintenant l'interface de nagios et vous devriez pouvoir voir les informations sur le client. Ceci termine le didacticiel sur la configuration de la surveillance Nagios à l’aide de SSH.

# 3. Configuration NRPE

**NRPE** (**N**agios **R**emote **P**lugin **E**xecutor) est un agent de supervision qui permet de récupérer les informations à distance. Son principe de fonctionnement est simple : il suffit d’installer le serveur NRPE sur la machine distante et de l’interroger à partir du serveur Nagios.

## Installer le dépôt EPEL

Les plugins Nagios ainsi que les agents NRPE sont fournis par le dépôt EPEL. Extra Packages for Enterprise Linux (EPEL) est un groupe d'intérêt spécial de Fedora qui crée, met à jour et gère un ensemble de packages de haute qualité pour Enterprise Linux, notamment RHEL, CentOS et Scientific Linux (SL), Oracle Linux (OL).

```
yum install epel-release -y
```

## Installer NRPE et NRPE-plugins

Pour effectuer des requêtes NRPE, le serveur et le client. Nagios aura besoin du plugin NRPE.

Exécutez la commande suivante sur les 2 machines :

```
yum install -y nrpe nagios-plugins-all
```

## Configurer l'agent NRPE

**Les manipulations à venir seront à appliquer sur chaque hôte possédant un agent NRPE.**

Pour que le serveur Nagios puisse récupérer des informations au sujet de son hôte, il faut lui donner l’accès au serveur NRPE dans le fichier **/etc/nagios/nrpe.cfg**.

Éditez le fichier **nrpe.cfg** :

```
vi /etc/nagios/nrpe.cfg
```

Repérez la ligne suivante :

```
allowed_hosts=127.0.0.1,::1
```

Ajoutez l’adresse IP du serveur Nagios.

Indiquez les substituts de commande appropriés, par exemple :


```
command[check_users]=/usr/lib64/nagios/plugins/check_users -w 5 -c 10
command[check_load]=/usr/lib64/nagios/plugins/check_load -r -w 8.0,7.5,7.0 -c 11.0,10.0,9.0
command[check_disk]=/usr/lib64/nagios/plugins/check_disk -w 15% -c 10% /
command[check_mem]=/usr/lib64/nagios/plugins/check_mem -w 75% -c 90%
command[check_total_procs]=/usr/lib64/nagios/plugins/check_procs -w 300 -c 400
command[check_swap]=/usr/lib64/nagios/plugins/check_swap -w 10 -c 5
```

## Firewall

Assurez-vous d'autoriser l'accès à l'agent NRPE via le pare-feu.

```bash
### FirewallD ###

firewall-cmd --permanent --add-service=nrpe
firewall-cmd --reload
```

## Démarrer et permettre à NRPE de s'exécuter au démarrage du système

```
systemctl start nrpe
systemctl enable nrpe
```

## Configuration de NRPE sur le serveur

Ce sera la dernière étape pour configurer la surveillance Nagios à l'aide de NRPE. Nous devons maintenant définir la définition des commandes pour utiliser check_nrpe pour la connexion aux clients. Pour ce faire, nous devrons éditer **commands.cfg**, situé dans le dossier **/usr/local/nagios/etc/objects/**. Ouvrir le fichier :

```
vim /usr/local/nagios/etc/objects/commands.cfg
```

Ajouter les commandes suivantes au fichier : 

```
# this command runs a program $ARG1$ with no arguments
define command {
        command_name    check_nrpe_1arg
        command_line    /usr/lib/nagios/plugins/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```
