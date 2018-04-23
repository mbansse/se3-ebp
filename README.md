# se3-ebp
Utilisation du PGI client-serveur EBP dans un environnement sambaedu


**Article en tout début de rédaction.**

* [Présentation](#présentation)
* [Mise en place du serveur Linux SQL](#mise-en-place-du-serveur-linux-sql)
     * [Installation du système d'exploitation](#installation-du-systeme-d-exploitation)
     * [Installation du moteur SQL](#installation-du-moteur-sql)
     * [Sécurisation du serveur MySQL](#securisation-du-serveur-mysql)
     * [Création d'un utilisateur pouvant lire et écrire dans les bases de données](#creation-dun-utilisateur-pouvant-lire-et-ecrire-dans-les-bases-de-donnees)


## Présentation
EBP est un logiciel type PGI. Il peut être utilisé de façon collaborative en mode client-serveur avec la mise en place d'un serveur dédié.

IL ne sera pas nécéssaire d'utiliser un windows serveur,d'acheter plusieurs NAS qui ne permettent qu'à 5 utilisateurs simultanés de travailler).

On installera sur un serveur dédié une ditribution  `open-source` Linux Server (Débian ou Ubuntu) avec un moteur mysql. Ce serveur sera placé dans le réseau pédagogique au même titre que le se3 et utilisera donc le domaine existant.

Il faudra créer ensuite un compte utilisateur/administrateur de bases mysql. C'est ce compte qui sera utilisé par les professeurs pour uploader/dupliquer/supprimer les bases que les élèves vont utiliser.

Les clients EBP sous  Windows se connecteront au serveur au moyen de raccourcis. Plusieurs élèves pourront ainsi travailler sur une même base de donnée et gérer chacun leur tâche de façon collaborative.

L'upload/suppression de bases EBP pourra se faire avec des outils graphiques sous Windows, rendant la gestion par un professeur non initié à Linux très simple.

## Mise en place du serveur Linux SQL

### Installation du système d'exploitation
L'installation décrite ici a été faite sur une debian server Stretch. Le plus simple est de télécharger l'iso netinstall disponible ici:
https://www.debian.org/CD/netinst/

Sauf serveur très vieux ou particulier, on choisira la version amd64:
https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-9.4.0-amd64-netinst.iso

On gravera le fichier iso téléchargé sur un cd. Il suffira de booter sur le cd et de se laisser guider. Une installation par clef usb est également possible. L'iso doit être placée sur la clef à l'aide d'un logiciel comme **unetbootin** .

-On choisir la mode d'installation **install** et non graphical install.
![01](images/install1.png)

- Choix de la langue: Français évidemment. Idem pour le pays et la configuration du clavier.
- Nom de machine: choisir un nom netbios avec moins de 15 caractères(et si possible sans caractères spéciaux à l'expetion du   tiret.
![02](images/install2.png)
- Domaine: Vous pouvez laisser cet espace **vide** si le serveur ne sera disponible qu'en interne (ce qui sera le cas ici).
- Mot de passe root: Choisir un mot de passe **complexe**. Il faudra l'entrer deux fois (et il n'apparait pas à l'écran).
- Choix d'un utilisateur: Choisir également un mot de passe solide [qui ne soit pas le même] que root.
- Partionnement: choisir d'utiliser le disque entier (convient ici puisqu'on mettra en place plus tard un système de sauvegarde externe des bases). On installera également tout le système dans une mme partition.
- Outil de gestion des paquets Debian: On utilisera pas d'autre analyse de cd.
![03](images/install3.png)
- Choix du miroir: Si on est débutant on choisira `France` puis `ftp.fr.debian.org`. Si le se3 sert de miroir local, on pourra choisir `saisie manuelle` (en haut de la liste), puis **nom d'hote**: http://172.20.0.2:9999 (en changeant l'ip par celle du se3). Le répertoire du miroir sera alors `/ftp.fr.debian.org/debian/` .
- Serveur proxy: Si l'établissement possède un serveur proxy (souvent le AMON), il faut l'indiquer ici.
![04](images/install4.png)
- Participation à des études anonymes: A vous de voir, mais à priori non.
- Choix des paquets à installer :  on ne cochera que les utilitaires usuels du système, ainsi que le serveur ssh. Tout autre choix est à décocher (surtout le choix d'une interface graphique qui ne sera d'aucune utilité pour un serveur sql).
![05](images/install5.png)
- Installer le grub:sur le secteur d'amorçage: OUI (ou le serveur ne pourra pas démarrer de façon autonome).
- Périphérique où sera installé grub: choisir le premier disque (/dev/sda)

L'installation se finalise. Le serveur va redémarrer. Penser à enlever le support d'installation pour que celle-ci ne recommence pas en boucle. 

On se connecte en root sur le serveur en entrant le login "root" et le mot de passe choisi au début.

Pour que le serveur garde la même ip, il faudra bien penser à faire une réservation d'adresse (on peut aussi modifier le fichier /etc/network/interfaces pour inscrire en dur l'adresse du serveur). 
```
nano /etc/network/interfaces
```

![06](images/install6.png)
On relance ensuite la connexion réseau par
```
service networking restart
```
On peut redémarrer le serveur pour plus de sureté.


### Installation du moteur SQL
Le serveur est maintenant acif, mais le service SQL n'est pas installé.

On se connecte en root sur le serveur puis on lnce l'installation du serveur SQL.
```
apt-get install mysql-server
```
Le serveur est maintenant doté d'un moteur mysql. A noter qu'aucun mot de passe n'a été demandé. Sur une Debian stretch le moteur SQL est mariadb.

### Sécurisation du serveur MySQL
Puisqu'aucun mot de passe n'a été demandé, il faut sécuriser l'accès au serveur en interdisant la connexion distante en root.

On lance la commande:

```
mysql_secure_installation
```

Initialement, il n'y a pas de mdp root pour mysql, on laissera donc la première demande vierge. On indiquera ensuite Y pour mettre un mot de passe solide Mysql

![07](images/mysql1.png)


On supprimera ensuite l'utilisateur anonyme en tapant Y pour "remove anonymous users"
![08](images/mysql2.png)
On désactivera ensuite l'accès "root" aux utilisateurs: ainsi, une connection "root" sera possible uniquement sur le serveur, mais impossible avec des outils de gestion MySQL comme phpmyadmin ou MySQL Workbench (sur Windows).

![09](images/mysql3.png)
On supprime également les bases de tests et autres fichiers inutiles.

### Création d'un utilisateur pouvant lire et écrire dans les bases de données
Il faut maintenant créer un nouvel utilisateur de bases mysql, car les enseignants n'auront pas accès au compte root.

On accède à la console mysql en tapant
```
mysql -u root -p
```
Ensuite on va indiquer qu'un nouvel utilisateur de base mysql doit être créé.

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

On pourra évidemment créer plusieurs utilisateurs. Une fois les opérations terminées on peut quitter la console Mysql.

```
exit
```

## Installation des clients EBP.
A venir.

Choisir le mode **Installation réseau**
image client1.png
Séléctionner ensuite le mode **client** seulement. 
image client2.png


### activation du logiciel

* Méthode à l'ancienne

Il est plus que probable que l'établissement n'utilise pas la dernière version des clients EBP. Lors de la première execution, il faudra activer le module manuellement.

Choisir **activer le logiciel**

Puis activer manuellement
image client4.png

Entrer les renseignements de licence tels qu'ils figurent sur le courrier d'EBP. Puis cliquer sur **activer le logiciel**.

Normalement un message doit indiquer que le logiciel est bien activé.
image client6.png

On precède de la même façon pour chacun des composants EBP.
* Sauvegarde/restauration du fichier licence.xml
Une fois tous les logiciels activés, on pourra sauvegarder le fichiere licence.xml placé dans 'C:\ProgramData\EBP'. 

Si on doit réinstaller les logiciels sur un autre type de poste, on procedra à l'installation, puis on copiera le fichier dans le répertoire du nouveau poste. 
Si la licence est valide, l'activation se fera sans avoir à tout entrer une nouvelle fois.

## Accès aux bases MySQL par les professeurs
Il est techniquement possible de faire toutes les opérations en ligne de comma,de sur le serveur, mais on pourra utiliser divers logiciels comme MySQL WOrkbench pour les opérations de gestion de base.

Il faut télécharger ce logiciel ici
https://dev.mysql.com/downloads/workbench/

A noter que la présence de .Netframework est nécéssaire pour utiliser MySQL Workbench, ainsi que Microsoft Visual C++ 2015

.Netframework 4.5.2 nécéssaire (https://www.microsoft.com/net/download/thank-you/net452?survey=false)

L'installeur MySQL permet d'ajouter tout un tas de modules, ici seul le module Workbench nous intéresse. 
Dans la partie `choosing a Setup Type`, on prendra `Custom` puis on choisira seulement mySQL Workbench.


## sauvegardes hebdomadaires des bases mysql
On va créer un répertoire savmysql à la racine du disque.
```
mkdir /savmysql
```
Ce répertoire va servir de point de montage d'un disque dur externe, interne, partage samba, nas...

On peut créer un script contenant ces lignes. Ce script appelé savmysql est placé dans /root et doit être executable
(chmod u+x /root/savmysql)
```
#!/bin/bash
mount -t cifs //172.20.0.11/savmysql /savmysql -o credentials=/root/credentials
mysqldump --user=root --password=mdprootmysql --all-databases | gzip > /savmysql/sauvegarde-`date +%Y-%m-%d-%H-%M`.sql.gz
#chmod -R 700 /savmysql/
cd /root
umount /savmysql
exit
```
A noter, il faut indique le mdp root "mysql" choisi lors de l'installation et non le mot de passe root du serveur.

L'utilisateur root va sauvegarder l'ensemble des bases compressées dans un fichier comportant la date et l'heure.
Ainsi, en cas de crash du serveur, il sera possible d'en refabriquer un avec les mmes paramètres et de restaurer les bases existantes.

Ce script peut-être lancé en tache cron de façon hebdomadaire.
```
crontab -e
```
On ajoute en bas du fichier
```
0 4 * * 0 /root/savmysql
```
Maintenant, le script est lancé tous les dimanches à 4h00 du matin. Vous disposez donc d'une sauvegarde complète hebdomadaire.
Il faudra de temps en temps en supprimer quelques unes ou le disque sera saturé (même si la compression est ici très eficace).

Les bases, une fois décompressées pourront être restaurées avec cette commande.

```
mysql --user=root --password=mdprootmysql < fichier_source.sql
```



