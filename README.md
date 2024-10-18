# tp03unix

Exercice : param`etres
j'ai cree fichier puis mis dedans script sauvegarde et donne permission puis lance script

```bash
#!/bin/bash

# Affiche le nombre de paramètres
echo "Bonjour, vous avez rentré" "$#" "paramètres."

# Affiche le nom du script
echo "Le nom du script est" "$0."

# Controle si il ya un 3eme parametre et l'affiche
if [ $# -ge 3 ]; then
    echo "Le 3ème paramètre est" "$3."
else
    echo "Moins de 3 paramètres fournis."
fi

# Affiche la liste des paramètres passes
echo "Voici la liste des paramètres :" "$@"
  ```

Exercice : v´erification du nombre de param`etres
#!/bin/bash

# Vérification du nombre de parametres passes
if [ "$#" -ne 2 ]; then
    echo "Erreur : il faut entrer 2 paramètres."
    exit 1 # Quitte le script avec un erreur
fi

# Concaténation des parametres passes
CONCAT="$1$2"

# Resultat
echo "La concaténation des deux mots est : $CONCAT"

si je lance le script
root@serveur1:~# ./concat.sh hey ciao
La concaténation des deux mots est : heyciao

avec erreur
root@serveur1:~# ./concat.sh hey
Erreur : il faut entrer 2 paramètres.
root@serveur1:~# echo $?
1

Exercice : argument type et droits

#!/bin/bash

# Vérifie si on a passe un parametre, sinon sors de l'execution avec un erreur
[ $# -ne 1 ] && echo "Erreur : 1 paramètre requis." && exit 1

# Vérifie si le parametre correspond à un fichier/repertoire, sinon sors de l'execution avec un erreur
[ ! -e "$1" ] && echo "Le fichier $1 n'existe pas." && exit 1

# Verifie le type de fichier
[ -d "$1" ] && TYPE="répertoire" || [ -f "$1" ] && TYPE="fichier ordinaire" || TYPE="type non reconnu"

# Affiche le type du fichier ou répertoire.
echo "Le fichier $1 est un $TYPE."

# Vérifie les droits
PERM=""; [ -r "$1" ] && PERM="lecture"
[ -w "$1" ] && PERM="$PERM écriture"
[ -x "$1" ] && PERM="$PERM exécution"

# Resultat qui dis par qui le fichier/repertoire passe en parametre est accessible et avec quel type de permission
echo "\"$1\" est accessible par $(whoami) en $PERM."

&& : Exécute la commande suivante si la précédente réussit.
|| : Exécute la commande suivante si la précédente échoue.

si je lance le script
root@serveur1:~# ./test-fichier.sh /etc
Le fichier /etc est un fichier ordinaire.
"/etc" est accessible par root en lecture écriture exécution.

script avec erreur
root@serveur1:~# ./test-fichier.sh /be
Le fichier /be n'existe pas.
root@serveur1:~# echo $?
1

Exercice : Afficher le contenu d’un r´epertoire
#!/bin/bash

# Vérifie parametre
if [ $# -ne 1 ]; then
    echo "Erreur : Inserez un parametre."
    exit 1
fi

# Vérifie si le paramètre passe est un répertoire valide
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

si je lance le script
root@serveur1:~# ./listedir.sh /boot
####### fichiers dans /boot/
/boot/System.map-6.1.0-25-amd64
/boot/initrd.img-6.1.0-25-amd64
/boot/vmlinuz-6.1.0-25-amd64
/boot/config-6.1.0-25-amd64
####### répertoires dans /boot/
/boot
/boot/grub

dans le cas d erreur
root@serveur1:~# ./listedir.sh /baaa
Le chemin spécifié n'est pas un répertoire ou n'existe pas.
root@serveur1:~# echo $?
1






