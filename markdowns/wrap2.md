# Word wrapping TextView - partie 2

Nous avons l'architecture pour notre `WordWrapView`, il est temps de nous intéresser à l'algorithme qui va réaliser l'affichage. Le principe est le suivant: nous allons lire le texte mot par mot en coupant sur les sauts de ligne et les espaces. Avant d'écrire le mot à l'écran, nous allons nous assurer qu'il y a la place de le mettre en comparant ses dimensions à l'espace restant. S'il n'y a pas la place, nous retournerons à la ligne, et si cette ligne est hors de l'écran, on s'arrêtera. C'est parti ?

