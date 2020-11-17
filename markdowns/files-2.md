# Des fichiers dans le simulateur

La compilation vers la calculatrice, puis l'upload du programme, prennent un peu de temps. Pour tester plus simplement et rapidement notre programme nous allons donc coder une solution qui liste les fichiers de l'ordinateur.

## La compilation conditionnelle

Nous allons coder une fonction `filesWithExtension` spécifique pour le simulateur. Pour cela nous allons utiliser les conditions du précompilateur qu'on a déjà rencontré pour les "include guard".

Quand le compilateur compile pour la calculatrice, la variable `DEVICE` est définie, quand on compile pour le simulateur, elle ne l'est pas. Nous allons utiliser cette information comme cela :\
```c++
#ifdef DEVICE
    // code spécifique à la calculatrice
#else
    // code spécifique au simulateur
#endif
```

Rajoutons ce bloc à notre fichier `utility.cpp` et déplaçons notre fonction `filesWithExtension` dans le bloc dédié à la calculatrice :
```c++
#include "utility.h"
#include <string.h>

namespace reader
{

bool stringEndsWith(const char* str, const char* pattern)
{
    int strLength = strlen(str);
    int patternLength = strlen(pattern);
    if (patternLength > strLength)
        return false;

    const char* strIter = str + strlen(str);
    const char* patternIter = pattern + strlen(pattern);

    while(*strIter == *patternIter)
    {
        if(patternIter == pattern)
            return true;
        strIter--;
        patternIter--;
    }
    return false;
}


#ifdef DEVICE

int filesWithExtension(const char* extension, External::Archive::File* files, int filesSize) 
{
    size_t nbTotalFiles = External::Archive::numberOfFiles();
    int nbFiles = 0;
    for(size_t i=0; i < nbTotalFiles; ++i)
    {
        External::Archive::File file;
        External::Archive::fileAtIndex(i, file);
        if(stringEndsWith(file.name, ".txt"))
        {
            files[nbFiles] = file;
            nbFiles++;
            if(nbFiles == filesSize)
                break;
        }
    }
    return nbFiles;
}

#else

#endif

}
```

## Parcourir un répertoire sur l'ordinateur

Quand nous travaillons sur le simulateur, nous avons accès aux librairies standards du C/C++. Nous allons donc les utiliser pour parcourir notre répertoire. Rajoutons quelques includes qui nous y donneront accès dans un bloc dédié au simulateur en haut du fichier (avec les autres includes). Nous utilisons la condition `#ifndef` plutôt que `#ifdef` ce coup ci, qui veut dire "si la variable n'est pas définie".
```C++
#ifndef DEVICE
#include <stdio.h>
#include <dirent.h> 
#include <sys/stat.h>
#endif 
```

La fonction `opendir()` permet d'ouvrir un répertoire, elle prend en paramètre son path et renvoie une structure `DIR`. Pour parcourir le répertoire, il faut appeler `readdir()` répétitivement. Cette fonction renvoie une `dirent` (une structure signifiant probablement "directory entry") pour chaque élément du répertoire (fichier ou sous répertoire). Quand elle renvoie `NULL` c'est qu'on est arrivé au bout. Quand on a finit avec le répertoire, c'est mieux de le fermer avec `closedir()`. On va donc utiliser ce jeu de fonction pour lister les fichiers du répertoire courant (celui duquel on aura lancé le simulateur).

```c++
int filesWithExtension(const char* extension, External::Archive::File* files, int filesSize) 
{
  dirent *file;
  DIR *d = opendir(".");
  int nb = 0;
  if (d) 
  {
    while ((file = readdir(d)) != NULL) 
    {
      if(stringEndsWith(dir->d_name, extension))
      {
        files[nb].name = strdup(file->d_name);//will probably leak
        nb++;
        if(nb == filesSize)
            break;
      }
    }
    closedir(d);
  }
  return nb;
}
```

Vous avez normalement presque toutes les cartes en main pour comprendre ce code. L'appel à `strdup()` mérite probablement quelques explications. La structure `External::Archive::File` stock le nom du fichier dans son membre `name`, mais `name` n'est pas un tableau mais un pointeur, cela signifie que le `name` pointe vers une chaîne de caractère qui doit "exister" aussi longtemps que l'instance de `External::Archive::File` existera. 

Sur la calculatrice, cela ne pose pas de problème, le nom du fichier se trouve dans une zone de la mémoire qui ne change que lorsqu'on upload une nouvelle archive TAR, il n'y a donc pas de danger à ce que `name` pointe cette zone de mémoire, elle sera toujours là. 

Sur l'ordinateur, c'est autre chose. Le nom du fichier est dans la structure `dirent` qui est "tenue" par la structure `DIR`. Ce "tenue" signifie que lorsque la structure `DIR` est supprimée, la structure `dirent` l'est aussi et sa mémoire est recyclée, on ne peut donc pas continuer à pointer dessus (en pratique rien ne l'empêche, mais le nom du fichier risque de changer).

La fonction `strdup()` permet de faire une copie d'une chaîne de caractère, nous dupliquons donc le nom depuis `dirent` pour en avoir une version qui sera tenue par notre structure `External::Archive::File`. Cela va fonctionner, mais ce n'est néanmoins pas propre car normalement quand la structure `External::Archive::File` est supprimée il faudrait libérer la mémoire occuper par la chaîne que nous avons dupliquée. Nous ne le ferons pas car pour notre usage quand la structure `External::Archive::File` est supprimée, c'est que nous quittons le simulateur.

## On essaie

Je vous laisse retrouver la commande pour builder sur votre plateforme. N'oubliez pas de créer quelques fichiers .txt dans le répertoire duquel vous lancez le simulateur.

