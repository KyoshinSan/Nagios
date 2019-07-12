# Les Alertes Nagios par Sms

L'un des points essentiels pour les plateformes de supervision est le système de notification. Par défaut, la grande majorité utilise la notification par email via les daemons postfix, mail, ou exim installés sur le serveur de supervision afin d'envoyer des notifications. Cependant tout ces daemons possèdent des inconvénients non négligeable:

- Nécessite d'avoir un serveur SMTP.
- Si le serveur de mail est éteint, on ne reçoit plus de notification.
- Les notifications par mail peuvent être considérées comme des spams et donc filtrer et bloquer.

De ce fait, on se retrouve souvent à considérer d'autres méthodes de notification afin de garantir une réaction plus rapide en cas de nécessiter. L'une d'entres elles est la notification par SMS.

C'est dans ce tutoriel que nous allons mettre en place ce service avec la plateforme de supervision Nagios.

## Prérequis :

Gammu permettent d'envoyer et de recevoir des messages via des passerelles GSM. Afin de pouvoir envoyer des SMS, il nous faut :

- Un modem GSM / GPRS. Nous utiliserons un ancien téléphone portable.
- Une carte SIM avec un numéro mobile attribué.
- Verifier que le modem est situé dans une zone de couverture du réseau GSM.

## Installation de Gammu

Cf. [Envoie de SMS avec Gammu (Nokia E71)](https://github.com/KyoshinSan/Gammu)

## Configuration de Nagios

Suite à l'installation de Gammu, nous allons configurer Nagios afin qu'il puisse nous notifier par SMS. Pour cela, modifions le fichier **commands.cfg** :

```
vi /etc/nagios/objects/commands.cfg
```

Dans notre fichier **commands.cfg**, nous allons utilisés les commandes **notify-host-by-email** et **notify-service-by-email** qui sont déjà présentent dans le répertoire de Nagios.

```
define command {
        command_name    notify-host-by-sms
        command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /usr/bin/gammu --sendsms TEXT $CONTACTPAGER$
}

define command {

        command_name    notify-service-by-sms
        command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /usr/bin/gammu --sendsms TEXT $CONTACTPAGER$
}
```

Cette commande est très simple. Nous envoyons le nom de l'hôte ayant un soucis, son statut, son adresse IP sur le réseau, l'heure et la date de la notification. Tout cela est envoyé vers notre $CONTACTPAGER$ qui n'est autre que notre contact SMS. Cependant nous devons définir ce $CONTACTPAGER$. Pour cela, nous allons ajouter un contact dans notre fichier de configuration **contacts.cfg** :

```
define contact {

    contact_name            nagiosadmin             ; Short name of user
    use                     generic-contact         ; Inherit default values from generic-contact template (defined above)
    alias                   Nagios Admin            ; Full name of user
    email                   nagiosadmin@exemple.com ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
    pager                   +336xxxxxxxx
    service_notification_commands   notify-service-by-email notify-service-by-sms
    host_notification_commands      notify-host-by-email notify-host-by-sms
}
```

Avant de terminer, nous devons nous assurés que nous n'avons pas d'erreur.

```
/usr/sbin/nagios -v /etc/nagios/nagios.cfg
```

Si nous n'avons pas d'erreur, nous pouvons redémarrer le service nagios afin que la notification par SMS soit activé.

```
systemctl restart nagios
```

Nous avons à présent un service de notification par SMS associé à notre fameuse plateforme de supervision Nagios.

## Les erreurs possibles

Nagios ne peut pas envoyer de sms, à voir dans les log quand il exécute les commandes pour envoyer les notifications.

```
output=Error opening device, it doesn't exist.
```

Assurez-vous que **nagios** (et **apache** si vous le souhaitez) dispose des droits d’écriture sur **/dev/ttyACM1**.

Vérifiez ceci en premier:

```
ls -l /dev/ttyACM1
```

En supposant que vous obtenez :

```
crw-rw----. 1 root dialout 166, 1 12 juil. 10:48 /dev/ttyACM1
```

Ensuite ajoutez l'utilisateur **nagios** au groupe **dialout** :

```
sudo usermod -a -G dialout nagios
```

Définissez le bit SUID sur la commande pour permettre à **nagios** d’exécuter gammu sous le compte **root** :

```
chmod 4755 /usr/bin/gammu
```

Essayez d'envoyer un sms avec le compte **nagios** :

```
su - nagios -s /bin/bash
echo "Ceci est un test." | gammu --sendsms TEXT 06xxxxxxxx
```
