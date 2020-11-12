# Création du squelette de l'application

Notre application va s'appeller `Reader`. Les applications sur la Numworks se trouve sous le répertoire `apps`. Chaque application a son propre répertoire. Nos allons donc créer notre répertoire `apps\reader`.

Dans ce répertoire nous allons créer le code de l'application. Toutes les applications sur la numworks utilisent la même classe principale: App. Nous allons imiter les autres applications.

En C/C++, le code d'une classe se répartit sur 2 fichiers (à la différence du java ou du python). On a d'abord un header reconnaissable à son extension .h ou .hpp qui déclare la classe, ses membres et ses fonctions. Ce fichier ne décrit pas ce que font les fonctions, il expose juste leur existance.

## Le header

Nous créons donc un fichier `apps\reader\App.h`

### Header guard

Les headers commencent et finissent toujours par un bout de code un peu particulier qu'il convient de comprendre:
```gcc:latest
#ifndef __APP__H__
#define __APP__H__
...
#endif
```
On appelle ce pattern des "include guard", ces instructions s'adressent au pre-compilateur et évite de déclarer plusieurs fois les mêmes classes.

### Namespace

Comme toutes les applications déclarent une class App, pour éviter les "collisions" de nom, chaque application déclare ses classes dans son propre "namespace". Nous allons donc utiliser notre propre namespace et mettre tout notre code dans un bloc tel que:
```gcc:latest
namespace reader
{

}
```

### La classe

En C++, une classe se déclare de la façon suivante:
```gcc:latest
class App
{

};
```

Attention à ne pas oublier le ";" final.


En réalité, nous allons dériver notre classe App d'une classe App fournie par numworks. Pour cela, nous avons déjà besoin d'inclure la déclaration de cette classe à notre fichier:
```gcc:latest
#include <escher.h>
```

Puis nous déclarons lors de la création de notre classe qu'elle hérite de la classe App de numworks. Cela nous évite d'écrire nous même une partie des fonctions dont toutes les applications ont besoin:
```gcc:latest
class App : public ::App {
};
```

### Descriptor et Snapshot

Malheureusement, bien que notre classe hérite d'une classe de Numworks, nous allons devoir écrire quelques lignes de codes un peu compliquées. Il s'agit de définir 2 classes internes, dérivant elles mêmes de 2 classes fournies par Numworks. La classe `Descriptor` permet d'indiquer le nom et l'icone de notre application. Le rôle de la classe `Snapshot` est moins clair pour moi.
Notre classe devient donc:
```gcc:latest
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
    Snapshot();
    App * unpack(Container * container) override;
    void reset() override;
    Descriptor * descriptor() override;
  };
};
```

et pour finir nous déclarons un constructeur à notre classe. Le constructeur est la fonction qui permet de créer une instance de notre classe.
```gcc:latest
App(Snapshot * snapshot);
```

### En résumé

Le fichier `apps\reader\App.h` aura donc cette tête là:
```gcc:latest runnable
#ifndef READER_H
#define READER_H

#include <escher.h>

namespace Sample {

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
    Snapshot();
    App * unpack(Container * container) override;
    void reset() override;
    Descriptor * descriptor() override;
  };
private:
  App(Snapshot * snapshot);

};

}

#endif

```

