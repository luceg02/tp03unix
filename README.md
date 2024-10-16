# tp03unix


j'ai cree fichier puis mis dedans script sauvegarde et donne permission puis lance script
#!/bin/bash

# Affiche le nombre de paramètres
echo "Bonjour, vous avez rentré" "$#" "paramètres."

# Affiche le nom du script
echo "Le nom du script est" "$0."

# Vérifie si le 3ème paramètre est présent et l'affiche
if [ $# -ge 3 ]; then
    echo "Le 3ème paramètre est" "$3."
else
    echo "Moins de 3 paramètres fournis."
fi

# Affiche la liste de tous les paramètres
echo "Voici la liste des paramètres :" "$@"


