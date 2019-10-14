#  :fr: Dictionnaires Grammalecte sans champs optionnels  :book: 
## Pourquoi ces dictionnaires ?
Au tout départ, je recherchais un dictionnaire français utilisable avec l'extension Alice - Spell Checking pour Brackets. 

Celui du projet Grammalecte.dic(fr) est très complet, mais le correcteur (ou la version du correcteur) orthographique utilisé par l'extension (`Typo.js`) ne lit pas correctement les mots ayant :
* des champs optionnels 
* mais pas de règles d'affixe
* exemple le mot `vroum po:interj`

## Les dictionnaires Hunspell
Les dictionnaires Grammelecte sont des dictionnaires Hunspell, chaque dictionnaire Hunspell est constitué de deux fichiers :
1. un fichier `mondictionnaire.dic` contenant une liste de mots avec ou non des règles d'affixe
2. un fichier `mondictionnaire.aff` contenant les déclinaisons de ces mots par ajout d'affixes (suffixe ou/et préfixe)

### Exemple

```
mondictionnaire.dic
-------------------
déjà <= mot sans règles d'affixe
poire/S.() <= le mot poire avec la règle pour le pluriels en s
poireau/X.() <= le mot poireau avec la règle pour le pluriels en x
http/S= <= déclinaison avec S comme suffixe
```

```
mondictionnaire.aff
-------------------
SFX S= Y 1 <= déclinaison du mot avec S comme suffixe
SFX S= 0 s [^sxz] <= ajout de la lettre s si le mot ne se termine pas par s ou x ou z
```

## Les dictionnaires Hunspell de Grammalecte

Ils sont disponibles sous licence Mozilla Public License 2.0 en 4 versions :

> Malgré les rectifications modestes apportées par cette réforme, la nouvelle orthographe suscite beaucoup de polémiques. Afin de satisfaire les exigences de chacun, quatre dictionnaires existent, respectant différemment cette réforme.

### Les 4 versions du dictionnaire français
#### Dictionnaire "Moderne"

Ce dictionnaire propose une sélection des graphies classiques et réformées, suivant la lente évolution de l'orthographe actuelle. Ce dictionnaire contient les graphies les moins polémiques de la réforme.

#### Dictionnaire "Classique" [recommandé]

Ce dictionnaire est une extension du dictionnaire « Moderne » et propose en sus des graphies alternatives, parfois encore très usitées, parfois tombées en désuétude.

#### Dictionnaire "Réforme 1990"

Ce dictionnaire ne connaît que les graphies nouvelles des mots concernés par la réforme de 1990.

#### Dictionnaire "Toutes variantes"

Ce dictionnaire contient les nouvelles et les anciennes graphies des mots concernés par la réforme de 1990.

### Les champs optionnels
Ces dictionnaires utilisent la possibilité d'ajouter des champs optionnels dans le fichier `mondictionnaire.dic` ce qui est conforme aux spécifications du format de fichier Hunspell.

#### Exemple
* Informations supplémentaires dans les champs optionnels

```
mondictionnaire.dic
-------------------
jolie/F.() po:adj <= jolie est un adjectif
jouet/S.() po:nom is:mas <= jouet est un nom masculin
```

* Extraction de la liste des prénoms

```
cat fr-classique.dic | grep po:prn | tail
Ziad po:prn is:mas is:inv
Zina po:prn is:fem is:inv
Zineb po:prn is:fem is:inv
Zinédine po:prn is:mas is:inv
Zita po:prn is:fem is:inv
Zoé po:prn is:fem is:inv
Zoey po:prn is:fem is:inv
Zohra po:prn is:fem is:inv
Zoroastre po:prn is:mas is:inv
Zosime po:prn is:mas is:inv
```

#### Inconvénients

L'utilisation de ces champs optionnels présentent les inconvénients suivants :
1. Au mieux, les correcteurs purement orthographiques n'utilisent pas ces informations, elles sont donc inutiles.
1. Au pire, les correcteurs ne lisent pas correctement les fichiers contenant ces champs optionnels.

## Les dictionnaires allégés (sans champs optionnels)

Ces dictionnaires allégés sont basés sur la dernière version disponible des dictionnaires, soit la version (6.4.1). 

### Comparaison avec les fichiers d'origines
Fichier | Original Taille | Allégé Taille | Delta %| Original Lignes | Allégé Lignes | Delta % 
:---|---:|---:|---:|---:|---:|---:
fr-classique.dic|2 413 552|1 220 742|-49,4%:chart_with_downwards_trend:|81 227| 80 205|-1,3%
fr-classique.aff| 401 541|258 111|-35,7%:chart_with_downwards_trend:|9 055| 8 812|-2,7%
**fr-classique.*** | **2 815 093**|**1 478 853**|**-47,5%** :chart_with_downwards_trend:|90 282|89 017|-1,4%

La taille du dictionnaire est "divisée par deux" et il en est de même pour les autres variantes.

### Le script de téléchargement et de modification des dictionnaires

```bash
VERSION="6.4.1"
ZIP="hunspell-french-dictionaries-v""$VERSION"".zip"
URL="https://grammalecte.net/download/fr/""$ZIP"
# wget -N $URL
unzip -o $ZIP -d $VERSION
cd $VERSION
dictionnaires=`ls *.dic`
for dictionnaire in $dictionnaires
do  # enlève les champs optionnels et les homonymes
    echo $dictionnaire
    mv $dictionnaire "tmp-""$dictionnaire"
    sed  -i 's/ .*$//' "tmp-""$dictionnaire"
    cat "tmp-""$dictionnaire" | uniq > $dictionnaire
    rm "tmp-""$dictionnaire"
    nombre_lignes=`wc -l < $dictionnaire`
    sed -i "1s/.*/$nombre_lignes/" "$dictionnaire"
done
affixes=`ls *.aff`
for affixe in $affixes
do  # enlève les commentaires et lignes vides
    echo $affixe    
    sed -i -r '/^(\s*|#.*)$/d' "$affixe"
    # enlève les champs optionnels
    sed  -i 's/ ..:.*$//' "$affixe"
done
cd ..
```

## Références
* [Alice - Spell Checking: Super alpha as-you-type spell checker for Adobe Brackets](https://github.com/JohnathonKoster/brackets-spellcheck)
> Linguistics adds a very powerful spell checking system to the Brackets editor. It offers the following features right out of the box:
> * Off-line spell checking :white_check_mark:
> * Support for multiple dictionaries :white_check_mark: 
> * Custom user dictionary profiles
> * Integrated user interface
> *Intelligent built in support for common file extensions
> * Intelligent support for uppercased words
> * Spell check within camelCasedWords
> * Spell check available within your code files
* [Grammalecte.dic(fr)](https://grammalecte.net/home.php?prj=fr)
> :book: :fr: Le projet vise à améliorer les dictionnaires orthographiques françaispour les logiciels libres, comme LibreOffice, OpenOffice, Firefox, Thunderbird, Evolution, Pidgin, Notepad++, Eclipse, etc., ainsi que tous les logiciels utilisant le correcteur orthographique Hunspell.
* [Forum Grammalecte](https://grammalecte.net/thread.php?prj=fr&t=827)
> Hunspell prévoit la possibilité d'adjoindre à chaque entrée des champs optionnels concernant la nature grammaticale, les flexions et autres informations potentiellement utiles.
Le dictionnaire français est l'un des rares à utiliser cette possibilité. Il y a aussi le dictionnaire hongrois et peut-être quelques autres (le russe ? le portugais ? je ne suis pas très sûr...)
* [ :male_detective: Typo.js: A client-side JavaScript spellchecker that uses Hunspell-style dictionaries](https://github.com/cfinke/Typo.js/)
* [ :male_detective: Hunspell > Manual pages](https://github.com/hunspell/hunspell/releases)
