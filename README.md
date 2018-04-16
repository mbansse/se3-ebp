# se3-ebp
utilisation du PGI client-serveur EBP dans un environnement sambaedu

**Article en tout début de rédaction.**

EBP est un logiciel type PGI. Il peut être utilisé de façon collaborative en mode client-serveur.

IL ne sera pas nécéssaire d'utiliser un windows serveur, ou d'activer un nouveau domaine sur le réseau pédagogique.

On installera sur un serveur dédié une ditribution Linux Server (Débian ou Ubuntu) avec un moteur mysql. Ce serveur sera placé dans le réseau pédagogique au même titre que le se3.
Il faudra créer ensuite un compte utilisateur/administrateur de bases mysql. 

Les clients EBP sous  Windows se connecteront au serveur au moyen de raccourcis. Plusieurs élèves pourront ainsi travailler sur une même base de donnée et gérer chacun leur tâche de façon collaborative.

L'upload/suppression de bases EBP pourra se faire avec des outils graphiques sous Windows, rendant la gestion par un professeur non initié à Linux très simple.


```
apt-get install mysql-server
```
Le serveur est maintenant doté d'un moteur mysql. Il ne reste plus qu'à sécuriser l'accès, et à créer un utilisateur.


```
mysql_secure_installation
```
Initialement, il n'y a pas de mdp root pour mysql, on laissera donc la première demande vierge.
On indiquera ensuite Y pour mettre un mot de passe solide Mysql
--> image mysql1

On supprimera ensuite l'utilisateur anonyme en tapant Y pour "remove anonymous users"
--> mysql3
On désactivera ensuite l'accès "root" aux utilisateurs: ainsi, une connection "root" sera possible uniquement sur le serveur.
--> mysql3
On supprime également les bases de tests.


Il faut maintenant créer un nouvel utilisateur de bases mysql, car les enseignants n'auront pas accès au compte root.
```
CREATE USER 'adminmysql'@'localhost' IDENTIFIED BY 'mysql123';
```
L'utilisateur est maintenant créé, il ne reste plus qu'à lui donner des droits de lecteure/écriture sur les bases.

```
GRANT ALL PRIVILEGES ON * . * TO 'adminmysql'@'localhost';
```
Il faut maintenant appliquer les différents changements effectués.
```
FLUSH PRIVILEGES;
```

On pourra évidemment créer plusieurs utilisateurs.


