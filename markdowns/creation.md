# Création du squelette de l'application

Notre application va s'appeller `Reader`. Les applications sur la Numworks se trouve sous le répertoire `apps`. Chaque application a son propre répertoire. Nos allons donc créer notre répertoire `apps\reader`.

Dans ce répertoire nous allons créer le code de l'application. Toutes les applications sur la numworks utilisent la même classe principale: App. Nous allons imiter les autres applications.

En C/C++, le code d'une classe se répartit sur 2 fichiers (à la différence du java ou du python). On a d'abord un header reconnaissable à son extension .h ou .hpp qui déclare la classe, ses membres et ses fonctions. Ce fichier ne décrit pas ce que font les fonctions, il expose juste leur existance.

## Le header

Nous créons donc un fichier `apps\reader\app.h`

### Header guard

Les headers commencent et finissent toujours par un bout de code un peu particulier qu'il convient de comprendre:
```c++
#ifndef __APP__H__
#define __APP__H__
...
#endif
```
On appelle ce pattern des "include guard", ces instructions s'adressent au pre-compilateur et évite de déclarer plusieurs fois les mêmes classes.

### Namespace

Comme toutes les applications déclarent une class App, pour éviter les "collisions" de nom, chaque application déclare ses classes dans son propre "namespace". Nous allons donc utiliser notre propre namespace et mettre tout notre code dans un bloc tel que:
```c++
namespace reader
{

}
```

### La classe

En C++, une classe se déclare de la façon suivante:
```c++
class App
{

};
```

Attention à ne pas oublier le ";" final.


En réalité, nous allons dériver notre classe App d'une classe App fournie par numworks. Pour cela, nous avons déjà besoin d'inclure la déclaration de cette classe à notre fichier:
```c++
#include <escher.h>
```

Puis nous déclarons lors de la création de notre classe qu'elle hérite de la classe App de numworks. Cela nous évite d'écrire nous même une partie des fonctions dont toutes les applications ont besoin:
```c++
class App : public ::App {
};
```

### Descriptor et Snapshot

Malheureusement, bien que notre classe hérite d'une classe de Numworks, nous allons devoir écrire quelques lignes de codes un peu compliquées. Il s'agit de définir 2 classes internes, dérivant elles mêmes de 2 classes fournies par Numworks. La classe `Descriptor` permet d'indiquer le nom et l'icone de notre application. Le rôle de la classe `Snapshot` est moins clair pour moi, je pense qu'elle permet de sauver la classe dans la mémoire de la numworks quand l'utilisateur quitte l'application pour pouvoir la restaurer quand il l'ouvre.
Notre classe devient donc:
```c++
class App : public ::App {
public:
  class Descriptor : public ::App::Descriptor {
  public:
    I18n::Message name() override;
    I18n::Message upperName() override;
    const Image * icon() override;
  };
  class Snapshot : public ::App::Snapshot {
  public:
    App * unpack(Container * container) override;
    Descriptor * descriptor() override;
  };
};
```

et pour finir nous déclarons un constructeur à notre classe. Le constructeur est la fonction qui permet de créer une instance de notre classe.
```c++
App(Snapshot * snapshot);
```

### En résumé

Le fichier `apps\reader\app.h` aura donc cette tête là:
```c++
#ifndef READER_H
#define READER_H

#include <escher.h>

namespace reader {

class App : public ::App {
public:
  class Descriptor : public ::App::Descriptor {
  public:
    I18n::Message name() override;
    I18n::Message upperName() override;
    const Image * icon() override;
  };
  class Snapshot : public ::App::Snapshot {
  public:
    App * unpack(Container * container) override;
    Descriptor * descriptor() override;
  };
private:
  App(Snapshot * snapshot);

};

}

#endif

```
## Les définitions

Une fois notre classe déclarée, il convient de coder son implémentation : ce qu'elle fait. Cela se fait dans un fichier .cpp. Créons donc `apps\reader\app.cpp`

Commençons par inclure quelques fichiers de définition:
```c++
#include "app.h"
#include "reader_icon.h"
#include "apps/apps_container.h"
#include "apps/i18n.h"
```

Le premier header "app.h" est le fichier que nous venons de créer. Je n'ai aucune idée de ce qu'est réellement le second `reader_icon.h`. `apps/apps_container.h` fournit un objet dont nous aurons besoin. Le dernier `apps/i8n.h` est un fichier contenant la traduction des chaines de caractères utilisées dans les applications. En effet, pour rendre l'internationnalisation plus simple, les applications évitent de contenir directement des chaines de caractères dans leur code, mais utilisent un label qui avec un peu de magie sera transformée en la bonne chaine de caractère selon la langue dans laquelle est configurée la calculatrice. Ce n'est pas totalement magique, il faudra bien traduire ces chaines quelques part.

Comme dans le header, nous allons mettre ensuite notre code dans notre namespace. Attention à ne pas mettre les includes dans le header. En effet, les includes recopient le contenu des headers dans notre fichier au moment de la compilation, mettre les includes dans notre namespace ajouterait ces définition à notre namespace ce que nous ne voulons pas.

```c++
namespace reader
{

}
```

### Descriptor

Puis au sein du namespace, nous allons commencer par fournir l'implémentation des fonctions de la classe `Descriptor`:

```c++
I18n::Message App::Descriptor::name() {
  return I18n::Message::ReaderApp;
}

I18n::Message App::Descriptor::upperName() {
  return I18n::Message::ReaderAppCapital;
}

const Image * App::Descriptor::icon() {
  return ImageStore::ReaderIcon;
}
```

La fonction `name` renvoie le nom de l'application, que nous allons définir ensuite dans un fichier de ressource. `upperName` renvoie le nom en capitale, et `icon` l'icone de l'application que nous allons également définir ensuite.

### Snapshot

L'implementation de la class `Snapshot` est un peu plus compliquée.

```c++
App * App::Snapshot::unpack(Container * container) {
  return new (container->currentAppBuffer()) App(this);
}

App::Descriptor * App::Snapshot::descriptor() {
  static Descriptor descriptor;
  return &descriptor;
}
```

La fonction `unpack` crée une instance de notre class App en lui donnant en paramètre la zone de mémoire à utiliser (qui est fournit par `container->currentAppBuffer()`).

La fonction `descriptor` renvoie elle le `Descriptor` de notre application en utilisant un pattern classique: le singleton. Il n'existe au sein de notre application qu'une seule instance du `Descriptor` celui déclaré à la ligne `static Descriptor descriptor;`. Le mot clé `static` lors de la déclaration de la variable indique que la variable continue d'exister même après la fin de la fonction et que si le programme repasse dans cette fonction il n'aura pas à déclarer une nouvelle variable mais réutilisera la variable du précédent appel.

### Constructeur

Il manque encore le constructeur de votre classe, qui se contentera d'appeler le constructeur de la classe dont il hérite:
```c++
App::App(Snapshot * snapshot) :
  ::App(snapshot, nullptr)
{
}
```


### En résumé

Le fichier `apps\reader\app.c` aura cette tête là:
```c++
#include "app.h"
#include "reader_icon.h"
#include "apps/apps_container.h"
#include "apps/i18n.h"


namespace reader {

I18n::Message App::Descriptor::name() {
  return I18n::Message::ReaderApp;
}

I18n::Message App::Descriptor::upperName() {
  return I18n::Message::ReaderAppCapital;
}

const Image * App::Descriptor::icon() {
  return ImageStore::ReaderIcon;
}

App * App::Snapshot::unpack(Container * container) {
  return new (container->currentAppBuffer()) App(this);
}

App::Descriptor * App::Snapshot::descriptor() {
  static Descriptor descriptor;
  return &descriptor;
}

App::App(Snapshot * snapshot) :
  ::App(snapshot, nullptr)
{
}

}
```


### Les fichiers de resources

Comme dit précédemment l'internationalisation se fait au moyen de fichiers de ressources. Il y en a un par langue. Le plus simple est de copié les fichiers .i18n d'une autre application dans votre répertoire `apps\reader`

puis de remplacer leur contenu par le votre :
```
ReaderApp = "Reader"
ReaderAppCapital = "READER"
```
et montrer vos talents en langue en traduisant "reader" dans toutes les langues.

### L'icone

Il vous faudra ensuite une icone de 55x56 au format png.
![Icone](../reader_icon.png)
à mettre également dans votre répertoire `apps\reader`

Il faut également déclarer votre icone dans le thème de la calculatrice. Pour cela
rajouter la ligne:\
`"apps/reader/reader_icon.png" : "apps/reader_icon.png",`
au fichier `themes\icons.json`

### L'intégration

Il faut ensuite inclure votre application aux applications à construire et embarquer sur la calculatrice.
Cela se fait dans le fichier `build\config.mak`

Rajouter "reader" à la ligne:\
`EPSILON_APPS ?= reader calculation rpn graph code statistics probability solver atom sequence regression settings external`
La position de votre application sur cette ligne déterminera sa position dans les applications de la calculatrice.

Ce fichier sert à configurer le "build" par défaut de la calculatrice. Vous pouvez ainsi changer le theme par défaut et d'autres paramètres.

### le Makefile

Pour compiler votre application, il vous faut un makefile, c'est un fichier qui déclare vos fichiers sources et dit avec quels outils ils se compilent.

Créer ainsi un fichier `apps\reader\Makefile` qui contiendra les lignes suivantes:
```Makefile
apps += reader::App
app_headers += apps/reader/app.h

app_sreader_src = $(addprefix apps/reader/,\
  app.cpp \
)

apps_src += $(app_sreader_src)

app_images += apps/reader/reader_icon.png

i18n_files += $(call i18n_without_universal_for,reader/base)

$(eval $(call depends_on_image,apps/reader/app.cpp,apps/reader/reader_icon.png))
```

La ligne `apps += reader::App` rajoute votre application à la liste des app de la calculatrice. `reader` correspond à votre namespace et `App` à votre classe principale (si votre application à un autre nom et namespace pensez y).
dans `app_headers` vous mettez vos headers et dans `app_sreader_src` vos fichier sources.


### Compilation

Il est temps de retourner à votre ligne de commande. Dans le répertoire `Omega`, on va commencer par nettoyer le précédent build, avec :\
`make PLATFORM=simulator device=windows MODEL="n0110" clean`

Il n'est pas necessaire de faire ce `make clean` avant chaque compilation, la compilation suivante, recompilera tout et sera donc longue. Mais quand vous faites des modifications à autre chose que des fichiers .c ou .h, cela peut être necessaire pour que tous les objets necessaires soient bien construits.

Vous pouvez ensuite tenter une compilation:
`make PLATFORM=simulator device=windows MODEL="n0110"`

et si tout va bien il n'y aura pas d'erreur... (c'est malheureusement rare d'arriver à tout faire du premier coup sans faute de frappe ou autre étourderie). En cas d'erreur, essayer de comprendre le message qui peut mettre sur la voie de ce qui ne va pas...

### Un premier test

Vous pouvez ensuite lancer le simulateur:
sous macos:\
`$ output/release/simulator/macos/epsilon.app/Contents/MacOS/Epsilon`

sous windows:\
`$ output/release/simulator/windows/epsilon.exe`

Normalement votre application devrait apparaitre dans le simulateur. Vous pouvez essayer de la lancer, mais elle va faire se crasher le simulateur ou votre calculatrice...


### On sauvegarde

Une bonne habitude à prendre est d'utiliser git pour sauvegarder votre code. Quand une version marche, submittez votre code dans git, cela vous permettra de revenir à une version qui marchait si vous êtes perdu.

Pour la suite, il va falloir patienter un peu...
