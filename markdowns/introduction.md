# Introduction

#### A qui s'adresse ce tuto?
Ce tuto s'adresse à des lycéens ayant acquis une calculatrice numworks (plutôt la version n0110 qui dispose de plus de mémoire que la version n0100) et qui souhaitent coder leur propre application.

Quelques connaissances de programmation dans un langage comme le python ou le java faciliteront certainement la compréhension de ces quelques pages. J'essayerais d'expliquer un peu les spécificités du C/C++ au fil de l'eau, mais si vous voulez vraiment apprendre ce langage, il vous faudra plus que ce tuto.

#### La Numworks

La Numworks se programme en C++, mais ne supporte pas toutes les librairies C/C++ qu'on a l'habitude de rencontrer, ce qui ne rend pas forcément le developpement très facile à un developpeur C++, mais cela simplifiera les choses pour un developpeutr débutant.

Il faut avoir en tête que la Numworks est une machine un peu "light" en terme de mémoire et vitesse de calcul par rapport à un ordinateur. Elle ne dispose pas d'un disque dur ni d'un système de fichier. On peut cependant uploader des fichiers dans une zone particulière de la mémoire ce qui permettra à notre programme d'y accéder très simplement.

#### L'application

Le but de l'application que nous allons construire ensemble est de réaliser une liseuse de fichier texte, logiciel qui n'est pas présent sur la numworks aujourd'hui. Cette petite application devrait vous permettre d'aborder assez de points pour vous permettre de développer votre propre application. 

Nous allons réaliser 2 écrans, l'un utilisant une "vue" standard pour lister les livres disponible sur la calculatrice, et une "vue" faite à la main pour afficher du texte à l'écran en codant manuellement le "word wrapping" et la scrollbar.

Nous allons également voir comment sauver des informations dans la mémoire de la calculatrice pour nous souvenir du dernier endroit lu dans chaque libre.

#### Moi

Pour information, je suis développeur C++, mais n'ai aucun lien avec Numworks. Le code de la numworks est public, et a fait l'objet de quelques forks (projets dérivés) dont [Omega](https://github.com/Omega-Numworks/Omega) duquel je vais partir. Le code bien que public est assez peu documenté, et Numworks ne fourni pas de SDK comme on peut trouver sur d'autres plateformes, ce qui rend la courbe d'apprentissage très raide. Ce que je vais vous présenter, je l'ai compris en étudiant le code des applications fournies avec la calculatrice, il se peut donc que je n'ai pas tout compris et que je raconte absolument n'importe quoi.

En developpant cette application, je me suis heurté à quelques difficultés qui m'ont coûté 80% du temps total. Vous en rencontrerez aussi, il suffit d'une petite faute de frappe, ou d'un petit oubli pour perdre des heures... la programmation est ainsi, quelque soit la plateforme ou le langage. Soyez patient et ne lachez pas l'affaire, ceux sont les erreurs qui font progresser.

Bon courage!

