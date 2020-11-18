# Environnement

#### Les outils

L'installation d'un environnement de développement est souvent une étape un peu longue. Pour développer pour la Numworks il vous faudra installer quelques pré-requis.

La première étape va être d'installer les outils de compilations. Pour cela suivre les indications de l'étape 1 (Installation du SDK) sur [Numworks](https://www.numworks.com/resources/engineering/software/build/)

Sous windows, cela n'a pas été aussi simple qu'espéré pour moi. Il faut d'abord installer [MSys](https://www.msys2.org/), pour cela j'ai dû désactiver mon antivirus et activer juste Windows Defender. MSys fournit une ligne de commande comme sous linux ou macos. A partir de là il faut installer les packages dont à besoin Numworks:\
`pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-freetype mingw-w64-x86_64-pkg-config mingw-w64-x86_64-libusb git make python`

Puis en prenant bien garde d'être dans votre répertoire "home" de MSys, normalement MSys s'ouvre directement à cet endroit, mais si vous vous êtes baladé depuis l'invite de commande, il convient d'y retourner (pour vous y rendre la commande est juste `cd`), il faut rajouter une ligne à un fichier de configuration qui modifiera votre PATH à chaque fois que vous lancerez MSys:\
`echo "export PATH=/mingw64/bin:$PATH" >> .bashrc`\
Pour information, le PATH est l'ensemble des répertoires où l'ordinateur cherchera vos programmes quand vous tapez une commande.

Ensuite, fermer et relancer MSys.

En général, on aime coder dans un editeur de texte un peu enrichit, un IDE (Integrated Development Environment). [Visual Studio Code](https://code.visualstudio.com/) est assez simple et complet, on le trouve sous linux, mac et windows, mais pour ce projet un bloc note et un terminal font l'affaire.

Une petite astuce pour ouvrir un terminal MSys dans Visual Studio Code, c'est de rajouter les 2 lignes suivantes dans le fichier de préférences (settings.json)
```json
    "terminal.integrated.shell.windows": "C:\\msys64\\usr\\bin\\bash.exe",
    "terminal.integrated.shellArgs.windows": ["--login", "-i"]
```

#### Le code

Une fois l'environnement de développement préparé, il vous faut cloner Omega sur votre ordinateur. Pour cela,  dans un terminal (MSys sous windows), exécutez la commande suivante :\
`$ git clone --recursive https://github.com/Omega-Numworks/Omega.git`\
Ne pas oublier le `--recursive` !

Cela créera un répertoire `Omega` dans le répertoire où vous vous trouvez, le code source sera dedans, et c'est dedans que nous allons travailler. Si vous souhaitez créer votre espace de travail ailleurs que dans le répertoire qui s'ouvre au lancement de votre terminal, les commandes pour vous déplacer dans l'arborescence en utilisant la ligne de commandes sont :\
`cd UnRepertoire` pour aller dans le répertoire `UnRepetoire` \
`cd ..` pour revenir au répertoire parent\
`mkDir Repertoire` pour créer un répertoire.\
Sous MSys, pour changer de disque, c'est par exemple `$ cd /d/`


#### La compilation

Je vous conseille de compiler une première fois le repository (l'ensemble du code que vous venez de télécharger) pour vous assurer que votre chaîne de compilation fonctionne.

Compiler consiste à transformer le code source d'une application en instructions binaires exécutable par un ordinateur.

Pour compiler pour le simulateur de votre plateforme, allez dans le répertoire de votre repository:\
`$ cd Omega`

puis sous macos:\
`$ make PLATFORM=simulator TARGET=macos MODEL="n0110"`

sous windows:\
`$ make PLATFORM=simulator TARGET=windows MODEL="n0110"`

ou sous linux:\
`$ make PLATFORM=simulator MODEL="n0110"`

>ATTENTION : plateforme en anglais ne prend pas de E entre le T et le F, ni à la fin ! (cette découverte m'a coûté une heure)


#### Un premier test

Vous pouvez ensuite lancer le simulateur:
sous macos:\
`$ output/release/simulator/macos/epsilon.app/Contents/MacOS/Epsilon`

sous windows:\
`$ output/release/simulator/windows/epsilon.exe`

sous linux:\
`$ output/release/simulator/linux/epsilon.bin`

Pour compiler pour la calculatrice vous trouvez toutes les informations sur [Omega](https://github.com/Omega-Numworks/Omega)

Si vous arrivez à compiler puis lancer le simulateur, on va pouvoir passer à la suite!
