# Demo1

## CSH (C shell)
- La syntaxe du shell C est similaire à celle du langage C.
- Il est courant d'utiliser `.csh` ou `.tcsh` comme extension pour les fichiers de script.
- Fournit des fonctions similaires à celles d'un langage de programmation, telles que des alias, des mécanismes d'historique, etc.

## Bash (Bourne Again Shell)
- Bash est un shell Unix populaire.
- Il est courant d'utiliser `.sh` comme extension pour les fichiers de script.
- BASH est compatible avec `sh` et ajoute de nombreuses nouvelles fonctionnalités, telles que l'édition en ligne de commande, la substitution de commandes, etc.

## Créer le répertoire démo1
```bash
mkdir demo1
cd demo1
```

## Utiliser CURL pour envoyer une requête HTTP GET
```bash
curl -O http://abu.cnam.fr/BIB/index.html
cat index.html
```

## Créer un fichier de script `download_abu_works.sh`
```bash
nano download_abu_works.sh
```

## Ajouter le code suivant dans le fichier de script
```bash
#!/bin/bash

# Créer abu répertoire si n'existe pas
mkdir -p abu

# URL de la page d'accueil de la bibliothèque ABU
seed="http://abu.cnam.fr/BIB/index.html"

# Définissez la variable d'environnement LC_ALL pour gérer les séquences d'octets illégales
export LC_ALL=C 

for oeuvre in $(curl -s $seed | grep "href" | grep cgi-bin | grep 'go\?' | sed -e 's/.*?\([^"]*\)".*/\1/g'); do
  o="abu/$oeuvre"
  echo "Getting $oeuvre into $o"
  curl -s -o $o "http://abu.cnam.fr/cgi-bin/donner_unformated?$oeuvre"
done

```

## Exécuter le script
```bash
chmod +x download_abu_works.sh
./download_abu_works.sh
ls -l abu/
```

## Supprimer les informations d'en-tête du fichier texte
Par exemple, si nous avons un fichier nommé gaspard2, exécutez la commande suivante pour afficher les 10 premières lignes et convertir l'encodage des caractères de Latin1 en UTF-8:

### Écrire le script `scorie`
```bash
nano scorie
```
```bash
#!/bin/bash

# Vérifier si l'argument est vide
if [ -z "$1" ]; then
  echo "Usage: $0 <file>"
  exit 1
fi

# lire le path du fichier
file=$1

# Supprimer les informations d'en-tête du fichier texte, jusqu'à ce que la ligne de délimitation corresponde
iconv -f latin1 -t UTF-8 "$file" | awk '/------------------------- DEBUT DU FICHIER/ {flag=1; next} flag'
```

```bash
chmod +x scorie

./scorie abu/gaspard2 | head -n 20 
```
Ce script utilise d'abord iconv pour convertir le fichier du codage latin1 en UTF-8, puis utilise tail -n +2 pour supprimer la première ligne. Cela devrait éviter les problèmes de codage des caractères et supprimer avec succès les informations d'en-tête du fichier.

## afficher le code des oeuvres qui contiennet un mot particulier vu au moins n fois
```bash
#!/bin/bash

# Vérifier si l'argument est vide
if [ "$#" -ne 3 ]; then
    echo "Usage: $0 seen <word> <min_count>"
    exit 1
fi

# Analyser les paramètres
command=$1
word=$2
min_count=$3

# Assurer le premier argument est "seen"
if [ "$command" != "seen" ];then
    echo "Unknown command: $command"
    echo "Usage: $0 seen <word> <min_count>"
    exit 1
fi

# Afficher les paramètres pour le débogage
echo "Searching for word: $word"
echo "Minimum count: $min_count"

# Compter le nombre d'occurrences du mot dans chaque fichier
grep -r -i -o -w "$word" abu/ | awk -v min_count="$min_count" -F: '
{
    counts[$1]++;
}
END {
    for (file in counts) {
        if (counts[file] >= min_count) {
            print file ":" counts[file];
        }
    }
}'
```

## Exécuter le script
```bash
chmod +x go
./go seen soleil 50
```