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

### Parcourir un répertoire sur l'ordinateur

Quand nous travaillons sur le simulateur, nous avons accès aux librairies standards du C/C++. Nous allons donc les utiliser pour parcourir notre répertoire. Rajoutons quelques includes qui nous y donneront accès dans un bloc dédié au simulateur en haut du fichier (avec les autres includes). Nous utilisons la condition `#ifndef` plutôt que `#ifdef` ce coup ci, qui veut dire "si la variable n'est pas définie".
```C++
#ifndef DEVICE
#include <stdio.h>
#include <dirent.h> 
#include <sys/stat.h>
#endif 
```

La fonction `opendir` permet d'ouvrir un répertoire, elle prend en paramètre son path et renvoie une structure `DIR`