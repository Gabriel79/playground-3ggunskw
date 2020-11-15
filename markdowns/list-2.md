# La liste des livres - deuxième partie

Notre application s'ouvre ! C'est un bon début. De façon générale, je vous recommande quand vous programmez de progresser par petites étapes en vérifiant à chaque fois que ce que vous avez fait marche, plutôt que de tout coder pour ne tester que tout à la fin. Si la marche est trop haute, vous risquez de perdre beaucoup de temps pour l'atteindre. Nous allons donc procéder ainsi. La prochaine étape va être d'afficher quelque chose dans notre `TableView`. Nous n'allons pas tout de suite essayer de lister les fichiers présents sur la calculatrice, si ca ne marchait pas nous ne saurions pas si ça ne marche pas car notre `TableView` ne marche pas ou si c'est la récupération des fichiers qui ne marche pas. 

## Stockage de la liste des fichiers

On va donc imaginer avoir réussi à lister les fichiers présents. Sur une plateforme plus "standard" que la Numworks, on aurait probablement listé les fichiers et remplit un `std::vector`, un tableau dont la taille peu augmenter au besoin. Malheureusement, la Numworks ne supporte pas l'allocation dynamique de mémoire (le fait de réclamer un peu plus de place pour notre tableau pendant l'exécution du programme). Il faut que la taille de notre tableau soit fixée à la compilation. Nous allons donc nous contentez de lister les 20 premiers fichiers textes, on est loin du Kindle mais le Kindle ne résout pas encore des équations.

La Numworks ne fournit pas de système de fichiers comme on peut en trouver sur un ordinateur. Les fonctions standard du C pour parcourir un système de fichier n'existe donc pas. Omega fourni une solution que nous étudierons plus tard. L'important pour l'instant est la structure `File` qui permet de décrire un fichier. Une structure est une sorte de classe dont tous les membres sont public. En générale les structures ne contiennent pas de méthodes, juste des attributs que l'on peut lire et écrire directement. On utilise une structure pour grouper ensemble des données. La structure `File` contient ainsi le nom du fichier, sa taille et son adresse dans la mémoire et une variable indiquant s'il s'agit d'un fichier exécutable.

Je vous reproduis ici sa définition (elle n'est pas à recopier dans notre projet!).
```c++
struct File {
    const char *name;
    const uint8_t *data;
    size_t dataLength;
    bool isExecutable;
};
```

### Le header

Nous allons stocker la description des fichiers trouvés sur la Numworks (les `File`) dans un tableau de 20 cases, que nous allons rajouter comme membre `private` de notre `ListBookController`:
```C++
static const int NB_FILES = 20;
External::Archive::File m_files[NB_FILES];
```
Plutôt que d'utiliser directement le chiffre 20, dans tout le code source, une bonne pratique est de définir une constante, ainsi si on désire plus tard changer cette valeur, il n'y aura qu'un endroit à modifier. Notre constante est donc `NB_FILES`, par convention en majuscule. Notre constante est constante grâce au `const`, cela signifie qu'on ne pourra pas modifier sa valeur dans le programme. Le nombre maximum de fichiers que nous supportons est un nombre entier, on utilise donc le type `int`. Enfin, le mot clé `static` qu'on a déjà rencontré dans un context différent, a cette fois un autre sens, il signifie que NB_FILES est un membre de la classe partagé par toutes les instances de notre classe, chaque instance n'aura pas son propre `NB_FILES`, comme la valeur est constante, on économise de la mémoire en utilisant la même constante pour toutes nos instance.

`External::Archive::File m_files[NB_FILES];` définit un tableau dont chaque élément sera un `External::Archive::File` où `File` est le nom de la classe et `External::Archive` le nom de son namespace (Omega n'a pas conservé la convention du code source de la Numworks qui écrit les noms de namespace en minuscule). Pour pouvoir utiliser cette classe, il faut inclure le header qui le définit :\
`#include <apps/external/archive.h>`

Notre tableau pourra donc contenir juqu'à 20 fichiers, mais nous avons besoin de savoir combien il en contient à un instant donné. Pour cela nous rajoutons donc un attribut à notre classe, toujours dans la section `private` :\
`int m_nbFiles = 0;` 

Votre fichier devrait maintenant avoir cette tête là
```c++
#ifndef __LIST_BOOK_CONTROLLER_H__
#define __LIST_BOOK_CONTROLLER_H__

#include <escher.h>
#include <apps/external/archive.h>

namespace reader
{

class ListBookController : public ViewController, public SimpleListViewDataSource, public ScrollViewDataSource
{
public:
    ListBookController(Responder * parentResponder);
    View* view() override;

    int numberOfRows() const override;
    KDCoordinate cellHeight() override;
    HighlightCell * reusableCell(int index) override;
    int reusableCellCount() const override;
private:
    TableView m_tableView;

    static const int NB_FILES = 20;
    External::Archive::File m_files[NB_FILES];
    int m_nbFiles = 0;
};
}
#endif
```
### L'implémentation

Commençons par simuler la présence d'un livre dans notre calculatrice, en remplissant nous même le tableau dans le constructeur de notre contrôleur.
```c++
ListBookController::ListBookController(Responder * parentResponder):
    ViewController(parentResponder),
    m_tableView(this, this)
{
    m_files[0].name= "Harry Potter 1.txt";
    m_nbFiles = 1;
}
```

`m_files` est un tableau de `File`, `m_files[0]` correspond a l'élément de la première case du tablean (les indices commencent à 0). `m_files[0].name` permet d'accèder directement à la variable `name` du fichier, on y recopie le nom d'un fichier. On ne remplit pas les autres membres de la structure car nous n'en avons pas besoin pour l'instant.

On n'oublie pas également d'indiquer que notre tableau contient maintenant un fichier, en modifiant `m_nbFiles`.

On va pouvoir maintenant modifier la fonction `numberOfRows()` pour quelle renvoie le nombre de fichier dans notre tableau :
```c++
int ListBookController::numberOfRows() const
{
    return m_nbFiles;
}
```

## Les cellules de la TableView

Nous avons maintenant du contenu à afficher dans notre `TableView`. Celle-ci est composée de "cellules" (`TableCell`). C'est nous qui devons les lui fournir, la `TableView` se contentant de gérer leur affichage au bon endroit. La `TableView`peut afficher un nombre infini de lignes, mais seulement quelques une sont visibles à un instant donné à l'écran, on va donc pouvoir "recycler" la cellule d'une ligne lorsqu'elle n'est pas visible à l'écran. C'est ce à quoi sert la méthode `reusableCell()` de notre contrôleur.

Souvenez-vous, nous avons du faire dériver notre classe `ListBookController` de `TableViewDataSource`, et en tant que data source de la `TableView` nous avons du implémenter différentes fonctions dont `reusableCell()` et `reusableCellCount()`. Pour l'instant nous n'avons pas vraiment codé ces fonctions. Il est temps de le faire.

### Retour vers le header

Vu la taille de l'écran de la numworks et la hauteur de nos cellules (que nous avons fixé à 50 dans la méthode `cellHeight()`) nous pouvons nous contenter de manipuler 6 cellules, la `TableView` s'occupera de les recycler pour afficher nos potentiels 20 fichiers. Définissons cela dans une constante dans la zone `private` de notre class `ListBookController` :\
`static const int NB_CELLS = 6;`

Numworks fournit des classes pour des cellules "classiques". Elles dérivent de `HighlightCell`, on peut les retrouver dans cette [documentation](https://udxs.me/EpsilonDocs/class_message_table_cell.html). Nous allons utiliser une `MessageTableCell`. Pour cela rajoutons donc un tableau de ces cellules à notre classe :\
`MessageTableCell m_cells[NB_CELLS];`

### L'implémentation

Nous pouvons maintenant coder pour de vrai `reusableCell()` et `reusableCellCount()` :
```c++
HighlightCell * ListBookController::reusableCell(int index)
{
    return &m_cells[index];
}
    
int ListBookController::reusableCellCount() const
{
    return NB_CELLS;
}
```

## Remplir une Cellule

### Le header

Nous fournissons maintenant des `TableCell` à la `TableView`. Il nous reste à remplir ces cellules avec le nom de nos fichiers. Pour cela il nous faut redéfinir une fonction. Nous rajoutons donc cette fonction :\
`void willDisplayCellForIndex(HighlightCell * cell, int index) override;`\
à notre controlleur. Cette fonction va être appelée par la `TableView` à chaque fois qu'elle voudra afficher une cellule. Cette fonction nous passe en paramètre la cellule que nous devons remplir.

Notre header devrait maintenant ressembler à ça:
```c++
#ifndef __LIST_BOOK_CONTROLLER_H__
#define __LIST_BOOK_CONTROLLER_H__

#include <escher.h>
#include "apps/external/archive.h"

namespace reader
{

class ListBookController : public ViewController, public SimpleListViewDataSource, public ScrollViewDataSource
{
public:
    ListBookController(Responder * parentResponder);
    View * view() override;

    int numberOfRows() const override;
    KDCoordinate cellHeight() override;
    HighlightCell * reusableCell(int index) override;
    int reusableCellCount() const override;
    void willDisplayCellForIndex(HighlightCell * cell, int index) override;
    
private:
    TableView m_tableView;

    static const int NB_FILES = 20;
    External::Archive::File m_files[NB_FILES];
    int m_nbFiles = 0;

    static const int NB_CELLS = 6;
    MessageTableCell m_cells[NB_CELLS];
};

}
#endif 
```

### L'implémentation

Nous allons donc rajouter la définition suivante à notre fichier d'implémentation :
```c++
void ListBookController::willDisplayCellForIndex(HighlightCell * cell, int index)
{
    MessageTableCell* myTextCell = static_cast<MessageTableCell*>(cell);    
    MessageTextView* textView = static_cast<MessageTextView*>(myTextCell->labelView());
    textView->setText(m_files[index].name);
    myTextCell->setMessageFont(KDFont::LargeFont);
}
```

Cette fonction reçoit donc en paramètre la cellule à remplir. Cette cellule est passée sous la forme d'un pointeur vers une `HighlightCell`. Ce type est le type parent de toutes les cellules. Or en réalité la `TableView` nous donne une des cellules qu'on lui a au préalable donné dans `reusableCell`, nous connaissons donc le type réel de la cellule que nous recevons, c'est une `MessageTableCell`. Nous allons donc forcer le compilateur à considérer la variable  `cell` comme un pointeur vers une `MessageTableCell`. Cette opération s'appelle un "cast", et ce fait avec ce code : `static_cast<MessageTableCell*>(cell)`. Nous en stockons le résultat dans un nouveau pointeur `myTextCell` de type `MessageTableCell` qui pointe vers la même cellule mais maintenant connu par le compilateur sous le bon type, ce qui nous permettra d'appeler dessus des méthodes qui n'existaient pas dans le type initial `HighlightCell`.

Pourquoi passer par une `HighlightCell`? la méthode `willDisplayCellForIndex(HighlightCell * cell, int index)` doit être implementée par tous les `TableViewDataSource`. Certaine `TableView` voudront afficher des images dans leur cellule, d'autres du texte, d'autre les 2... la méthode doit donc prendre une cellule la plus générique possible en paramètre pour marcher dans tous les contextes. Attention, un cast ne permet pas de transformer tout en n'importe quoi ! C'est bien parce que nos cellules (`m_cells` dans le `ListBookController` sont de type `MessageTableCell` qu'on peut faire le cast vers ce type).

Maintenant qu'on a une `MessageTableCell`, on peut en récupérer sa vue au moyen de la méthode `labelView()`

Travail en cours !