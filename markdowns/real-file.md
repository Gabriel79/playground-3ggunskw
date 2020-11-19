# Un vrai fichier

## Dans le ReadBookController

Nous avons pu vérifier que notre vue affichant du texte s'ouvrait. On va maintenant essayer d'afficher le véritable contenu de notre fichier. Sur la calculatrice c'est très simple, le contenu du fichier se trouve déjà dans la structure `External::Archive::File` dans `data`, mais sur le simulateur ce n'est pas le cas. Rajoutons tout de suite le code dans le `ReadBonController` pour passer le contenu du fichier à la `PointerTextView`:
```c++
void ReadBookController::setBook(const External::Archive::File& file)
{
    m_readerView.setText(reinterpret_cast<const char*>(file.data));
}
```
Notez, que le ficher n'est pas forcément un fichier texte, le pointeur `data` pointe vers un tableau de `uint8_t` (des `int` positifs sur 8 bits). Il faut dire au compilateur que ces données sont à interpréter comme du texte. Pour caster un pointeur d'un type à un autre qui n'ont pas de relation d'héritage entre eux, nous utilisons un `reinterpret_cast`.

C'est tout ce qu'il faut pour tester sur la calculatrice.

## Lire un fichier dans le simulateur

On pourrait lire le fichier au moment de son ouverture dans le `ReadBookController` mais il faudrait "polluer" le code du contrôleur avec un code spécifique au simulateur. Il y aurait bien des solutions propres mais elles imposeraient de modifier le code d'Omega, ce que je préfère éviter. Nous allons donc lire les fichiers dès que nous les trouvons à l'ouverture du `ListBookController`, au niveau de la gestion de la mémoire c'est vraiment pas terrible, mais à priori le but du simulateur est plus de servir d'outil de développement qu'autre chose.

Nous allons donc rajouter une fonction `fillFileData` dans notre fichier `apps\reader\utility.cpp`. Cette fonction est dite `static` cela signifie (pour une fonction, et non pour une méthode d'une classe) qu'elle ne sera appelable que depuis le fichier dans lequel on se trouve, et uniquement par des fonctions situées après dans le fichier. Nous rajoutons donc cette fonction dans le bloc dédié aux fonctions du simulateur, juste avant `filesWithExtension` qui y fera appel.

```c++
static void fillFileData(External::Archive::File& file)
{
```
Pour éviter la copie, on passe le paramètre par référence.

On commence par initialiser le `file` avec des 0 au cas où quelque chose se passait mal, notre `file` ne contiendrait pas n'importe quoi. En C/C++, si un constructeur n'initialise pas un objet ou une variable, la mémoire est laissée non-initialisée.
```c++
    file.data = nullptr;
    file.dataLength = 0;        
```

Nous allons commencer par lire les informations sur le fichier, notamment sa taille, pour savoir quelle quantité de mémoire il va être nécessaire de réserver pour stocker le contenu du fichier. La fonction `stat()` permet de remplir une structure `stat` avec ces informations. Si une erreur se produisait, la fonction `stat()` renverrait un nombre différent de 0, on sortirait alors de notre fonction sans lire le fichier. Notez qu'on est obligé de déclarer `info` comme un `struct stat` et non juste comme un `stat` pour que le compilateur ne mélange pas la structure `stat` et la fonction `stat`.
```c++
    struct stat info;
    if (stat(file.name, &info) != 0) 
    {
        return;
    }   
```
Nous avons maintenant la taille du fichier en octet dans `info.st_size`. Nous allons réserver un tableau de caractère de la taille du fichier pour stocker son contenu. Pour réserver de la mémoire pour un tableau, on utilise le mot clé `new` et on spécifie la taille du tableau entre `[ ]`. Si l'allocation de mémoire échouait (des fois que la taille de votre fichier soit trop grande...), on sortirait de notre fonction.
```c++
    unsigned char* content = new unsigned char[info.st_size];
    if (content == NULL) 
    {
        return ;
    }   
```

Nous ouvrons ensuite le fichier en lecture binaire "rb" (on va le lire d'un coup, et non ligne par ligne comme on ferait avec un fichier texte). Si l'ouverture échoue, on sort de la fonction.
```c++       
    FILE *fp = fopen(file.name, "rb");
    if (fp == NULL) 
    {
        return ;
    }
```
Pour finir, on lit le contenu du fichier en un seul appel et on stocke le contenu du fichier dans `content`. Notez que `content` est un pointeur vers la mémoire allouée par le `new`. On copie donc ce pointeur dans notre structure `External::Archive::File`. On n'oublie pas de fermer le fichier avant de sortir de la fonction.
```c++        
    fread(content, info.st_size, 1, fp);        
    fclose(fp);
    file.data = content;
    file.dataLength = info.st_size;        
}    
```

Voilà la fonction en un seul bloc :
```c++
static void fillFileData(External::Archive::File& file)
{    
    file.data = nullptr;
    file.dataLength = 0;   

    struct stat info;
    if (stat(file.name, &info) != 0) 
    {
        return;
    }   
    
    unsigned char* content = new unsigned char[info.st_size];
    if (content == NULL) 
    {
        return ;
    }   
    FILE *fp = fopen(file.name, "rb");
    if (fp == NULL) 
    {
        return ;
    }
    
    fread(content, info.st_size, 1, fp);        
    fclose(fp);
    file.data = content;
    file.dataLength = info.st_size;            
}
```

Nous allons appeler cette fonction juste après avoir trouvé notre fichier dans la fonction `filesWithExtension`
```c++
...
    if(stringEndsWith(file->d_name, extension))
    {
      files[nb].name = strdup(file->d_name);//will probably leak
      fillFileData(files[nb]);
...
```

et voilà... on compile, et on essait. Normalement, ça marche.

## Normalisation des fichiers textes

Si votre fichier texte contient des caractères accentués, vous avez peut-être constaté que ces caractères ne s'affichent pas. En effet, la Numworks ne supporte qu'un jeu limité de caractères. En général, sur ordinateur, les caractères accentués sont encodés avec un code particulier pour chaque caractère. Sur la Numworks, les caractères accentués sont recomposés en combinant le caractère non accentué et son accent. Il faut donc convertir les fichiers textes d'un encodage vers l'autre. 

Nous allons coder un petit script en python qui va nous permettre de normaliser nos fichiers texte. Alors, je ne suis pas développeur python, donc ce n'est peut-être pas le code python le plus élégant, mais il semble faire le job. Le script prend en paramètre le nom du fichier texte à normaliser, il va écrire le fichier normalisé dans un fichier temporaire puis renommer ce fichier temporaire avec le nom initial. Le fichier initial va donc être modifié. Dès fois que quelque chose se passe mal faîtes en donc une copie avant d'exécuter le script!

```python
import sys
import unicodedata
import argparse
import io
import shutil

filename = sys.argv[1]

print("Normalization of "+filename)

output = open(filename+".tmp", "wb")

with io.open(filename, "r", encoding='utf-8') as file:    
    for line in file:
        unicodeLine = unicodedata.normalize("NFKD", line)        
        output.write(unicodeLine.encode("UTF-8"))        
output.close()
shutil.move(filename+".tmp",filename)
```
Le script est assez simple, il parcourt le fichier d'entrée ligne par ligne, normalise la ligne avec une fonction de `unicodedata` et ré-écrit la ligne.

Pour exécuter ce script, il vous faut python sur votre ordinateur et lancer :\
`$ python normalize.py fichier.txt`


