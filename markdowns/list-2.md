# La liste des livres - deuxième partie

Notre application s'ouvre ! C'est un bon début. De façon générale, je vous recommande quand vous programmez de progresser par petites étapes en vérifiant à chaque fois que ce que vous avez fait marche, plutôt que de tout coder pour ne tester que tout à la fin. Si la marche est trop haute, vous risquez de perdre beaucoup de temps pour l'atteindre. Nous allons donc procéder ainsi. La prochaine étape va être d'afficher quelque chose dans notre `TableView`. Nous n'allons pas tout de suite essayer de lister les fichiers présents sur la calculatrice, si ca ne marchait pas nous ne saurions pas si ça ne marche pas car notre `TableView` ne marche pas ou si c'est la récupération des fichiers qui ne marche pas. 

## Stockage de la liste des fichiers

On va donc imaginer avoir réussi à lister les fichiers présents. Sur une plateforme plus "standard" que la Numworks, on aurait probablement listé les fichiers et remplit un `std::vector`, un tableau dont la taille peu augmenter au besoin. Malheureusement, la Numworks ne supporte pas l'allocation dynamique de mémoire (le fait de réclamer un peu plus de place pour notre tableau pendant l'exécution du programme). Il faut que la taille de notre tableau soit fixée à la compilation. Nous allons donc nous contentez de lister les 20 premiers fichiers textes, on est loin du Kindle mais le Kindle ne résout pas encore des équations.

La Numworks ne fournit pas de système de fichiers comme on peut en trouver sur un ordinateur. Les fonctions standard du C pour parcourir un système de fichier n'existe donc pas. Omega fourni une solution que nous étudierons plus tard. L'important pour l'instant est la structure `File` qui permet de décrire un fichier. Une structure est une sorte de classe dont tous les membres sont public. En générale les structures ne contiennent pas de méthodes, juste des attributs que l'on peut lire et écrire directement. On utilise une structure pour grouper ensemble des données. La structure `File` contient ainsi le nom du fichier, sa taille et son adresse dans la mémoire.

Nous allons stocker la description des fichiers trouvés sur la Numworks (les `File`) dans un tableau de 20 cases, que nous allons rajouter comme membre de notre `ListBookController`:
```C++



