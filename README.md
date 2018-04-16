# se3-ebp
utilisation du PGI client-serveur EBP dans un environnement sambaedu

**Article en tout début de rédaction.**

EBP est un logiciel type PGI. Il peut être utilisé de façon collaborative en mode client-serveur.

IL ne sera pas nécéssaire d'utiliser un windows serveur, ou d'activer un nouveau domaine sur le réseau pédagogique.

On installera sur un serveur dédié une ditribution Linux Server (Débian ou Ubuntu) avec un moteur mysql. Ce serveur sera placé dans le réseau pédagogique au même titre que le se3.
Il faudra créer ensuite un compte utilisateur/administrateur de bases mysql. 

Les clients EBP sous  Windows se connecteront au serveur au moyen de raccourcis. Plusieurs élèves pourront ainsi travailler sur une même base de donnée et gérer chacun leur tâche de façon collaborative.

L'upload/suppression de bases EBP pourra se faire avec des outils graphiques sous Windows, rendant la gestion par un professeur non initié à Linux très simple.
