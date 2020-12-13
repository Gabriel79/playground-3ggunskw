# Introduction

#### A qui s'adresse ce tuto ?

Ce tuto s'adresse à des lycéens ayant acquis une calculatrice Numworks (plutôt la version n0110 qui dispose de plus de mémoire que la version n0100) et qui souhaitent coder leur propre application. Je vais donc donner quelques détails sur les outils, la programmation,... qui pourraient sembler assez inutiles à des développeurs un peu plus agés.

Quelques connaissances de programmation dans un langage comme le python ou le java faciliteront certainement la compréhension de ces quelques pages. J'essayerai d'expliquer un peu les concepts de programmation orientée objet et les spécificités du C/C++ au fil de l'eau, mais si vous voulez vraiment apprendre ce langage, il vous faudra plus que ce tuto.

Plutôt que d'expliquer le code final de l'application, je vais détailler le processus de création de l'application, en progressant petit à petit vers ma solution (qui n'est probablement ni la seule, ni la meilleure).

#### La Numworks

La Numworks se programme en C++, mais ne supporte pas toutes les librairies C/C++ qu'on a l'habitude de rencontrer, ce qui ne rend pas forcément le développement très facile à un développeur C++, mais cela simplifiera les choses pour un débutant.

Il faut avoir en tête que la Numworks est une machine un peu "light" en termes de mémoire et de vitesse de calcul par rapport à un ordinateur. Elle ne dispose pas d'un disque dur ni d'un système de fichier. On peut cependant uploader des fichiers dans une zone particulière de la mémoire ce qui permettra à notre programme d'y accéder très simplement.

#### L'application

Nous allons construire ensemble une liseuse de fichiers texte, avec l'idée de pouvoir lire des ebooks sur la calculatrice, logiciel qui n'est pas présent sur la Numworks aujourd'hui. Cette petite application devrait vous fera aborder assez de points pour vous permettre de développer vos propres applications. 

Nous allons réaliser 2 écrans, l'un utilisant une "vue" standard pour lister les livres disponibles sur la calculatrice, et une "vue" faite à la main pour afficher du texte à l'écran en codant manuellement le "word wrapping" et la scrollbar.

Nous allons également voir comment sauver des informations dans la mémoire de la calculatrice pour nous souvenir du dernier endroit lu dans chaque fichier texte.

#### Votre avis

Si vous êtes bloqués quelque part, ou si des points méritent plus d'explications, n'hésitez pas à poster un commentaire !

Je sais que lorsqu'on donne un cours ou une présentation et qu'il n'y a pas de questions, c'est en général que les gens n'ont rien compris... Donc si je suis à 1000 lieux de la cible, dites moi, ou si vous arrivez à faire tourner votre propre app sur votre Numworks grâce à ce tuto, n'hésitez pas à "liker" !

#### Moi

Pour information, je suis développeur C++, mais n'ai aucun lien avec Numworks. Le code de la Numworks est public, et a fait l'objet de quelques forks (projets dérivés) dont [Omega](https://github.com/Omega-Numworks/Omega) duquel je vais partir. Le code bien que public, est assez peu documenté, ce qui rend la courbe d'apprentissage très raide. Ce que je vais vous présenter, je l'ai déduit en étudiant le code des applications fournies avec la calculatrice, il se peut donc que je n'ai pas tout compris et que je raconte absolument n'importe quoi (dans ce cas, postez un commentaire!).

En développant cette application, je me suis heurté à quelques difficultés qui m'ont coûté 80% du temps total. Vous en rencontrerez aussi, il suffit d'une petite faute de frappe, ou d'un petit oubli pour perdre des heures... la programmation est ainsi, ce sont les erreurs qui font progresser.

Bon courage !

