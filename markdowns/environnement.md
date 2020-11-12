# Environnement

L'installation d'un environnement de developpement est souvent une étape un peu longue. Pour développer pour la numworks il vous faudra installer quelques pré-requis.

La première étape va être d'installer les outils de compilations. Pour cela suivre les indications de l'étape 1 (Installation du SDK) sur [Numworks](https://www.numworks.com/resources/engineering/software/build/)

Un IDE relativement simple et complet pour un débutant est [Visual Studio Code](https://code.visualstudio.com/), on le trouve sous linux, mac et windows, mais pour ce projet un bloc note est un terminal font l'affaire.

Une fois l'environnement de développement préparé, il vous faut cloner Omega sur votre ordinnateur. Pour cela executez la commande suivante dans un terminal:
`$ git clone --recursive https://github.com/Omega-Numworks/Omega.git`

Je vous conseille de compiler une première fois le repository pour vous assurer que votre chaine de compilation fonctionne.

Pour compiler pour le simulateur de votre plateforme, allez dans le répertoire de votre repository:
`$ cd Omega`
puis sous macos:
`$ make PLATFORME=simulator TARGET=macos MODEL="n0110"`
sous windows
`$ make PLATFORME=simulator TARGET=windows MODEL="n0110"`
ou sous linux:
`$ make PLATFORME=simulator MODEL="n0110"`

>ATTENTION : plateforme en anglais ne prend pas de E entre le T et le F (cette découverte m'a coûté une heure)

Vous pouvez ensuite lancer le simulateur:
sous macos
`$ output/release/simulator/macos/epsilon.app/Contents/MacOS/Epsilon`
sous windows


Pour compiler pour la calculatrice vous trouvez toutes les informations sur [Omega](https://github.com/Omega-Numworks/Omega)

Si vous arrivez à compiler puis lancer le simulateur, on va pouvoir passer à la suite!
