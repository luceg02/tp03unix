#TP03 UNIX LUCE GIOVANNETTI
Pour tous les exercices, je crée les fichiers et j'écris dedans avec la commande nano, puis je donne les permissions avec chmod +x.

## Exercice : paramètres

```bash
#!/bin/bash

# Affiche le nombre de paramètres
echo "Bonjour, vous avez rentré" "$#" "paramètres."

# Affiche le nom du script
echo "Le nom du script est" "$0."

# Contrôle s'il y a un 3ème paramètre et l'affiche
if [ $# -ge 3 ]; then
    echo "Le 3ème paramètre est" "$3."
else
    echo "Moins de 3 paramètres fournis."
fi

# Affiche la liste des paramètres passés
echo "Voici la liste des paramètres :" "$@"
```

## Exercice : vérification du nombre de paramètres
- script
```bash
#!/bin/bash

# Vérification du nombre de paramètres passés
if [ "$#" -ne 2 ]; then
    echo "Erreur : il faut entrer 2 paramètres."
    exit 1 # Quitte le script avec une erreur
fi

# Concaténation des paramètres passés
CONCAT="$1$2"

# Résultat
echo "La concaténation des deux mots est : $CONCAT"
```

- Exécution du script

Lancement avec succès :
```bash
root@serveur1:~# ./concat.sh hey ciao
La concaténation des deux mots est : heyciao
```

Avec erreur :
```bash
root@serveur1:~# ./concat.sh hey
Erreur : il faut entrer 2 paramètres.
root@serveur1:~# echo $?
1
```

## Exercice : argument type et droits

- script
```bash
#!/bin/bash

# Vérifie si on a passé un paramètre, sinon sort de l'exécution avec une erreur
[ $# -ne 1 ] && echo "Erreur : 1 paramètre requis." && exit 1

# Vérifie si le paramètre correspond à un fichier/répertoire, sinon sort de l'exécution avec une erreur
[ ! -e "$1" ] && echo "Le fichier $1 n'existe pas." && exit 1

# Vérifie le type de fichier
[ -d "$1" ] && TYPE="répertoire" || [ -f "$1" ] && TYPE="fichier ordinaire" || TYPE="type non reconnu"

# Affiche le type du fichier ou répertoire.
echo "Le fichier $1 est un $TYPE."

# Vérifie les droits
PERM=""; [ -r "$1" ] && PERM="lecture"
[ -w "$1" ] && PERM="$PERM écriture"
[ -x "$1" ] && PERM="$PERM exécution"

# Résultat qui dit par qui le fichier/répertoire passé en paramètre est accessible et avec quel type de permission
echo "\"$1\" est accessible par $(whoami) en $PERM."
```

- Exécution du script

Lancement avec succès :
```bash
root@serveur1:~# ./test-fichier.sh /etc
Le fichier /etc est un fichier ordinaire.
"/etc" est accessible par root en lecture écriture exécution.
```

Script avec erreur :
```bash
root@serveur1:~# ./test-fichier.sh /be
Le fichier /be n'existe pas.
root@serveur1:~# echo $?
1
```

## Exercice : Afficher le contenu d’un répertoire

- script
```bash
#!/bin/bash

# Vérifie paramètre
if [ $# -ne 1 ]; then
    echo "Erreur : Insérez un paramètre."
    exit 1
fi

# Vérifie si le paramètre passé est un répertoire valide
if [ ! -d "$1" ]; then
    echo "Le chemin spécifié n'est pas un répertoire ou n'existe pas."
    exit 1
fi

Repertoire="$1"

# Affiche les fichiers
echo "####### fichiers dans $Repertoire/"
find "$Repertoire" -maxdepth 1 -type f

# Affiche les sous-répertoires
echo "####### répertoires dans $Repertoire/"
find "$Repertoire" -maxdepth 1 -type d ! -name '.'
```

- Exécution du script

Lancement avec succès :
```bash
root@serveur1:~# ./listedir.sh /boot
####### fichiers dans /boot/
/boot/System.map-6.1.0-25-amd64
/boot/initrd.img-6.1.0-25-amd64
/boot/vmlinuz-6.1.0-25-amd64
/boot/config-6.1.0-25-amd64
####### répertoires dans /boot/
/boot
/boot/grub
```

Dans le cas d'erreur :
```bash
root@serveur1:~# ./listedir.sh /baaa
Le chemin spécifié n'est pas un répertoire ou n'existe pas.
root@serveur1:~# echo $?
1
```

## Exercice : Lister les utilisateurs

- script
```bash
#!/bin/bash

# Lire chaque utilisateur du fichier /etc/passwd
for user in $(cut -d: -f1 /etc/passwd); do
    uid=$(id -u "$user")

    # Vérifie si l'UID est supérieur à 100, si oui print
    if [ "$uid" -gt 100 ]; then
        echo "Nom d'utilisateur : $user, UID : $uid"
    fi
done
```

- Exécution du script

Lancement avec succès :
```bash
root@serveur1:~# ./listenoms.sh
Nom d'utilisateur : nobody, UID : 65534
Nom d'utilisateur : systemd-network, UID : 998
Nom d'utilisateur : systemd-timesync, UID : 997
Nom d'utilisateur : avahi-autoipd, UID : 101
Nom d'utilisateur : sshd, UID : 102
```

Avec `awk` :
```bash
#!/bin/bash
# Parcourt le fichier /etc/passwd, vérifie que l'uid est supérieur à 100 et affiche le premier champ (nom de l'utilisateur) et le troisième champ (UID).
awk -F: '$3 > 100 { print $1, $3 }' /etc/passwd | while read nom uid;
do
        echo "Nom utilisateur : $nom, UID : $uid"
done
```

- Exécution du script avec `awk`

Lancement avec succès :
```bash
root@serveur1:~# ./listenoms.sh
Nom utilisateur : nobody, UID : 65534
Nom utilisateur : systemd-network, UID : 998
Nom utilisateur : systemd-timesync, UID : 997
Nom utilisateur : avahi-autoipd, UID : 101
Nom utilisateur : sshd, UID : 102
```

## Exercice : Mon utilisateur existe-t-il

- script
```bash
#!/bin/bash

# Vérifie le nombre de paramètres
if [ $# -ne 1 ]; then
    echo "Erreur : vous devez fournir un nom d'utilisateur ou un UID"
    exit 1
fi

p=$1

# Vérifie si le paramètre fourni est un nom d'utilisateur, si oui cherche l'UID
uid=$(awk -F: -v user="$p" '$1 == user { print $3 }' /etc/passwd)
if [ -n "$uid" ]; then
    echo "L'utilisateur $p existe avec UID : $uid"
    exit 0
fi

# Vérifie si le paramètre fourni est un UID, si oui cherche le nom
login=$(awk -F: -v id="$p" '$3 == id { print $1 }' /etc/passwd)
if [ -n "$login" ]; then
    echo "L'UID $p appartient à l'utilisateur : $login"
    exit 0
fi

# Gestion erreur : le paramètre ne correspond ni à un nom ni à un UID
echo "Aucun utilisateur trouvé avec le nom ou l'UID : $p"
exit 1
```

- Exécution du script

Avec un UID :
```bash
root@serveur1:~# ./userexist.sh 998
L'UID 998 appartient à l'utilisateur : systemd-network
```

Avec un nom :
```bash
root@serveur1:~# ./userexist.sh nobody
L'utilisateur nobody existe avec UID : 65534
```

Avec une erreur :
```bash
root@serveur1:~# ./userexist.sh
Erreur : vous devez fournir un nom d'utilisateur ou un UID
root@serveur1:~# echo $?
1
```

## Exercice: Création utilisateur

- script
```bash
# Vérifier si le groupe existe, sinon le créer
if ! getent group "$gid" > /dev/null; then
    echo "Le groupe avec GID $gid n'existe pas. Création du groupe..."
    groupadd -g "$gid" "group_$gid"
    if [ $? -ne 0 ]; then
        echo "Erreur : Échec de la création du groupe."
        exit 1
    fi
fi

# Créer le répertoire home
mkdir -p "$home"

# Ajouter le nouvel utilisateur
useradd -m -d "$home" -u "$uid" -g "$gid" -c "$commentaires" -s /bin/bash "$login"
if [

 $? -ne 0 ]; then
    echo "Erreur : La création de l'utilisateur '$login' a échoué."
    exit 1
fi

# Définir le mot de passe pour le nouvel utilisateur
passwd "$login"

echo "Félicitations ! Le compte utilisateur '$login' a été créé avec succès."
```

- Exécution du script

Lancement avec succès :
```bash
root@serveur1:~# ./_create_user.sh pippo
Aucun utilisateur trouvé avec le nom ou l'UID : pippo
Nom: pippo
Prénom: paperino
UID: 102030
GID: 102030
Commentaires: Test utilisateur
Le groupe avec GID 102030 n'existe pas. Création du groupe...
Groupe 'group_102030' créé avec succès.
Le répertoire /home/pippo n'existe pas.
Félicitations ! Le compte utilisateur 'tom' a été créé avec succès.
```

- Vérification de la création de l'utilisateur `pippo`

```bash
root@serveur1:~# ./listenoms.sh
Nom utilisateur : nobody, UID : 65534
Nom utilisateur : systemd-network, UID : 998
Nom utilisateur : systemd-timesync, UID : 997
Nom utilisateur : avahi-autoipd, UID : 101
Nom utilisateur : sshd, UID : 102
Nom utilisateur : pippo, UID : 102030
```

## Exercice : Lecture au clavier

- script
Essayer les commandes indiquées :
```bash
root@serveur1:~# nano lectureclavier.sh
root@serveur1:~# chmod +x lectureclavier.sh
root@serveur1:~# ./lectureclavier.sh
entrer votre nom: luce
votre nom est luce
```

### Commande `file`

```bash
root@serveur1:~# file ex.txt
ex.txt: ASCII text
```

### Commande `more`

Comment quitter `more` ? Pour quitter `more`, on appuie sur "q".  
Comment avancer d’une ligne ? Pour avancer d'une ligne, on appuie sur la touche "Entrée" ou sur la flèche vers le bas.  
Comment avancer d’une page ? Pour avancer d'une page, on appuie sur la touche "espace".  
Comment remonter d’une page ? Pour remonter d'une page, on appuie sur la touche "b".  
Comment chercher une chaîne de caractères ? Pour rechercher une chaîne de caractères, on appuie sur "/", puis sur "Entrée". On peut appuyer sur "n" pour passer à l'occurrence d'après.
```
- script
```bash
#!/bin/bash

#controler nombre de parametres
if [ "$#" -ne 1 ]; then
echo "Inserez un paramètre."
        exit 1
fi

rep="$1"

#Verification sur le repertoire pour voir s il existe
if [ ! -d "$rep" ]; then
        echo "Aucun repertoire $rep trouvé"
        exit 1
fi

#parcourir le repertoire
for fichier in "$rep"/*; do

#verification du fichier pour voir si c'est de type text
if file "$fichier" | grep -q "text" && [ "${fichier##*.}" != "sh" ]; then
        read -p "Voulez vous visualiser le fichier $fichier ? (o/n): " r
        if [ "$r" = "0" ] || [ "$r" = "o" ]; then
                more "$fichier"
        fi
fi
done
```

- execution du script
```bash
root@serveur1:~# ./view_file.sh shell
Voulez-vous visualiser le fichier shell/ex.txt ? (o/n): 0

hello world
abc

byeeee
```

# Exercice : Appréciation

- script

```bash
#!/bin/bash

while true; do
    read -p "Entrez une note (ou 'q' pour quitter) : " note

    # Vérification si l'utilisateur veut quitter
    if [ "$note" = "q" ]; then
        echo "Programme terminé."
        exit 0
    fi

    # Vérifier la note et afficher le message correspondant
    if (( note >= 16 && note <= 20 )); then
        echo "Très bien."
    elif (( note >= 14 && note < 16 )); then
        echo "Bien."
    elif (( note >= 12 && note < 14 )); then
        echo "Assez bien."
    elif (( note >= 10 && note < 12 )); then
        echo "Moyen."
    elif (( note < 10 )); then
        echo "Insuffisant."
    else
        echo "Erreur: la note insérée n'est pas valide, insérez un nombre entre 0 et 20."
    fi
done
```

- Exécution 

```bash
root@serveur1:~# ./appreciation.sh
Entrez une note (ou 'q' pour quitter) : 18
Très bien.
Entrez une note (ou 'q' pour quitter) : 16
Très bien.
Entrez une note (ou 'q' pour quitter) : 15
Bien.
Entrez une note (ou 'q' pour quitter) : q
Programme terminé.
```
