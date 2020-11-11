# Environnement

Pour l'instant je ne vais pas détailler les détails de la configuration d'un environnement de développement.

Je recommande, de développer sous macos ou linux qui fournissent assez simplement une chaine de compilation permettant de compiler le code pour la Numworks.

Un IDE relativement simple et complet pour un débutant est Visual Studio Code, mais pour ce projet un bloc note est un terminal font l'affaire.

Une fois l'environnement de développement préparé, il vous faut cloner Omega sur votre ordinnateur. Pour cela executez la commande suivante dans un terminal:
`$ git clone --recursive https://github.com/Omega-Numworks/Omega.git`

Je vous conseille de compiler une première fois le repository pour vous assurer que votre chaine de compilation fonctionne.

Pour compiler pour le simulateur de votre plateforme, allez dans le répertoire de votre repository:
`$ cd Omega`
puis sous macos:
`$ make PLATFORME=simulator TARGET=macos MODEL="n0110"`
ou sous linux:
`$ make PLATFORME=simulator MODEL="n0110"`

>ATTENTION : plateforme en anglais ne prend pas de E entre le T et le F (cette découverte m'a coûté une heure)

Vous pouvez ensuite lancer le simulateur:
`$ output/release/simulator/macos/epsilon.app/Contents/MacOS/Epsilon`

Pour compiler pour la calculatrice vous trouvez toutes les informations sur [Omega](https://github.com/Omega-Numworks/Omega)
