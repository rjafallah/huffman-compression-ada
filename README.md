# Codage de Huffman en Ada

Implémentation complète de la **compression sans perte de Huffman** en Ada : construction de l'arbre de Huffman, génération des codes, compression, puis décompression avec reconstruction de l'arbre à partir du fichier compressé.

Projet réalisé en équipe dans le cadre du cours de Programmation impérative (ENSEEIHT, Toulouse).

---

## Principe

L'algorithme de Huffman attribue des codes **courts aux symboles fréquents** et des codes **longs aux symboles rares**, ce qui réduit la taille totale sans aucune perte d'information.

Le compresseur travaille sur un **fichier binaire** (suite de `0` et `1`, 8 bits par caractère) pour être indépendant du type de données. Deux petits convertisseurs permettent donc de passer d'un texte à sa représentation binaire et inversement.

```
 texte.txt
    │  text_to_binary
    ▼
 texteBINARY.txt
    │  compresser                       ──▶ affiche l'arbre et les codes de Huffman
    ▼
 texteBINARY.txt.hff        (fichier compressé : en-tête + données)
    │  decompresser
    ▼
 texteBINARY.txt.hff.d      (binaire décompressé, identique à texteBINARY.txt)
    │  binary_to_text
    ▼
 texteTEXT.txt              (texte d'origine retrouvé)
```

## Algorithmes et structures de données

| Étape | Méthode |
|-------|---------|
| Comptage des symboles | Table de hachage (`TH`) : clé = code ASCII binaire du symbole, valeur = nombre d'occurrences |
| Symbole de fin | Symbole spécial `$` (0 occurrence) ajouté pour marquer la fin du fichier |
| Construction de l'arbre | File de nœuds triée : on fusionne itérativement les deux nœuds de plus faible poids |
| Génération des codes | Parcours récursif de l'arbre (gauche = `0`, droite = `1`), stockés dans une 2ᵉ table de hachage (`THCharCode`) |
| En-tête du fichier compressé | Symboles dans l'ordre du **parcours infixe** de l'arbre + structure de l'arbre |
| Décompression | Lecture de l'en-tête, reconstruction de l'arbre, puis décodage bit à bit |

## Structure du projet

```
.
├── src/
│   ├── compresser.adb / compression.ads / compression.adb         # Compression
│   ├── decompresser.adb / decompression.ads / decompression.adb   # Décompression
│   ├── th.ads / th.adb                  # Table de hachage (symbole → occurrences)
│   ├── thcharcode.ads / thcharcode.adb  # Table de hachage (symbole → code de Huffman)
│   ├── text_to_binary.adb               # Convertisseur texte → binaire
│   └── binary_to_text.adb               # Convertisseur binaire → texte
├── tests/                               # Fichiers d'exemple
├── raffinages.pdf                       # Conception par raffinages successifs
└── rapport.pdf                          # Rapport complet du projet
```

## Prérequis

- Compilateur **GNAT** (GNU Ada)
  - Ubuntu / Debian : `sudo apt install gnat`
  - Windows : [GNAT Community / Alire](https://alire.ada.dev/)

## Compilation

```bash
cd src
gnatmake -gnatwa compresser.adb
gnatmake -gnatwa decompresser.adb
gnatmake -gnatwa text_to_binary.adb
gnatmake -gnatwa binary_to_text.adb
```

## Utilisation

Exemple complet, depuis le dossier `src/` avec un fichier `exemple.txt` :

```bash
./text_to_binary  exemple.txt                 # → exempleBINARY.txt
./compresser      exempleBINARY.txt           # → exempleBINARY.txt.hff
./decompresser    exempleBINARY.txt.hff       # → exempleBINARY.txt.hff.d
./binary_to_text  exempleBINARY.txt.hff.d     # → exempleTEXT.txt
```

Vérifier que la décompression est sans perte :

```bash
cmp exemple.txt exempleTEXT.txt && echo "Aller-retour OK"
```

> Les fichiers d'entrée doivent avoir l'extension `.txt`. Les fichiers de sortie sont créés à côté du fichier d'entrée. Le programme `compresser` affiche l'arbre de Huffman et la table des codes obtenue.

## Exemple de résultat

Sur `tests/binaryInput3TEXT.txt` (texte répétitif de 3 361 caractères) :

| Fichier | Taille (caractères `0`/`1`) |
|---------|-----------------------------|
| Binaire d'origine | 26 881 |
| Compressé (`.hff`, en-tête inclus) | 12 644 |

Soit une réduction d'environ **53 %**, avec un texte retrouvé strictement identique après décompression. Sur de très petits fichiers, l'en-tête (symboles + arbre) peut dépasser le gain : la compression de Huffman devient intéressante dès que le texte est assez long ou répétitif.

## Limites et pistes d'amélioration

- Les fichiers `.hff` stockent les bits sous forme de **caractères `0` et `1`** (pédagogique, facile à inspecter) : pour un gain réel sur disque, il faudrait écrire les bits regroupés par octets.
- Le tri de la file de nœuds est un tri simple (quadratique) ; une file de priorité (tas) serait plus efficace.
- Pas de gestion des fichiers vides ou de très grande taille.

## Technologies

Ada (types accès, tables de hachage, récursivité, programmation par contrat avec `Pre` / `Post`), GNAT.
