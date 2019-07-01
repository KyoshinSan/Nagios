# Nagios Centos 7
Nagios est l’outil de contrôle open source le plus largement utilisé. Il nous aide à surveiller les services et les applications exécutées sous Windows, Linux, les routeurs et d’autres périphériques réseau.

Avec l’aide de Nagios, on peut surveiller les services et attributs de base. Nous pouvons accéder à Nagios à l’aide de l’interface Web fournie avec le kit et la configuration doit être effectuée au niveau du fichier.

## Type de configuration

1. Configuration Normal
2. Configuration check_by_ssh
3. Configuration NRPE

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

Créez un compte d'utilisateur (nagiosadmin) pour vous connecter à l'interface Web de Nagios. Rappelez-vous le mot de passe que vous avez attribué à cet utilisateur - vous en aurez besoin plus tard.

```
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

Redémarrez le serveur Web Apache pour que les nouveaux paramètres prennent effet.

```
systemctl restart httpd
systemctl enable httpd
```

# 1. Configuration Normal 
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

Screenshot

La console Nagios ressemblera à celle ci-dessous.

Screenshot

Cliquez sur **Host** dans le volet de gauche pour obtenir une liste des systèmes surveillés par Nagios. Nous n’avons ajouté aucun hôte à Nagios, il surveille donc simplement l’hôte local lui-même.

Screenshot

Cliquez sur **Services** dans le volet de gauche pour obtenir le statut des services surveillés avec Nagios.

Screenshot

# 2. Configuration check_by_ssh
