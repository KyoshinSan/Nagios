# Les Alertes Nagios par Sms

L'un des points essentiels pour les plateformes de supervision est le système de notification. Par défaut, la grande majorité utilise la notification par email via les daemons postfix, mail, ou exim installés sur le serveur de supervision afin d'envoyer des notifications. Cependant tout ces daemons possèdent des inconvénients non négligeable:

- Nécessite d'avoir un serveur SMTP.
- Si le serveur de mail est éteint, on ne reçoit plus de notification.
- Les notifications par mail peuvent être considérées comme des spams et donc filtrer et bloquer.

De ce fait, on se retrouve souvent à considérer d'autres méthodes de notification afin de garantir une réaction plus rapide en cas de nécessiter. L'une d'entres elles est la notification par SMS.

C'est dans cet article que nous allons mettre en place ce service avec la plateforme de supervision la plus connue, Nagios.
