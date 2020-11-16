# Lister les fichiers

Nous arrivons maintenant à afficher une liste de fichiers dans une `TableView`. Il nous reste à accéder réellement aux fichiers de la Numworks. Souvenons nous que nous ne voulons pas afficher n'importe quels fichiers, uniquement les fichiers .txt.

Omega fournit une API (Application Programming Interface, un ensemble d'objet et fonctions) pour accéder à des fichiers stockés dans une archive TAR dans une zone libre de la mémoire de la Numworks. Nous n'avons pas réellement besoin de savoir comment ce stockage fonctionne. L'API dans `apps/external/archive.h` fournit 2 fonctions qui vont nous suffire:
```c++
size_t numberOfFiles();
bool fileAtIndex(size_t index, File &entry);
```

Nous allons utiliser ces fonctions pour parcourir tous les fichiers et filtrer ceux dont le nom finit par ".txt". Malheureusement, ni Omega ni Numworks ne fournissent de fonction pour filtrer les fichiers selon leur extension, ou je n'ai pas trouvé...

Nous allons coder nous même cette fonction que nous appellerons `stringEndsWith`. L'algorithme est assez simple, nous allons partir de la fin de l'extension et de la fin du nom du fichier et comparer les caractères en descendant vers le début de l'extension. Si les 2 caractères sont différents alors la fonction renvoie `false` sinon on continue jusqu'au début de l'extension.

Voici la signature de la fonction :
```c++
bool stringEndsWith(const char* str, const char* pattern)
{
```

### Les chaînes de caractères

En C++, les chaînes de caractères sont représentés par des instances de la classe `std::string`, malheureusement cette classe n'est pas disponible sur la Numworks. Nous devons donc utiliser la représentation C. En C, une chaîne de caractère est stockée dans un tableau de `char` terminé par un 0 (la valeur 0, par le caractère '0'). En pratique, quand on déclare une chaîne de caractères, on utilise rarement un tableau (`char chaine[NB]`) mais un pointeur vers le premier caractère `char*`. Notre fonction ne devant pas modifier les chaînes de caractères qu'elle manipule, on s'en assure en les préfixant de `const`. Si on essayait de modifier un caractère, le compilateur produirait une erreur. C'est le contenu de ce qui est pointé qui ne peut pas être modifié, le pointeur, lui peut pointer vers autre chose si on le souhaite. 

Nous allons tout d'abord calculer la longueur de chacune des chaînes de caractères