# Environnement

#### Les outils

L'installation d'un environnement de développement est souvent une étape un peu longue. Pour développer pour la Numworks il vous faudra installer quelques pré-requis.

La première étape va être d'installer les outils de compilations. Pour cela suivre les indications de l'étape 1 (Installation du SDK) sur [Numworks](https://www.numworks.com/resources/engineering/software/build/)

Sous windows, cela n'a pas été aussi simple qu'espéré pour moi. Il faut d'abord installer [MSys](https://www.msys2.org/), pour cela j'ai dû désactiver mon antivirus et activer juste Windows Defender. MSys fournit une ligne de commande comme sous linux ou macos. A partir de là il faut installer les packages dont à besoin Numworks:\
`pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-freetype mingw-w64-x86_64-pkg-config mingw-w64-x86_64-libusb git make python`

Puis en prenant bien garde d'être dans votre répertoire "home" de MSys (pour vous y rendre la commande est juste `cd`), il faut rajouter une ligne à un fichier de configuration qui modifiera votre PATH à chaque fois que vous lancerez MSys:\
`echo "export PATH=/mingw64/bin:$PATH" >> .bashrc`\
Pour information, le PATH est l'ensemble des répertoires où l'ordinateur cherchera vos programmes quand vous tapez une commande.

Ensuite, fermer et relancer MSys.

Un IDE relativement simple et complet pour un débutant est [Visual Studio Code](https://code.visualstudio.com/), on le trouve sous linux, mac et windows, mais pour ce projet un bloc note et un terminal font l'affaire.

Une petite astuce pour ouvrir un terminal MSys dans Visual Studio Code, c'est de rajouter les 2 lignes suivantes dans le fichier de préférences (settings.json)
```json
    "terminal.integrated.shell.windows": "C:\\msys64\\usr\\bin\\bash.exe",
    "terminal.integrated.shellArgs.windows": ["--login", "-i"]
```

#### Le code

Une fois l'environnement de développement préparé, il vous faut cloner Omega sur votre ordinateur. Pour cela,  dans un terminal (MSys sous windows), exécutez la commande suivante :\
`$ git clone --recursive https://github.com/Omega-Numworks/Omega.git`\
Ne pas oublier le `--recursive` !


#### La compilation

Je vous conseille de compiler une première fois le repository pour vous assurer que votre chaîne de compilation fonctionne.

Pour compiler pour le simulateur de votre plateforme, allez dans le répertoire de votre repository:\
`$ cd Omega`

puis sous macos:\
`$ make PLATFORME=simulator TARGET=macos MODEL="n0110"`

sous windows:\
`$ make PLATFORME=simulator TARGET=windows MODEL="n0110"`

ou sous linux:\
`$ make PLATFORME=simulator MODEL="n0110"`

>ATTENTION : plateforme en anglais ne prend pas de E entre le T et le F (cette découverte m'a coûté une heure)


#### Un premier test

Vous pouvez ensuite lancer le simulateur:
sous macos:\
`$ output/release/simulator/macos/epsilon.app/Contents/MacOS/Epsilon`

sous windows:\
`$ output/release/simulator/windows/epsilon.exe`

Pour compiler pour la calculatrice vous trouvez toutes les informations sur [Omega](https://github.com/Omega-Numworks/Omega)

Si vous arrivez à compiler puis lancer le simulateur, on va pouvoir passer à la suite!
